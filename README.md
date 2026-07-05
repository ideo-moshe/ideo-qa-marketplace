# ideo-qa-marketplace

A Claude plugin marketplace hosting the **ideo-qa** plugin (skills `ideo-qa:open` and `ideo-qa:align`).

## Repo layout
```
ideo-qa-marketplace/
├── .claude-plugin/
│   └── marketplace.json      # catalog; lists ideo-qa via a relative path
└── ideo-qa/                  # the plugin
    ├── .claude-plugin/plugin.json
    ├── skills/open/SKILL.md
    ├── skills/align/SKILL.md
    └── README.md
```
The relative-path source (`"./ideo-qa"`) only resolves when the marketplace is added via **git** (Bitbucket/GitHub/etc.), which is the intended path.

## Publish to Bitbucket (one time)
```bash
cd ideo-qa-marketplace
git init && git add . && git commit -m "ideo-qa v0.1.0"
git remote add origin https://bitbucket.org/<workspace>/ideo-qa-marketplace.git
git push -u origin main
```

## Use it (Claude Code)
```bash
/plugin marketplace add https://bitbucket.org/<workspace>/ideo-qa-marketplace.git
/plugin install ideo-qa@ideo-qa-marketplace
/reload-plugins
```
Private repo: works with your normal git credentials (SSH key or HTTPS token) — if `git clone` works in your terminal, this works.

## Pull updates (Daniel)
After changes are pushed to Bitbucket:
```bash
/plugin marketplace update ideo-qa-marketplace
/reload-plugins
```
For hands-off background updates on a private repo, set `BITBUCKET_TOKEN` (app password or repo access token) in the shell environment.

## Shipping a new version
1. Edit `ideo-qa/skills/*/SKILL.md` (and bump `version` in `ideo-qa/.claude-plugin/plugin.json`).
2. Commit and push.
3. Everyone runs `/plugin marketplace update` (or it auto-updates if a token is set).

## Note on non-CLI testers
The Claude **app**'s org-level auto-sync marketplace is GitHub-only today. If QA testers who don't use the CLI need automatic updates, mirror this repo to GitHub and connect it under Organization settings > Plugins, or have them self-install the `.plugin` file until then.
