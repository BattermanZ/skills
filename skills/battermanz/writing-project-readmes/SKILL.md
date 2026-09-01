---
name: writing-project-readmes
description: Use when creating, rewriting, auditing, or improving a README for a self-hosted app, developer tool, CLI, service, or library, or when the user asks whether a README is good or how to make one better.
---

# Writing Project READMEs

A README is the first and often the only thing a person reads before deciding to run your project. Treat it as the front page of the product, not as documentation. Its job is to get the right person to a working install fast, and to make the wrong person leave without filing an issue.

This skill is opinionated toward self-hosted apps, developer tools, CLIs, services, and libraries. It is a flexible checklist, not a fixed template. Pick the sections the project actually needs and drop the rest.

## When to use

- Writing a README from scratch for a project.
- Rewriting or tightening an existing README.
- Auditing a README and reporting what to fix (run the rubric at the bottom).

## The first 30 seconds decide everything

Above the fold, a reader must be able to answer four questions without scrolling far:

1. What is this?
2. Is it for me?
3. Can I run it?
4. Who made it and how do I learn more?

So the top of the README, in order, is usually:

- A one-line pitch that names the differentiator, not a generic category. "A self-hosted, agent-native web app for your Markdown vault" beats "A notes app."
- Two or three sentences that make the pitch concrete (what it does, for whom, and the core promise or trust model).
- A visual near the top: a screenshot, GIF, or diagram. People skim visually, and a picture right under the pitch earns the scroll.
- Badges are optional. Use them when the project is public and each one carries a real signal (it runs, it ships, it is secure, the license), and skip them otherwise. Never add decorative badges.

## Section menu

Include what fits, in roughly this order. Not every project needs every section.

- **Highlights / What You Get**: a bulleted list of the actual selling points. This is the most important section after the pitch.
- **Who it is for, and who it is not**: see negative scoping below.
- **Quick start**: the shortest path to a running instance, with copy-paste commands that match the defaults. Get them running before they can feel overwhelmed.
- **Data and safety model**: for anything that touches user data, grants write access, or runs an agent. State the safe defaults plainly.
- **Configuration**: tables with Default and Purpose columns so people can configure without reading prose.
- **Usage**: the common things people do, with examples.
- **Troubleshooting**: written from the symptom the user sees.
- **Security notes, running from source, API reference, license**: keep these short in the README and link out for the long versions.

## Patterns that separate great from generic

Most README guides stop at structure. These are the moves that make a README trustworthy:

- **Negative scoping.** Say what the project is not. "Not a hosted sync service, not a multi-user platform, not a replacement for Obsidian." This is one of the highest-value sentences you can write. It stops wrong-fit users before they become wrong-fit issues.
- **Safety and defaults as a first-class section**, placed before deep configuration, with the reassuring defaults stated outright ("MCP is disabled by default", "delete moves files to trash, it does not erase them"). For anything that can touch or destroy data, lead with the trust posture.
- **Troubleshooting from the symptom, with the real cause.** Write the entry the way the user experiences the failure ("App starts but cannot write", "It created a starter vault I did not expect"), then give the actual root cause, not the tidy one.
- **Operational honesty.** Call out the sharp edges other projects hide: "demo mode does not rate-limit, put a proxy in front", "this needs its own token because it bypasses the web auth layer", "date ranges are not part of the 2.3.0 contract yet". Naming what does not exist yet prevents support churn and builds trust.
- **Push depth out of the README.** Full API references, exhaustive env tables, and dev-environment setup belong in linked docs or a docs site, not in the README body. Keep the README a lean narrative that links to detail.

## Voice

Write plainly, for a competent person who has never seen the project. Define terms and acronyms on first use. Every command should be copy-paste ready and match the defaults shown elsewhere in the file.

Do not use em-dashes. Use a comma, a colon, parentheses, or two sentences instead.

Ban AI-speak and marketing filler. Avoid: "seamless", "effortless", "leverage", "unlock", "delve", "robust", "powerful", "cutting-edge", "game-changer", "elevate", "supercharge", "in today's fast-paced world", and the "X is not just Y, it is Z" construction. If a sentence would survive being said out loud to a colleague, keep it. If it sounds like a landing page, cut it.

Prefer short declarative sentences. State facts, defaults, and limits. Let the reader trust you because you are precise, not because you are enthusiastic.

## Review rubric

When auditing a README, score it against these and report the gaps, most important first:

1. Can a stranger answer "what is this and is it for me" in 30 seconds?
2. Is there a visual near the top?
3. Is there a Highlights list of concrete selling points?
4. Is there negative scoping (what it is not)?
5. Does quick start get to a running instance with copy-paste commands?
6. If it touches data or grants access, is there a safety/defaults section before deep config?
7. Is troubleshooting symptom-first?
8. Does it push heavy reference (full API, all env vars, dev setup) to links instead of bloating the body?
9. Is it honest about limits and sharp edges?
10. Voice: plain language, no em-dashes, no AI-speak, commands match defaults.

## Common mistakes

- Opening with installation before the reader knows what the thing is or whether they want it.
- A generic pitch ("a modern, powerful tool for X") that names a category instead of a difference.
- Burying or omitting the visual.
- No negative scoping, so the wrong users show up.
- Treating the README as full documentation, so it becomes long and no section is findable.
- Troubleshooting written from the maintainer's mental model instead of the user's symptom.
- Marketing voice and em-dashes that make the project read as less serious than it is.
