# ESCaiDO Workbench UI — React integration (2026-10-24)

## Where things live
- Source: `workbench/ui/` (React 18 + TS + Vite). Builds to `workbench/static/ui/`,
  served by the FastAPI app at **/ui**. The older vanilla page stays at `/`.
- Backend: `workbench/server.py` (FastAPI, SQLite/WAL). Port 12000.
- Typed adapter: `workbench/ui/src/lib/api.ts` — the ONLY place that calls fetch.
- Live events: `workbench/ui/src/lib/useEventStream.ts` (SSE, dedupe by event id).
- Browser tests: `workbench/ui/e2e/*.py` (Playwright, real Chromium, no mocks).
  `seed_10k.py` creates the 10,000-message conversation for pagination tests.

## Build/run
    cd workbench/ui && npm install && npm run build      # tsc -b && vite build
    python -m workbench.run                              # serves /ui and /api

## Bugs that only showed up when driving a real browser
1. **Toast never auto-dismissed** — messages stacked forever.
2. **Scroll anchoring**: setting scrollTop inside `requestAnimationFrame` raced
   React's commit of prepended nodes, so loading older messages jumped to top.
   Fix = `useLayoutEffect` keyed on the message array + a saved scrollHeight.
3. **RTL drawer**: `inset-inline-end` resolves to the LEFT edge in RTL, so a
   `translateX(+102%)` drawer parked itself partly on screen (~85px visible).
   Use `inset-inline-start` when the transform pushes in the positive X direction.
4. **Compose queue polluted by history**: reloading a conversation put past
   uploads back into the send queue, silently re-sending them. Keep the pending
   queue and the attachment history as separate state.
5. **Offline never detected**: an open EventSource survives a network drop, so
   SSE `onerror` is not a trustworthy connectivity signal. Poll `/api/health`
   and listen to online/offline events; AND the result with the SSE state.
6. **CSS class mismatch**: the prototype markup (`message-row`, `app-shell`)
   and my earlier stylesheet (`msg`, `.sidebar`) did not overlap, so the React
   build rendered completely unstyled. Always diff className usage against the
   stylesheet before believing a "design" is applied.

## Verification results (this run)
- build clean; 13/13 core e2e, 7/7 resilience, 5/5 attachments.
- 10,000 messages: initial DOM = 40 rows; DOM grows 40 -> 200 only when paging;
  frame time ~39ms; search across all 10k returns hits. No freeze.
- 0 JS errors and 0 failed requests in every run.

## Still not connected (and the UI says so)
Live voice input, a managed browser/terminal session, MCP/skill discovery.
Buttons for these are labelled unavailable rather than faking success.

## Session 2 — "models not connected" (same day)

### The three real causes
1. **Latency, not wiring.** `gemini-3.8-flash` was FIRST in every role chain but
   returns HTTP 429. Every message burned 20-37s before failover, which reads as
   "the model isn't connected". Measured live in parallel:
   `nemotron-3.5-super-vl 463ms | gpt-oss-20b 467ms | minimax-m3.1-flash 687ms |
   glm-5.3 4495ms | glm-5.3-flash 134320ms (unusable) | gemini-3.8-flash 429`.
   Fix = reorder chains fastest-first + `PER_MODEL_TIMEOUT=45` so one slow model
   cannot stall the chain. Result: 37s -> 1.5s.
2. **Vision was never wired.** `_attachment_context` only told the model that a
   binary file existed ("لم يُحلَّل محتواه"). Images must be sent as
   `{"type":"image_url","image_url":{"url":"data:<mime>;base64,..."}}` parts, and
   an image must FORCE the vision role.
3. **Wrong app at the root.** React was only mounted at `/ui`; `/` still served
   the old vanilla prototype. The user was using the prototype. Root now serves
   React; the old page moved to `/legacy`.

### Gotcha: reasoning models look like failures
`nemotron-3.5-super-vl` is a reasoning model: it fills `reasoning_content` first,
so a 3000-token cap at a 45s timeout ALWAYS times out. Keep a `REASONING` set,
cap those models at ~700 tokens, give them `VISION_TIMEOUT` (120s), and read
`reasoning_content` when `content` is null. Same trick needed for any
`*thinking*`/`*-vl` model added later.

### New endpoints
- `GET/PUT /api/settings` (whitelist of keys) — theme, accent, lang, density,
  radius, font_scale, animations, default_role. Table `settings(scope,key,value)`.
- `POST /api/models`, `DELETE /api/models/{id}` — the API key is deliberately
  NOT accepted from the browser; only provider/model_id/base_url are stored.
- `GET/PUT /api/skills` — DB-backed skill registry.

### Theme architecture
`html[data-theme=light]` re-declares the SAME CSS variables as the dark base, so
no component needs per-theme rules. `--accent-ink` is computed from accent
luminance so text on any accent stays readable. Measured light contrast 14.35:1.
Also `data-density`, `data-radius`, `data-animations` on `<html>`.

### Verification
12/12 settings+chat checks, all 41 visible buttons clicked with 0 JS errors,
6/6 model roles OK, vision "أحمر" in 1.6s, Arabic reply in ~2s.

## Session 3 — media, modes, professional output, phone layout

### Video and audio actually work now
- Audio: OpenAI-style `{"type":"input_audio","input_audio":{"data":b64,"format":"wav"}}`.
  `inline_data` is REJECTED by the Gemini OpenAI-compatible endpoint (400).
- Video: a `data:video/mp4;base64,...` URL inside the normal `image_url` part works.
- Only the Gemini models accept these parts. NVIDIA models ignore them.
  Keep `MEDIA_CAPABLE` in sync; anything else silently answers about the wrong thing.
- Verified: sine-wave WAV -> correct frequency/duration/timbre in a table;
  a 2-colour test MP4 -> correct colour order + "two tones" + duration.

### Model roster is data-driven from a live probe
The provider catalogue advertises many models that return **410 Gone**
(`meta/llama-3.3-70b`, `openai/gpt-oss-120b`, `qwen/*`, `deepseek-*` ...). Seeding
from the catalogue created dead entries. `DEFAULT_MODELS` now lists only models
verified by an actual call. `laguna-xs` is fastest (198ms) but intermittently
returns **503 ResourceExhausted**, so it is a fallback, not a chain lead.

### Performance: the bundle was the "lag"
Entry chunk was 1054 kB because mermaid (~1.5 MB) was imported eagerly.
Fix = `import('mermaid')` on first diagram + `manualChunks` for react/markdown/icons.
Entry chunk is now **47 kB**; DOMContentLoaded 1054 kB -> **124 ms**.

### Real phone bug found
`rightOpen` defaulted to `true`, and the work panel is a fixed full-height overlay
under 920px — it covered the whole screen and swallowed taps meant for the top bar
(the drawer button was unreachable). Now it starts closed on small screens and has
a `panel-scrim` to dismiss by tapping outside.

### Modes
`ChatIn.depth` (brief/normal/detailed) and `ChatIn.search` (off/deep/max) shape the
system prompt. Depth chips + search chip live in the composer toolbar.

## Session 4 — council, auto-replace, UI repair

### The conversation really did vanish (found the cause)
An old `@media (max-width:650px)` block set `.work-panel{inset:0;width:100%}` and
`.chat-column{visibility:hidden}`. On a phone that hid the entire thread and
overlapped the newer 920px drawer rules. Removing the block fixed it. Also the
panel was rendered WITHOUT the `.open` class, so it stayed at `translateX(-102%)`
and every control inside was unreachable.

### Phone drawer geometry
`width:100vw` at <=420px left zero scrim, so tapping outside could not dismiss the
panel. Now 88vw at <=420px and 86vw under 920px, leaving a 43-50px strip.
The panel is anchored LEFT in RTL — tests must tap the RIGHT edge.

### Fonts
The sheet used 8-10px in 41 places. Nothing below 11px now, and 12-12.5px inside
panels on touch screens. That was most of the "primitive" impression.

### Closed drawer must be inert
The sidebar was only translated off-screen, so ~24 buttons stayed focusable and
hit-testable. `.sidebar{visibility:hidden}` + `.sidebar.sidebar-open{visibility:visible}`.

### Council (workbench/council.py)
Five lenses (fact/logic/edge/risk/improve), chosen per role; `fast`,
`vision_alt` and `code_fallback` never get one. Each lens is pinned to a DIFFERENT
model from `DIVERSITY_POOL` — five critics on one model share its blind spots and
the panel is worthless. Critics run concurrently, then the lead synthesises and
must end with "تعديلات مقبولة: N". A failed critic retries twice then benches the
model. Measured: 4 distinct models, ~49s for code+detailed, and the synthesis
visibly tightened the code.

### Auto-replace
`council.record_failure` counts consecutive failures; at 2 the model is benched
(`status='benched'`) and the best healthy model OF THE SAME ROLE is promoted to
the front of the chain. Same-role matching is deliberate: a reasoning model must
never be replaced by a cheap chat model. Ranking prefers models with proven calls,
then lowest measured latency from the usage table.

### Kilo: keyless extra quota
`api.kilo.ai/api/gateway/v1` serves `nvidia/nemotron-3-ultra-550b-a55b:free`
without any key. A bogus-path control returned HTTP 400 (not 200 'OK'), proving
the reply is genuine and not sandbox-proxy interception — the failure mode that
made models.github.ai look like it worked. `_headers` must NOT send an empty
Bearer token; `has_key` returns True for `KEYLESS_BASE` providers.

## Session 5 — real browser + deep research

### Browser (workbench/browser.py)
Playwright + `/usr/bin/chromium` (the managed download is absent). Verified:
github.com/OpenHands HTTP 200, 290 links, 27 buttons, 11,309 chars.
Live view = CDP `Page.startScreencast` (jpeg q55, everyNthFrame 2) → WebSocket
`/api/browser/live`, measured 30 frames / 6s. CDP emits NOTHING for an idle page,
so seed a frame on connect and re-send the last frame as a heartbeat or the
viewer is blank. crawl4ai does NOT work here: it needs Playwright-managed
browsers and its BrowserConfig has no executable_path.
traffic note: `--disable-blink-features=AutomationControlled` + a real UA.

### Deep research (workbench/research.py)
GitHub + GitLab + Hugging Face only (primary sources). ~400 pages/min measured.
THREE bugs found the hard way:
1. `verdict_of(out)` inside the `out.update({...})` literal ran BEFORE the values
   existed → saw days_since_push=9999 → dropped everything. Assign, then judge.
2. REST `/repos/x/issues` returns mostly PULL REQUESTS in busy repos
   (browser-use: 38/40). Use GraphQL: exact `totalCount` for open/closed in ONE
   call, plus most-discussed issue bodies. Real resolution rates: browser-use 89%,
   SeleniumBase 99%.
3. Word-count sentiment over issue text is worthless as a primary signal —
   issue text is inherently negative. Use a health score: resolution rate 0.40,
   open-pressure 0.25, recency 0.15, popularity 0.10, licence 0.10, sentiment
   0.10. >=0.70 keep, >=0.45 unclear, else drop.
`search/issues` gives a transient 403 at its 30/min budget → retry 3x.

## Session 6 — modes, Alexa, real mic, device-adaptive UI

### The mode chips were decorative
`api.chat()` never sent `depth`, `search` or `council`, although server.py had
accepted `depth` and `council` all along. So the three chips (مختصر/مفصّل/بحث)
changed NOTHING. `api.chat` now takes an options object and App sends depth,
search and council. The three cycling chips were replaced by one named menu of 8
modes (سريع/محادثة/مفصّل/بحث/بحث عميق/ماكس/بناء/تحليل وسائط).

### Mode menu anchoring
The menu was absolutely positioned to the trigger and overflowed to -4px at
360px. On <=920px it is now `position:fixed;left:8px;right:8px` so it can never
leave the viewport. Verified at 360/390/414.

### Alexa (real)
`/api/transcribe` sends the recorded clip to an audio model. VERIFIED:
gemini-3-flash identified a 440Hz tone as silence, then transcribed a real
command correctly. The transcription prompt MUST be strict ("write down ONLY the
spoken words… if no words output فارغ") or the model answers the speech instead
of transcribing it. `/api/alexa` runs the command and returns the reply text,
which the browser speaks locally so there is no second round trip. Three voice
profiles (بنت/رجل/مهندس) are applied via SpeechSynthesis pitch/rate.
BUG: `store.create_conversation()` returns the id STRING, not a dict — using
`conv["id"]` raised TypeError and 500'd the endpoint.

### Real microphone
The old mic button only flashed "not connected". It now records via MediaRecorder
and inserts the transcription into the composer. Verified recording starts and
stops.

### Device adaptation
main.tsx reads the real device before first paint: `data-dev`
(phone/tablet/desktop), `data-touch`, and `--vh` from the measured innerHeight
(mobile URL bars change innerHeight, so a fixed 100vh overflows). CSS sizes chat
text, padding and tap targets per device, plus `env(safe-area-inset-*)`.

### Duplicates removed
The top bar had a second "النماذج" button duplicating the panel tab; removed. Top
bar is now only الإعدادات / السمة / المساعدة.

## Session 7 — screen control, self-repair, site builder, bridge, MCP, Telegram

### Screen control
The live frame is letterboxed (object-fit:contain), so clicks must map through the
drawn rectangle, not the element box, or every click lands in the wrong place.
Added click_at (normalised 0..1), scroll, key. A "تجميد" (freeze) button stops the
stream so the view holds still while you aim — without it the image moves under the
cursor. Verified: 1204 links / 20 buttons on github.com/trending, click dispatched.

### Self-repair (workbench/repair.py)
Snapshot to .repair/ → write → VERIFY → roll back on failure. Verified for real:
it rewrote a test module, ast.parse passed, and the new code ran correctly. A
deliberately broken replacement was rejected and the original restored. Blocked
paths (.repair, data/, .git, static/, node_modules) can never be written.

### Site builder (workbench/sites.py)
Two stages: previews then build. Measured problems:
- max_tokens=8000 TRUNCATES at ~3200 chars; 4000 fails outright. Fixed with
  chunked continuation (~4000 per call, up to 4 rounds).
- A 45s per-model timeout killed full-page generation. Added a `timeout` param to
  call_with_failover/call_model; sites use 180s.
- 3 previews sequentially = 220s (browser gave up). asyncio.gather → 105s.
Verified: 3 designs, real Arabic content in an iframe, final build 9402 chars.

### Bridge (workbench/bridge_agent.py + /ws/bridge)
CONSENT-BASED by design, and it says so: the user runs the agent, types `yes`,
every command prints in their terminal, no arbitrary shell, no persistence, no
autostart. The connection goes OUT from their machine, so nothing can reach a
machine that is not running the agent. Unapproved screenshot/fetch are refused.
Verified with a real websocket client: status, info, disk, big_files, screenshot.

### MCP server (workbench/mcp_server.py)
stdio JSON-RPC, 5 tools (browse, research, site_preview, repair, ask). Verified
initialize + tools/list + a real tools/call.

### Telegram
Real and NOT intercepted (bogus method → 404, getMe → @escaidoaiopd_bot).
Webhook set to /api/telegram/webhook. Commands: /ask /repair /preview /sites
/browse /pc /status. BUG: httpx was not imported in server.py → NameError.

### UI
4 new tabs (بناء/إصلاح/جهازي + المتصفح controls) → 9 tabs total. Phone tab strip
scrolls horizontally instead of squashing. 20/20 across 5 viewports.
