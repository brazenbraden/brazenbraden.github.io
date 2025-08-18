---
layout: post
title: JuggleBee’s Great Leap - Rebuilding a Rails 4 App in Rails 8 (Part 1)
crawlertitle: Why We Started Fresh on Rails 8 and How We Got There
summary: How I rebuilt JuggleBee from Rails 4.2 to Rails 8 in one giant leap — modernizing models, JavaScript, background jobs, and deployment along the way.
date: 2025-07-30
categories: posts
tags: ['jugglebee', 'rails']
bg: "post-jugglebee-upgrade-part1.png"
---

[JuggleBee](http://www.jugglebee.com) was born in 2015. It was Namibia's first online auction platform and is still one of the biggest today. I built it with Ruby on Rails 4.2 on Ruby 2.2. It sat there, rock-solid and stubbornly stable, only needing the occasional dust-off and minor tweak over the years. At various points in time, I thought about upgrading the various versions; however, I was stuck between a rock and a hard place — in order to upgrade Ruby, I had to upgrade Rails, and vice versa. The idea of incremental upgrades felt like a nightmare — two battlefronts to fight on, Ruby on one side and Rails on the other.

Fast forward to 2025. With the arrival of my daughter, I acquired a smattering of free time interspersed with sleepless nights. The perfect time to finally bite the bullet, bring JuggleBee into the 21st century (you know what I mean), and get these upgrades done. After much pondering and planning, only one realistic option remained... incremental upgrades would take months with constant patching between every version bump; I would have to take the leap from Rails 4.2 straight to Rails 8, with Ruby hopping from 2.2 to 3.4.3. It was quite the journey, and what follows is a breakdown of the challenges and learnings I made along the way.

## Starting Fresh

Not to blow my own horn or anything, but I was in a pretty good place with this migration plan. The codebase of JuggleBee was intentionally built with self-contained business logic components, defining a clear separation of concerns. The models and controllers were not overly bloated, with liberal usage of "service objects" performing the more complicated logic, and an added separation of logic between my controllers and views using the "view decorator" pattern. This architecture made the copy-pasta approach almost… dare I say… fun. But let’s not kid ourselves — a jump from Rails 4.2 to 8 is no walk in the park.

As part of the `rails new` process (after installing the latest Ruby and necessary Rails gems), and before the true code migration could begin, I just needed an empty, runnable project with the necessary gems installed. I appended my old `Gemfile` with the auto-generated one, removing all locked versions to ensure the latest compatible versions were installed. I also identified irrelevant and dead gems, removing those and replacing them with alternatives where necessary (more detail on this later).

Reading into each of the default gems specified by the `rails new` generator gave me a good understanding of what the current Rails stack looks like. I didn’t need all of it, so some were commented out, but I did take some of the new suggestions into account. `Puma`, for example, replaced `Unicorn`, which I was previously using. I could also get rid of `God`, in the non-divine sense, as I no longer needed to monitor my server state manually. All in all, this whole process didn’t take more than an hour or so.

## Models, Controllers, Services & Views (in that order)

These being purely Ruby code made it the easiest place to start. Even though we jumped up from Ruby 2.2 to 3.4.3, not all that much in the core language changed. Twas Rails that was the most behind. For example, JuggleBee was developed before the abstract `ApplicationRecord` class made its way into the Rails architecture!

### Models

The models were the easiest to migrate over, as they already had a smattering of unit tests behind them. First, though, we had to get the database up and running. Spoiler alert: I was also updating our Postgres version from **9.6** to **17.5** as part of this big update. The old migration files were fossils — brittle, outdated, and utterly unfit for a modern Rails 8 app. The simplest solution was to drop a `pg_dump` of the current database (just on my local copy of the now legacy JuggleBee) and import it, generating the `schema.rb`. I could now copy across my models and test them out in the `rails console`.

Some of the updates required to get things up and running were:

- `belongs_to` is now a required atribute. I have a home-grown version of polymorphism which allows a `Listing` to be either a `Product` or `Auction` and the `belongs_to` on these associations are optional, so I had to update these associations accordingly (***side note**: a refactor waiting to happen*). So,
```ruby
belongs_to :auction
belongs_to :product
```
becomes
```ruby
belongs_to :auction, optional: true
belongs_to :product, optional: true
```

- I had a couple of instances of `update_attributes` (very selective. Given they bypass validations, they were used with extreme caution) which is now deprecated and had to be changed to `update`.

- Deprecated validation methods such as `validates_presence_of :name` needed updating to `validates :name, presence: true` sort of changes.

As for the views, the gems I used (`haml-rails`, `redcarpet`, `simple_form`, `cocoon`, etc) are all still actively maintained and required no changes at all — a very pleasant surprise.

### Controllers

Next up were the controllers. Thanks to my use of service objects for the bulk of complex logic, migrating controllers was mostly painless. Rails 8 is stricter, and that’s where most of my changes were required. While bringing over controllers, I also began migrating the service objects they called upon, which are POROs and required almost no changes.

The main updates included:

- Params passed to background jobs now must be valid JSON. This was especially relevant for controllers triggering email or background tasks.
- Controller `filter`s were renamed to `action`s (`before_filter` → `before_action`).
- Controller hooks like `before_action` should come after the method they reference, not at the top of the file (a style guideline enforced in modern Rails apps).
- `redirect_to :back` was replaced with `redirect_back fallback_location: root_path` (or an appropriate fallback path).
- The `responders` gem is no longer included in Rails, so `respond_with` calls were rewritten using `respond_to` blocks.

All in all, the MVC migration was more about cleaning things up and embracing modern Rails conventions rather than fighting major incompatibilities.

## JavaScript Modernization

This is where things started to get spicy — the kind of spicy where you wonder if you just bit off more than you can chew. So much has changed in the front-end assets side of things. JuggleBee leverages **jQuery** (thankfully I dodged the *CoffeeScript* bullet) with Sprockets as the asset pipeline. Modern Rails favours `importmaps` as the default asset manager and pipeline, and keeping with the spirit of rejuvenating JuggleBee and bringing it up to speed, I signed up to it too. This allowed me to replace a bunch of gems which added various front-end features, such as `chartkick`, `masonry` and `bootstrap` with the importmaps `pin` alternative. Some libraries had to remain as CDN imports in my layout file, due to the order at which these JavaScripts got initialized and loaded by the DOM.

My suite of JavaScript classes also needed a bit of love and attention. The Sprockets
```javascript
//= require
```
statements were replaced with
```javascript
import
```

and my classes now had to `export` themselves so that they could be loaded as ES modules. All the files had to be relocated from `app/assets/javascripts/` to `app/javascript/`. The way we add our JavaScript base file into our application view layout also had to change slightly, going from
```erb
<%= javascript_include_tag "application" %>
```
to
```erb
<%= javascript_importmap_tags %>
```

Even though `jQuery` is considered a bit long in the tooth, given the plethora of really solid UI frameworks like Vue and React, I opted to keep my reliance on it for the time being. It still works, is actively maintained, and trying to replace it now would be a whole new kettle of fish. An adventure for another day.

## Background Jobs & Scheduling

This area was one of the biggest wins in the entire migration. JuggleBee had the “classic” background processing setup: **Sidekiq + Redis + whenever** (for cron). It worked fine for years, but it always felt a little… heavy, like lugging around a toolbox just to tighten a single screw. Sidekiq required its own container, Redis was a hungry beast that always felt like it was asking for more memory, and `whenever` meant fiddling with cron jobs and ensuring everything stayed in sync across deploys. It worked, but it was yet another moving part in an already aging stack.

Enter **SolidQueue** — a gem that ships with Rails 8, built to replace Sidekiq-like setups with something that is both simpler and more tightly integrated with ActiveJob. The real beauty? **No Redis.** SolidQueue uses the same database you already have (Postgres in our case) to manage job queues, which meant I could drop an entire service and reclaim a chunk of server memory.

Here’s what this meant for JuggleBee:
- No more Sidekiq configuration, container management, or Redis headaches.
- The `whenever` gem (used for cron scheduling) could be thrown out too, thanks to SolidQueue’s built-in `recurring.yml`.
- Background jobs now feel like a natural part of Rails — no extra glue required.

To put it in perspective, my stack went from:
```text
Rails App + Sidekiq + Redis + whenever + cron
```

**to**:
```text
Rails App + SolidQueue
```

That’s it. Beautiful, minimal, and a lot less to maintain.

Another *chef’s kiss* moment was how little code I had to change. Most jobs migrated over with minimal tweaks — the biggest adjustment being that job arguments **must be JSON-serializable**. This meant replacing keyword arguments with hashes like:
```ruby
EmailJob.perform("MyEmailClass", {"from" => current_user.name, "subject" => "Hello there"})
```
instead of:
```ruby
EmailJob.perform("MyEmailClass", from: current_user.name, subject: "Hello there")
```

It was one of those rare upgrades where less really is more. Dropping Redis and cron alone made this whole migration feel worth it.

## Infrastructure & Deployment

For years, deployment was powered by a custom shell script I wrote. It wasn’t flashy, but it was reliable and got the job done. It handled atomic releases, cleaned up old deployments, built Docker images, restarted containers, and even maintained a cache for faster updates. For a single-developer project like JuggleBee, it was the perfect balance of simplicity and control.

Here’s the core of that script:

```bash
#!/bin/bash

branch=${1:-master}
cwd=/home/ubuntu
repo_url=your_repo_here
revision=master
keep=5
timestamp=$(date +%s)
cache=$cwd/.git_cache
releases=$cwd/releases
mkdir -p $releases
previous=$releases/$(ls $releases/ -t | head -n 1)
release=$releases/$timestamp

if [ ! -d $cache ] ; then
        git clone $repo_url $cache
fi

echo "downloading latest source code from $branch..."
mkdir -p $release
(cd $cache && git gc --prune=now && git remote prune origin && git fetch origin && git reset --hard origin/$branch && git archive $revision | tar -x -c $release)

echo "release timestamp is: $timestamp ($release)"

echo 'building docker images...'
/bin/bash -c "(cd $release && docker compose -f docker-production.yml build)"

if [ $? -eq 0 ]
then
        if [[ $previous != "" ]] ; then
                echo 'stopping previous containers...'
                /bin/bash -c "(cd $previous && docker compose -f docker-production.yml stop && docker ps -a -q --filter='status=exited' | xargs docker rm)"
        fi

        echo 'restarting application...'
        /bin/bash -c "(cd $release && docker compose -f docker-production.yml up -d)"

        echo 'deleting old releases...'
        (cd $releases && (ls -t | head -n $keep; ls) | sort | uniq -u | xargs rm -rf)

        echo 'creating symlink.'
        rm $cwd/current-release
        ln -s $release $cwd/current-release

        echo 'deploy complete.'
fi
```

Honestly, this script could have kept soldiering on for years… until **Kamal** walked in like the cool new kid who makes your old tricks look ancient. It wasn’t broken — far from it — but Kamal is like having a personal DevOps engineer in your terminal. It takes all the things I cared about (Dockerized deploys, atomic releases, rollbacks) and adds extra superpowers I didn’t even realize I needed.

## Introducing Kamal 2

Kamal is a server provisioning and deployment tool built specifically (but not limited to) Rails apps. It automates a ton of tasks that my script didn’t — and one of the most significant changes? **No more Nginx.**

Kamal ships with a built-in Traefik proxy that handles all your routing and SSL certificates out of the box. No more tinkering with `nginx.conf` files or restarting a separate webserver. One less dependency chewing through RAM and one less thing to break at 2 am.

Here’s what Kamal brings to the table:

- **`kamal setup`**: installs Docker, Git, and all required dependencies on your server.
- **`kamal deploy`**: builds your Rails app into a Docker image, pushes it to your registry, and runs it on the server.
- **Traefik proxy with SSL (via Let’s Encrypt)**: you get HTTPS without lifting a finger.
- **Accessory management**: want Postgres or Redis? Kamal spins them up as Docker containers.
- **Scaling**: need more capacity? add another container in deploy.yml and Kamal will route traffic automatically.

The impact of this was huge. I went from managing Docker builds, old release directories, and an Nginx webserver, to just running one command. There’s something almost unsettling about how smooth it is — I felt like I was missing steps at first, like forgetting my keys when leaving the house.

> Wait, that’s it? Did I actually deploy? Did I forget something? Oh… it’s done already. Huh.

This wasn’t just a convenience win — it was a dependency detox. Between dropping Nginx here and killing Redis with SolidQueue earlier, my infrastructure got a lot lighter, faster, and cheaper to run. Kamal didn’t just evolve my old script — it rendered half of its logic unnecessary.

## Half Way There

With the new Rails 8 foundation in place, JuggleBee has already shed a lot of its old skin. We’ve modernized the models and controllers, swapped out a creaky asset pipeline for importmaps, retired Sidekiq, Redis, and cron in favor of SolidQueue, and traded a homegrown deployment script (solid as it was) for the convenience and power of Kamal 2. The app is lighter, faster to deploy, and much easier to maintain — but we’re not done yet.

Getting the Rails 8 app running and deployable was only **half the battle**. Now comes the rest of the journey:
- Migrating data from a legacy PostgreSQL 9.6 database to a fresh PostgreSQL 17.5 instance.
- Replacing CarrierWave with ActiveStorage and migrating all user-uploaded files into a new S3 bucket.
- Locking down credentials with Rails encrypted secrets and Bitwarden CLI.
- Cleaning up leftover code smells, finalizing test coverage, and polishing the overall performance of the new stack.

That’s some of what Part 2 will cover — all the finishing touches, the unexpected hurdles, and the final steps that took JuggleBee from a Rails 4 relic to a lean, modern, and production-ready Rails 8 app.

**Stay tuned for Part 2, where the migration story continues.**
