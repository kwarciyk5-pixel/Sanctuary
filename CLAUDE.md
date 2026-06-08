# Sanctuary — Project Notes

Personal daily-tracker PWA. Single HTML file with embedded CSS + JS, stored in localStorage. No build step, no framework, no dependencies. Served from GitHub Pages.

## User context

- **User is non-technical.** They don't use the terminal. Walk them through GUI steps when something needs their hands; do everything else yourself.
- They've asked: **after any code change, commit and push to GitHub automatically.** Show the commit message and a one-line summary of what changed each time. Don't ask permission for every push — the standing approval was given.

## Where things live

- **Working folder:** `/Users/thettant/Documents/GitHub/Sanctuary/` — this is the git repo. ALL edits go here.
- **GitHub repo:** https://github.com/kwarciyk5-pixel/Sanctuary (branch `main`)
- **Live site:** https://kwarciyk5-pixel.github.io/Sanctuary/ (updates ~1 min after push)
- **Old folder** `/Users/thettant/Documents/Sanctuary/` is stale — ignore it. User may delete it eventually.

## Git workflow

Credentials are in macOS Keychain (`credential.helper = osxkeychain`), set up via GitHub Desktop. Plain `git push` from terminal works without prompting.

Standard sequence after editing:
```
git -C /Users/thettant/Documents/GitHub/Sanctuary add <files>
git -C /Users/thettant/Documents/GitHub/Sanctuary commit -m "short message"
git -C /Users/thettant/Documents/GitHub/Sanctuary push
```

Author is already configured: `kwarciyk5-pixel <kwarci.yk5@gmail.com>`. Do not change.

## App architecture (index.html)

Single `<script>` tag holds all JS. Key globals around line 1178:

- `STORAGE_KEY = 'sanctuary_v1'` — localStorage key. **Never rename** (would orphan user's data).
- `PASSCODE = '5555'` — lock screen passcode.
- `appData = {}` — populated from localStorage; shape includes `days[YYYY-MM-DD]`, `healthEntries`, tasks, etc.

Render functions (each tab has its own): `renderHome`, `renderStats`, `renderTasks`, `renderHealth`, `renderMoney`, `renderLists`, plus helpers like `renderCalendarStrip`, `renderTimeline`, `renderDailySummary`.

Import/export: `importData()` accepts either a single day object **or** an array of day objects (batch). `importSingleDay(data)` does the actual merge and returns count of new entries added. `exportData()` dumps full `appData` as JSON.

## Code conventions

- **ASCII straight quotes only.** No curly/smart quotes anywhere in source. Verify with:
  ```
  python3 -c "d=open('/Users/thettant/Documents/GitHub/Sanctuary/index.html',encoding='utf-8').read(); print(sum(d.count(c) for c in '“”‘’'))"
  ```
  Must print `0`.
- **Backward compatibility is critical.** Old localStorage data and old single-day JSON imports must keep working. Additive changes only unless explicitly asked otherwise.
- **No dependencies.** Don't add npm/CDN scripts. Single-file app.

## What NOT to touch without explicit ask

- Lock screen logic (passcode, dots) — works, user relies on it
- Tab navigation switching
- `STORAGE_KEY` value
- CSS (it's hand-tuned)
- Export function (`exportData`) — round-trips must stay clean

## Verifying changes

No `node`, `deno`, or `bun` is installed on this machine. To syntax-check JS:

```
python3 -c "
import re
d=open('/Users/thettant/Documents/GitHub/Sanctuary/index.html',encoding='utf-8').read()
m=re.findall(r'<script[^>]*>(.*?)</script>', d, re.DOTALL)
open('/tmp/sanc.js','w').write(m[0])
"
/System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Helpers/jsc -e "const src = readFile('/tmp/sanc.js'); try { new Function(src); print('PARSE OK'); } catch(e) { print('PARSE ERROR:', e); }"
```

Must print `PARSE OK`. For runtime testing, user opens the live site or local file in a browser and tries the feature.

## Recent work

- **2026-06-08:** Added batch import support. `importData()` now accepts a JSON array of day objects in addition to single-day objects. Split out `importSingleDay(data)` helper that returns added-entry count. Single-day status message stays in `importData` (the helper doesn't touch `statusEl`, which only exists in `importData`'s scope). Old single-day imports unchanged.

## Style for talking to the user

Concise, direct. No filler ("Great question!", "Absolutely!"). No emojis unless they use them. Walk through GUI steps numbered, with the exact button text in **bold** so they can scan. If something is risky or irreversible, flag it before doing it.
