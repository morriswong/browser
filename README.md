# browser

A fast headless browser skill for [Claude Code](https://claude.ai/claude-code). Persistent Chromium daemon — first call ~3s startup, then ~100ms per command. Navigate pages, click elements, fill forms, take screenshots, test responsive layouts, and more — all from Claude Code.

## Features

- **Persistent Chromium daemon** — starts once (~3s), then ~100ms per command
- **Ref-based element selection** — use `snapshot` to get `@e1`, `@e2` refs instead of fragile CSS selectors
- **Snapshot diffing** — see exactly what changed after an action with `snapshot -D`
- **Annotated screenshots** — labeled screenshots with ref overlays for bug reports
- **Responsive testing** — mobile, tablet, and desktop screenshots in one command
- **Cookie import** — bypass login walls by importing cookies from Chrome, Arc, Brave, or Edge
- **State persistence** — cookies, tabs, and login sessions persist between calls
- **Full interaction** — click, fill, select, hover, type, upload files, handle dialogs

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- macOS, Linux, or WSL
- [Bun](https://bun.sh) (installed automatically by `setup` if missing)

## Install

### Option 1: Git clone (recommended)

```bash
git clone https://github.com/morriswong/browser.git ~/.claude/skills/browser
cd ~/.claude/skills/browser && ./setup
```

### Option 2: Claude Code plugin marketplace

```
/plugin marketplace add morriswong/browser
```

> **Custom config directory?** If you use a custom `CLAUDE_CONFIG_DIR` (e.g. `~/.claude-personal`), clone into `$CLAUDE_CONFIG_DIR/skills/browser` instead.

## Quick Start

Once installed, use `/browser` in Claude Code. Here are common patterns:

**Check a page:**
```
/browser goto https://example.com
/browser text
/browser console --errors
```

**Test a user flow (snapshot + ref pattern):**
```
/browser goto https://app.com/login
/browser snapshot -i              # see interactive elements with @refs
/browser fill @e3 "user@test.com"
/browser fill @e4 "password"
/browser click @e5                # submit
/browser snapshot -D              # diff: what changed?
```

**Take a screenshot:**
```
/browser screenshot /tmp/page.png
/browser responsive /tmp/layout   # mobile + tablet + desktop
```

**Import cookies to bypass login walls:**
```
/browser cookie-import-browser chrome --domain .github.com
```

## Full Command Reference

See [SKILL.md](SKILL.md) for the complete list of commands, snapshot flags, and usage patterns.

## Troubleshooting

- **First run is slow?** The setup script installs Playwright Chromium (~100MB). Subsequent runs start in ~3s.
- **Site shows CAPTCHA/bot detection?** Use `cookie-import-browser` to import your real browser cookies.
- **macOS Keychain dialog?** Click "Allow" when prompted for Chrome Safe Storage access during cookie import.
- **Binary missing after update?** Re-run `./setup` to rebuild.

## License

MIT
