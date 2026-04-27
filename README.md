# AI Foundations — Starter Repo

This is the starter repository for **AI Foundations: Architecting the Next Generation of Apps**. Clone it, rename it, and make it yours.

[![Fork this template](https://img.shields.io/badge/Fork_this_template-24292e?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Bumbolio/agentic-coding-starter/fork)
[![View syllabus](https://img.shields.io/badge/View_syllabus-a8361a?style=for-the-badge)](https://bumbolio.github.io/agentic-coding-starter/)

## What's in here

```
.
├── CLAUDE.md                          # Project context Claude reads on every session
├── README.md                          # This file
├── setup.sh                           # Run once — renames claude-config to .claude
├── docs/
│   ├── PRD.md                         # Template for your Product Requirements Document
│   ├── DESIGN.md                      # Template for your UI/UX design brief
│   └── BUILDPLAN.md                   # Template for your phased, context-window-sized build plan
└── claude-config/                     # Will become .claude/ after running setup.sh
    ├── settings.example.json          # Example hooks config (rename to settings.json)
    └── skills/
        ├── pm-interview/              # Have Claude interview you to produce a PRD
        ├── design-brief/              # Have Claude interview you to produce a design brief
        ├── build-plan/                # Slice the project into phases that fit one session each
        ├── rubber-duck-quiz/          # Quiz yourself on code before committing
        ├── explain-this-code/         # Have Claude teach you code at the right depth
        ├── architecture-diagram/      # Generate + critique Mermaid diagrams
        └── cloudflare-product-picker/ # Pick the right CF product for a need
```

> **Why `claude-config` instead of `.claude`?** Folders starting with `.` are hidden by default on macOS and Windows, so people downloading the starter often think the skills are missing. We ship it visible. Run `./setup.sh` to rename it to `.claude` where Claude Code expects to find it.

## Getting started

1. **Fork or clone this repo**, then rename it to match your project.
2. **Run `./setup.sh`** to move `claude-config/` into place as `.claude/`. (On Windows without a bash shell, just rename the folder manually from `claude-config` to `.claude`.)
3. **Install Claude Code** if you haven't (see course materials).
4. **Install Wrangler:** `npm install -g wrangler` and then `wrangler login`.
5. **Open Claude Code in the project directory:** `claude`.
6. **Run the PM interview** to build your PRD:
   ```
   > Use the pm-interview skill to help me write a PRD for [your idea]
   ```
7. **Save the PRD** to `docs/PRD.md`. Claude will update it as you go.
8. **Run the design-brief interview** before writing UI code:
   ```
   > Use the design-brief skill to help me fill out docs/DESIGN.md
   ```
   Defaults are React + Headless UI + Tailwind. The brief captures any deviation.
9. **Bootstrap your app** with `npm create cloudflare@latest` once the PRD and design brief are clear enough.
10. **Run the build-plan interview** after bootstrapping, before writing real features:
    ```
    > Use the build-plan skill to help me fill out docs/BUILDPLAN.md
    ```
    Output is 3–6 phases sized to fit one Claude Code session each. Re-run it when the plan drifts from reality.

## The skills

Use them like this:

| When you're... | Use the skill... |
|---|---|
| Starting a new project or feature | `pm-interview` |
| About to build UI for the first time | `design-brief` |
| About to start coding a project | `build-plan` |
| About to commit a chunk of AI-written code | `rubber-duck-quiz` |
| Staring at code you don't fully grasp | `explain-this-code` |
| Planning architecture or reviewing your own | `architecture-diagram` |
| Unsure which Cloudflare product to use | `cloudflare-product-picker` |

You don't have to name the skill — Claude will pick it up from context. But saying "use the rubber-duck-quiz skill" forces it.

## Enabling hooks

Hooks make Claude run your test suite automatically so you catch regressions in the moment. After running `setup.sh`:

```bash
cp .claude/settings.example.json .claude/settings.json
```

Then either keep the default `Stop` hook (runs tests when Claude finishes a turn — recommended) or swap in the `PostToolUse` hook (runs after every edit — stricter).

## Course rules

- **No `--dangerously-skip-permissions` until week 3.** Read every diff.
- **Run `rubber-duck-quiz` before every meaningful commit.** Claude can draft the commit message and run `git commit` for you — that's encouraged. The rule is that you must understand the change well enough to defend it before it lands.
- **Architecture diagrams get regenerated every week.** If it no longer matches the code, something drifted.
- **Two videos required, both linked in this README:**
  - A **PRD explainer video** (≤5 min) recorded after week 1.
  - A **demo video** (≤5 min) recorded by week 4 — the deployed app plus 2–3 file-level decisions.

## Your project links

_Replace these once you deploy and record:_

🔗 **Live site:** [https://your-project.pages.dev](https://your-project.pages.dev)
🎥 **PRD video:** [your-prd-video-link]
🎥 **Demo video:** [your-demo-video-link]
