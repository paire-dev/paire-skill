---
name: paire
description: Brief a pull request so a human can grasp it, check it matches the ask, and accept it in under a minute. Effects, not diffs. Use for "explain this PR", "brief PR 1409", "what does this PR do", or any PR number, URL, or branch.
argument-hint: '<pr-number | pr-url | branch>'
---

# Paire

Produce a brief that lets a human understand and accept a pull request in under a minute. The reader has three jobs, in order: grasp what changed, check that the agent built what was asked, then review the code. The file follows that order. The Top part serves the first two jobs and reads like a product note. The Deep part below it serves the third and holds every path, line number, and evidence glyph. The reader knows the domain but did not watch the work. They accept effects, not diffs. The brief must be far smaller than the change. The Deep part keeps the Paire app's impact-item vocabulary, before, after, evidence as file:lines, confidence, so a brief can be imported later.

Target: `$ARGUMENTS`. A PR number, URL, or branch. With no argument, the current branch's open PR. A branch with no PR yet works the same, keyed by branch name in place of the number, and the Top part makes a first PR description.

## Living document

The brief is a file, edited on every run, not a chat message. One file per PR at `~/.paire/<owner>-<repo>/pr-<n>.md`, with `<owner>-<repo>` from `gh repo view --json nameWithOwner -q .nameWithOwner | tr / -`. Create the directory if missing.

- **First run.** Write the full brief. Reply in chat with the Top part and the path. Never the Deep part.
- **Later runs.** Read the file first. Its `<!-- paire ... -->` line holds the full head SHA and the base last briefed; work only from what changed since, per Update range below. Edit the file in place, then reply in chat with the Since last brief block and the path, nothing more.
- **Stable numbers.** An effect keeps its number for life. New effects take the next number. An effect the new head no longer produces is struck through with `removed in <short sha>`, never deleted or renumbered. Rows sort divergent-first, so a row moves between runs. Refer to a row by its number, never by its position.
- **Decided rows are frozen.** The Deep part's Before and After are the record and never change. The Now cell is a rendering of that record and may be reworded. To change a decided row's meaning, add a new row and mark the old one `superseded by #N`. After a rebase, re-cite the line numbers; that is evidence moving, not a state change.
- **The reader's lines are read-only.** Never rewrite a line the reader wrote or a box they ticked. A ticked question moves, with their note verbatim and today's date, from Questions to Decided as ✅ accepted. A question the reader answers with a concern becomes 💬 commented: on request, post their note as an inline PR comment on the effect's evidence lines, in their voice, and record the link. Decided is the acceptance record.
- **Since last brief.** One block at the top on every update, replaced each run. 5 to 8 lines: new commits, state changes as `#7 🔍 Unverified → ✅ Verified`, threads opened or closed, questions answered, questions added. Only a glyph change is a state change. A consequence phrase changing on its own is not.
- **Old layout.** A file with no `## Deep` heading predates this layout. Rebuild it in this layout on its next run. Numbers, Decided lines, and every line the reader wrote survive. Evidence, root cause, Trust, and Map move into the Deep part. A row the rules below no longer count as an effect is struck through with `dropped, not an effect`. Write `layout updated` in Since last brief.

## Voice

Three rules outrank everything else here.

- **Concise.** Every sentence earns its place. Cut a sentence that restates another. Cut a number that changes nothing.
- **Direct.** Lead with the conclusion. Say "the diff shows", "the author states", or "I could not verify". Never "it seems" or "arguably".
- **Plain.** 15 words per sentence at most. One idea per sentence. Use the words the product and the team already use. Never coin a noun for the brief: "readiness rule" becomes "when the screenshot is taken", "capture path" becomes "PDF export and thumbnail", "non-additive schema" becomes "removes or changes a column".

Test for the Top part: a new teammate on day one understands every line above the Deep part. A line that needs the codebase to make sense moves to the Deep part or goes.

Run the writing rules at the end of this file as a last pass over the whole brief.

## Output

Two parts in one file. The Top part is what the reader sees in chat. The Deep part is there when they choose to dig, keyed by effect number.

| Part | Block | Budget | Reader's job |
| --- | --- | --- | --- |
| Top | Since last brief | 5 to 8 lines, updates only | What moved since I last looked? |
| Top | Headline | 1 sentence, after-state | Grasp |
| Top | Problem, Fix | 1 sentence each, 2 at most | Grasp |
| Top | Your concerns | 3 lines max, omitted when empty | Grasp |
| Top | Effects | Ask line, then one table | Alignment |
| Top | Decisions | 2 to 3, one line each | Alignment |
| Top | Questions before merge | 1 line each | Alignment |
| Top | Decided | the reader's record | Alignment |
| Deep | Root cause | 1 sentence, 2 for a second independent cause | Review |
| Deep | Effect detail | one table, then the concern pointers | Review |
| Deep | Coverage | 1 to 2 lines | Review |
| Deep | Trust | CI, load-bearing test, transcript, findings, mismatches | Review |
| Deep | Map | 6 files max | Review |

Prose above the Effects table, the headline through the concerns, is 8 lines at most.

## Your concerns

Not a review. What you tripped over while reading for the Grasp lines and the Effects table. Do not go looking. Three lines at most.

A line qualifies only if all three hold:

- You can point at the file and line.
- You can say what is wrong in one sentence with no "might", "arguably", or "consider".
- A second reader sees it within a minute once pointed at it.

Anything that fails goes to the Effects table as ❓ Suspected, or gets cut. Opinions on style, structure, or approach never appear here. Format at the top: `- C1. <one sentence in product words>.` The pointer, `C1 👁 observed path:start-end`, goes in the Deep part under the effect detail table.

What qualifies: a bot finding marked resolved with no code change. A test that mocks away the thing it asserts on. A PR body claim the diff contradicts. A swallowed error on a path the PR body calls guaranteed. A cap or loop bound that is off. Concerns are fixes, not decisions, so they do not repeat as questions.

## Effects model

**Ask.** What was requested, in one plain sentence. Build it from the whole base-to-head range: the linked task or issue, every commit message, the PR body. Never from the transcript, which shows the session and not the pull request. It opens the Effects block. It is the contract: a row matches it or falls outside it. With no task linked, the PR body's stated scope is the ask, and say so. The Problem sentence comes from the same sources and names the problem the whole pull request solves. On an update, keep the Ask unless the task, a commit message, or the PR body changed the scope.

**Types.** Product: a user can do something new, or behaviour changed. System: API, schema, dependencies, performance, security. Operational: cost, observability, deployment, rollback, reliability. Execution: an action the agent took outside the repo with a consequence, such as a deploy, a migration run, a posted comment, money spent, or a call to an external system. Read the transcript only for these. A rebase, a push, an opened pull request, or a check run is not an effect. If the transcript is unavailable, one Transcript line in Trust says so, never a row.

**Rows.** One row per outcome someone can notice after merge: a user, an operator, another system, or a teammate who later changes one place and finds another moved. Two rows that differ only in wording are one row. "Unchanged" is never a row. A rename or an extraction nobody notices is not a row; acknowledge it in Coverage. Never invent numbers, counts, or behaviour the diff does not support.

**Now cell.** 15 words at most, product words, stating what is true after merge. New behaviour has no before. When behaviour is removed or reversed, fold the old behaviour into the wording and stress the difference with "now", "no longer", "instead of", or "still". Not "Chartless report PDF times out after a minute → it exports" but "A report with no chart now exports as PDF instead of timing out."

**States.** Two axes, in contract and evidence, with the no-evidence cells split by the agent's claim.

| State | In contract | Evidence | Agent claims | Question it triggers |
| --- | --- | --- | --- | --- |
| ✅ Verified | ✅ yes | ✅ yes | ✅ done | none |
| 🔍 Unverified | ✅ yes | ❌ no | ✅ done | Accept on the agent's word, or require proof? |
| ✂️ Dropped | ✅ yes | ❌ no | ❌ not done | Accept the narrower scope? |
| ⚠️ Unexpected | ❌ no | ✅ yes | ✅ done | Accept the scope expansion? |
| ❓ Suspected | ❌ no | ❌ no | ❓ might | Investigate first, or accept the risk? |

**Consequence.** A row is consequential if any one holds. Irreversible: deletes data, a schema change that cannot roll back, an external system, money. Accumulates: grows per event, request, or publish. Reach: users, paths, services, or cost the Ask did not name. Consequence sorts the table and gates the questions. It is not a column at the top. It shows there as a phrase of six words at most, after the state, naming the thing at stake: `🔍 Unverified, reaches every chart report`, `⚠️ Unexpected, deletes brand rows`, `✅ Verified, grows per publish`. Never the test's name. A row that passes no test carries the state alone. In the Deep part it is a glyph. Pick the strongest: 💥 irreversible over 📈 accumulates over 🌐 reach.

**Order.** Rows that are not ✅ Verified come first, by consequence strength, and rows with no consequence close that group. Then the ✅ Verified rows, #1 first, then Type in legend order. Effect #1 is the one the pull request exists for and keeps that number; if it is divergent it leads the first group. If the headline and #1 disagree, one of them is wrong. Struck rows sort last. Flat list, no group headings. Numbers never change with the order.

**Questions.** One per consequential row that is not ✅ Verified. One per unresolved review thread not already in Your concerns. One if the required check failed or never ran on the head SHA.

**Evidence.** Deep part only. Cite `path:start-end`, comma-separated, three at most, using changed line numbers. Prefix with confidence: 👁 observed when the diff, a test, CI, or the transcript shows it; 💭 inferred when you deduced it and nothing points at it directly; ❓ unknown when you cannot tell. An author's claim is not evidence; it lives in State as 🔍 Unverified. Line numbers are read from the head file or from `-U0` hunk headers, never guessed.

**Coverage.** Deep part only. Every changed file is cited by an effect or a concern, or acknowledged in one line with a reason: lockfile, generated, formatting, snapshot, rename, no outcome to notice. A file you did not read is not acknowledged; read it or mark it ❓ unknown. Uncovered must reach 0 before the brief is done.

**Glyphs.** Use exactly these and no other emojis. Always glyph plus label, as in `✅ Verified`, `👤 Product`, `📈 accumulates`, `❌ no`. Never a bare glyph, never a bare label. Every yes/no, done/not done, passed/failed, or fixed/not fixed value anywhere in the brief uses the three glyphs from the last row, in the States table, the CI line, the thread status, and the Consequence column alike. Never a bare "yes" or "no"; write `✅ yes` or `❌ no`.

| Column | Glyphs |
| --- | --- |
| Type | 👤 Product, 🧩 System, 🏗 Operational, 🤖 Execution |
| State | ✅ Verified, 🔍 Unverified, ✂️ Dropped, ⚠️ Unexpected, ❓ Suspected |
| Consequence, Deep part only | 💥 irreversible, 📈 accumulates, 🌐 reach, ❌ no |
| Evidence, Deep part only | 👁 observed, 💭 inferred, ❓ unknown |
| Decided | ✅ accepted, 💬 commented |
| Any yes/no | ✅ yes, done, passed, fixed. ❌ no, not done, failed, rejected. ❓ unknown, might, unverifiable |

## Process

Do not write until step 7.

1. **Check for an existing brief** at the path above. If it exists, read it and take `head`, `base`, and `updated` from its `<!-- paire ... -->` line. This run is an update. If the file has no `## Deep` heading, this run also rebuilds it, per Old layout.
2. **Gather in parallel.** Metadata with full commit messages, inline review threads including bot findings and replies, top-level comments, CI for the head commit, the diff. On an update, replace `gh pr diff` with Update range below, and read only threads and commits newer than the recorded `updated` timestamp.

   ```bash
   gh pr view <n> --json title,body,baseRefName,headRefName,headRefOid,commits,changedFiles,author
   gh api "repos/{owner}/{repo}/pulls/<n>/comments" --paginate
   gh api "repos/{owner}/{repo}/issues/<n>/comments"
   gh pr checks <n>
   gh pr diff <n> --stat && gh pr diff <n>
   git diff -U0 origin/<baseRefName>...origin/<headRefName>   # hunk headers carry head line numbers
   ```

   Confirm the required check has a run for `headRefOid`. A check that never ran looks identical to one that passed. To read a file at the PR head without checking out: `git fetch origin <headRefName>` then `git show origin/<headRefName>:<path>`.

   **Update range.** The brief describes the PR's change against its base, so an update is the difference between two such changes, never a plain diff between two heads. A plain `old..new` counts a merge from the base as PR work and breaks after a rebase.

   ```bash
   git fetch origin <baseRefName> <headRefName>
   git cat-file -e <old-head> 2>/dev/null || git fetch origin <old-head>
   git range-diff origin/<baseRefName> <old-head> <new-head>
   ```

   `range-diff` matches commits across a rebase and shows, per commit, what is new, dropped, or edited. Read only the files it names. If the old head cannot be fetched, fall back to `gh api "repos/{owner}/{repo}/compare/<old-head>...<new-head>"`. If that fails too, re-brief from scratch, keep the effect numbers and the reader's lines, and say so in Since last brief. If `base` changed, say that too.

3. **Write the Ask and the Problem.** Sources: the linked task or issue, every commit message in the range, the PR body. Never the transcript. Open the linked task if there is one. Set the body aside. On an update, keep both unless one of those sources changed the scope.
4. **Find the one mechanism.** Read the two or three files the diff makes largest or newest. Write the root cause in one sentence. If you cannot, read more. On an update, revisit only if a changed file touches it.
5. **List effects from the diff**, before rereading the author's claims. Then, if a transcript exists, scan it only for actions outside the repo with consequences, one 🤖 Execution row each. Nothing else comes from the transcript. Tag type, state, consequence, and evidence for each row. On an update, re-tag existing rows and add new ones.
6. **Reconcile.** Reread the PR body. Effects only one side names are findings. Classify every review thread: ✅ fixed in commit N, ❌ rejected with a reason, or ❓ resolved with no visible fix or reply. Note what you tripped over for Your concerns.
7. **Write the Top part first.** Headline, Problem, Fix, concerns, Ask, the table in Order, Decisions, then the Questions from consequential rows that are not ✅ Verified, from unresolved threads, and from CI. Then write the Deep part from the same rows. On an update, edit in place per Living document and write Since last brief last, from the difference between the old file and the new.
8. **Cut, then run the writing rules.** Above the Deep part, remove every path, line number, identifier, coined noun, and glyph other than Type, State, and Decided. Remove numbers that change nothing and sentences that restate. 15 words per sentence. Read the Top part as a new teammate on day one. Never cut a reader's line.
9. **Finalize.** Do not reply until all hold: uncovered files 0; every Deep row has evidence with a confidence glyph; every Top row has a Deep row and the same state; every Now cell 15 words or fewer; prose above the table 8 lines or fewer; Decisions 3 or fewer; no row describes session activity; every consequential row that is not ✅ Verified has a question; every review thread classified; glyph plus label in every cell; writing rules run over the whole file. Fix, then check again.

## Flows and diagrams

Use a flow when a sequence or a causal chain is the point and a sentence would bury it. A flow replaces prose; it never repeats it. Flows live in the Deep part and in a Decision. A Top sentence carries one idea and needs no chain.

- **Inline arrow chain** for a linear flow. 3 to 5 steps, one line, product words: `duplicate → copies the pointer → both charts write one row`. Fits in the root cause or a Decision.
- **Mermaid** only in the Deep part, and only when the flow branches or has two actors and an arrow chain cannot carry it. 6 nodes max per diagram, `flowchart LR` for a pipeline or layers, `graph` for how components relate. Split anything bigger into several small diagrams, one per stage, each with a one-line caption saying what it shows. Never one big diagram.
- Arrows live only inside a flow chain, never as a connector in a sentence.

## Rules

- The PR body is claims. The diff is evidence. Attribute every statement.
- Paths, line numbers, code identifiers, and evidence glyphs appear only in the Deep part.
- Never coin a noun for the brief. Above the Deep part, use the words the product and the team already use.
- Session activity is never an effect. A rebase, a push, an opened pull request, or a check run is not a row.
- A decision needs a viable rejected alternative. Rationale not found in diff, commits, or PR body is marked "(rationale not stated)", never invented.
- Numbers go in tables, never in prose, and only when they change what the reader does.
- A bot finding marked resolved with no visible fix is a finding. It goes to Your concerns if the diff shows it, otherwise to Questions.
- Reuse the author's reviewer guide for the Map if it matches the diff. Say you did.
- A Map row says why this position, opening with "Start here because", "Review next because", or "Check last because". Never a restatement of the file's contents.
- Stop when the content stops.

## Writing rules

Copied from the unslop skill. Run them as the last pass. Ask "what makes this obviously machine-written?" and fix it.

**Content.** Puffery ("pivotal", "testament to", "landscape") becomes what happened. Vague attribution ("experts believe") names the source or goes. Trailing -ing phrases ("ensuring...", "highlighting...") go. Promotional words ("robust", "seamless", "groundbreaking") become neutral description.

**Language.** Replace AI vocabulary with the plain word: additionally, crucial, delve, enhance, fostering, garner, interplay, intricate, leverage, pivotal, showcase, underscore, utilize. Fancy "is" ("serves as", "boasts", "features") becomes "is" or "has". "Not just X, but Y" states the point. No forced groups of three. No synonym cycling; pick one word and repeat it. Abstract metaphor nouns (substrate, wedge, vector, surface, scaffolding, primitive, north star, flywheel, harness) become the concrete word.

**Style.** No em dashes, no en dashes, no parentheses as asides. End the sentence or use a comma. Colons only before a list or an example. Bold a lead-in that ends in a period and is followed by new detail; never a whole sentence, never every noun. Sentence case headings. Straight quotes. No decorative emojis; the legend glyphs are the only emojis in the brief.

**Filler.** "In order to" becomes "To". "Due to the fact that" becomes "Because". "It is important to note that" is deleted. Hedging stacks ("could potentially possibly") become "may" or a fact. Chatbot phrases ("I hope this helps", "Let me know", "Great question") are deleted. No generic conclusions; end on the last fact.

**Plain speech.** Say what it does, not how it feels: name the mechanism or the number. One idea per sentence; split anything the reader must reread. 15 words per sentence at most above the Deep part. Active voice, naming the actor: "the compiler validates queries", not "queries are validated". Cut adverbs or use the number: "significantly faster" becomes the measured delta. Use, help, many, if.

**Voice.** Have a view in Your concerns and Decisions; react to the fact instead of listing pros and cons. Vary rhythm: a short sentence, then one that takes its time. Be specific: not "this is risky" but "one CSV copy per publish per chart, forever". First person is fine for what you checked: "I could not find a change for this."

## Template

```markdown
# PR <number>: <title>

Head <short sha>, updated <YYYY-MM-DD>
<!-- paire head=<full sha> base=<baseRefName> updated=<ISO 8601> -->

## Since last brief

<updates only. new commits; `#7 🔍 Unverified → ✅ Verified`; threads opened or closed; questions answered; questions added; `layout updated`>

<Headline: one sentence, what is true after merge>

**Problem.** <the problem the whole pull request solves, as a user saw it>
**Fix.** <the design move, not the edit>

## Your concerns

- C1. <one sentence, product words>.

## Effects

Ask: <one plain sentence of what was requested>. From <task link | PR body, no task linked>.

| # | Now | Type | State |
| --- | --- | --- | --- |
| 2 | <what is true now, stressing the difference, 15 words max> | 👤 Product | 🔍 Unverified, <consequence phrase> |
| 1 | <the effect the pull request exists for> | 👤 Product | ✅ Verified |

## Decisions

1. **Chose A over B.** Because C.

## Questions before merge

- [ ] <one per consequential row not ✅ Verified; one per unresolved thread not in Your concerns; one if CI failed or never ran on head>

## Decided

- [x] ✅ accepted. <question as asked>. <reader's note, verbatim>. <YYYY-MM-DD>
- [x] 💬 commented <PR comment link>. <question as asked>. <reader's note, verbatim>. <YYYY-MM-DD>

## Deep

**Root cause.** <the one mechanism; a second sentence only for a second independent cause>

### Effect detail

| # | Before | After | Consequence | Evidence |
| --- | --- | --- | --- | --- |
| 1 | <before, or "none, new behaviour"> | <after> | ❌ no | 👁 observed <path:start-end>, <path:start-end> |

Concerns: C1 👁 observed <path:start-end>.

### Coverage

<n> changed files, <n> cited in Effect detail or Concerns, <n> acknowledged, 0 uncovered.
Acknowledged: <path> (<reason>), <path> (<reason>)

### Trust

**CI.** <required check>, ran on head SHA <✅ yes | ❌ no>, result <✅ passed | ❌ failed>
**Load-bearing test.** <name once; why it bites; fails pre-fix <✅ yes | ❌ no | ❓ unknown>, per <author | me>>
**Transcript.** <read, no actions outside the repo | read, see #n | unavailable, outside-the-repo actions not assessed>

| Finding | Source | Status |
| --- | --- | --- |
| | | ✅ fixed in commit N / ❌ rejected: <reason> / ❓ resolved, no visible fix or reply |

**Mismatches.** <PR body vs diff, or "none found">

### Map

| Order | File | Why here |
| --- | --- | --- |
| 1 | <smallest file carrying the idea> | Start here because <the ask or risk that makes it the first stop> |
```
