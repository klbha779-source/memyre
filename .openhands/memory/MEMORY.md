<!-- MEM:HEADER -->
# Skaido Memory — ESCaiDO
<!-- /MEM:HEADER -->

<!-- MEM:STATE -->
## الحالة الحالية
_2026-10-25 11:08_

_updated 2026-10-24_

**SECOND WORKSTREAM — Agent Skaido AI Workbench (live now):** the React+FastAPI app in
`workbench/` is served on **port 12000 → https://work-1-dhtwigrnoupljkbx.prod-runtime.all-hands.dev/**
(no Vercel). Phases 1–3 done & committed: conversation-centric UI, live task cards in the message
stream, and an in-conversation design studio (4 designs A–D, revisions, approval, then build).
`python3 -m workbench.tests_e2e` = **26/26 passing**. Model roster must stay `733e33360f11c8bf`.
Full API shapes, measured timings and pitfalls: `2026-10-23.md` → "Agent Skaido AI (Phases 1–3)".

**Phase:** Iraqi TTS on **LAHGTNA** (Kaggle + Telegram bot). The older **Habibi/F5-IRQ + ComfyUI**
path is **SUPERSEDED** — do not cite it as the live stack (older entries below still mention it;
that contradiction was the cause of confusing advice). Archived copy: `detail/memory-archive-2026-10-24.md`.

**Working:** env cell + deep-clone cell + Telegram bot cell (`tools/kernel_cells/`, notebook
`kaggle_tts/escdo_tts.ipynb`). 7 voices load on a T4. Duration buttons + scene parser pass tests.
**Not proven:** no audio sample has ever been listened to. The CUDA `device-side assert` cause is
**NOT established** (see below). Do not claim the TTS is fixed.
**Next:** get the full traceback + what the audio actually sounds like, then diagnose for real.
<!-- /MEM:STATE -->

<!-- MEM:FACTS -->
## حقائق دائمة
- **COUNCIL + AUTO-REPLACE 2026-10-24**: `workbench/council.py` runs a 5-lens panel (fact/logic/edge/risk/improve) per role; each lens is pinned to a DIFFERENT model …
- **MEDIA + ROSTER 2026-10-24**: Audio works ONLY via `input_audio` (`inline_data` → 400 on the Gemini OpenAI endpoint); video works as a `data:video/mp4;base64` URL …
- **WORKBENCH MODELS 2026-10-24**: "the models aren't connected" was really LATENCY — `gemini-3.8-flash` was first in every role chain but returns 429, so each messag…
- **UI INTEGRATION 2026-10-24**: The React RTL workbench UI now lives in `workbench/ui/` and is served at **/ui** (build output `workbench/static/ui/`); the old vanil…
- MODEL_COMPARISON.md, FREE_API_PROVIDERS.md, AGENT_ARCHITECTURE.md, SECURITY_AND_LIMITATIONS.md.
- **REPORTS written to `reports/` (2026-10-24):** ESCaiDO_BRAIN_RESEARCH.md (main, Arabic),
- **REVISED CAPACITY 2026-10-24:** Gemini 100/day + NVIDIA ~100-400/day + Kilo 200/hr ⇒ **~300-700 req/day** vs the 1000-2000 target. Improved ~3x with the NVIDIA key…
- **GEMINI KEY (measured 2026-10-24) -> `detail/gemini-key-measurements-2026-10-24.md`:** works via OpenAI-compatible `/v1beta/openai/chat/completions` (Bearer) so it…
- **NVIDIA KEY (measured 2026-10-24) -> `detail/nvidia-key-measurements-2026-10-24.md`:** works. Only 4/85 models usable: **`z-ai/glm-5.3`** (free HERE — plans claime…
- and push detail into `detail/`. Check pointers resolve (`grep -o 'detail/.*\.md'`).
- **Memory budget: ~6000 chars combined**; over budget drops lines from the TOP. Keep this file lean
- AGENT/QUOTA AUDIT 2026-10-24 -> `detail/free-quota-evidence-2026-10-24.md` + `detail/agent-audit-2026-10-24.md`: Kilo is the only keyless free API (200/hr/IP = 3.33…
- PLAN v3 (`docs/plan-tts-v3.md`) is authoritative; v2 and FINAL-PLAN.md are superseded. HARD FACT: reference audio is TRUNCATED (`ENC_COND_LEN=6*16000`=6.0s, `DEC_CO…
- Details: `detail/telegram-bot-2026-10-23.md`.
- Cell templates keep `__TOKEN__`/`__CHAT__`/`__REF_IDS__`, rendered by `tools/make_cell.py`.
- TELEGRAM BOT: @escaidoaiopd_bot, chat_id 5399915161. Token ONLY in `~/.escdo/telegram.json` (600).
- sob moan pause breath stutter hes weep throat_clear.
- [gasp] [groan] [mumble] [sniff] [cough] [clear_throat] [inhale] [UH]. Invalid: laugh scream shout
- FIX: `_itu.clean_tags = lambda t: t`. Valid: [whisper] [shhh] [laughter] [giggle] [cry] [sigh]
- rewrites `[tag]`->`<tag>` and the vocab has no `<tag>`, so tags shredded to chars.
- EMOTION TAGS 2026-10-23 -> `detail/emotion-tag-rootcause-2026-10-23.md`: `iraqiTextUtil.clean_tags()`
- `"no"` (Norwegian code reused). `"iq"` emits an `[iq]` token with no normalization. Needs an A/B test.
- **`language_id="iq"` is NOT wired to the Iraqi normalizer.** `tokenizer.py` maps the normalizer to
- `max_text_tokens=2048`, `max_speech_tokens=4096`, vocab 2454, stop_token 6562.
- post-processing layer (librosa time_stretch; beyond ~20% it distorts, so split the text instead).
- cleanliness, not length. There is NO duration control in `generate()` — scene timing needs a
- `DEC_COND_LEN=10*24000` (10.0s). Cloning from hours of audio is useless; quality comes from
- **HARD MODEL LIMITS (Lahgtna):** reference audio is TRUNCATED — `ENC_COND_LEN=6*16000` (6.0s) and
- ESCaiDO ON VERCEL (2026-10-24): LIVE at https://escaido-workbench.vercel.app (stable alias -> latest production). Preview: https://escaido-workbench-giicx8h5y-klbha…
- **BIG FEATURE SET 2026-10-24**: self-repair (workbench/repair.py) = snapshot → write → VERIFY → auto-rollback; blocked paths .repair/data/.git/static/node_modules. …
- **MODES + ALEXA + MIC 2026-10-24**: the three mode chips were DECORATIVE — `api.chat()` never sent `depth`/`search`/`council` although server.py accepted them. Now …
- **REAL BROWSER + DEEP RESEARCH 2026-10-24**: `workbench/browser.py` drives `/usr/bin/chromium` via Playwright (goto/click/type/read/links/screenshot) and streams CD…
<!-- /MEM:FACTS -->