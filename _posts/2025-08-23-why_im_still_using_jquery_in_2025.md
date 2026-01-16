---
layout: post
title: "Why I'm Still Using jQuery in 2025 (And Not Even Sorry)"
crawlertitle: "A Pragmatic Look at Keeping jQuery in a Modern Rails 8 App"
summary: "Why I chose to keep our custom jQuery framework alive in a modern Rails 8 world — and how it still holds up."
date: 2025-08-19
categories: posts
tags: ['JuggleBee', 'Javascript', 'jQuery']
bg: "post-jquery2025.png"
---

While grinding through the monumental JuggleBee upgrade, one of the recurring debates was: what do we modernize, and what do we leave the heck alone? Plenty got a shiny new coat of paint (see my earlier posts for all the juicy details), but one area I deliberately left untouched was the frontend JavaScript. These days, jQuery is seen as a relic of a bygone era — a noble steed well past its sell-by date. Hotwire now ships as the Rails 8 default, and heavyweight contenders like React and Vue have taken over the frontend spotlight.

## Why I Kept It

So why didn’t I replace it with Hotwire or Stimulus when doing the big Rails 8 overhaul?

Because honestly... it still *slaps*.

Sure, jQuery has fallen out of favour, and yes, I could’ve followed the modern path. But here’s the reality:

* **It still works. Flawlessly.**
* **The architecture is solid.** It keeps behavior modular and scoped. No random scripts firing across unrelated pages.
* **It’s expressive and readable.** You can look at the markup and know exactly which components will be applied.
* **Rewriting it would’ve cost me weeks** — and yielded very little gain for the effort.

It’s easy to chase shiny new frameworks. But this setup? It’s already solved the very problems frameworks like Stimulus are trying to address — just in our own way, before they existed.

> "Technical debt isn't defined by age. It's defined by pain."

This jQuery-based framework causes me **zero** pain. It’s clean, consistent, and extendable. And I’m not paying a complexity tax for keeping it around.

## A Beautiful Framework

Back in the yonder years, when development on JuggleBee first commenced, jQuery was king. "Write less, do more" as the slogan proudly declared. These were the days before Turbo, pre-SPAs, when frameworks like React were still internal tools being tinkered on at Facebook. Even then, I liked my code clean, purposeful, and targeted. Every function should do its job — and only its job — exactly where it’s meant to. The typical jQuery spaghetti of the time grated on me.

Take this scenario: I’ve got a lovely `Select2` dropdown on the listings page to filter by category. All good. But why is that whole library still being pulled in and initialized on the About Us page, which has no dropdowns in sight? This was a symptom of how jQuery scripts were commonly written — everything ran everywhere. I wanted modularity, not mayhem. And so, the “Behaviour Framework” was born.

So what exactly did we build?

In essence, it's a lightweight, declarative behaviour system — think of it like a primitive version of Stimulus or Alpine. Components declare themselves in the markup, and a central engine takes care of the wiring when the DOM loads. This means zero manual JS in our views, and zero unnecessary script execution across unrelated pages.

Let me show you how it works...

At the heart of it all is the `Behaviour` class. Its role is simple but powerful: register UI components and apply them only when needed — and only on the elements that require them. Here's a high-level look at how it works under the hood:

```javascript
// Note: classes didnt exist so the "Class" we're calling below is a custom implementation
Framework.Behaviour = new Class({
  filters: {},
  addFilter:function(name, instantiator){
    this.filters[name] = instantiator;
  },
  addFilters:function(filters){
    for (const filter in filters){
      this.addFilter(filter, filters[filter]);
    }
  },
  apply:function(wrapper){
    this._scan(wrapper, $.proxy(this._apply, this));
  },
  unapply:function(wrapper){
    this._scan(wrapper, function(target, behaviourClassName){
      var behaviourResult = target.data('getBehaviourResult')(behaviourClassName);
      if (typeof behaviourResult.deinit === 'function'){
        behaviourResult.deinit();
      }
    });
  },
  // rest of the implementation ...
});
```

This allows us to define small, focused components — like a date picker — in total isolation. No global bleed. No script soup. Just self-contained behaviour.

One of the earliest components we wrote using this system was actually said date picker. At the time, we were using Bootstrap and jQuery UI plugins — each with their own quirks. So we wrapped them up into their own isolated modules, making them easier to configure and reuse across different forms, pages, and even admin views.

Here’s what the class for that looked like:

```javascript
Component.DatePicker = new Class({
    options: {
        startDate: '04/07/2015',
        format: 'dd/mm/yyyy',
        todayHighlight: false,
        weekStart: 0
    },
    init:function(element, options){
        this.setOptions(options);
        element.datepicker(this.options);
    }
});
```

Once defined, we simply register our component with the behaviour system so it knows how to wire it up when the time comes:

```javascript
var App = {
  behaviour: new Framework.Behaviour
};

App.behaviour.addFilters({
  'Component.DatePicker': {
    setup: function (element, options) {
      return new Component.DatePicker(element, options);
    }
  }
)

$(function(){
  var body = $('body');
  App.behaviour.apply(body);
})
```

Now comes the beauty of it — in your HAML or HTML, you declare the behaviour directly in your markup using a `data-behaviour` attribute. When the page loads, only the relevant components are instantiated, and only for the elements that need them.

```haml
  = listing.input :start_date,
    as: :string,
    label: 'Start Date',
    placeholder: "e.g. #{example}",
    input_html: { value: (localize(date, format: '%d/%m/%Y')),
      data: {behaviour: 'Component.DatePicker', 'options-component.datepicker' => '{"startDate" : "' + example + '"}'}
```

And just like that, our behaviours are neatly attached, scoped, and easy to manage.

It’s clean. It’s obvious. And it means onboarding a new team member doesn’t require a 2-hour walkthrough of where the JavaScript magic lives — it’s all in the markup and component files.

This approach meant zero wasted cycles, faster page loads, and a far more maintainable codebase. Behaviourally-aware pages, declarative setup, and modular control over every UI component. In today’s parlance, it’s not far off what Stimulus or Vue’s component system offers — just built years earlier and tailored to our exact needs.

This is the short version — enough to give you a taste of how it all works. In a follow-up post, I’ll take a deep dive into the full implementation, including sibling systems like the `EventBus`, `Trigger` logic, Adapters, and more.

Credit where it’s due: this wasn’t a solo effort. The framework was shaped with the brilliant Paul Schwarz, and together we built something that — dare I say — still holds up surprisingly well in 2025.

## jQuery Isn’t Dead. It’s Just Retired.

Do I recommend jQuery for new apps in 2025? Of course not.
But for apps that already have it — and especially those with a system like this layered on top — there's no real reason to tear it out just to chase trends.

This is the kind of code you keep *because* it works — not in spite of it.

## Coming Up Next

In Part 2, I’ll take you on a deep dive into the guts of the Behaviour framework: how components are hydrated, how the `EventBus` enables decoupled comms between modules, how `Trigger` handles custom DOM events, and more.

It’s a little old-school, sure — but it's also still slick as hell.
