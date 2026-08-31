# AGENTS.md

## Project

This repository is Braden King's personal site and blog, published with Jekyll at `brazenbraden.com`. It uses a customised version of the Voyager theme.

- `index.md` is the CV/about page.
- `_posts/` contains the main "Ramblings" blog.
- `_whenthemudcame/` is an archived game-dev collection. Its source is intentionally retained, but it is not currently published.
- `_drafts/` contains unfinished writing; do not publish, rename, or substantially rewrite a draft unless asked.
- `_layouts/`, `_includes/`, `_sass/`, and `assets/` implement the site and theme.
- `_data/comments/` contains Staticman comments.

Preserve unrelated work. In particular, check `git status` before editing and treat untracked drafts as user-owned work.

## Local development

Install dependencies with:

```sh
bundle install
```

Preview the site with:

```sh
bundle exec jekyll serve
```

Build it as production with:

```sh
./build.sh
```

The generated `_site/` directory is ignored and should not be committed. For content, layout, configuration, JavaScript, or Sass changes, run the production build before handing off when dependencies are available.

## Content conventions

Main posts live at `_posts/YYYY-MM-DD-slug.md` and normally use this front matter:

```yaml
---
layout: post
title: "Visible post title"
crawlertitle: "Concise browser and crawler title"
summary: "Short social/share description"
date: YYYY-MM-DD
categories: posts
tags: ['tag']
bg: "image-in-assets-images.ext"
---
```

When The Mud Came entries use `categories: game` and backgrounds below `assets/images/whenthemudcame/`. The collection is currently archived with `output: false`; do not republish it unless asked. Keep `bg` relative to `assets/images/`. In post bodies, the established portable image form is:

```markdown
![Alt text]({{site.images}}/path/to/image.png)
```

Liquid, Markdown, and small amounts of inline HTML are all established patterns. Use fenced code blocks with a language identifier. Keep links descriptive, and use horizontal rules only where they help pace a long article.

## Writing voice

When drafting or editing in Braden's voice:

- Use British English: `modernise`, `favourite`, `behaviour`, and similar spellings.
- Write in the first person and ground claims in concrete work, decisions, bugs, measurements, or lessons learned.
- Lead with motivation and trade-offs, then explain implementation with practical examples.
- Prefer clear, conversational prose over corporate language or abstract thought-leadership language.
- Use dry humour and playful analogies as seasoning, especially around technical frustration; do not force a joke into every section.
- Be candid about mistakes and limitations. Explain what changed after evidence, review, staging, or runtime testing.
- Favour pragmatic, incremental, maintainable solutions. Newer technology is not automatically better, and older technology is not automatically technical debt.
- Use short emphatic sentences occasionally for rhythm, alongside fuller explanatory paragraphs.
- End by drawing out the practical lesson or next direction rather than merely repeating the introduction.

Technical Ramblings should be detailed enough to teach from: show representative code, name the constraints, and connect architecture to delivery or maintenance. Game-dev posts should connect implementation choices to player experience, world feel, scope, and design intent.

Preserve the author's personality during copy-editing. Fixing grammar or clarity should not flatten contractions, asides, rhetorical questions, or deliberately informal phrasing. Do not invent personal experiences, metrics, project history, or opinions; flag missing facts with a clear placeholder or ask for them.

## Site changes

- Prefer focused edits over broad theme rewrites.
- Maintain compatibility with GitHub Pages/Jekyll 3.x–4.x as constrained by `vouager.gemspec` and `Gemfile.lock`.
- Keep URLs stable. `_config.yml` uses `/:categories/:title/`, and the custom collection has its own dated permalink.
- `future: true` is intentional, so future-dated posts are included in builds.
- Comments depend on `page.comments` defaults and the Staticman data/includes; preserve that path unless the task explicitly changes comments.
- Do not edit generated files in `_site/`.
