---
name: align
description: Bring an EXISTING Jira bug up to the team's bug-quality standard and update it in place — rewriting the title, itemizing findings, adding complete repro steps, moving essential detail out of screenshots into text, and appending a Prompt to Fix. For QA testers working entirely in chat, with no codebase access and no CLI. Trigger whenever someone points at a bug that already exists and wants it improved — a bug key like DRUS-1234 or a Jira URL plus "fix up / clean up / align / standardize / refine / bring to standard / get this ready for devs"; a complaint that a ticket is missing repro steps or a Prompt to Fix; a reopened bug that needs tightening; or a batch request like "get all my open bugs to standard" (the team's alignment / יישור קו task). This is NOT for creating a brand-new bug from scratch (use ideo-qa:open) and NOT for planning a story's implementation (use refine-jira-task). Output is always polished English.
---

# Align an Existing Bug (ideo-qa:align)

A tester points at a bug that's already in Jira — one key, or a whole batch of open ones. This skill reads it, measures it against the team's standard, rewrites the weak parts, and updates it in place, landing it in the **Open** column for the lead's review.

The bar is a **first-try fix**: a developer (or Claude) reading only the rewritten bug should understand the whole problem and fix it without questions. If a bug gets reopened because the fix missed, the assumption is the description wasn't clear or complete enough — so aligning a bug means making it stand entirely on its own in text.

## Skill version & maintenance

`ideo-qa:align` — **version 0.2.0** · author **Moshe Edri**.

Anyone on the team may edit this skill. **On every change: bump this version (semver), put your own name as author on the line above, and set the same version in `ideo-qa/.claude-plugin/plugin.json`.** Every bug this skill updates is stamped with this skill's name + version + author (see step 7), so a bug can be traced to the exact skill version that last aligned it.

## Scope — the Ideo board only

"Bugs" here means the **Ideo board and nothing else** — DRUS board **2328**. JQL can't target a board directly, so reproduce its scope in every search (batch cleanup and the split duplicate check):

```
issuetype = Bug AND (assignee in (
  "712020:dd995c71-ac8b-45e1-95e3-193b3b372b28",   /* Kobi Moshe */
  "712020:8407108d-db7b-40e5-b995-c55689c0b686"    /* Moshe Edri */
) OR assignee is EMPTY)
```

Ignore DRUS bugs outside that scope — they belong to other boards and are not ours.

## The team's bug standard

Measure every bug against all of these:

1. **Title and description carry the full scope.** Summary and description together reflect the complete amount of work. Nothing essential lives elsewhere.
2. **No scope creep through comments.** Same-component findings fold into the description; a different defect or new scope is a separate bug.
3. **Every finding is listed, each with its own repro steps.** An explicit itemized list, each item with clear, complete, self-contained steps — numbered, from a known starting state to the wrong behavior plus what was expected.
4. **Screenshots illustrate, they don't inform.** Assume the reader can't see images. Everything needed to understand or reproduce lives in text.
5. **Polished English — but UI strings stay verbatim.** Clean English prose, except literal on-screen text (field names, button/menu labels, tab names, exact values, error messages), which stays **verbatim in its original language (usually Hebrew), quoted, never translated** — «חיפוש», not "Search". An English gloss in parentheses on first mention is fine; the Hebrew literal is the canonical reference a developer matches against.
6. **Reopened/updated bugs land in the "Open" column** for the lead's review before work resumes.
7. **Two closing checks, in this same chat, in order** — the plain-language gate, then the Prompt to Fix (see the workflow).

## When to split into a separate bug

An existing bug is sometimes secretly several problems. Split when:
- It bundles **two or more unrelated defects** → one bug per defect.
- A finding reads like a **regression** — used to work, now doesn't — *even without certainty*. Regressions get their **own** bug, **linked** to the original. Suspicion is enough.
- Findings have grown past the bug's original **scope** → separate bug, don't keep inflating this one.

Same-component, in-scope findings just get itemized into this description. When you split, say what and why, and get the tester's nod first.

## Stop points — pause and wait for the tester

Four moments in this flow are **hard stops**. At each one, present what you have, then stop and wait for the tester's reply. Never chain past a stop on your own, and never do two in one turn — each is a separate pause.

1. **Before splitting.** The moment the bug looks like more than one problem — multiple defects, a suspected regression, or findings grown past its original scope — stop. Lay out the proposed split (what you'd file as separate tickets, and why) and wait. Do not rewrite or create anything until the tester agrees.
2. **Before opening any new bug from a split — check it isn't already open.** Whenever the split would create a *new* ticket, first search DRUS's open bugs for the same issue; if a likely match exists, show it and ask whether to align that one instead of opening a duplicate. (Doesn't apply to the bug you're already aligning — only to new ones you'd spin off.)
3. **The plain-language checkpoint (the "10-year-old" test).** After rewriting, show your simple explanation and ask the tester whether it captures the whole problem. Wait for their answer. This is a real gate: if they say something's missing — or if explaining it plainly made *you* guess or gloss over anything — go back, fix the rewrite, and show the explanation again. Only move on once it holds.
4. **Before writing to Jira.** Show the complete rewritten bug — findings, repro steps, and Prompt to Fix — and ask for an explicit yes to update the ticket. Wait. A partial "looks good but…" is not a yes. In batch cleanup this stop applies **per bug** — one yes each, never a blanket approval.

If the tester tries to skip a stop ("just update it all"), still pause there — the stops protect them from pushing a half-formed bug to development.

## Where the bug text lives (DRUS specifics)

This project does **not** use Jira's built-in `description` field for bugs — it's almost always empty. The real content lives in custom fields:

- **`customfield_10104` — "Bug Description"**: the primary field. This is what you read, measure, rewrite, and where the `Prompt to Fix` section goes. Treat this as "the description" everywhere below.
- **`customfield_10138` — "Bug Description Mobile"**: mobile-specific write-up. If the bug is about mobile (or has a mobile facet), read and update this one too, with its own findings and repro steps.

Ignore the native `description` field unless both custom fields are empty and the content genuinely sits there. (Field IDs are specific to this Jira site; if a call ever reports the field doesn't exist, re-fetch with `expand="names"` and match the labels "Bug Description" / "Bug Description Mobile" to their current `customfield_*` IDs.)

**Environment naming:** `qa.drushim.io` is the **QA** environment — always call it "QA", never "staging". Staging is a *separate* environment on a different route; the two are not interchangeable. If an existing bug mislabels QA as staging (or vice versa), fix it during alignment. If a finding is genuinely on staging or elsewhere, keep that as-is with its real URL, and ask the tester if it's unclear.

## Workflow

### 0. Connect once
Call `getAccessibleAtlassianResources` once for the `cloudId`; reuse it. If several sites return, ask which by name.

### 1. Read the bug — narrow fields only
Full fetches are huge. Whitelist what you need, and read the custom Bug Description fields (not the native `description`):
```
getJiraIssue(
  cloudId,
  issueIdOrKey,
  fields=["summary", "customfield_10104", "customfield_10138", "status", "labels", "priority", "issuelinks", "attachment", "comment", "resolution"],
  responseContentFormat="markdown"
)
```
`customfield_10104` is "Bug Description"; `customfield_10138` is "Bug Description Mobile". Read `comment` too — on this team, findings and scope creep tend to pile up there rather than in the description.

### 2. Measure against the standard
Check all seven items. Typical gaps: title doesn't cover everything in the body; findings aren't itemized; repro steps missing or partial; essential detail only in a screenshot; no `Prompt to Fix`. Note every gap.

**⏸ Stop point 1 — before splitting.** If the bug is really several bugs, or a finding reads like a regression, stop here: present the proposed split (rules above) and wait for the tester's agreement before you rewrite anything.

### 3. Rewrite the weak parts
Keep what the tester already got right; fix the rest into this shape:

```markdown
**Summary (title):** <concise, covers the full scope>

**Environment:** <app/URL, browser/device, account or state if relevant>

**Findings**

1. **<short name>**
   - Steps to reproduce:
     1. <known starting state>
     2. <action>
   - Expected: <…>
   - Actual: <…>

2. **<next finding>** …

**Notes:** <helpful extras — never the only home for essential info>

## Prompt to Fix
<filled in at step 5>
```

If a screenshot held essential detail, ask the tester to describe it and write it into the steps before updating.

### 4. Plain-language gate ("explain to a 10-year-old")
**⏸ Stop point 2.** Explain the rewritten bug simply, as if to a curious 10-year-old, and show that explanation to the tester. Ask whether it captures the *entire* problem, then wait. If they confirm and it also reads complete to you, continue. If they flag a gap — or if explaining it plainly made *you* gloss over or guess at something — fix step 3 and show the explanation again. This is the real quality check.

### 5. Prompt to Fix
In this same chat, write a short, direct prompt a developer could paste to Claude to fix the bug — symptom, location, expected behavior, referencing the findings. Put it at the end of the **Bug Description field** (`customfield_10104`) under a heading exactly `Prompt to Fix`. It goes in that field, never a comment.

### 6. Show the draft and get approval
**⏸ Stop point 3.** Show the full rewritten bug and ask plainly, e.g. "Ready for me to update `<KEY>` with this?" Wait for a clear yes. Revise on request. Don't update on a partial "looks good but…".

### 7. Update it, and land it in Open
First, append the **provenance stamp** to the end of the rewritten Bug Description (after `Prompt to Fix`), matching the "Skill version & maintenance" block above, so the bug records the skill version that last aligned it:

```
---
_Aligned by `ideo-qa:align` v0.2.0 · author: Moshe Edri_
```

If a previous `ideo-qa:*` stamp is already there, replace it with this one rather than stacking. Then write the rewritten content into the Bug Description field (and the mobile field if the bug has a mobile facet) — not the native `description`:
```
editJiraIssue(
  cloudId,
  issueIdOrKey,
  fields={ "summary": <title>, "customfield_10104": <full Bug Description incl. Prompt to Fix + provenance stamp> },
  contentFormat="markdown"
)
```
For a mobile bug, also set `"customfield_10138": <mobile write-up>`. Send content as markdown. If the custom field rejects markdown (some rich-text custom fields need Atlassian Document Format), rebuild that field's value as ADF and retry with `contentFormat="adf"`.

Also add the `ideo-qa-align-v0-2-0` label (matching this version) — read the issue's current `labels` first and write them back **plus** this one, so you don't drop existing labels. The footer version and the label version must always match this skill's version at the top of the file.

Then move it to Open if it isn't already (`transition` is an object):
```
getTransitionsForJiraIssue(cloudId, issueIdOrKey)   # find the transition named "Open"
transitionJiraIssue(cloudId, issueIdOrKey, transition={ "id": <id> })
```
**Reopened bugs often refuse to transition** because they still carry a Resolution from when they were closed. If the move is blocked, clear it first:
```
editJiraIssue(cloudId, issueIdOrKey, fields={ "resolution": null })
```

### 8. Batch cleanup (the alignment / יישור קו task)
For "get all my open bugs to standard", find them **within the Ideo board scope**, then handle **one at a time with approval each** — never bulk-edit silently:
```
searchJiraIssuesUsingJql(
  cloudId,
  jql='project = DRUS AND issuetype = Bug AND status = Open
       AND (assignee in ("712020:dd995c71-ac8b-45e1-95e3-193b3b372b28", "712020:8407108d-db7b-40e5-b995-c55689c0b686") OR assignee is EMPTY)
       ORDER BY created DESC',
  fields=["summary", "status"]
)
```
Show the list first and confirm scope before working through it. Never align a bug outside the Ideo board — those belong to other boards.

### 9. Split & link regressions
For a split, each *new* bug you'd spin off gets the duplicate check first (**Stop point 2**): before creating it, search the Ideo board's open bugs for the same issue and confirm with the tester it isn't already filed —

```
searchJiraIssuesUsingJql(
  cloudId,
  jql='project = DRUS AND issuetype = Bug AND statusCategory != Done
       AND (assignee in ("712020:dd995c71-ac8b-45e1-95e3-193b3b372b28", "712020:8407108d-db7b-40e5-b995-c55689c0b686") OR assignee is EMPTY)
       AND (summary ~ "<terms>" OR text ~ "<terms>" OR cf[10104] ~ "<terms>") ORDER BY updated DESC',
  fields=["summary", "status", "customfield_10104"]
)
```

Judge real overlap (same page/element/symptom, not just shared words). If an open bug already covers it, align that one instead of creating a duplicate. Otherwise create the new bug through ideo-qa:open's create step. Link a suspected regression to its original:
```
getIssueLinkTypes(cloudId)
createIssueLink(cloudId, inwardIssue=<new>, outwardIssue=<original>, type="Relates")
```

### 10. Hand off
Confirm in plain terms: which key(s) you updated, that they're in Open awaiting the lead's review, any links made. Then stop.

## Hard rules
- The bug must stand alone in text; if reproducing it needs a screenshot, it isn't aligned.
- Every finding gets its own complete repro steps.
- `Prompt to Fix` goes in the Bug Description field (`customfield_10104`), under that exact heading, at the end — never a comment.
- Never grow a bug through comments; new defect / new scope / suspected regression → separate bug.
- Suspicion of regression is enough to split and link.
- Only align bugs on the **Ideo board** (board 2328: assignee in Kobi Moshe / Moshe Edri, or unassigned). Every search — batch cleanup and split duplicate checks — is scoped to that board; never touch a bug outside it.
- Never spin off a new bug without first searching the Ideo board's open bugs for one that already covers it.
- Every aligned bug is **provenance-stamped** with this skill's name + version + author: the footer line in the Bug Description **and** the `ideo-qa-align-v<version>` label (preserve existing labels). Bump the version (here + in `plugin.json`) on every edit, and keep the stamp in sync.
- Output is polished English, **but** keep UI labels, field names, and on-screen text verbatim in Hebrew (quoted, untranslated) — «חיפוש», not "Search". Translating them loses the developer's reference.
- Always show the rewrite and get a clear yes before updating Jira — one confirmation per bug, even in batch.
- Read issues with a narrow field list.
- Don't perform the lead's review; your job ends when the bug is in Open, complete and to standard.

## When you hit a snag
| Situation | What to do |
|---|---|
| Essential info only in a screenshot | Ask the tester to describe it; write it into the steps before updating. |
| Bug is really several defects | Propose a split; confirm, then align/file each. |
| A finding looks like a regression | File it separately, link to the original. Suspicion is enough. |
| A split-off finding already has an open bug | Show the match; align that bug instead of opening a duplicate. |
| `getJiraIssue` response too large | Narrow the `fields` list further. |
| Reopened bug won't move to Open | It likely still has a Resolution — clear it with `editJiraIssue(fields={"resolution": null})`, then transition. |
| No transition to Open from current status | Tell the tester; leave it and note where it landed. |
| Markdown renders wrong in Jira | Rebuild the body as ADF and retry with `contentFormat="adf"`. |
| Bug Description rewritten after you saved it | The project may run automation on these fields — flag it; the standard's Prompt to Fix lives in the Bug Description field. |
| Tester asks to bulk-update everything at once | Do them one at a time with a yes each. The gate protects them. |

## Example triggers
- "Clean up DRUS-1487 so it meets the new standard"
- "This ticket is missing repro steps and a Prompt to Fix, fix it"
- "Align https://…/browse/DRUS-1490 to the standard"
- "Get all my open bugs ready for the devs"
- "Reopened DRUS-1502 — tighten it up and put it back in Open"
