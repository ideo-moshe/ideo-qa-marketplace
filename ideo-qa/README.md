# ideo-qa

QA bug workflow for testers who work entirely in chat — no codebase, no CLI. Both skills read and write Jira directly through the Atlassian (Rovo) MCP connector, so a tester just describes what they saw and Claude handles the rest.

## Skills

- **`ideo-qa:open`** — file a *new* bug. Turns a tester's raw finding (any language) into one clean English bug and creates it in the **Open** column.
- **`ideo-qa:align`** — bring an *existing* bug up to standard. Reads a bug (or a batch of open bugs), rewrites the weak parts, and updates it in place, landing it in **Open**. Covers the team's one-time alignment (יישור קו) task.

Both end the same way: draft → plain-language "explain to a 10-year-old" gate → `Prompt to Fix` appended to the description → show the tester → explicit approval → write to Jira → hand off for the lead's review.

## The standard both skills enforce

1. Title + description carry the full scope; nothing essential lives elsewhere.
2. No scope creep through comments — same-component findings fold into the description; a different defect or new scope is a separate bug.
3. Every finding is itemized, each with its own complete repro steps.
4. Screenshots illustrate only; all essential detail is written in text.
5. Polished English, whatever the input language.
6. New/reopened bugs land in the Open column for the lead's review.
7. The two closing checks (plain-language gate, then Prompt to Fix) run in one chat session.

Suspected regressions are filed as separate bugs, linked to the original — suspicion is enough. Before opening any new bug, both skills first search DRUS's open bugs and surface likely matches, so a finding that's already tracked gets aligned rather than duplicated.

## Requirements

- The **Atlassian** connector enabled in Claude, with write access to the DRUS project.

## DRUS specifics (hardcoded in both skills)

- Project key **`DRUS`**, issue type **`Bug`**.
- Bug content lives in custom fields, not the native `description` (which stays empty): **`customfield_10104` = "Bug Description"** (primary; holds the write-up and the Prompt to Fix) and **`customfield_10138` = "Bug Description Mobile"** (mobile facet). Both skills read/write these.
- If field IDs ever change, re-fetch a bug with `expand="names"` and re-map the "Bug Description" / "Bug Description Mobile" labels.

## Notes for maintainers

- The standard is duplicated inside each skill's `SKILL.md` on purpose, so each skill runs self-contained. If it changes, update both `skills/open/SKILL.md` and `skills/align/SKILL.md`.
- Still open for this team: (1) whether automation rewrites the Bug Description custom fields — if so, the Prompt-to-Fix-in-field rule needs revisiting; (2) the exact "Open" status/transition name to hardcode the transition id.
