# Project instructions for Claude Code

This is the "plugin-registry" project — a GitHub-native PocketMine-MP plugin
registry (GitHub org: poggit-alternative-test for now, migrating to axolotl-pm
later) that replaces Poggit using only free GitHub features.

Before doing any work in this repo, read:
- `ARCHITECTURE.md` — binding design decisions and constraints
- `ROADMAP.md` — phased task breakdown

Rules:
- Treat ARCHITECTURE.md §2 as constraints, not suggestions. Do not silently
  reinterpret or work around them.
- Do not implement anything listed under ARCHITECTURE.md §8 (non-goals)
  unless the current task explicitly asks for it.
- If a task needs a decision not covered in ARCHITECTURE.md, stop and ask —
  especially anything touching auth, token scope, or what gets executed vs.
  only read.
- Apply least privilege to every permission, token scope, and GitHub Actions
  `permissions:` block.
- Stick to the decided stack: React + Vite + TypeScript + Tailwind CSS,
  GitHub Actions for CI, GitHub Pages for hosting.
- Work on ONE phase from ROADMAP.md at a time, exactly as scoped in that
  phase's checklist. Don't pull in later-phase work even if it seems
  convenient to bundle in.
- Before marking a phase's checklist item done, re-read ARCHITECTURE.md §6
  (security model) if the item touches auth, tokens, or Actions workflows.