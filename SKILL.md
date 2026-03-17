---
name: browser
description: |
  Fast headless browser for testing, dogfooding, and interacting with web pages. Navigate URLs,
  click/fill/select elements, take screenshots, check responsive layouts, inspect console errors,
  test forms, handle dialogs, and assert element states. Persistent Chromium daemon: first call
  ~3s startup, then ~100ms per command. Use when asked to: test a website, verify a deployment,
  dogfood a user flow, check a page, take a screenshot, fill a form, QA a feature, or interact
  with any web page. Also use when user says "browse", "open", "go to", or gives a URL to check.
---

# /browse: Fast Headless Browser

Persistent headless Chromium. First call auto-starts (~3s), then ~100ms per command.
State persists between calls (cookies, tabs, login sessions).

## Setup (run once per session before any browse command)

```bash
B=~/.claude/skills/browser/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

If `NEEDS_SETUP`: tell the user "The browse skill needs a one-time build (~30 seconds). OK to proceed?" Then STOP and wait for confirmation. Once confirmed:

```bash
cd ~/.claude/skills/browser && ./setup
```

If `bun` is not installed, the setup script will install it automatically.

## How It Works

The `$B` binary talks to a persistent Chromium server over localhost HTTP. The server starts on first use and stays alive for 30 min of idle time. Every command after startup is a fast HTTP call (~100ms).

The key concept is **ref-based element selection**: run `snapshot` to get an accessibility tree where every element has a ref like `@e1`, `@e3`. Then use those refs in subsequent commands (`click @e3`, `fill @e4 "hello"`). This is faster and more reliable than CSS selectors.

## Core Patterns

### 1. Verify a page loads correctly
```bash
$B goto https://yourapp.com
$B text                          # content loads?
$B console --errors              # JS errors?
$B network                       # failed requests?
$B is visible ".main-content"    # key elements present?
```

### 2. Test a user flow (the snapshot + ref pattern)
```bash
$B goto https://app.com/login
$B snapshot -i                   # see all interactive elements with @refs
$B fill @e3 "user@test.com"     # use refs — no CSS selectors needed
$B fill @e4 "password"
$B click @e5                     # submit
$B snapshot -D                   # diff: what changed after submit?
$B is visible ".dashboard"       # success state present?
```

### 3. Verify an action worked
```bash
$B snapshot                      # baseline
$B click @e3                     # do something
$B snapshot -D                   # unified diff shows exactly what changed
```

### 4. Visual evidence for bug reports
```bash
$B snapshot -i -a -o /tmp/annotated.png   # labeled screenshot with ref boxes
$B screenshot /tmp/bug.png                # plain screenshot
$B console --errors                       # error log
```

### 5. Find all clickable elements (including non-ARIA)
```bash
$B snapshot -C                   # finds divs with cursor:pointer, onclick, tabindex
$B click @c1                     # interact via @c refs
```

### 6. Assert element states
```bash
$B is visible ".modal"
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"
$B is checked "#agree-checkbox"
$B is editable "#name-field"
$B is focused "#search-input"
$B js "document.body.textContent.includes('Success')"
```

### 7. Test responsive layouts
```bash
$B responsive /tmp/layout        # mobile + tablet + desktop screenshots
$B viewport 375x812              # or set specific viewport
$B screenshot /tmp/mobile.png
```

### 8. Test file uploads
```bash
$B upload "#file-input" /path/to/file.pdf
$B is visible ".upload-success"
```

### 9. Test dialogs
```bash
$B dialog-accept "yes"           # set up handler
$B click "#delete-button"        # trigger dialog
$B dialog                        # see what appeared
$B snapshot -D                   # verify deletion happened
```

### 10. Compare environments
```bash
$B diff https://staging.app.com https://prod.app.com
```

### 11. Show screenshots to the user
After `$B screenshot`, `$B snapshot -a -o`, or `$B responsive`, always use the Read tool on the output PNG(s) so the user can see them. Without this, screenshots are invisible.

### 12. Unblock sites with cookie import
Sites like Google block headless browsers with CAPTCHAs. Import the user's real cookies to bypass:
```bash
$B cookie-import-browser chrome --domain .google.com
```
**Key details:**
- Use **dot-prefixed domains** (`.google.com` not `google.com`) — cookies are stored under the root domain with a leading dot
- Supported browsers: `chrome`, `arc`, `brave`, `edge`
- First run triggers a macOS Keychain dialog ("Allow access to Chrome Safe Storage") — tell the user to click Allow
- Works for any site: `.github.com`, `.notion.so`, `.slack.com`, etc.
- After import, navigate normally — the session carries the user's auth

**When to use:** If a site returns a CAPTCHA, login wall, or bot detection page, ask the user: "This site is blocking the headless browser. Want me to import your cookies from Chrome to bypass it?"

## Snapshot Flags

The snapshot is the primary tool for understanding and interacting with pages.

```
-i        --interactive           Interactive elements only (buttons, links, inputs) with @e refs
-c        --compact               Compact (no empty structural nodes)
-d <N>    --depth                 Limit tree depth (0 = root only, default: unlimited)
-s <sel>  --selector              Scope to CSS selector
-D        --diff                  Unified diff against previous snapshot (first call stores baseline)
-a        --annotate              Annotated screenshot with red overlay boxes and ref labels
-o <path> --output                Output path for annotated screenshot (default: /tmp/browse-annotated.png)
-C        --cursor-interactive    Cursor-interactive elements (@c refs — divs with pointer, onclick)
```

All flags can be combined freely. `-o` only applies when `-a` is also used.
Example: `$B snapshot -i -a -C -o /tmp/annotated.png`

**Ref numbering:** @e refs are assigned sequentially (@e1, @e2, ...) in tree order.
@c refs from `-C` are numbered separately (@c1, @c2, ...).

After snapshot, use @refs as selectors in any command:
```bash
$B click @e3       $B fill @e4 "value"     $B hover @e1
$B html @e2        $B css @e5 "color"      $B attrs @e6
$B click @c1       # cursor-interactive ref (from -C)
```

**Output format:** indented accessibility tree with @ref IDs, one element per line.
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

Refs are invalidated on navigation — run `snapshot` again after `goto`.

## Full Command Reference

### Navigation
| Command | Description |
|---------|-------------|
| `goto <url>` | Navigate to URL |
| `back` | History back |
| `forward` | History forward |
| `reload` | Reload page |
| `url` | Print current URL |

### Reading
| Command | Description |
|---------|-------------|
| `text` | Cleaned page text |
| `html [selector]` | innerHTML of selector, or full page HTML if no selector |
| `links` | All links as "text -> href" |
| `forms` | Form fields as JSON |
| `accessibility` | Full ARIA tree |

### Interaction
| Command | Description |
|---------|-------------|
| `click <sel>` | Click element |
| `fill <sel> <val>` | Fill input |
| `select <sel> <val>` | Select dropdown option by value, label, or visible text |
| `hover <sel>` | Hover element |
| `type <text>` | Type into focused element |
| `press <key>` | Press key (Enter, Tab, Escape, ArrowUp/Down, etc.) |
| `scroll [sel]` | Scroll element into view, or scroll to page bottom |
| `wait <sel\|--networkidle\|--load>` | Wait for element, network idle, or page load (15s timeout) |
| `viewport <WxH>` | Set viewport size |
| `upload <sel> <file> [file2...]` | Upload file(s) |
| `cookie <name>=<value>` | Set cookie on current page domain |
| `cookie-import <json>` | Import cookies from JSON file |
| `cookie-import-browser [browser] [--domain d]` | Import cookies from Chrome, Arc, Brave, or Edge |
| `header <name>:<value>` | Set custom request header |
| `useragent <string>` | Set user agent |
| `dialog-accept [text]` | Auto-accept next dialog |
| `dialog-dismiss` | Auto-dismiss next dialog |

### Inspection
| Command | Description |
|---------|-------------|
| `js <expr>` | Run JavaScript expression and return result |
| `eval <file>` | Run JavaScript from file (path must be under /tmp or cwd) |
| `css <sel> <prop>` | Computed CSS value |
| `attrs <sel\|@ref>` | Element attributes as JSON |
| `is <prop> <sel>` | State check (visible/hidden/enabled/disabled/checked/editable/focused) |
| `console [--clear\|--errors]` | Console messages (--errors filters to error/warning) |
| `network [--clear]` | Network requests |
| `dialog [--clear]` | Dialog messages |
| `cookies` | All cookies as JSON |
| `storage [set k v]` | Read/write localStorage + sessionStorage |
| `perf` | Page load timings |

### Visual
| Command | Description |
|---------|-------------|
| `screenshot [--viewport] [--clip x,y,w,h] [sel\|@ref] [path]` | Save screenshot |
| `pdf [path]` | Save as PDF |
| `responsive [prefix]` | Screenshots at mobile/tablet/desktop |
| `diff <url1> <url2>` | Text diff between pages |

### Snapshot
| Command | Description |
|---------|-------------|
| `snapshot [flags]` | Accessibility tree with @refs (see Snapshot Flags above) |

### Tabs
| Command | Description |
|---------|-------------|
| `tabs` | List open tabs |
| `tab <id>` | Switch to tab |
| `newtab [url]` | Open new tab |
| `closetab [id]` | Close tab |

### Server
| Command | Description |
|---------|-------------|
| `status` | Health check |
| `stop` | Shutdown server |
| `restart` | Restart server |

### Meta
| Command | Description |
|---------|-------------|
| `chain` | Run commands from JSON stdin: `[["cmd","arg1",...],...]` |
