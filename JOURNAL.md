# Development Journal

A running log of what gets done on zemingwang.de.

---

## 2026-05-25 (Day 1)

### Done
- Installed local dev environment: Homebrew, Node.js v26, VS Code with 5 core extensions
- Configured Git identity and SSH key for GitHub authentication
- Cloned the repository from `git@github.com:iknownothinglol/zemingwang-de-website.git` to `~/Code/`
- Installed and authenticated Claude Code CLI (Sonnet 4.6 via Claude Pro)
- Created CLAUDE.md — a project guide for AI-assisted development
- Set up .gitignore for project hygiene
- Made first AI-assisted commits and pushed to main

### Learned
- AI agents do not always complete multi-step instructions in full. Always verify the actual state after Claude reports "done":
  1. Check files exist locally (`ls -la`)
  2. Check git status (`git status`, `git log`)
  3. Check the remote on GitHub via browser
- `git commit` saves locally; `git push` is a separate step that syncs to GitHub. Confusing them is a common mistake.

### Tomorrow
- Apply for GitHub Student Pack with RWTH email (for free Copilot Pro)
- Plan the modularization refactor for index.html
