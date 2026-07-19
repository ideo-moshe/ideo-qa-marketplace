---
name: open
description: Turn a QA tester's raw finding into a brand-new Jira bug that meets the team's bug-quality standard, then create it directly in Jira in the Open column. For testers who work entirely in chat, have no codebase access, and don't use the CLI. Trigger whenever someone wants to open, file, report, write, raise, or draft a NEW bug; pastes fresh findings or repro steps for something that isn't a ticket yet; or says things like "I found a bug", "the page breaks when…", "help me report this", "write this up as a bug". Trigger even without the word "bug" — a described defect counts. This is NOT for improving a bug that already exists in Jira (use ideo-qa:align for that) and NOT for planning how engineers implement a story (use refine-jira-task). Output is always polished English regardless of input language.
---

# Open a Bug (ideo-qa:open)

A tester describes something they saw — usually informally, often in Hebrew. This skill turns it into one clean, complete, English bug in Jira, created in the **Open** column where the team lead reviews it before development.

The whole point is a **first-try fix**: a developer (or Claude) reading only this bug should understand the entire problem and fix it without asking questions. The team's working assumption is that if a bug gets reopened because the fix missed, the description wasn't clear or complete enough. So "done" means the bug stands entirely on its own in text.

## Skill version & maintenance

`ideo-qa:open` — **version 0.3.0** · author **Moshe Edri**.

Anyone on the team may edit this skill. **On every change: bump this version (semver), put your own name as author on the line above, and set the same version in `ideo-qa/.claude-plugin/plugin.json`.** Every bug this skill creates is stamped with this skill's name + version + author (see step 7), so any bug can be traced back to the exact skill version that produced it.

## Scope — the Ideo board only

"Bugs" here means the **Ideo board and nothing else** — DRUS board **2328**. JQL can't target a board directly, so reproduce its scope in every search (the duplicate check especially):

```
issuetype = Bug AND (assignee in (
  "712020:dd995c71-ac8b-45e1-95e3-193b3b372b28",   /* Kobi Moshe */
  "712020:8407108d-db7b-40e5-b995-c55689c0b686"    /* Moshe Edri */
) OR assignee is EMPTY)
```

Ignore DRUS bugs outside that scope — they belong to other boards and are not ours. Because every new bug is assigned to you, the creator (a board member), it stays on the board (see step 7).

## The team's bug standard

Every bug must satisfy all of these:

1. **Title and description carry the full scope.** The summary and description together reflect the complete amount of work. Nothing essential lives anywhere else.
2. **No scope creep through comments.** Never grow a bug with comments. Same-component findings fold into the description; a different defect or new scope is a separate bug.
3. **Every finding is listed, each with its own repro steps.** An explicit itemized list of everything to fix. Each item gets clear, complete, self-contained steps — numbered, from a known starting state to the observed wrong behavior plus what was expected.
4. **Screenshots illustrate, they don't inform — and they never carry the element's identity or the design source.** Assume the reader can't see images. Every detail needed to understand, locate, or reproduce the bug is written in text; screenshots are extra, never a substitute. Two things in particular must be in words, never left to a picture — each has cost a real bug:
   - **The exact element.** Name it unambiguously — its on-screen label / `aria-label`, or a CSS selector — plus the precise screen and state (route or overlay, empty vs filled, focused vs at-rest). "the × near the «אזור» field" is **not** enough: on DRUS-3822 that ambiguity made two developers resize the *wrong* × (the field's clear icon) instead of the real target (the search-overlay close button, `hero-shell` `.closeBtnMobile`, `aria-label` «סגור»). Write the element so a developer can `document.querySelector` it.
   - **The Figma source, as a node-id.** Any claim about expected size, spacing, color, or layout cites the design **in text**: the Figma **file key + node-id** of the frame that defines the expected state — e.g. «Expected per Figma `TG4LDbDtMnHCa7CDHNljsA` → frame "iPhone 13 & 14 - 85", node `3522:94953`: close × is 13×13». A screenshot of Figma is **not** a citation; paste the node-id so the developer opens the *same* frame instead of guessing (and can confirm it's the current design, not a superseded one). If a node-id can't be produced, the expected value isn't verifiable — treat it as a blocker, not an optional extra.
5. **Polished English — but UI strings stay verbatim.** Testers may write in Hebrew or rough English; the bug that gets created is clean, professional English. The one exception: literal on-screen text — field names, button and menu labels, tab names, exact values, error messages — is kept **verbatim in its original language (usually Hebrew), in quotes, never translated.** The field is «חיפוש», not "Search"; a developer matches against the real label, and translating it loses that reference. A short English gloss in parentheses is fine on first mention (e.g. «חיפוש» (the free-text search field)), but the Hebrew literal is the canonical reference.
6. **New bugs land in the "Open" column.** That's where the lead reviews them before work starts. You create the bug in Open and hand off — the review is a human step, not yours.
7. **Two closing checks, in this same chat, in order** — the plain-language gate, then the Prompt to Fix (both detailed in the workflow below).

## When to split into a separate bug

You are the guard against one bloated ticket that's really several problems. Before drafting, split when:

- The notes describe **two or more unrelated defects** → one bug per defect.
- A finding smells like a **regression** — something that used to work and now doesn't — *even without certainty*. Regressions get their **own** bug, **linked** to the original they likely broke. Suspicion is enough.
- New findings push past the **scope** the title/description described → new bug, not a bigger one.

Findings in the *same component* that don't expand scope are the exception — they just become another item in the same description. When you split, tell the tester what and why, and get their nod before creating several tickets.

## Stop points — pause and wait for the tester

Four moments in this flow are **hard stops**. At each one, present what you have, then stop and wait for the tester's reply. Never chain past a stop on your own, and never do two of these in one turn — each is a separate pause.

1. **Before splitting.** The instant the finding looks like more than one bug — multiple defects, a suspected regression, or work beyond the original scope — stop. Lay out the proposed split (what you'd file as separate tickets, and why) and wait. Do not draft or create anything until the tester agrees to the split.
2. **Before proposing a new bug — check it isn't already open.** Search DRUS's open bugs for the same issue and, if a likely match exists, show it to the tester and ask whether their finding is that bug (align it) or genuinely new. Never create a bug without this check — a duplicate is exactly the mess this process removes.
3. **The plain-language checkpoint (the "10-year-old" test).** After drafting, show your simple explanation and ask the tester whether it captures the whole problem. Wait for their answer. This is a real gate: if they say something's missing — or if explaining it plainly made *you* guess or gloss over anything — go back, fix the draft, and show the explanation again. Only move on once it holds.
4. **Before writing to Jira.** Show the complete bug — findings, repro steps, and Prompt to Fix — and ask for an explicit yes to create it. Wait. A partial "looks good but…" is not a yes; revise and ask again.

If the tester tries to skip a stop ("just file it"), still pause there once — the stops exist to protect them from a half-formed bug.

## Where the bug text lives (DRUS specifics)

This project does **not** use Jira's built-in `description` field for bugs — leave it empty. The bug write-up goes in custom fields:

- **`customfield_10104` — "Bug Description"**: the primary field. Your drafted write-up and the `Prompt to Fix` section go here. Treat this as "the description" everywhere below.
- **`customfield_10138` — "Bug Description Mobile"**: use this when the bug is about mobile, with its own findings and repro steps.
- **`customfield_10010` — "Sprint"**: new bugs go into the current sprint, **IDEO Sprint 5** (the project's final sprint). Jira sets this by the sprint's numeric **id**, not its name, so resolve the id at runtime (see the create step) — never hardcode a number, since ids differ per sprint.
- **`customfield_10112` — "Need Release"**: a Yes/No select. Every bug this skill creates sets it to **`{ "value": "No" }`** (see the create step).

Project key is **`DRUS`**, issue type **`Bug`**. (Field IDs are specific to this Jira site; if a create/edit ever reports the field doesn't exist, re-fetch an existing bug with `expand="names"` and match the labels "Bug Description" / "Bug Description Mobile" to their current `customfield_*` IDs.)

**Environment naming:** `qa.drushim.io` is the **QA** environment — always call it "QA", never "staging". Staging is a *separate* environment on a different route; the two are not interchangeable. If a finding is actually on staging (or anywhere else), say so explicitly and give that URL. When in doubt about which environment the tester meant, ask rather than assume.

## Workflow

### 0. Connect once
Call `getAccessibleAtlassianResources` a single time for the `cloudId`; reuse it all chat. If several sites return, ask which one by name.

### 1. Gather the finding
Turn informal notes into a complete finding — pull from what they gave you first, ask only about what's genuinely missing. For each distinct defect you need: what's wrong (observed), what should happen (expected), where (page/screen/component, environment, relevant account or state), and clean steps from a known start to the wrong behavior.

If they mention a screenshot, gently remind them the bug must stand on its own in text, and ask them to describe in words whatever it showed (error text, which field, which value). For any visual/appearance finding, this is where you capture the two things a picture can't carry (standard #4): the **exact element** (its label / `aria-label` or a CSS selector, plus screen + state) and the **Figma node-id** of the frame that defines the expected result. If they can't produce a node-id, tell them the expected value can't be verified and help them get one before you draft — don't file an appearance bug whose "expected" lives only in an image.

**⏸ Stop point 1 — before splitting.** If the notes look like more than one bug, a regression, or out-of-scope work, apply the split rules now: present the proposed split and wait for the tester's agreement before you draft anything.

### 2. Check for an existing bug first
**⏸ Stop point 2.** Before proposing a new bug, make sure an open one doesn't already cover it. Build a search from the finding's distinctive terms — the page/screen, the element, the symptom — and query DRUS's still-open bugs:

```
searchJiraIssuesUsingJql(
  cloudId,
  jql='project = DRUS AND issuetype = Bug AND statusCategory != Done
       AND (assignee in ("712020:dd995c71-ac8b-45e1-95e3-193b3b372b28", "712020:8407108d-db7b-40e5-b995-c55689c0b686") OR assignee is EMPTY)
       AND (summary ~ "<terms>" OR text ~ "<terms>" OR cf[10104] ~ "<terms>") ORDER BY updated DESC',
  fields=["summary", "status", "customfield_10104"]
)
```

The `assignee in (…) OR assignee is EMPTY` clause is the Ideo-board scope (see "Scope — the Ideo board only") — keep it on every search so duplicates are judged only against our board. Read the top few candidates' summary + Bug Description and judge *real* overlap — same page, same element, same symptom, not just shared words. Then:
- **Likely already covered** → show the tester the match(es) and ask whether their finding is one of them. If yes, don't open a duplicate: switch to aligning that bug (ideo-qa:align), or, if it's the same component and in scope, add the finding to it.
- **Genuinely new** → say so briefly ("nothing open seems to cover this") and continue.

Wait for the tester's decision before creating anything. If step 1 split the finding into several bugs, run this check for **each** one.

### 3. Draft the bug
Write summary and description in clean English:

```markdown
**Summary (title):** <concise, covers the full scope in one line>

**Environment:** <app/URL, browser/device, account or state if relevant>

**Findings**

1. **<short name>**
   - Steps to reproduce:
     1. <known starting state>
     2. <action>
     3. <action>
   - Expected: <what should happen>
   - Actual: <what happens instead>

2. **<next finding>** …

**Notes:** <helpful extras — never the only place essential info lives>

## Prompt to Fix
<filled in at step 5>
```

One finding just means the list has one item.

### 4. Plain-language gate ("explain to a 10-year-old")
**⏸ Stop point 3.** Before anything reaches Jira, explain the drafted bug simply, as if to a curious 10-year-old — what's broken, where, what should happen instead. Show that explanation to the tester and ask whether it captures the *entire* problem. Then wait. If they say yes and it also reads complete to you, continue. If they flag a gap — or if explaining it plainly forced *you* to gloss over or guess at something — go back to step 3, fix it, and show the explanation again. This is the main quality check, not a formality.

### 5. Prompt to Fix
In this same chat, write a short, direct prompt a developer could paste to Claude to fix the bug — symptom, location, expected behavior, referencing the findings. Put it at the end of the **Bug Description field** (`customfield_10104`) under a heading exactly `Prompt to Fix`. It goes in that field, never a comment.

### 6. Show the draft and get approval
**⏸ Stop point 4.** Show the complete bug in chat and ask plainly, e.g. "Ready for me to create this in Jira?" Wait for a clear yes. Revise on request and ask again. Don't create on a partial "looks good but…".

### 7. Create it — straight into Open, assigned to you, Need Release = No, provenance-stamped
First, **resolve two things once per session:**

1. **Your own Jira `accountId`** via `atlassianUserInfo` — every new bug is **assigned to you, the creator**, which also keeps it on the Ideo board.
2. **The sprint id.** New bugs belong in the current sprint (IDEO Sprint 5). The Sprint field takes the sprint's numeric id, not its name, so read it off an issue already in the open sprint:

```
searchJiraIssuesUsingJql(
  cloudId,
  jql='project = DRUS AND sprint in openSprints() ORDER BY updated DESC',
  fields=["customfield_10010"],
  maxResults=1
)
```

In the returned issue's `customfield_10010` array, take the entry whose `state` is `active` and whose `name` is "IDEO Sprint 5", and use its `id`. If several IDEO sprints are open, match by name. If nothing comes back (e.g. no issue is in the sprint yet), tell the tester and either ask for the sprint or create without it and flag that it needs setting — don't guess an id.

Before building the call, append the **provenance stamp** to the end of the Bug Description (after the `Prompt to Fix` section) so the bug records which skill version produced it — matching the "Skill version & maintenance" block above:

```
---
_Filed by `ideo-qa:open` v0.3.0 · author: Moshe Edri_
```

`createJiraIssue` puts custom fields (Bug Description, Sprint, Need Release) in `additional_fields`; the native `description` param stays unused. Priority, labels, and assignee also go in `additional_fields`. Land the bug in Open in the same call via the `transition` parameter:

```
getTransitionsForJiraIssue(cloudId, <any bug in this project>)   # find the "Open" transition id, once
createJiraIssue(
  cloudId,
  projectKey="DRUS",
  issueTypeName="Bug",
  summary=<title>,
  additional_fields={
    "customfield_10104": <full Bug Description incl. Prompt to Fix + provenance stamp>,  # markdown; mobile field is customfield_10138
    "customfield_10010": <IDEO Sprint 5 id>,        # numeric sprint id from above
    "customfield_10112": { "value": "No" },         # Need Release — always No
    "assignee": { "id": <your accountId from atlassianUserInfo> },   # bug is opened on you
    "priority": { "name": <priority> },
    "labels": [ "ideo-qa-open-v0-3-0", ... ]         # provenance label: skill + version (bump with the version)
  },
  contentFormat="markdown",
  transition={ "id": <Open transition id> }
)
```

The version in the footer and in the `ideo-qa-open-v…` label must always match this skill's current version (top of file). If `customfield_10112` ("Need Release") is ever reported missing, re-map it via `expand="names"` like the other custom fields.

Two fallbacks: if the custom field rejects markdown (some rich-text fields need Atlassian Document Format), rebuild that value as ADF and retry with `contentFormat="adf"`. If the create screen doesn't expose the Bug Description or Sprint field, create the bug first, then set them with `editJiraIssue(fields={ "customfield_10104": ..., "customfield_10010": <sprint id> })`.

If the create screen rejects the transition, create the bug, then move it:
```
transitionJiraIssue(cloudId, issueIdOrKey, transition={ "id": <Open transition id> })
```

If there's no "Bug" issue type, ask which type to use.

### 8. Split & link regressions
When step 1 produced a split, run each new bug through steps 2–7 (including the duplicate check). For a suspected regression, link the new bug to the original:
```
getIssueLinkTypes(cloudId)                        # pick a type, e.g. "Relates"
createIssueLink(cloudId, inwardIssue=<new>, outwardIssue=<original>, type="Relates")
```
Tell the tester what you linked and why.

### 9. Hand off
Confirm in plain terms: the key(s) created, the sprint they landed in (IDEO Sprint 5), that they're in Open awaiting the lead's review, and any links made. Then stop — the review and the fix are separate steps.

## Hard rules
- The bug must stand alone in text; if reproducing it needs a screenshot, it isn't done.
- Every finding gets its own complete repro steps.
- `Prompt to Fix` goes in the Bug Description field (`customfield_10104`), under that exact heading, at the end — never a comment.
- Same-component findings → description; new defect / new scope / suspected regression → separate bug.
- Suspicion of regression is enough to split and link.
- Never create a bug without first searching open bugs for one that already covers it — **scoped to the Ideo board** (board 2328: assignee in Kobi Moshe / Moshe Edri, or unassigned). We only track that board.
- New bugs are set to the current sprint (IDEO Sprint 5) using its resolved numeric id — never a guessed number; if the id can't be resolved, flag it rather than invent one.
- Every new bug is **assigned to you, the creator** (`assignee` = your accountId from `atlassianUserInfo`) — it's opened on you and stays on the Ideo board.
- Every new bug sets **Need Release (`customfield_10112`) to `{ "value": "No" }`**.
- Every new bug is **provenance-stamped** with this skill's name + version + author: the footer line in the Bug Description **and** the `ideo-qa-open-v<version>` label. Bump the version (here + in `plugin.json`) on every edit, and keep the stamp in sync.
- Output is polished English, **but** keep UI labels, field names, and on-screen text verbatim in Hebrew (quoted, untranslated) — «חיפוש», not "Search". Translating them loses the developer's reference.
- Always show the draft and get a clear yes before creating in Jira.
- Don't perform the lead's review; your job ends when the bug is in Open, complete and to standard.

## When you hit a snag
| Situation | What to do |
|---|---|
| Tester leans on a screenshot for essential info | Ask them to describe its content in words; write it into the steps. Don't file until it's in text. |
| Notes describe several defects | Propose one bug per defect; confirm, then file each. |
| A finding looks like a regression | Flag it, file it separately, link to the original. Suspicion is enough. |
| Don't know the project key | Ask by name, or take it from a key/URL they gave. Don't guess. |
| No "Bug" issue type | Read the project's issue types; if none, ask which type to use. |
| Create screen rejects the transition | Create the bug, then transition it to Open separately. |
| Markdown renders wrong in Jira | Rebuild the body as ADF and retry with `contentFormat="adf"`. |
| Bug Description field rewritten after you saved it | The project may run automation on these fields — flag it; the standard's Prompt to Fix lives in the Bug Description field. |
| Can't resolve the sprint id (nothing in the open sprint yet) | Ask the tester for the sprint, or create without it and flag that it needs setting. Don't guess an id. |
| A search finds an open bug that looks like the same issue | Show it to the tester and ask: same bug (align it) or new? Don't create a duplicate on your own. |
| Tester asks to skip the draft and just file it | Show the draft anyway and get the yes. The gate protects them. |

## Example triggers
- "I found a bug — the login page freezes after I submit, help me open it"
- "Write this up as a bug: [rough notes]"
- "Raise a ticket for this, the save button does nothing"
- "I think this used to work — file it as a regression"
- "העמוד קורס כשלוחצים על שמור, תפתח על זה באג" (output in English)
