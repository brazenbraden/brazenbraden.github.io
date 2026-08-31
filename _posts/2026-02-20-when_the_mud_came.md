---
layout: post
title: "When The Mud Came: My First Proper Dive into Game Development"
crawlertitle: "What I Learned Building My First Game Prototype"
summary: "How a long-held ambition became When The Mud Came: a relaxing cleaning game, a playable Unity prototype, and my first proper dive into game development."
date: 2026-02-20
categories: posts
tags: ['Game Dev', 'Unity', 'When The Mud Came']
bg: "when_the_mud_came.png"
---

I’ve always wanted to make a game. In fact, games are what first pulled me towards software development in the first place. But there is a rather large gap between *wanting* to make a game and finding the time, energy, and frankly ridiculous range of skills needed to actually build one.

Bills need paying. Careers take shape. Families expand. Before you know it, that game idea has spent another decade sitting quietly in the back of your head, patiently waiting for you to stop making excuses.

Midway through 2025, I found myself with a suddenly much emptier calendar and decided it was finally time to have a go. Not to build the next sprawling open-world masterpiece or begin a twelve-year early-access campaign, but to make something tangible. Something playable. Something that proved I could take an idea beyond a folder full of half-finished experiments.

That something became *When The Mud Came*.

<img src="{{site.images}}/whenthemudcame/intro/main_menu.png" alt="When The Mud Came main menu" />

This is the story of the first few months of that project: what I wanted it to be, how the prototype took shape, and what building a game taught me about the sizeable gulf between making systems work and making a world feel good to play in.

---

## Building a “dad game”

We all have our preferred gaming experiences. I’ve traditionally been an FPS guy — popping heads down scope while hidden in the shadows. Then my family grew, my free time shrank, and the ability to disappear beneath a pair of headphones for an uninterrupted evening became something of a historical curiosity.

I found myself drifting towards slower, more satisfying games. Games without constant time pressure, where progress is visible and I can stop playing without abandoning eleven strangers halfway through a competitive match.

I started calling them “dad games”.

The idea behind *When The Mud Came* grew from that change. I wanted something you could play with your favourite music on, or a television show burbling away in the background. Something built around the satisfaction of cleaning, steady incremental upgrades, and watching a lifeless world gradually recover because of your work.

The obvious inspiration was *PowerWash Simulator*, combined with the visible world-changing progression of *Planet Crafter* and the reliable little dopamine taps of incremental games. The core loop became:

1. Clean muck from the environment.
2. Earn money for cleaning.
3. Spend that money on better equipment and abilities.
4. Reach places and clean things that were previously inaccessible.
5. Watch colour and life return to the world.

Simple enough to explain. Much less simple to build.

## Then the mud came

The player begins stuck in a painfully familiar routine: wake up, go to work, eat a TV dinner, repeat.

Then the news reports that a mysterious volcano has appeared from nowhere and covered half a continent in thick brown sludge. Towns are evacuated, dust fills the air, and conveniently timed adverts begin promising good money for anyone willing to help with the cleanup.

You happen to have an old power washer in the garage. Maybe this is life finally throwing you a nickel. You load up the truck and head towards the volcano.

The first stop is a roadside services area on Route 66, containing a petrol station, shop, motel, bus stop, and children’s play area. It became the setting for the prototype and a small demonstration of the wider idea. Newspapers tell the public story of the disaster, scientific journals investigate the mud itself, and notes from a slightly unhinged stranger suggest that something less obvious may be lurking behind it all.

Is the volcano alien? The result of a black-site experiment? A workplace health and safety incident that has become profoundly difficult to hide?

All perfectly reasonable possibilities.

The story gave the cleaning a destination, but the real test was always the loop itself. If spraying muck from a wall wasn’t satisfying, no amount of mysterious volcano lore was going to save it.

---

## Making progress visible

The Route 66 map begins brown, dirty, and trapped beneath a dust storm. As the player cleans each zone, the environment slowly comes back to life. The dust clears, clouds lift, sunlight breaks through, and greenery returns.

<div style="display: flex">
  <img src="{{site.images}}/whenthemudcame/intro/full_dirty.png" alt="Route 66 covered in mud" style="width: 50%;" />
  <img src="{{site.images}}/whenthemudcame/intro/full_clean.png" alt="Route 66 after being cleaned" style="width: 50%;" />
</div>

That transformation is central to the game. A progress bar can tell you that an area is 73% complete, but the world itself should be the better progress bar. The player should be able to stop, look around, and immediately see the difference they have made.

Cleaning also earns money for upgrades. Some improve the power washer, making the work quicker or more efficient. Others unlock physical abilities such as jumping, double jumping, or “dirt vision” for finding those last stubborn flecks of muck that have apparently signed a long-term lease on the underside of a bench.

The prototype had a complete beginning and end. Clean the full location, find the required story material, unlock the vehicle, and leave through a short cutscene. It was rough, inconsistent, and used placeholder assets in places, but it existed. More importantly, it was fully playable.

That distinction mattered to me. It wasn’t a polished vertical slice and it certainly wasn’t content-complete. It was a test of three things:

- Is cleaning the muck satisfying?
- Does watching colour return to the world feel rewarding?
- Do the upgrades make you want to clean just one more thing?

Everything else hung off those questions.

## From a level to a place

Once the core loop worked, a different class of problem became obvious. The map did the right things mechanically, but it didn’t always communicate them well.

Invisible walls stopped the player without explanation. The road ended at the edge of the playable area. The surrounding environment dropped away too sharply, reminding you that this was a box containing some game objects rather than a small part of a larger world.

The next phase wasn’t about adding more mechanics. It was about making the existing space feel deliberate.

I expanded the terrain beyond the playable boundary, continued the road into the distance, added roadside trees, and placed the volcano on the horizon. Fences and physical barriers gave the invisible boundaries a reason to exist. Greenery generation became more selective so that roads looked like infrastructure rather than unusually flat gardens.

![The mud volcano visible beyond Route 66]({{site.images}}/whenthemudcame/v0.2/volcano.png)

The player still couldn’t travel beyond the level, but now the world appeared to continue without them.

Lighting, fog, and atmosphere received the same attention. Fog moved across the ground instead of sitting motionless. Additional lights helped guide the eye and gave the scene more shape. Cleaned zones received extra assets so they felt transformed rather than merely meeting a number behind the scenes.

Individually, these were small changes. Together, they did an enormous amount of work.

That was one of my first significant game-development lessons: correctness isn’t enough. A boundary can function perfectly and still feel wrong. A zone can technically be complete and still look unfinished. The game knows what happened, but unless it communicates that through visuals, sound, movement, and feedback, the player doesn’t care.

Or, put more simply: if the player reaches 100% and doesn’t *feel* it, something’s wrong.

---

## Removing the clever one-offs

While I was wrestling with world-building, another very familiar problem was quietly forming in the code.

Early in a project, hard-coding is tempting. Sometimes it’s even the correct choice. You need to find out whether an idea works before spending three weeks designing a framework for it.

Want the game to end when the map is clean? Add a conditional. Want a readable to unlock something? Add another conditional. Want a notification for a secret? Fire a toaster and move on.

This helped get the prototype into a playable state, but every new feature began asking for its own logic, rules, and UI glue. The game was becoming a collection of clever one-offs, and clever one-offs have a nasty habit of introducing their extended families once you’re not looking.

This was where my regular software-development instincts became useful.

### Interactions became triggers

Originally, interacting meant pressing a button to read some text. That worked for journals and newspapers, but it didn’t provide much of a foundation for a responsive world.

I rebuilt interaction so that it represented a trigger rather than a fixed outcome. An interactive object could now toggle lights, play an animation, emit particles, begin a transition, or dispose of itself after one use. New behaviour could plug into an existing pipeline instead of demanding a completely new code path.

<div style="display: flex">
  <img src="{{site.images}}/whenthemudcame/v0.3/interact1.jpg" alt="Interacting with an object in When The Mud Came" style="width: 50%;" />
  <img src="{{site.images}}/whenthemudcame/v0.3/interact2.jpg" alt="An interaction triggering movement in the world" style="width: 50%;" />
</div>

The individual effects weren’t the important bit. The important bit was that interaction had become a reusable idea. That is the difference between a prop you can click and a world that can respond.

### The endgame became a quest

The prototype technically had a quest, although only in the loosest possible sense. Reach 100% clean, read the correct documents, unlock the vehicle, and finish the game. It was completely bespoke and impossible to reuse without copying the whole flow.

I replaced it with a generic quest system composed from tasks, definitions, conditions, and sequences. Instead of hard-coding what happens, the system describes what progress looks like.

![The reusable quest system in action]({{site.images}}/whenthemudcame/v0.3/quest.jpg)

The original ending then became just another quest. I also added a side quest to prove the structure could support something beyond its first use case.

Notifications went through a similar cleanup. Several narrowly focused feedback systems became one generic event-driven layer. Interactions and quests could raise customised notifications without each inventing a new way to tell the player something had happened.

None of this produced the sort of screenshot that makes people gasp at a release announcement. It was a capability update rather than a content update. But it gave future-me a fighting chance of adding stories, objectives, and reactive objects without the project collapsing beneath accumulated special cases.

Players should never need to know that architecture exists. Ideally, they only notice that the world behaves consistently.

## Learning outside my lane

I came into the project after more than a decade of backend and web development. Architecture, state, data, tests, and maintainable systems were familiar territory. Art, 3D modelling, shaders, particles, lighting, animation, audio, and the numerous dark arts involved in making a virtual surface convincingly dirty were not.

That was precisely the point.

Working in an unfamiliar discipline forced me to look at familiar engineering principles from a different direction. Small, incremental changes still mattered. Reusable systems still mattered. Avoiding premature abstraction still mattered. But games add another test that normal business software doesn’t face in quite the same way:

> Does this *feel* right?

A technically elegant cleaning shader is useless if cleaning feels fiddly. A generic interaction system is valuable only if it helps the world respond clearly. A perfectly calculated completion state falls flat if the player can’t see or hear the result.

The project also reinforced the value of building the smallest complete thing. A finished prototype taught me more than a grand design document ever could. It exposed the rough surfaces, awkward boundaries, missing feedback, and architectural shortcuts because I could actually play through them rather than imagining how they might work one day.

## Where the mud settled

*When The Mud Came* grew from a long-held ambition into a playable prototype with a complete loop, an evolving world, progression, story, interactions, quests, and the beginnings of an architecture that could support something larger.

It is not finished. There were—and still are—plenty of places it could go: additional locations, deeper upgrades, new cleaning mechanics, more cohesive stories, puzzles, and secrets. There is also an enormous amount of polish between a promising prototype and a game ready for a proper release.

For now, the project is resting. That doesn’t mean it has been abandoned or that those months were wasted. Quite the opposite. It proved that I could build a game, taught me an entirely new set of disciplines, and gave me a much clearer understanding of the kind of games I want to make.

Perhaps I’ll return to the mud one day. I hope I do. But even if its next chapter takes a while to arrive, it has already done something important: it turned “I’ve always wanted to make a game” into “I made one.”

And that feels like a pretty good place to leave the truck parked for now.

The prototype and its original development updates can still be found on [itch.io](https://brazenbraden.itch.io/when-the-mud-came).
