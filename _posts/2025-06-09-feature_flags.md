---
layout: post
title: Feature Flags | Controlled Chaos in Production
crawlertitle: Feature Flags | Controlled Chaos in Production
summary: Simple feature flag management
date: 2025-06-06
categories: posts
tags: ['feature flags']
bg: "feature_flags.jpg"
---
Feature flags are a way of life when developing software. They let you deploy code to production without immediately exposing it to users — like flipping a switch without blowing the fuse.

In practice, they're used to:

- Ship features incrementally
- Test changes in production with internal users
- Run A/B tests or multivariate experiments
- Deliver bespoke functionality to specific customers (not my favourite approach, but hey, sometimes we have to pick our battles)

Feature flags introduce flexibility — but also the responsibility to manage them well. Left unchecked, they can become a tangle of forgotten toggles, quietly rotting your codebase from within.

Let’s take a look at one of the more robust feature flag services out there.

## LaunchDarkly
One of the more popular services (at least in my limited experience) is [LaunchDarkly](https://launchdarkly.com/). It allows you to create and manage feature flags, define targeting rules (e.g., which users get what), and view flag usage analytics — including which flags are no longer being used.

That last bit is more useful than it sounds. Stale flags can linger for months or even years, adding dead weight to your app and making the logic harder to follow. Being able to identify and prune them is a big plus.

Here’s a basic integration pattern for using LaunchDarkly in a Rails app:

```ruby
module FeatureFlag
  class << self
    def for(entity)
      @entity = entity
      self
    end

    def feature_a? = check_for("feature-a", default: true) # this default will make sense shortly
    def feature_b? = check_for("feature-b")
    def feature_c? = true # hard-coded flag (sometimes needed)

    private

    attr_reader :entity

    def check_for(flag_name, default: false)
      return false unless entity.present?

      ld_client.variation(flag_name, entity_context(entity), default)
    end

    def ld_client
      @ld_client ||= Rails.configuration.ld_client
    end

    def entity_context(entity)
      LaunchDarkly::LDContext.create(
        {
          key: entity.identifier, # a unique identifier for the entity
          kind: entity.type       # usually "user", but could be "account", etc.
        }
      )
    end
  end
end
```

This pattern gives you a central place to check feature availability in your app. You might wire this into your controllers, services, or views like so:

```ruby
if FeatureFlag.for(current_user).feature_a?
  render :new_feature_version
else
  render :legacy_version
end
```

It’s readable, testable, and keeps your flag logic out of the weeds.

## Handling Flags in Test and Development Environments
Since LaunchDarkly is an external service that communicates via HTTP, you definitely don’t want your test suite making live requests every time it runs. That’s just asking for flakiness and slow feedback loops.

Instead, we can use a mock client in the test environment to simulate feature flag behavior without hitting the actual API. Here’s a simple example I popped into our `lib` folder:

```ruby
module LaunchDarkly
  class MockLdClient
    def initialize(*) = true

    def initialized? = true

    def variation(_key, _user, default) = default
  end
end
```

This mock client always returns the `default` value passed into the `variation` method — making it trivial to control feature behavior explicitly in your tests.

But wait — why not just put LaunchDarkly into `"offline"` mode in the test environment, like we do below for development?

Good question. While `offline: true` does prevent real network calls, it still requires the full LaunchDarkly SDK to be initialized — which adds unnecessary overhead and dependencies to your test suite. By using a lightweight mock client instead, you get a few key benefits:

- **Faster tests** — no SDK initialization, no config loading, just a plain Ruby object
- **Zero external dependencies** — the mock removes the need to load the LaunchDarkly gem at all in tests
- **Simpler control** — it's easier to stub or extend the mock for edge cases if needed
- **Isolation** — your tests won't break due to SDK upgrades or changes in default behaviors

In short, the mock client keeps your test environment lean and focused. It’s a small abstraction that pays for itself in speed and stability.

With that in place, in your LaunchDarkly initializer (`config/initializers/launch_darkly.rb`), you can wire it in like so:

```ruby
require "#{Rails.root}/lib/launch_darkly/mock_ld_client.rb"

case Rails.env
when "test"
  Rails.configuration.ld_client = LaunchDarkly::MockLdClient.new("FAKE API KEY")
when "development"
  config = LaunchDarkly::Config.new(offline: true)
  Rails.configuration.ld_client = LaunchDarkly::LDClient.new(ENV['LD_KEY'], config)
else
  Rails.configuration.ld_client = LaunchDarkly::LDClient.new(ENV['LD_KEY'])
end
```

In the **test** environment, we avoid any HTTP calls by using the mock client.

In **development**, we set LaunchDarkly into `offline` mode. This disables all outbound network traffic and ensures your app doesn't hang or fail due to a misconfigured API key. You can still control the outcome of flags by setting the default values in your FeatureFlag methods — which is usually more than enough for local dev.

This setup gives you full control and isolation in non-prod environments, with zero reliance on external calls. Win-win.

## Some Other Options
While this post focuses on LaunchDarkly, there are plenty of other options — some open-source, some more opinionated, and some simpler to get going with.

Here are a few worth considering:

- [Flipper](https://github.com/flippercloud/flipper) – Powerful, flexible, and actively maintained. Can use a variety of backends (Redis, ActiveRecord, etc.) and integrates with a slick UI via Flipper Cloud.
- [Rollout](https://github.com/FetLife/rollout) – A battle-tested gem that’s been around for a while. Simpler API, great for percentage rollouts and user targeting.
- [Flipflop](https://github.com/voormedia/flipflop) – Great for toggling features via a web UI. Good for admin-driven flags or feature previews.

Which one to use depends on your needs — whether you want self-hosted vs. SaaS, need advanced targeting and analytics, or just want a simple way to toggle features in development.

## Parting Thoughts
Feature flags give you control, flexibility, and faster feedback loops — but like anything powerful, they need to be managed responsibly. Treat them like temporary scaffolding, not permanent fixtures. Set reminders to clean them up. Document what each flag does and who owns it. And when possible, limit the number of active flags in your system at a given time.

If you’ve had war stories with forgotten flags or misconfigured rollouts, I’d love to hear them. Otherwise, happy toggling.
