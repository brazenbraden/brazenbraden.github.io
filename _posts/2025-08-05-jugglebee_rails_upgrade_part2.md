---
layout: post
title: "JuggleBee’s Great Leap – Data Migration, ActiveStorage, and Production Readiness (Part 2)"
crawlertitle: "How we migrated data, moved to ActiveStorage, and shipped to production on Rails 8"
summary: "Step-by-step on taking a Rails 4.2 app live on Rails 8: Postgres 17.5 migration, 50k image move to ActiveStorage/S3, secure credentials via Bitwarden, and Kamal-powered deploys to Hetzner."
date: 2025-08-05
categories: posts
tags: ['jugglebee', 'rails']
bg: "post-jugglebee-upgrade-part2.png"
---

Now that we had a deployable app, I set up a staging server using my new favourite provider, [Hetzner](https://www.hetzner.com/) (*not a sponsor, lol*), spun it up, and witnessed a “fully functional” skeleton of JuggleBee. Stage 1 of the migration plan was complete.

Now came the real meat and potatoes of the project — actually hydrating the new app with all of its production data. We’re talking over 50,000 images, countless database records, and security credentials… the works. This is the part where a missed step could mean broken listings, missing files, or downtime. No pressure.

> In case you missed **Part 1**, we covered:
> - Why I started fresh on **Rails 8** instead of doing incremental upgrades.
> - Migrating core **models/controllers/services** to modern Rails conventions.
> - Swapping Sprockets for **importmaps** and modernizing the JavaScript setup.
> - Replacing **Sidekiq + Redis + whenever** with **ActiveJob + SolidQueue**.
> - Moving deployments to **Kamal 2** with Traefik and Let’s Encrypt.

**Catch up on it here:** [JuggleBee’s Great Leap – Rebuilding a Rails 4 App in Rails 8 (Part 1)](https://www.brazenbraden.com/posts/jugglebee_rails_upgrade_part1/)

Now—back to the post.

---

## The Daunting Database Migration

Legacy JuggleBee had been coasting along happily on Postgres 9.6 — which, back in 2016, was the latest and greatest. Fast forward to today and it’s long past its sell-by date, with official support ending in November 2021. Upgrading wasn’t just about keeping up with the times; Postgres 17.5 brings some serious perks:

- Native JSON field type support.
- Incremental backups with `pg_basebackup --incremental`, slashing backup time and storage usage.
- Noticeable performance gains from better memory management, indexing, and partitioning.
- Improved observability via enhanced `EXPLAIN` options and richer stats views.
- …and more.

It was a no-brainer to upgrade, but the version gap meant one wrong move could corrupt thousands of user, auction, and invoice records. That made reliable backups non-negotiable before touching a single byte. Luckily, I already had a manual backup process baked into my deploy routine. Here’s the script (*feel free to steal any of the scripts in my posts*):

```bash
#!/bin/bash

DB_CONTAINER=($(sudo docker ps | grep postgres | awk '{print $NF}'))
BACKUP_NAME=dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql
BACKUP_FOLDER=db_backups
KEEP=10

# Dump the database into a backup sql file
echo "Creating the database dump..."
docker exec -t $DB_CONTAINER pg_dumpall -c -U postgres > $BACKUP_NAME
echo "Database dump created."

echo "Moving database backup to backup folder..."
mkdir -p $BACKUP_FOLDER
mv $BACKUP_NAME $BACKUP_FOLDER
echo "Backup moved to backup folder."

echo "Deleting old releases..."
(cd $BACKUP_FOLDER && (ls -t | head -n $KEEP; ls) | sort | uniq -u | xargs rm -rf)
echo "Backup stored."
```

One catch: `pg_dumpall` (used above) tries to dump *everything* — including Postgres cluster-wide settings — which can be version-specific and cause headaches on import. For a major version jump, you want `pg_dump`, which targets a single database and avoids the version mismatch landmines.

For the migration, I only needed to change the `docker` command we executed - with the added ENV vars, that is:

```bash
DB_USER=db_user
DB_PASSWORD=db_password
DATABASE_NAME=database_name

# ...

docker exec -e PGPASSWORD=$DB_PASSWORD -t $DB_CONTAINER \
    pg_dump -U $DB_USER $DATABASE_NAME > $BACKUP_NAME
```

With the new backup in hand, importing into Postgres 17.5 went off without a hitch — no edits, no drama. Honestly, I was braced for a fight given the version gap, but the data was simple enough (and avoided fancy Postgres features) that it just… worked. A rare and pleasant surprise in a project full of moving parts.

## Listing Image Migration


Unlike the data migration before it, migrating all the listing images (just shy of 50,000!) came with its own set of headaches. The legacy setup used CarrierWave with a custom uploader and fixed S3 paths, which worked fine at the time but wasn’t exactly future-proof. Since I was already upgrading to Rails 8, it made sense to tackle this technical debt head-on and move to ActiveStorage — effectively retiring these gems:

- `carrierwave`
- `fog-aws`
- `mini_magick`

… and replacing them with `image_processing` for post-processing.

In the old days, uploads required creating an “Uploader” class and mounting it on a model. For example:

```ruby
# app/models/image.rb
class Image < ActiveRecord::Base
  mount_uploader :aws, ImageUploader
end

# app/uploaders/image_uploader.rb
class ImageUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick
  storage :fog

  process resize_to_fill: [850, 850]

  version :thumb do
    process resize_to_fill: [280, 280]
  end

  def store_dir
    "listings/#{model.listing_id}/#{model.id}"
  end
end
```

With ActiveStorage, uploaders and hard-coded S3 paths are gone. You simply declare the attachment in your model (after setting up `config/storage.yml`):

```ruby
class Image < ApplicationRecord
  has_one_attached :file do |attachable|
    attachable.variant :thumb, resize_to_limit: THUMB_DIMENSIONS, preprocessed: true
  end
end
```

That takes care of *future* uploads — but what about migrating the 50,000+ legacy images stored in the old bucket? The `image.aws` field plus CarrierWave’s `store_dir` gave me the exact file path to each image. I then wrote a script to:

1. Download each image from the legacy bucket.
2. Re-upload it through ActiveStorage so it generates its own hashed storage path.
3. Skip already-migrated images to allow safe re-runs.

Here’s the script in all its glory (run synchronously to avoid AWS rate limits — which meant a cool 4 hours of migration time):

```ruby
#!/usr/bin/env ruby

require_relative '../config/environment'
require 'aws-sdk-s3'
require 'stringio'

OLD_S3_BUCKET = Rails.application.credentials.dig(:old_aws, :bucket)
OLD_S3_REGION = Rails.application.credentials.dig(:old_aws, :region)
OLD_S3_ACCESS_KEY_ID = Rails.application.credentials.dig(:old_aws, :s3_key)
OLD_S3_SECRET_ACCESS_KEY = Rails.application.credentials.dig(:old_aws, :s3_secret)

# Optional: skip already migrated images
SKIP_EXISTING = true

s3 = Aws::S3::Client.new(
  region: OLD_S3_REGION,
  access_key_id: OLD_S3_ACCESS_KEY_ID,
  secret_access_key: OLD_S3_SECRET_ACCESS_KEY
)

puts "Starting image migration..."

Image.find_each do |image|
  begin
    if SKIP_EXISTING && image.file.attached?
      puts "Skipping Image #{image.id} (already attached)"
      next
    end

    # Construct old S3 key from uploader logic - your path here
    key = "listings/#{image.listing_id}/#{image.id}/#{File.basename(image[:aws].to_s)}"

    obj = s3.get_object(bucket: OLD_S3_BUCKET, key: key)

    image.file.attach(
      io: StringIO.new(obj.body.read),
      filename: File.basename(key),
      content_type: obj.content_type || 'image/jpeg'
    )

    puts "Migrated Image #{image.id} (Listing #{image.listing_id})"
  rescue Aws::S3::Errors::NoSuchKey => e
    warn "Image #{image.id} missing in old bucket: #{key}"
  rescue => e
    warn "Failed to migrate Image #{image.id}: #{e.class} - #{e.message}"
  end
end

puts "Migration script finished"
```

Four hours later, we were in business — all images were now managed by ActiveStorage.
Future uploads are handled automatically, thumbnails are generated on demand, and the whole setup is cleaner and easier to maintain.

## Credentials & Security

You might have noticed the use of `Rails.application.credentials` in the previous script. That’s new territory — Rails encrypted credentials didn’t exist back in 4.2.

Back then we had `secrets.yml`, which stored everything in plain text and was meant to be excluded from Git. In theory, you’d inject it during deploy with whatever tooling you used. In practice… I was lazy. My deploys were manual, the repo was private, and I just checked them straight in. **Big no-no.** Thankfully, Bitbucket never got hacked — but let’s just say this setup wouldn’t pass any kind of security audit.

Rails 5.2 introduced encrypted credentials — finally, a way to store secrets in plain text *locally* while Rails encrypts them with a master key. That master key stays out of Git and gets injected during deploy. Much better.

Fast forward to today, and Kamal (my de facto deployment tool) takes it a step further with [built-in support for pulling secrets](https://kamal-deploy.org/docs/commands/secrets/) straight from password managers like Bitwarden, 1Password, or LastPass. That means my Rails master key, database credentials, and other environment variables can live safely in Bitwarden, and Kamal just grabs them during deploy. No manual copying, no “where did I put that key?” moments, and no plain-text secrets hanging around in the repo.

Here’s an example using Bitwarden via the CLI. This snippet fetches all secrets and assigns them to variables in `.kamal/secrets`:

```bash
SECRETS=$(kamal secrets fetch --adapter bitwarden-sm all)

KAMAL_REGISTRY_PASSWORD=$(kamal secrets extract KAMAL_REGISTRY_PASSWORD $SECRETS)
RAILS_MASTER_KEY=$(kamal secrets extract RAILS_MASTER_KEY $SECRETS)
POSTGRES_USER=$(kamal secrets extract POSTGRES_USER $SECRETS)
POSTGRES_PASSWORD=$(kamal secrets extract POSTGRES_PASSWORD $SECRETS)
```

The only catch: you’ll need to have the Bitwarden CLI installed and authenticated on every machine you deploy from. It’s a one-time setup, but totally worth it for the peace of mind of never having to worry about leaked secrets again.

## Lessons Learned

Looking back, a few big takeaways stood out:

- **Scripts are your best friend.** Whether it’s a database dump, image migration, or credentials setup, having a repeatable script means you can run it, tweak it, and run it again without reinventing the wheel. It also removes a ton of mental overhead when you’re juggling multiple moving parts.

- **Test migrations in staging — thoroughly.** Moving to PostgreSQL 17.5 could have been a disaster if I’d gone straight into production. Running the full migration end-to-end in a safe environment gave me confidence and caught subtle issues before they ever touched live data.

- **Expect the cleanup phase.** Even after the “big ticket” migrations were done, staging revealed a laundry list of smaller fixes — broken image links, JavaScript load order issues, outdated gem calls, and other gremlins. None of these were major individually, but together they took nearly two weeks to iron out. Build that buffer into your timeline.

- **Take the opportunity to ditch technical debt.** CarrierWave, fog-aws, Sidekiq, Redis… all of these had served me well for years, but replacing them with ActiveStorage and SolidQueue means fewer dependencies to maintain, fewer services to monitor, and a cleaner architecture.

- **Security deserves equal billing with features.** Rails encrypted credentials plus Bitwarden CLI integration finally dragged my secrets management into the modern age. No more plain-text secrets in Git. Enough said.

- **Modern Rails is worth the leap.** From importmaps to Kamal, the tooling around Rails 8 makes deployment, scaling, and maintenance dramatically simpler than what I was doing in 2015.

## Closing thoughts

And with that, JuggleBee officially joined the modern Rails era.

This wasn’t just a version bump — it was a full-stack rejuvenation.
The app runs faster, the infrastructure is leaner, and I finally have a deployment pipeline I can trust at 2 a.m. without crossing my fingers.

Was it worth skipping incremental upgrades? Absolutely. Starting fresh on Rails 8 gave me a clean slate to rebuild only what mattered, using tools that will keep JuggleBee humming for years to come.

If you’ve been sitting on a legacy Rails app, staring down the upgrade path and wondering if it’s worth the effort — it is. Just plan your migrations, test them in staging, and embrace the opportunity to shed some old weight while you’re at it.

[JuggleBee’s](https://www.jugglebee.com) now live on **Rails 8 / Ruby 3.4.3 / PostgreSQL 17.5**, with ActiveStorage handling thousands of images, Bitwarden keeping our secrets safe, and Kamal making deployments a one-command affair.

Now it’s time to step away from the terminal, grab a cold beer, and actually enjoy the thing I’ve been rebuilding over the last month — until the next big upgrade comes along.
