# 🔓 Unclosed for LeetCode

> open bracket, **no closure.** fr fr

A small Chrome extension (Manifest V3) that stops LeetCode's editor from auto-closing
`(` `[` `{` `"` `'` (and `` ` ``). You type it, you own it. It also throws a confetti
party every time a submission comes back **Accepted**.

| You type | Without Unclosed | With Unclosed |
|---|---|---|
| `(` | `()` | `(` |
| `{` | `{}` | `{` |
| `[` | `[]` | `[` |
| `"` | `""` | `"` |
| `'` | `''` | `'` |

## Features

- **Live on/off toggle.** Flip it in the popup or press **Alt+Shift+U**. Every open LeetCode
  tab switches straight away, with no reload. Turning it on gives you a sparkle burst 🔓✨.
- **Fixes the code editor and the testcase box.** The code editor is Monaco and the
  testcase input is CodeMirror 6. Both are handled.
- **Celebrates Accepted.** Confetti cannons, sparkles and an `ACCEPTED` badge with a random
  line ("ate that, no crumbs 💅"). It only fires for real submissions, not for "Run".
  You can turn it off separately.
- **Doesn't get in the way.** Indentation, syntax highlighting, shortcuts, copy/paste, Run
  and Submit all work as normal. Typing a closing bracket yourself inserts exactly one.
- **Lightweight.** No libraries, no remote code, no tracking. The only permission is `storage`.

## Install (unpacked)

1. Open `chrome://extensions` in Chrome (or any Chromium browser: Edge, Brave, Arc).
2. Turn on **Developer mode** (top right).
3. Click **Load unpacked** and pick this `unclosed` folder (the one with `manifest.json`).
4. Pin the Unclosed icon from the puzzle-piece menu.
5. Open any problem on `https://leetcode.com/problems/...`. If a LeetCode tab was already
   open before you installed, refresh it once.

To change the shortcut, go to `chrome://extensions/shortcuts`.

## How it works

LeetCode's editors are JavaScript objects owned by the page. A normal content script can't
reach them, so `src/page-hook.js` runs in the page's **MAIN world** at `document_start`.

**Monaco (code editor)**
- It doesn't assume `window.monaco` already exists. It sets a trap on the property so it
  catches Monaco the moment Monaco's loader publishes it. A `MutationObserver` scanner (with
  an AMD `require` fallback) is the backup.
- It uses `monaco.editor.onDidCreateEditor` plus `getEditors()` to cover every editor,
  including the new ones LeetCode creates when you move between problems or switch language.
- It applies `autoClosingBrackets`, `autoClosingQuotes` and `autoSurround` = `"never"`.
- It wraps each editor's `updateOptions`. That way, if LeetCode re-applies its own settings,
  auto-closing can't sneak back on. When you toggle off, the page's original values come back.

**CodeMirror 6 (testcase box)**
CodeMirror's `closeBrackets` extension can't be reconfigured from outside, so this works at
the input level. When you press an opening key, the keystroke becomes a normal
`input.type` transaction that inserts just that character. `closeBrackets` never runs, so
nothing gets inserted and then deleted afterwards. IME composition and Ctrl/Cmd/Alt
shortcuts are left alone.

**CodeMirror 5** (legacy, in case it ever shows up): `setOption('autoCloseBrackets', false)`.

**Accepted detection**
It watches LeetCode's own `/problems/<slug>/submit/` and
`/submissions/detail/<id>/check/` responses without modifying them. A result counts as
Accepted when `status_code === 10` or `status_msg === "Accepted"`. "Run" results
(`runcode_…`) are ignored. As a fallback it also reads the result panel's
`data-e2e-locator="submission-result"`. You get one party per submission.

## Files

```
unclosed/
├── manifest.json        MV3 manifest (leetcode.com only)
├── background.js        keyboard shortcut + "OFF" toolbar badge
├── src/
│   ├── page-hook.js     MAIN world: editor config, input handling, submit watcher
│   ├── content.js       isolated world: settings bridge (chrome.storage <-> page)
│   └── fx.js            confetti, sparkles, toast, ACCEPTED badge (shadow DOM + canvas)
├── popup/               popup UI (toggle, celebrate switch, preview button, status)
└── icons/               16 / 32 / 48 / 128 px
```

## Testing checklist

1. Open a problem. The popup footer should say `connected · Monaco ✓`.
2. In **C++**, **Python** and **JavaScript**, type `( { [ " '`. Each should appear once.
3. Type `foo(x)`. You should get exactly `foo(x)`, with no doubled `)`.
4. In the **Testcase** tab, type `"([`. No closers should appear.
5. Press **Alt+Shift+U**. Auto-close comes back right away, and a toast says so. Press it
   again and the sparkles come back.
6. Go to another problem with the Next/Prev arrows, without refreshing. It should still work.
7. Click **Preview the party 🎉** in the popup, or submit a correct solution.

## Troubleshooting

- **Nothing happens:** refresh the LeetCode tab once after installing or updating the extension.
- **Popup says "open a LeetCode problem to connect":** the active tab isn't on `leetcode.com`.
- **Too much motion:** if your OS has *Reduce motion* turned on, the celebration is just a
  few gentle sparkles. Or switch off "Celebrate Accepted".
- Only `leetcode.com` is supported. `leetcode.cn` isn't included.
