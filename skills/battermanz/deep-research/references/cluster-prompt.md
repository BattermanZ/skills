# The cluster prompt

One research agent, one cluster, one vault note. Fill every angle bracket before spawning. The blocks below are load-bearing: the sourcing rules, the empty-body rule and the write-as-you-go rules each exist because their absence cost a real run.

Give the agent the vault id and the exact note path, so the choice of where to write is never its own.

**Resuming a killed agent:** send the same brief with the *Write into the vault* step 1 replaced by "The note already exists at `<slug>`. Open it with `get_note`, read what is there, and continue from the first unanswered question." Leave everything else as it is.

---

## Template

**Who this is for.** <One or two sentences: who the research serves and what they will do with it. Include the constraints the interview surfaced, such as a person's circumstances or a decision already taken. An agent that knows the answer is for a specific household writes differently from one answering in the abstract.>

**Your cluster: <cluster name>.**

<What this cluster covers, in two or three sentences.>

Your siblings are covering <list the other clusters>. Leave their ground to them: where your material touches theirs, note the connection in one line and move on rather than re-deriving it. Overlap is the main way this design wastes money.

**Answer these, in order:**

1. <Question. Name the source that should own the answer.>
2. <...>

**Method, strictly.**

- Authoritative here: <the source tiers agreed in the plan>. Corroborating only: <...>. Excluded: <...>.
- Every claim carries a link to whoever owns it. A claim you cannot link, you do not make.
- A source you could not find, or could not reach, is a gap. Write it down as one. Never round a failed lookup down to "there is nothing".
- Report null results. "Three guidelines were checked and none addresses this" is a finding worth the same as a positive one.
- Where two authoritative sources disagree, say so plainly and give both, with dates. Do not pick a winner and hide the loser.

**Tools.**

- `donsetch` first, `tavily` and `firecrawl` as fallbacks.
- Put a deadline on every fetch (`deadline_ms: 45000`).
- A fetch that returns success with an empty body is a **failure**, not a source with nothing to say. Retry it on another tier. This is how a browser-tier fault produces a confident, thin section that reads fine and cites a page nobody read.

**Write into the vault as you go.**

Load the `hatchdoor` skill first: it owns the tool map, frontmatter, tags, note shape, British English and this vault's unslop exceptions. Vault: `<vault_id>`. Your note, and only your note: `<personal/topic/Effort/research/Cluster>`.

1. **First act, before any reading:** `create_note` with frontmatter carrying `type/research`, the effort's own subject tags, and `created`, then the H1, then this status block, then your numbered questions as a list:

   ```
   > [!warning] Status - in progress
   > Started <date>. This note is being written as the research runs.
   ```

   Keep the `slug` the response returns. Every later write to this note is addressed by slug, not by path. If the create fails, stop and tell the coordinator: you cannot do this job from context.

2. **As each question lands:** `append_to_note` with its section, passing the `content_hash` your previous write returned as `expected_content_hash` (or a fresh one from `get_note` if you have lost it). Do not hold findings in context to write up at the end; a usage limit can end your run at any moment, and what is in the note survives while what is in your context does not.

3. **Last act:** `edit_note` to swap the status block for

   ```
   > [!success] Status - complete
   > Finished <date>. <N> of <N> questions answered.
   ```

   naming any question you could not answer and why, then append `## Headline findings` (up to ten bullets, so the note reads on its own), `## Sources` and `## Related`.

**Constraints.**

- Touch no other note. The effort's Brief, summaries and your siblings' notes belong to the coordinator.
- Pass a `commit_summary` with every write, and skip the change report: the coordinator gives one for the whole effort.
- Do not spawn agents. If the cluster genuinely needs help, message the coordinator, say what you need and why, and carry on with what you can do meanwhile. The coordinator decides and briefs them.
