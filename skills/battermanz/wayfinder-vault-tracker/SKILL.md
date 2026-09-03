---
name: wayfinder-vault-tracker
description: Wayfinder's vault issue tracker — how a /wayfinder effort lives in the BatterNotes vault (Hatchdoor) as map, ticket, and research notes. Reach it when charting a map, working or resolving a ticket, or writing the destination outcome of any non-coding effort.
---

# Issue tracker: Hatchdoor vault

Routing is the **subject test**: an effort about coding/apps tracks inside its own repo; every other wayfinder effort — life, hardware, household, fleet decisions — lives in the BatterNotes vault under `wayfinder/<effort>/`. This file maps wayfinder's tracker operations onto vault notes. All vault access goes through the Hatchdoor MCP tools, per the `hatchdoor` skill.

## Layout

- **Map**: `wayfinder/<effort>/<Effort> — Map`
- **Ticket**: `wayfinder/<effort>/issues/NN — <short title>`, numbered from `01`
- **Research file**: `wayfinder/<effort>/research/RNN — <title>`, NN matching its ticket

Start a map or ticket from its template — `_system/templates/wayfinder-map` and `_system/templates/wayfinder-ticket` in the vault. The templates are the authoritative note shapes: frontmatter and tags, the map's sections, the ticket's `Type:` / `Status:` / `Blocked by:` lines. Refer to tickets by name, as wikilinks.

## Operations

- **Frontier**: a ticket is on the frontier when open, unblocked (every `Blocked by:` ticket resolved), and unclaimed; first by number wins. List `wayfinder/<effort>/issues/` to scan — and during or just after a write burst, per-note reads are authoritative, `get_tree` is not (Hatchdoor #226).
- **Claim**: set `Status: claimed` and write it before any work.
- **Resolve**: append the answer under `## Answer` (above `## Related`), set `Status: resolved`, then append one line — gist + wikilink — to the map's Decisions so far. When the evidential footing needs stating, open the answer with `> [!success] Verification status — sourced` or `> [!warning] Verification status — unverified`: who ran it, and what a consumer must re-check.
- **HITL task checklists**: a HITL task hands the human a precise checklist — a GFM task list (`- [ ]`), tickable in the note.
- **New tickets, fog, out of scope**: as the wayfinder skill directs — next free number, `Blocked by:` wired in a second pass.
- **Research agents** write findings to their `research/RNN — …` note and resolve their own ticket; the charting session indexes answers on the map.
- **One ticket per session**, research excepted.

## The destination outcome

Ticket resolutions are **waypoints**: while the effort is live, a decision's only home is its ticket. Only when the map fully resolves — nothing left to decide — is the decision reached. Then:

1. Write the **destination outcome** (the spec or decision the effort was finding its way to) as a note in its domain home, routed like any other note, linking back into the map and key tickets.
2. Set the map to `status/done` and link the outcome from it.
3. The effort folder stays in place under `wayfinder/` — the resolved tickets and research are its evidence trail.

## Commit summaries

Every write: `wayfinder(<effort>): <what happened>` — the effort's audit trail in the vault's git history.

## Session harness

Claude Code sessions for these efforts run in `~/coding/wayfinding` (see its `AGENTS.md`).
