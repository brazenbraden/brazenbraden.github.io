---
layout: post
title: The jQuery Behaviour Framework That Refused to Die — A Deep Dive
crawlertitle: The Forgotten jQuery Framework Still Powering JuggleBee
summary: In 2015 we built a jQuery behaviour framework for JuggleBee. A decade later, it still runs in production — here’s how and why it’s worth remembering.
date: 2025-09-22
categories: posts
tags: ['jugglebee', 'jQuery', 'JavaScript']
bg: "post-javascript-behaviour-framework.png"
---
Back in 2015, when JuggleBee first came to life, the front-end landscape looked like another planet. React was just cutting its teeth, Vue was the “new kid” only a few people had heard of, and jQuery was still the Swiss Army knife everybody had in their pocket. If you wanted to toggle a menu or slap a fade effect on something, jQuery was your friend.

Out of that world came this small **JavaScript Behaviour framework**. It wasn’t built to compete with the heavyweights — it was built to solve a very specific itch I had: I wanted to apply targeted behaviours to components only if they actually existed on the page. Why should my app load the JavaScript for a fancy checkbox if there wasn’t a checkbox in sight? Back then, that kind of “pay only for what you use” thinking wasn’t baked into anything — so I rolled my own.

If you want to follow along in detail, the full source lives on GitHub here: [**brazenbraden/javascript_behaviour_framework**](https://github.com/brazenbraden/javascript_behaviour_framework). The blog breaks it down concept by concept, but the repo is where you’ll see all the code in one place. Note: this repo only includes a small subset of the components that are in active use by JuggleBee right now.

*Quick disclaimer before we go any further:* this framework is not something you should actually use in 2025 (seriously, don’t). Modern frameworks give you all of this and more with a lot less fuss. What follows is purely **historical documentation**: a look at how we solved front-end headaches a decade ago, with some light archaeology thrown in for fun.

---

## The Core Framework (& lib)

At the centre of it all are three small files that gave us the scaffolding:

1. **`Framework.Behaviour.js`** — a behaviour system that lets you attach little JavaScript classes to DOM elements using `data-behaviour`.
2. **`Framework.EventBus.js`** — a tiny pub/sub bus that let components talk to each other without knowing each other.
3. **`Framework.Trigger.js`** — a declarative way to wire up DOM events via `data-trigger`.

Supporting those are a handful of utility files in the `lib/` folder — `Class.js`, `Selectors.js`, `String.js`. Think of them as the duct tape that held it all together.

Let’s go through them one by one.

### Framework.Behaviour

If you’ve used Stimulus, this will feel oddly familiar — except Stimulus didn’t exist yet. With Behaviours, you drop an attribute like this into your markup:

```html
<div data-behaviour="Component.Toggle"></div>
```

Then, when you call `Framework.Behaviour.apply(document.body)`, the framework scans the DOM, finds that element, and wires up the matching component class. The mapping of `Component.Toggle` → actual JavaScript happens through something called a “filter”:

```js
Framework.Behaviour.addFilter('Component.Toggle', {
  setup: function(target, options) {
    // initialise behaviour here
    return { deinit: () => { /* teardown */ } };
  }
});
```

The clever part isn’t just the setup — it’s that the framework keeps track of what’s been applied. Each element gets a little data store (`appliedBehaviours`) you can query later:

```js
$('#my-div').data('getBehaviourResult')('Component.Toggle');
```

So you don’t just get automatic setup and teardown — you also get introspection. You can peek into an element and see which behaviours are hanging off it. It’s hacky (jQuery `$.data` was abused a bit), but it worked, and it gave us a primitive component system before components were the cool thing.

### Framework.EventBus

Ah yes, the EventBus. Nothing screams “2010s front-end” like a hand-rolled pub/sub system.

The idea is simple: instead of components calling each other directly, they throw events onto the bus, and anyone listening can react.

```js
const id = Framework.EventBus.subscribe('chat', 'message', (data) => {
  console.log('New chat message:', data);
});

Framework.EventBus.publish('chat', 'message', { text: "Hello world" });
```

Events are just `channel:event` strings (`chat:message` here). Underneath, it’s basically two objects: one to hold callbacks, one to hand out IDs so you can unsubscribe. Nothing fancy, but it decoupled things just enough that we weren’t constantly tangling components together.

Today you’d reach for RxJS, or use the browser’s built-in `EventTarget`, or just lean on your framework’s state management. Back then, this was our lifeline.

### Framework.Trigger

Behaviours are about attaching classes to DOM elements. Triggers are about wiring up DOM events declaratively.

Instead of writing a dozen `$('#button').on('click', ...)` calls, you drop attributes into your markup:

```html
<button
  data-trigger="event"
  data-trigger-options-event='{"channel":"ajax","event":"load","data":{"id":42}}'>
  Load item 42
</button>
```

And in JS:

```js
Framework.Trigger.register('click', 'event', function(e, element, options) {
  e.preventDefault();
  Framework.EventBus.publish(options.channel, options.event, options.data);
});
```

With one `Framework.Trigger.apply(document.body)`, you’ve got delegated listeners on your wrapper. Whenever an event bubbles up, it checks for `data-trigger` attributes and dispatches the right callback.

It’s like Angular directives if you squint — declarative, reusable, but lighter and scrappier. Perfect for the Bootstrap + jQuery era.

### The Libs

These little helpers filled gaps in JavaScript at the time:

- **Class.js** — a pre-ES6 “class” helper that sets defaults, merges options, calls `init`. Ugly now, but kept everything consistent.
- **Selectors.js** — added goodies like `:regex` selectors, a `$.findWrapper` for finding the nearest ancestor, and a couple of URL/query helpers.
- **String.js** — gave us `"Hello {name}".substitute({ name: "Braden" })`. Yes, basically reinventing template strings.

They’re tiny, but they kept showing up in every project until we finally bundled them in here.

### And That's Our Core

Individually, these files don’t look like much. Together, they gave us a skeleton to build on. Without it, the app would’ve been a swamp of one-off jQuery snippets. With it, we had structure: declarative attachment, decoupled messaging, tidy DOM wiring, and a few quality-of-life hacks. It wasn’t glamorous, but it saved our sanity.

---

## Components (and the Road to Adapters)

Once you’ve got behaviours, you need actual *things* to apply. Enter **components**. These are the reusable features you attach to DOM elements.

### Component.Tabs

The simplest one in the repo is `Component.Tabs`. All it does is hook into Bootstrap’s tab plugin:

```js
Component.Tabs = new Class({
  options: {
    tab$: '.nav > li > a'
  },
  init:function(element, options){
    this.setOptions(options);
    element.find(this.options.tab$).on('click', function(e){
      e.preventDefault();
      $(this).tab('show');
    });
  }
});
```

Drop it onto your markup like this:

```html
<div data-behaviour="Component.Tabs">
  <ul class="nav nav-tabs">
    <li><a href="#first">First tab</a></li>
    <li><a href="#second">Second tab</a></li>
  </ul>

  <div class="tab-content">
    <div id="first" class="tab-pane">Content A</div>
    <div id="second" class="tab-pane">Content B</div>
  </div>
</div>
```

That’s it — tabs just work. Declarative, simple, done.

### Component.DatePicker

So far our components have been pretty bare-bones. `Tabs` didn’t need any options at all. But one of the nice touches in this framework was that components could accept configuration via `options`, giving you flexibility without rewriting the component itself.

Here’s `Component.DatePicker`, which wraps a Bootstrap Datepicker:

```js
import Class from "lib/Class"

var Component = Component || {};

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

export default Component.DatePicker;
```

Notice how it defines a default `options` object, then merges in anything passed via the DOM before initialising the plugin. That means you can tweak behaviour per-instance directly from your markup.

For example, here’s what it looks like in plain HTML (converted from our Rails HAML):

```html
<div class="input string required birth_date">
  <label for="birth_date">Birth Date</label>
  <input type="text"
         name="user[birth_date]"
         id="birth_date"
         placeholder="e.g. 04/07/2015"
         value="04/07/2015"
         data-behaviour="Component.DatePicker"
         data-options-component.datepicker='{"startDate" : "04/07/2015"}'>
</div>
```

When `Framework.Behaviour.apply(document.body)` runs, it picks up the `data-behaviour` attribute, reads the extra JSON blob in `data-options-component.datepicker`, merges it into the defaults, and initialises the datepicker with the right settings.

This is where the framework starts to shine — you don’t just slap behaviours onto DOM elements, you can also **customise them declaratively**. It made the HTML self-describing, which was a big win for readability and maintainability at the time.

### Component.Notifier

But not all components are that trivial. A more interesting one is `Component.Notifier`, which incorporates the EventBus and delegates the actual UI work to a `notifier` object:

```js
Component.Notifier = new Class({
    notifier: null,
    eventBus: null,
    init:function(notifier, eventBus){
        this.notifier = notifier;
        this.eventBus = eventBus;
        this.initialiseHandlers();
    },
    initialiseHandlers:function(){
        this.eventBus.subscribe('notification', 'alert', $.proxy(this.notify, this));
    },
    notify:function(data){
        if (data){
            this.notifier.alert(data);
        }
    }
});
```

Two ideas show up here:

1. **Dependency injection** — we don’t hard-code a library; we accept a `notifier` implementation.
2. **Event-driven design** — the component listens on the EventBus for `notification:alert` and delegates the UI work to the `notifier`.

This naturally leads into **Adapters** — swappable wrappers that let us swap out implementations without disturbing the rest of the system.

---

## Adapters

Adapters are the bridge layer of the framework. They let us hide the messy details behind a stable, predictable interface, so we could hot-swap one implementation for another without touching the rest of the code. Sometimes that meant wrapping a library like Toastr, other times it meant rolling our own variations — the point was flexibility.

```js
Adapter.Toastr = new Class({
    options: { /* toastr config */ },
    init:function(){
      toastr.options = this.options;
    },
    alert:function(data){
        if(typeof toastr[data.type] == 'function' ){
            toastr[data.type](data.message, '', this._getOptions(data));
        }
    },
    _getOptions:function(data){
        return data.options?.sticky ? { timeOut: 0 } : {};
    }
});
```

Now, when the EventBus fires a `notification:alert`, `Component.Notifier` calls `notifier.alert(...)`, and the adapter decides what to do. Don’t like Toastr anymore? Swap it for `Adapter.Noty` or `Adapter.SweetAlert`. Nothing else changes.

This may look small, but it’s one of the most forward-thinking patterns in the whole framework. It gave us dependency inversion before we even knew that was the term.

---

## Modules

Components are tied to DOM elements. Adapters wrap libraries. **Modules** are for bigger browser-wide concerns.

Take `Module.History`, which wraps the HTML5 History API. Instead of every component fiddling with `pushState` and `popstate`, you centralise it:

```js
const historyModule = new Module.History({
  tab: (options) => { showTab(options.id); },
  modal: (options) => { openModal(options.slug); }
});

historyModule.start();

// Push a new state when switching tabs:
historyModule.change({ tab: { id: "billing" } }, "Billing", "/account/billing");
```

Now, when the user hits “back,” the module intercepts `popstate` and replays the correct UI state. It’s the same “wrap and tame” idea as adapters, but applied to browser APIs.

---

### Where It All Comes Together: App.js

All the bits and pieces we’ve looked at so far — behaviours, triggers, event bus, components, adapters, modules — don’t do much in isolation. They need a conductor. That’s what `App.js` is: the entrypoint that wires everything together and kicks the framework into life.

```js
var App = {
  behaviour: new Framework.Behaviour,
  trigger: new Framework.Trigger,
  eventBus: new Framework.EventBus,
  modules: {
    history: new Module.History({
      publish: function(options){
        App.eventBus.publish(options.channel, options.event, options.data);
      }
    })
  }
};
```

A few things are happening here:
- We create single instances of `Behaviour`, `Trigger`, and `EventBus`.
- We initialise global modules, like `History`, and plug them straight into the bus.
- Everything gets collected under one `App` object so the rest of the code has a consistent place to reach for them.

Then, when the DOM is ready, `initApp()` runs:

```js
$(function(){ initApp(); })

function initApp() {
  var body = $('body');
  App.trigger.apply(body);
  App.behaviour.apply(body);

  $.ajaxSetup({
    headers: { 'X-CSRF-Token': $('meta[name="csrf-token"]').attr('content') }
  });
}
```

That’s the moment the framework springs to life. It applies triggers and behaviours across the page, and even handles Rails’ CSRF token setup for Ajax calls. From there, the filters you registered (e.g. `Component.Tabs`, `Component.DatePicker`, `Component.Notifier`) are automatically bound to the right DOM nodes.

Finally, `App.js` is where you register behaviours and triggers globally:

```js
App.behaviour.addFilters({
  'Component.Notifier': {
    setup: function() {
      return new Component.Notifier(Adapter.Toastr, App.eventBus);
    }
  },
  'Component.Tabs': {
    setup: function(element, options) {
      return new Component.Tabs(element, options);
    }
  }
});

App.trigger.register('click', 'modal', function (e, element) {
  e.preventDefault();
  $(element.attr('data-target')).modal('show');
});
```

By the time all of that is done, the system is fully wired:
- DOM is decorated with behaviours.
- EventBus is ready to publish/subscribe.
- Triggers are listening for declarative `data-trigger` attributes.
- Modules like `History` are active.

In other words: `App.js` is the bootloader. It doesn’t contain any features itself, but it orchestrates the whole framework so the rest of the pieces can just do their jobs.

---

## Final Thoughts

### A Snapshot in Time

This whole framework is a time capsule. It’s from an era when everyone was duct-taping jQuery plugins together and then wondering why their apps turned into spaghetti. React and Vue weren’t mainstream yet, so if you wanted order, you built it yourself.

- **Behaviours** were our proto-components.
- **EventBus** was Redux before Redux.
- **Triggers** were our scrappy Angular directives.
- **Adapters and Modules** were our poor man’s DI and service layers.

It wasn’t “modern,” but it solved the exact same problems we’re still solving today: modularity, decoupling, swappability, and declarative markup.

### Why Bother in 2025?

So why am I dredging this up now? Two reasons.

First: because it still lives inside **JuggleBee**. When I upgraded the app to Rails 8, I had to pick my battles. This framework still worked, and rewriting it just to say we don’t use jQuery anymore would’ve been like ripping out the plumbing in your house just because copper pipes aren’t cool anymore.

Second: because it’s worth remembering. The tools look dated, but the instincts were spot on. Looking at this code today is like looking at cave paintings — rough around the edges, but you can see the DNA of the things we use now.

For me, it’s also a reminder that “good enough” architecture can quietly power an app for a decade, while entire hype cycles come and go. Sometimes the smartest move in a migration isn’t ripping everything out, it’s knowing what to leave alone.

### Credit Where It’s Due

And finally, credit where it’s due: this wasn’t all me. **Paul Schwarz** was deeply involved in shaping the behaviour system, and he later polished it into a standalone library called [**bee**](https://github.com/paulschwarz/bee).

Bee is like the glow-up version of what you’ve just read: declarative `data-bee` attributes, neat little filters, clean init/deinit semantics. If you want to see how these ideas look when they’re stripped down and packaged properly (instead of bolted into my messy app), check it out.

In other words: this framework may be a relic, but it’s a relic with lineage. And for JuggleBee, it’s still quietly doing its job — proof that sometimes the code you wrote in 2015 doesn’t need to be replaced, just understood.

If you’ve got your own long-forgotten frameworks, I’d love to hear about them — it’s always fun comparing notes from the trenches. Bonus points if it's still running in Production!
