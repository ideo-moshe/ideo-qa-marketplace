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

## Install & update (Claude Code)

Marketplace: `ideo-qa-marketplace` · plugin: `ideo-qa`.

**First time** (never installed):

```
/plugin marketplace add ideo-moshe/ideo-qa-marketplace
/plugin install ideo-qa@ideo-qa-marketplace
/reload-plugins
```

Then the skills are available as `/ideo-qa:open` and `/ideo-qa:align`.

**Pull a new version** (already installed):

```
/plugin marketplace update ideo-qa-marketplace
/reload-plugins
```

⚠️ Third-party marketplaces default to **auto-update OFF**, so `marketplace update` refreshes the metadata but may not bump the installed plugin on its own. Either:

- **Enable auto-update once** — `/plugin` → **Marketplaces** → select `ideo-qa-marketplace` → **Enable auto-update**; after that a `marketplace update` pulls new versions automatically; or
- **Force it** — `/plugin install ideo-qa@ideo-qa-marketplace` (same scope), then `/reload-plugins`.

Verify the active version in `/plugin` → **Installed** (or `/plugin list`). You can also confirm from Jira: bugs filed by a given version carry that version in the Bug Description footer and the `ideo-qa-open-v<version>` / `ideo-qa-align-v<version>` label.

## Scope — the Ideo board only

Both skills care about **one board only: the Ideo board, DRUS board 2328.** JQL can't target a board, so every search reproduces its scope: `issuetype = Bug AND (assignee in "Kobi Moshe" (`712020:dd995c71-ac8b-45e1-95e3-193b3b372b28`) / "Moshe Edri" (`712020:8407108d-db7b-40e5-b995-c55689c0b686`) OR assignee is EMPTY)`. Bugs outside that scope belong to other boards and are ignored. (Same scope as the app's `/fix-bugs` skill.)

## DRUS specifics (hardcoded in both skills)

- Project key **`DRUS`**, issue type **`Bug`**.
- Bug content lives in custom fields, not the native `description` (which stays empty): **`customfield_10104` = "Bug Description"** (primary; holds the write-up, the Prompt to Fix, and the provenance stamp) and **`customfield_10138` = "Bug Description Mobile"** (mobile facet). Both skills read/write these.
- **`customfield_10112` = "Need Release"** (Yes/No): `open` sets every new bug to **No**.
- **`customfield_10010` = "Sprint"**: new bugs go to the current sprint (IDEO Sprint 5), resolved by id at runtime.
- New bugs are **assigned to the creator** (`assignee` = the runner's accountId from `atlassianUserInfo`), which keeps them on the Ideo board.
- If field IDs ever change, re-fetch a bug with `expand="names"` and re-map the labels.

## Skill versioning & provenance

- Each skill carries its own **`version` + `author`** in the "Skill version & maintenance" block at the top of its `SKILL.md`. Anyone may edit a skill; **on every change, bump that version (semver), set your name as author, and match the version in `.claude-plugin/plugin.json`.**
- Every bug a skill writes is **provenance-stamped** so it traces back to the exact skill version that produced it: a footer line in the Bug Description (e.g. `Filed by ideo-qa:open v0.2.0 · author: …`) **and** a queryable Jira label (`ideo-qa-open-v0-2-0` / `ideo-qa-align-v0-2-0`). Filter bugs by that label to find everything a given skill version created.

## Notes for maintainers

- The standard is duplicated inside each skill's `SKILL.md` on purpose, so each skill runs self-contained. If it changes, update both `skills/open/SKILL.md` and `skills/align/SKILL.md`.
- The Ideo-board scope, the DRUS field IDs, and the version stamp are duplicated in both skills too — keep them in sync.
- Still open for this team: (1) whether automation rewrites the Bug Description custom fields — if so, the Prompt-to-Fix-in-field rule needs revisiting; (2) the exact "Open" status/transition name to hardcode the transition id.
