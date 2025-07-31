---
bg: "fam.jpg"
layout: page
title: "BrazenBraden"
crawlertitle: "About BrazenBraden"
summary: "Who am I?"
---

## About Me

I’m a backend-focused software engineer with over a decade of experience, specialising in Ruby on Rails and the art of untangling legacy codebases. I’ve spent the last several years helping teams rescue brittle systems, introduce testing, and modernise their infrastructure without burning everything to the ground. I care deeply about clean architecture, developer happiness, and leaving codebases better than I found them.

Outside of work, I’m a dad of two, a lifelong gamer, and the kind of guy who gets weirdly excited about deleting thousands of lines of obsolete code.

---

## Links

* GitHub _(mostly private repos)_ -> [GitHub](https://github.com/brazenbraden)
* LinkedIn -> [LinkedIn](https://www.linkedin.com/in/bradenhmking/)
* JuggleBee -> [JuggleBee](https://www.jugglebee.com)

---

## Summary

Senior backend engineer specialising in Ruby, with a strong focus on legacy codebases, architectural refactors, and sustainable development practices. Over the past five years, I've helped teams untangle complex systems, introduce test coverage and modern conventions, and regain control of their development workflow. I'm particularly effective in environments where the code is messy, brittle, or under-documented — bringing clarity, structure, and calm to the chaos.

## Core Technologies

Ruby on Rails, PostgreSQL, Sidekiq/SolidQueue, JavaScript (ES6+), jQuery, Docker, Redis, AWS (S3, SQS), Kamal, CI/CD (CircleCI, GitHub Actions), ActiveStorage

---

## Work Experience

### Kallidus (Sapling Team)

**Position:** Senior Backend Ruby Engineer

**Duration:** July 2023 – May 2025

**Revamped and optimised the CircleCI pipeline for our test suite, reducing build times from ~50 minutes to under 20 minutes**

Restructured the pipeline from a single, heavily-parallelised job into multiple discrete jobs categorised by test type, which enabled quicker reruns of failed tests. Introduced fast-running code quality checks (Reek, Rubocop) early in the pipeline, and implemented test coverage enforcement to prevent untested code from merging. These improvements significantly reduced CI credit usage, accelerated developer feedback loops, and improved code quality across the board.

**Refactored a legacy reporting system into a clean, extensible framework using the Builder pattern**

Inherited complex, brittle code and incrementally transformed it into a robust, testable architecture. The new design standardised how reports were built and dramatically improved maintainability. A new report I implemented using this system was delivered in weeks, demonstrating how the refactor enabled faster development and cleaner handoff.

**Led the transition to a trunk-based development workflow, streamlining releases and improving collaboration**

Moved the team from a slow, single-branch QA process to weekly releases on the main branch, reducing QA overhead and enabling faster, more reliable delivery. Established git conventions for branch naming and commit messages, integrated with GitHub to generate automated release notes, and documented the new workflow to ensure adoption across the team.

---

### Lifestream (previously BigSofa Technologies Ltd)

**Position:** Backend Tech Lead

**Duration:** May 2017 – July 2023

**Re-architected a monolithic Rails platform into a scalable microservice-based system**

Led the transition from a tightly coupled legacy application to a modular API-driven architecture, retiring over 40,000 lines of legacy code. Designed and built a suite of microservices, including a language transcription engine (integrating multiple third-party providers), a Python-based facial recognition and anonymisation service using YOLO models, and a media processing pipeline for automated batch workflows.

**Strengthened platform infrastructure with improved authentication and data architecture**

Implemented a standalone identity provider and SSO gateway to centralise authentication across services. Introduced Neo4j to model media relationships for deeper client insights and improved dashboard performance by integrating DynamoDB for frequently queried data.

**Defined and documented team workflows to improve collaboration and delivery**

Introduced structured processes for pull requests, issue tracking, design reviews, and deployments, helping the team ship faster without sacrificing quality. Designed the team’s GitHub Projects setup for task scoping, sprint planning, and timeboxing. Facilitated weekly dev meetings to promote knowledge sharing and maintained a StackOverflow Teams knowledge base to preserve team knowledge and improve onboarding.

---

### JuggleBee  – [jugglebee.com](https://jugglebee.com)

**Position:** Cofounder and Lead Engineer

**Duration:** Mar 2014 – Present

**Designed and launched Namibia’s first online auction platform, now serving over 13,000 users**

Built JuggleBee from the ground up in collaboration with a business partner from the property auction world. Delivered a full-featured online marketplace where users can list both products and time-based auctions, complete with real-time bidding logic, secure checkout, and automated invoicing. Since launch, the platform has hosted over 12,000 listings and generated more than 5,000 invoices, establishing itself as a trusted digital marketplace in a previously offline-dominated industry.

**Overhauled and modernised the full JuggleBee stack to bring it from Rails 4 to Rails 8**

Led a comprehensive, production-safe upgrade of a legacy codebase spanning over a decade. Introduced modern Rails conventions, rewrote background processing from Sidekiq to SolidQueue, migrated file storage from CarrierWave to ActiveStorage with S3, and transitioned from a fragile VPS setup to a fully containerised Kamal deployment. Also rebuilt key areas of the app for maintainability, including controller refactors, credential management, and database migrations from PostgreSQL 9.4 to 17.

---

### Westcosoft

**Position:** Lead Software Engineer

**Duration:** Nov 2012 – Jan 2017

**Transitioned a sprawling desktop system to a modular Ruby on Rails architecture**

Joined initially to develop modules in Xojo for a custom-built trucking ERP system, including workshop, tire management, and petty cash. As the limitations of the platform became clear, I helped evaluate alternative tech stacks and led the transition to Ruby on Rails, rebuilding the suite as a monolithic app structured with mountable engines to preserve modular boundaries.

**Rebuilt core logistics tooling for workshop, finance, and fleet management**

Designed and implemented multiple mission-critical modules from scratch, covering workshop operations, inventory control, tire lifecycle tracking, staff reimbursements, and more — all aimed at streamlining internal logistics for a growing trucking business.

**Drove a large-scale rewrite that laid the foundation for future Rails work**

Although the project was ultimately shelved under a release-only-when-complete philosophy, the experience solidified my skills in Rails, domain modeling, and modular app design — and directly influenced the architecture behind JuggleBee.

---

### Early Career

**Taught and freelanced simultaneously before moving into agency work**

Before transitioning into Rails, I began my career in South Africa as a lecturer at EMC College, teaching Electronics, Computer Hardware and Software, and Data Networking & Communication. During that time, I also ran Griffin Studios, a freelance web design business focused on PHP and MySQL sites. After relocating to Kenya, I joined M&S Studio, where I continued building CMS-driven websites and deepened my experience in client-facing development.
