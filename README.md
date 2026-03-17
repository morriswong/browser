# browser

A fast headless browser skill for [Claude Code](https://claude.ai/claude-code). Persistent Chromium daemon — first call ~3s startup, then ~100ms per command.

## Install

```bash
git clone https://github.com/morriswong/browser.git ~/.claude/skills/browser
cd ~/.claude/skills/browser && ./setup
```

Requires [Bun](https://bun.sh) (installed automatically by `setup` if missing).

## Usage

Once installed, use `/browser` in Claude Code to navigate pages, click elements, fill forms, take screenshots, and more. See [SKILL.md](SKILL.md) for the full command reference.
