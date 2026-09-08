---
name: wayfinder-vault-tracker
description: Wayfinder's vault issue tracker — how a /wayfinder effort lives in the BatterNotes vault (Hatchdoor) as map, ticket, and research notes. Reach it when charting a map, working or resolving a ticket, or writing the destination outcome of any non-coding effort.
---

# Issue tracker: Hatchdoor vault

Routing is the **subject test**: an effort about coding/apps tracks inside its own repo; every other wayfinder effort — life, hardware, household, fleet decisions — lives in the BatterNotes vault under `wayfinder/<effort>/`. This file maps wayfinder's tracker operations onto vault notes. All vault access goes through the Hatchdoor MCP tools, per the `hatchdoor` skill.

## Layout

- **Map**: `wayfinder/<effort>/<Effort> - Map`
- **Ticket**: `wayfinder/<effort>/issues/NN - <short title>`, numbered from `01`
- **Research file**: `wayfinder/<effort>/research/RNN - <title>`, NN matching its ticket. **`R00` is charting-time research**, gathered before any ticket existed and matching none — every effort so far has needed one.

Note titles take ` - ` as the separator, never an em dash, per the `hatchdoor` skill. It is a documented exception to this vault's prose rules and it applies to every note of an effort.

Start a map, ticket or research note from its template — `_system/templates/wayfinder-map`, `wayfinder-ticket` and `wayfinder-research` in the vault. The templates are the authoritative note shapes: frontmatter and tags, the map's sections, the ticket's `Type:` / `Status:` / `Blocked by:` lines. Refer to tickets by name, as wikilinks.

## Operations

- **Frontier**: a ticket is on the frontier when open, unblocked (every `Blocked by:` ticket resolved), and unclaimed; first by number wins. List `wayfinder/<effort>/issues/` to scan — and during or just after a write burst, per-note reads are authoritative, `get_tree` is not (Hatchdoor #226).
- **Claim**: set the ticket's **Status** row to `claimed` and write it before any work, on the ticket and in the map's Open tickets row.
- **Resolve**: append the answer under `## Answer` (above `## Related`), set the **Status** row to `resolved`, then move the ticket's row off the map's Open tickets board into Decisions so far — one line, gist + wikilink — in the same write burst. When the evidential footing needs stating, open the answer with `> [!success] Verification status — sourced` or `> [!warning] Verification status — unverified`: who ran it, and what a consumer must re-check.
- **The map's tables hold one line per row.** A correction that needs paragraphs lands in the ticket it corrects; the map's Corrections column carries only `✓ re-checked by [[…]]` or `✗ superseded by [[…]]`.
- **Ticket graph**: the map's optional Mermaid flowchart, drawn once blocking chains run deeper than one ticket. It spans every ticket, resolved ones included, on three channels: **shape** is status (rounded resolved, square open-but-blocked, hexagon on the frontier), **stroke colour** is mode (amber `#d97706` HITL, teal `#0891b2` AFK), **stroke weight** marks the frontier. Style strokes, never fills, so it survives both themes, and repeat the mode in the node label so nothing depends on colour. Redraw the affected nodes in the same burst that claims a ticket, resolves one, or wires a `Blocked by:` line.
- **Mode decides who can work it**, and it is a ticket field of its own, not a parenthetical on Type. An **AFK** ticket a session closes alone; a **HITL** one only resolves through the live exchange, and the agent never stands in for the human. A HITL task hands the human a precise checklist — a GFM task list (`- [ ]`), tickable in the note. The map's board shows it as `<type> · <mode>`.
- **Closed ≠ resolved**: a ticket ruled out of scope takes `Status: closed` with a `> [!warning]` callout saying why, leaves the board, and is recorded in the map's **Out of scope**, never in Decisions so far.
- **Blocking edges are mirrored on both tickets, and every one is a link.** A `Blocks:` entry on one side needs the matching `Blocked by:` on the other, or the board and the graph go wrong in opposite directions. Write each as an aliased wikilink so a blocker is one click away, never a bare number: `~~[[02 - Full ticket title|02]]~~` when resolved, `**[[12 - Full ticket title|12]]**` when live. Inside a table cell the alias pipe must be escaped — `[[Full title\|NN]]` — or it splits the cell; Hatchdoor resolves the link correctly either way. Reframings that arrive as an upstream ticket resolves go under the question's `### Reframings` heading, dated and newest first, rather than rewriting the question in place.
- **New tickets, fog, out of scope**: as the wayfinder skill directs — next free number, `Blocked by:` wired in a second pass, a row added to the map's Open tickets board.
- **Research agents** write findings to their `research/RNN - …` note and resolve their own ticket; the charting session indexes answers on the map. A research note is a document, not tracker state: fixed frame (provenance line, verification callout, short version, what still needs confirming, sources), free middle. Number its sections only when the ticket's question was numbered, and treat a published section number as a permanent address — other notes cite `research/NN §3`, so append on revision, never renumber.
- **One ticket per session**, research excepted.

## The destination outcome

Ticket resolutions are **waypoints**: while the effort is live, a decision's only home is its ticket. Only when the map fully resolves — nothing left to decide — is the decision reached. Then:

1. Write the **destination outcome** (the spec or decision the effort was finding its way to) as a note in its domain home, routed like any other note, linking back into the map and key tickets.
2. Set the map to `status/done`, link the outcome from its `Reached` callout, and update the effort's Status in [[Wayfinder]] — that row mirrors the map's tag and goes stale otherwise.
3. The effort folder stays in place under `wayfinder/` — the resolved tickets and research are its evidence trail.

## Commit summaries

Every write: `wayfinder(<effort>): <what happened>` — the effort's audit trail in the vault's git history.

## Session harness

Claude Code sessions for these efforts run in `~/coding/wayfinding` (see its `AGENTS.md`).
