# CLAUDE.md — Callrecordingapp

## Project Overview

**Voice Recorder Pro** is a standalone, zero-dependency HTML5 progressive web application for recording and managing audio calls. It runs entirely in the browser with no backend, build step, or package manager. The full application lives in a single file: `call-recorder.html.html`.

The app is bilingual (Arabic/English) with full RTL/LTR layout switching, and integrates Claude AI for recording analysis and Q&A.

---

## Repository Structure

```
Callrecordingapp/
├── call-recorder.html.html     # Entire application (HTML + CSS + JS)
├── .github/
│   └── workflows/
│       └── static.yml          # GitHub Actions: auto-deploy to GitHub Pages
└── CLAUDE.md                   # This file
```

There are no other source files, packages, or build artifacts.

---

## Tech Stack

| Layer       | Technology                                   |
|-------------|----------------------------------------------|
| Markup      | HTML5 (semantic, single-file SPA)            |
| Styling     | CSS3 with CSS custom properties (variables)  |
| Logic       | Vanilla JavaScript (ES2020, no frameworks)   |
| Audio       | MediaRecorder API (WebRTC)                   |
| Storage     | `localStorage` (recordings + settings)       |
| AI          | Anthropic Claude API (REST, client-side)     |
| Fonts       | Google Fonts — Cairo + Space Mono            |
| Deployment  | GitHub Pages via GitHub Actions              |

---

## Application Architecture

The app is a single-page application (SPA) with three tabs, all rendered in one HTML file.

### Pages / Tabs

| Tab ID          | Page ID              | Purpose                                      |
|-----------------|----------------------|----------------------------------------------|
| `tab-record`    | `page-record`        | Record audio, save/discard, Claude Q&A panel |
| `tab-recordings`| `page-recordings`    | Browse, search, play, analyze, delete        |
| `tab-settings`  | `page-settings`      | Language toggle, API key setup, quality       |

Navigation is controlled by `showPage(name)` which swaps `.active` CSS class on pages and tabs.

### State (Global Variables)

```js
isRecording    // bool — current recording state
mediaRecorder  // MediaRecorder instance
audioChunks    // Blob[] — raw audio chunks during capture
timerInterval  // setInterval handle for recording timer
seconds        // int — elapsed recording seconds
recordings     // Array<RecordingObject> — loaded from localStorage on init
currentLang    // 'ar' | 'en' — active language
currentBlob    // Blob | null — the just-recorded audio, before saving
apiKey         // string — Claude API key from localStorage
audioElements  // { [id]: HTMLAudioElement } — cached audio players
```

### Recording Object Schema

```js
{
  id: number,          // Date.now() timestamp (unique ID)
  name: string,        // Contact name / title
  note: string,        // Free-text note
  date: string,        // Locale-formatted date string
  time: string,        // Locale-formatted time string (HH:MM)
  duration: number,    // Recording length in seconds
  size: string,        // Human-readable size e.g. "128 KB"
  url: string,         // Object URL (ephemeral, session-only)
  blobData: string     // Base64 data URL — persisted in localStorage
}
```

### localStorage Keys

| Key              | Value                                            |
|------------------|--------------------------------------------------|
| `recordings`     | JSON array of recording objects (base64 blobs)   |
| `lang`           | `'ar'` or `'en'`                                 |
| `claudeApiKey`   | The user's Anthropic API key (`sk-ant-…`)        |

---

## Key Functions Reference

### Recording Flow

| Function           | Purpose                                              |
|--------------------|------------------------------------------------------|
| `toggleRecord()`   | Start/stop recording via `getUserMedia` + MediaRecorder |
| `updateTimer()`    | Increment `seconds` and update `#timer` every 1s    |
| `saveRecording()`  | Convert blob to base64, push to `recordings`, persist to localStorage |
| `discardRecording()` | Clear `currentBlob` and hide save form             |

### Playback

| Function              | Purpose                                           |
|-----------------------|---------------------------------------------------|
| `togglePlay(id)`      | Play/pause audio; stops any other playing audio   |
| `seekAudio(e, id)`    | Click-to-seek on progress bar                     |

### Recordings List

| Function              | Purpose                                           |
|-----------------------|---------------------------------------------------|
| `renderRecordings(filter)` | Re-render the recordings list with optional search filter |
| `filterRecordings()`  | Read `#searchBar` value and call `renderRecordings` |
| `updateCount()`       | Update the badge showing total recording count    |
| `shareRec(id)`        | Use Web Share API or fall back to `.webm` download |
| `deleteRec(id)`       | Remove from array, update localStorage, re-render |

### AI Integration

| Function           | Purpose                                              |
|--------------------|------------------------------------------------------|
| `askAI()`          | Send user question to Claude API with recent recordings as context |
| `analyzeRec(id)`   | Ask Claude to summarize/analyze a specific recording |
| `showApiModal()`   | Show the API key entry modal                         |
| `saveApiKey()`     | Persist API key to localStorage                      |

**Claude API call details:**
- Endpoint: `https://api.anthropic.com/v1/messages`
- Model: `claude-sonnet-4-20250514`
- `max_tokens`: 500 (Q&A) / 400 (analysis)
- API key sent as `x-api-key` request header (client-side, user-provided)
- `anthropic-version`: `2023-06-01`

### Internationalisation

| Function       | Purpose                                                      |
|----------------|--------------------------------------------------------------|
| `tr(key)`      | Returns the translated string for `currentLang`             |
| `applyLang()`  | Updates all visible text nodes and HTML dir/lang attributes  |
| `toggleLang()` | Switches `currentLang`, saves to localStorage, calls `applyLang()` |

All UI strings live in the `t` object with two top-level keys: `t.ar` (Arabic) and `t.en` (English).

---

## CSS Design System

CSS custom properties defined in `:root`:

```css
--bg:       #0a0a0f    /* Page background */
--surface:  #12121a    /* Cards, header */
--surface2: #1a1a26    /* Inputs, secondary surfaces */
--border:   #2a2a3d    /* Border colour */
--accent:   #ff3b5c    /* Red — recording state, active tab */
--accent2:  #7b5ea7    /* Purple — AI, primary buttons */
--green:    #00e676    /* Unused reserved */
--blue:     #448aff    /* AI gradient, secondary accent */
--text:     #e8e8f0    /* Primary text */
--text2:    #8888aa    /* Secondary/muted text */
```

Layout: fixed max-width of `420px`, centred — optimised for mobile.

Fonts:
- **Cairo** (Arabic-capable, 300/400/600/700/900 weights) — UI text
- **Space Mono** (400/700 weights) — timer display

---

## Deployment

### GitHub Actions (`.github/workflows/static.yml`)

- **Trigger:** Push to `main` branch, or manual `workflow_dispatch`
- **Action:** Uploads the entire repository root to GitHub Pages
- **Concurrency:** One deployment at a time; in-progress deployments are never cancelled
- **Permissions required:** `pages: write`, `id-token: write`

The deployed URL serves `call-recorder.html.html` directly — no index.html rename is done in the workflow, so the URL will include the filename.

---

## Development Conventions

### Editing the Application

Since everything is in `call-recorder.html.html`, all changes (markup, styles, logic) go in that one file. Sections are clearly separated:
1. `<style>` block — all CSS
2. HTML body — structure and static content
3. `<script>` block at the bottom — all JavaScript

### Adding Features

1. **New UI string:** Add keys to both `t.ar` and `t.en`, then reference with `tr('key')` and update `applyLang()` to apply the text to the DOM element.
2. **New page/tab:** Add a `<div class="tab" id="tab-X">` and `<div class="page" id="page-X">` pair, then register the tab click to call `showPage('X')`.
3. **New recording field:** Add the property to the object in `saveRecording()` and update `renderRecordings()` to display it.
4. **New AI feature:** Follow the `askAI()` / `analyzeRec()` pattern — `fetch` to the Claude API, show loading dots, render the response text.

### LocalStorage Constraints

Recording blobs are stored as base64 data URLs. `localStorage` has a typical browser limit of ~5–10 MB. Recordings will silently fail to persist if this limit is exceeded (`try/catch` wraps the `setItem` call in `saveRecording`).

### No Build Process

There is no transpilation, bundling, minification, or linting toolchain. Edit the file and open it directly in a browser, or push to `main` to deploy via GitHub Pages.

### Testing

There is no automated test suite. Manual testing should cover:
- Recording start/stop in Chrome, Firefox, Safari (MediaRecorder codec support varies)
- Save and reload (localStorage persistence)
- Language toggle (RTL/LTR layout)
- AI Q&A with a valid Anthropic API key
- Share (Web Share API on mobile, download fallback on desktop)

---

## Browser Compatibility Notes

- **MediaRecorder:** Supported in all modern browsers. Safari requires iOS 14.5+. Output format is `audio/webm` (Chrome/Firefox) or browser default.
- **Web Share API:** Mobile only for most browsers; desktop Chrome 89+ partially supports it.
- **localStorage:** Available in all modern browsers; may be disabled in private/incognito on some browsers.
- **RTL layout:** Uses `dir="rtl"` on `<html>` and CSS `dir` toggling — no polyfills needed.

---

## Security Notes

- The Claude API key is stored in `localStorage` in plain text. This is intentional for a client-side-only app but means it is accessible to any JavaScript running on the same origin.
- API calls to `api.anthropic.com` are made directly from the browser. CORS is supported by the Anthropic API for browser clients.
- No user data is transmitted anywhere except to the Anthropic API (only recording metadata and notes — never the raw audio blob).
