Refactoring Isn’t Rewriting — Again
1. Introduction — Why this still needs saying

- Start with a short story or example — e.g. a team “refactoring” a feature and emerging six months later with something totally new (and broken).
- Set the premise: Refactoring is not a rewrite. It’s the art of improving structure without changing behaviour. Yet teams keep confusing the two.
- Tone: slightly exasperated but amused — “We’ve been over this for 20 years, yet somehow we still treat ‘refactor’ as a greenlight for ‘start over’.”

2. How we got here

- Trace the misunderstanding:
  1. Agile made “refactoring” sound glamorous.
  2. IDEs made “rename method” and “extract class” look like big work.
  3. Developers crave novelty — rewriting feels creative, refactoring feels like cleaning.
- Explain that the word refactor has been stretched beyond its original meaning: from small continuous improvements to full-on rewrites disguised as maintenance.

3. True refactoring — the art of restraint

- Define it properly:
  1. Small, incremental, reversible.
  2. Driven by tests and feedback.
  3. Goal: improve readability and maintainability, not add features.
- Bring in examples:
  1. Extracting service objects.
  2. Replacing conditionals with polymorphism.
  3. Introducing dependency injection to reduce coupling.
- Emphasise discipline — “You should be able to stop at any commit and the app still works.”

4. The rewrite temptation

- Discuss the “if I just start from scratch it’ll be cleaner” mindset.
- Explore why rewrites often fail:
  1. You throw away the hard-won domain knowledge baked into the messy old code.
  2. You underestimate hidden dependencies and edge cases.
  3. You stop delivering business value for months.
- Quote Joel Spolsky’s “Things You Should Never Do” as a touchstone — but add your own modern context (e.g., how even in 2025, we still repeat that mistake).

5. Refactoring as a continuous habit

- Contrast “refactoring sprint” with habitual tidying.
- Explain how you integrate refactoring:
  1. Before adding a new feature, clean up the area you’ll touch.
  2. Commit small, verifiable improvements.
  3. Use tests as guardrails — never refactor blind.
- Mention how this mindset turned JuggleBee’s Rails 4 → 8 upgrade from a scary rebuild into a structured, testable evolution.

6. Closing — A cleaner tomorrow, one class at a time

- End with a reminder:
  1. Refactoring is maintenance, not reinvention.
  2. The best refactors are invisible to the user but transformative to the developer.
- Optional punchline: “You know it was a good refactor when nobody notices — except the next developer who reads the code and doesn’t swear.”
