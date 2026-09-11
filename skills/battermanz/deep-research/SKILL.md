---
name: deep-research
description: Research a question deeply with a coordinator and parallel agents, landing the whole dossier in the vault.
disable-model-invocation: true
platforms: [linux, macos, windows]
---

# deep-research

You are the **coordinator**. Interview the user, cut their question into **clusters**, brief one research agent per cluster, and turn what comes back into notes that live in the vault. The agents read. You judge.

**Only you spawn, and every brief you write says so.** Cluster agents, the verifier, anything else you send out. An agent that wants help messages you and asks; you decide against the number already running, and if you grant it you say how many children, pin their model, and repeat the guardrails from the parent's brief in theirs, because guardrails do not survive being paraphrased by a parent mid-run. The clause lives in [`references/cluster-prompt.md`](references/cluster-prompt.md), which is why a brief written by hand loses it. On 2026-09-10 the verification brief was written by hand: it spawned six children nobody was counting, and four of them outlived the kill of their parent by fifteen minutes.

Load the `hatchdoor` skill before writing anything into the vault. It owns filing, tags, frontmatter, note shape, linking, British English, this vault's unslop exceptions and the change report. This skill covers only what it adds.

## Is this the right tool

Judge after the interview, not before: the scope that makes a question splittable is usually what the interview surfaces.

- **Three or more lines of enquiry that do not overlap:** run it. "What should someone eat postpartum" split into six without strain.
- **One line of enquiry:** answer the user directly and spawn nothing. "What is the RDA for iron while breastfeeding" does not split at all.
- **A technical question about code, APIs or specs:** hand it to the `research` skill, which is the single-agent version that writes into a repo.

Four to eight clusters is the usual range. Below four, the fan-out costs more than it returns. Above eight, the clusters are probably slicing the same ground.

## 1. Interview, one question per message

Ask one question per message and wait for the answer. Ask only a question whose answer would change the plan. Settle everything else yourself, from the vault or from the question, and say what you assumed. Stop at four questions regardless of what is still unknown.

Worth asking, because only the user knows: who the research is for, what their situation actually is, what is already ruled in or out, and whether they want a **practical note** alongside the evidence. A practical note is a do-this checklist. Some questions have one behind them and some have none, so ask rather than assume; if the user has no preference, write one.

Search the topic for an existing effort before you plan, so you know whether this is a re-run.

## 2. Show the plan, wait for a yes

State, in one message:

- The clusters, and why the question cuts that way.
- The **source tiers** for this subject: which bodies count as authoritative here, which sources are corroborating only, and which are excluded. Look this up rather than asserting it; naming the right authorities for an unfamiliar field is itself research. For each authoritative body, say how its current edition will be confirmed, because clinical and safety guidance is revised on its own cycle and the superseded edition stays online, indexed and confident. Two rules hold for every subject and are not up for negotiation: a claim carries a link to whoever owns it, and a source that could not be found or reached is reported as a gap.
- Whether this extends an existing effort or starts a new one (see **Re-runs**).
- Your assumptions.

No cost estimate. Wait for approval. A trim or a redirect becomes the new plan and needs no second yes.

## 3. Open the effort in the vault

One folder per effort, under the topic the filing router picks. The effort name is the subject in a few words, with no date: `Postpartum nutrition`, not `2026-09-10 - Postpartum nutrition`. Only a re-run of a superseded effort carries a date.

```
personal/<topic>/<Effort>/
  <Effort> - Brief          type/research    research-brief
  <Effort> - Evidence       type/reference   research-evidence
  <Effort> - Practical      type/reference   research-practical   (only if wanted)
  research/
    <Cluster>.md            type/research    research-cluster
```

The last column is the note's starting template, under `_system/templates/`. Each one carries the shape and the reason for it; follow them rather than reinventing a layout per effort.

This is deeper than the vault's usual two levels, and deliberate: an effort is a set of notes that only make sense together. If the topic folder itself does not exist yet, ask before creating it.

Write the Brief now, before spawning. It holds the question as the user asked it, the interview answers, the cluster plan, the source tiers, the date, and links to everything the effort produces. It is what makes the effort legible a year later, when the practical note says to do something and nobody remembers whether that was general advice or specific to this person. It is also the recovery anchor: a session that comes back to a half-finished effort reads the Brief first and picks up from there.

## 4. Fan out

Brief each agent from [`references/cluster-prompt.md`](references/cluster-prompt.md).

- **Four at once.** That is a concurrency limit, not a budget. Nine clusters run in three waves.
- **Run a wave to completion before opening the next.** A wave is over when every one of its notes reads complete, not when the last agent returns.
- **Confirm the writes land.** Once a wave's agents have had a few minutes, check that every cluster note exists and has grown. An agent that cannot write to the vault will keep reading happily and hand you a confident summary from its context, which is the failure this whole design exists to prevent. A cluster with no note is a broken agent: stop, say so, and fall back to having it write a scratchpad file you import yourself.
- **Watch for stalls.** The status callout and the note's length are your liveness signal. An agent whose note has not grown in twenty minutes is stuck, not thinking.
- **Opus**, unless the user has granted Fable for this effort.
- **Save the documents.** Fetch each authoritative guideline whole, to a file, before reading it, and grep the file. A broken search returns confident nonsense, and "nothing found" is indistinguishable from "the search broke"; a document on disk makes a null provable. It also makes §7's third pass a grep rather than a second round of fetching.
- **Preflight.** Run `donsetch doctor` before the first spawn, and let it clear what it finds. Concurrent agents have left a stale browser profile lock behind, after which fetches return success with an empty body and an agent writes a thin section believing it read the page. If it cannot be cleared, say so and expect the fallback tiers to carry the run.
- **Only you spawn**, per the rule at the top. Put it in the brief in so many words.

## 5. Resume rather than restart

A usage limit can kill every running agent at once. Each cluster note carries a status callout at the top, so recovery is a read, not a guess:

1. Read the top of each cluster note.
2. Anything marked complete is done. Leave it.
3. For the rest, spawn a fresh agent with the same brief, **with step 1 of its write instructions replaced**: the note already exists, so it opens with `get_note` on the slug you give it, and continues from what is there. An agent that runs the original step 1 will either fail on `create_note` or overwrite the partial work it was sent to save.

## 6. Synthesise

Read every cluster note in full with `get_note`, then write the Evidence note yourself, and the Practical note if one was wanted.

Synthesis is the one step that stays in a single head. The cross-cluster disagreements only surface when one context holds all of them at once: a table of where two national guidelines contradict each other is worth more than either cluster's summary of itself.

The Evidence note carries the numbers, the disagreements, the provenance and the gaps, and every claim in it names the cluster note it came from. That citation is what makes the next step checkable. The Practical note carries what to do, and is allowed to be directive, because that is what it was commissioned for.

## 7. Check before you report

- **Structural.** Every cluster note marked complete, and every claim in the summaries traceable to one. A half-written note from a killed agent must not quietly become evidence: resume it per §5 and re-synthesise, or drop what rests on it and say in the Evidence note's gaps that the cluster is unfinished.

Verification runs in three passes, cheapest first. Most errors die in the first two. Run them in order and take only what survives to the next.

1. **Your summaries against the cluster notes.** You have just read all of them, so this costs nothing, and it is where most errors are: a number transcribed wrong, a hedge dropped, an interpretation added during synthesis that no cluster note supports. Go over every claim carrying a number and every claim that tells the user to do something. Those are the two kinds that change what the user does. **A number you cannot find in any cluster note is deleted, not checked.** It came from you. On 2026-09-10 one such figure was invented outright and reached the note meant for a babysitter.

2. **Ask the agents what they read.** Resume each cluster agent report-only: no fetching, no searching, no spawning, no editing, one reply. Ask it to confirm its cluster's claims that reached the summaries, from what it actually read. It still holds that. On 2026-09-10 four agents did this in about two minutes each with zero tool calls, roughly 90k for the lot, against 900k when the same agents covered the same ground by re-reading sources.

3. **Back to the source, narrowly.** Only for a claim that survived passes 1 and 2 and still looks wrong, or one carrying an instruction someone else will act on, such as anything on a note to be handed to a babysitter or a grandparent. If the document was saved to a file during the run (§4), this is a grep rather than a fetch. Brief one agent, name the claims, and say it re-reads the source rather than trusting the citation, reports in its reply, edits nothing and spawns nothing.

- Fix what verification finds, then say what was found and fixed. A discrepancy you decide not to act on goes in the Evidence note's gaps, not in a silence.
- **State the coverage.** The Evidence note says how many of its claims have been independently checked and how many rest on the agent that wrote them. A year later, "checked" and "written down confidently" look identical.

## 8. Register and report

Update the Brief with links to every note the effort produced. Link the summaries into the topic hub, and into the topic's research dashboard, creating it from `_system/templates/research-dashboard` if there is none.

Then one `hatchdoor` change report for the whole effort, with a clickable link per note. The agents write without reporting; you report once, for all of it. Alongside it, tell the user what the research could not settle, what the verifier caught, and which of your assumptions still stand.

## Manners

Report findings and status. Ask before advising on anything outside the research question, and never tell the user what to do with their time.

## Re-runs

- **Same question, one more angle:** add a cluster to the existing effort, write its note alongside the others, revise the summaries in place.
- **Question asked again because the situation moved:** a new dated effort whose Brief links back and says what changed. The old effort stays exactly as it is.

Overwriting an effort is out. The raw layer exists to be audited, and a summary silently rewritten a year later is worth less than no summary. Decide which case applies at plan time and say so, so the user can overrule you before anything is written.

## Why it is shaped this way

Every rule above that looks fussy is paid for by a run that went wrong. The design came out of [Three silent failures in a ten-agent research run](https://hatchdoor.batterlan.cc/v/bb7e4994-d1b3-4b04-aecf-bf7ed5418f02/n/2026-09-09-three-silent-failures-in-a-ten-agent-research-run) on 2026-09-09; the spawn rule and the two-layer verification come from its first outing, [Verifying the expensive layer instead of the cheap one](https://hatchdoor.batterlan.cc/v/bb7e4994-d1b3-4b04-aecf-bf7ed5418f02/n/2026-09-10-verifying-the-expensive-layer-instead-of-the-cheap-one) on 2026-09-10.
