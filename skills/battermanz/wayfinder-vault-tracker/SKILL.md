---
name: wayfinder-vault-tracker
description: Issue-tracker adapter that stores wayfinder efforts (maps, decision tickets, research) as notes in the BatterNotes vault via Hatchdoor. Use whenever a /wayfinder session's effort lives in the vault rather than a repo — any non-coding effort — for charting, working a ticket, or writing the destination outcome.
---

# Issue tracker: Hatchdoor vault

Wayfinder efforts about anything **other than coding/apps** live in the BatterNotes vault, not in a repo (the subject test: coding/apps efforts track inside their own codebase; everything else is decision-work, and decisions live vault-side). This document maps wayfinder's tracker operations onto vault notes. The wayfinder skill itself is unchanged — only the storage layer differs from the local-markdown tracker.

All reads and writes go through the Hatchdoor MCP tools, following the `hatchdoor` skill's conventions (discover the vault with `list_vaults`; fresh `expected_content_hash` before every mutation; `commit_summary` on every write; British English).

## Layout

One folder per effort under the vault's `wayfinder/` root — the working area for in-flight efforts, deliberately outside both domain roots:

- **Map**: `wayfinder/<effort>/<Effort> — Map` — the Destination / Notes / Decisions-so-far / Not-yet-specified / Out-of-scope body.
- **Ticket**: `wayfinder/<effort>/issues/NN — <Question title>`, numbered from `01`. The number lives in the note title; refer to tickets by title, as wikilinks.
- **Research file**: `wayfinder/<effort>/research/NN — <Title>`, NN matching its ticket. Research agents write findings here and resolve their own ticket; they do not edit the map.

## Frontmatter and tags

- Every tracker note (map, ticket, research) carries exactly `type/wayfinder` as its type tag.
- The **map alone** also carries exactly one project status tag: `status/active | simmering | sleeping | done`. Tickets and research notes never carry `status/*` — their lifecycle is body lines (below), which can hold the prose a tag cannot ("resolved, with corrections — see banner…").
- Every tracker note ends with a `## Related` section (always last): tickets and research files link at least their map; the map links the relevant domain hub/MOC notes.

## Ticket body lines

Same grammar as the local-markdown tracker, near the top of the ticket body:

- `Type:` — `research` / `prototype` / `grilling` / `task`, with HITL/AFK qualifiers in prose where useful.
- `Status:` — `open` / `claimed` / `resolved`, freely annotated ("resolved, with corrections 2026-09-02 — …").
- `Blocked by:` — ticket numbers; a ticket is unblocked when every listed ticket is `resolved`. Strike resolved blockers in place (`~~06~~`). `Blocks:` on the blocker side is welcome but optional.

## Operations

- **Frontier**: list `wayfinder/<effort>/issues/` (`get_tree` on that folder — but never trust `get_tree` during or just after a write burst; per-note results are authoritative); a ticket is on the frontier when open, unblocked, and unclaimed; first by number wins.
- **Claim**: set `Status: claimed` in the ticket and write it before any work.
- **Resolve**: append the answer under an `## Answer` heading (above `## Related`), set `Status: resolved`, then append a context pointer — one line, gist + wikilink — to the map's Decisions-so-far.
- **New tickets / fog / out-of-scope**: as the wayfinder skill directs; create ticket notes with the next free number, wire `Blocked by:` in a second pass.
- **One ticket per session**, research tickets excepted — exactly as upstream wayfinder rules.

## The destination outcome

Ticket resolutions are **working state**, not domain decisions. Only when the map fully resolves — nothing left to decide — is the decision reached. Then:

1. Write the **destination outcome** (the decision/spec the effort was finding its way to) as a proper note in its domain home, routed like any other note (`personal/…` or `homelab/…` per the vault router), linking back into the map and key tickets for the trail.
2. Set the map to `status/done` and link the outcome note from the map.
3. The effort folder stays in place — resolved tickets and research remain the linkable evidence trail. Never copy ticket answers into domain notes while the effort is live.

## Commit summaries

Every write uses the grammar `wayfinder(<effort>): <what happened>` — e.g. `wayfinder(family-car): resolve ticket 05 — the EV and petrol branches, measured`. This is the effort's audit trail in the vault's git history.

## Session harness

Claude Code sessions for vault-wayfinder efforts run in the `wayfinding` repo (`~/coding/wayfinding`) — a thin harness holding instructions and agent memory only. Effort data never lives there, nor anywhere outside the vault.
