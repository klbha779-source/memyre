# MEMORY.md archive before consolidation (2026-10-24)

<!-- MEM:HEADER -->
# Project memory — ESCaiDO
<!-- /MEM:HEADER -->

<!-- MEM:STATE -->
_updated 2026-10-23T12:20Z_

**Phase:** Iraqi TTS — LIVE ENGINE IS **LAHGTNA** (Kaggle + Telegram bot), superseding the Habibi/ComfyUI path below
**✅ Done:**
- P1 memory · P2 tool-first hook · P3 consistency guard · P4 hook hardened · P5 YouTube/media
- **TTS a1:** `tools/make_ref.py` cuts a 12s ref (skydo 19–31s, score 30–42s, both ~97% speech)
- **TTS b1/b2/c1/c2:** `tools/build_tts_workflow.py` + `comfy/workflows/iraqi_tts_skydo.json` → first Iraqi clip generated, scored, delivered
**▶️ Next:**
- score a `F5-IRQ` test, then voicebank for non-cloned characters
- Kaggle GPU bridge (m8) — CPU generation of one line takes ~5 min
**❓ Open:**
- `ref_text` for skydo/score are Whisper drafts (`ref_text_verified: false`) — need review
- Telegram token NOT set; delivery is via public ComfyUI `/view` URL
<!-- /MEM:STATE -->

<!-- MEM:LEDGER -->
- **D13** 2026-10-23: The dialect problem is SOLVED without any new model: use the **Habibi dataset's own Iraqi speakers** (`SWivid/Habibi`, config IRQ, 2198 clips, speaker_id 0..274, 3-12s each, Apache-2.0) as reference clips for every non-cloned character, so the SAME Habibi-IRQ engine voices the whole scene and the accent never drifts. Do NOT use VoxCPM2 text voice-design for them (generic Arabic = two accents in one scene). — because "other characters must also be 100% Iraqi" was the blocking requirement
- **D14** 2026-10-23: The ComfyUI TTS path WORKS headlessly: `tools/build_tts_workflow.py` posts an API graph (LoadAudio -> F5TTSAudioInputs(model=F5-IRQ, model_type=F5TTS_v1_Base) -> SaveAudio) to `/prompt`; the graph is saved to `comfy/workflows/iraqi_tts_<char>.json`. Deliver results via the PUBLIC ComfyUI `/view?filename=<f>&type=output` URL, and the file must sit in `ComfyUI/output/`. — because Telegram token is unset and the 12000 gateway (comfy_gateway.py) has no `/api/report` route (405)
- **D15** 2026-10-23: FOUR environment traps break Iraqi TTS; each cost a full debug cycle: (1) `LoadAudio` reads `ComfyUI/input/` ROOT, not `input/audio/`; (2) **FFmpeg was missing** → torchcodec "Could not load libtorchcodec" (fix `sudo -n apt-get install -y ffmpeg`; plain apt-get has no dpkg lock); (3) resemblyzer needs `pip install "setuptools<81"` (81+ dropped pkg_resources); (4) faster-whisper chokes on file paths (av version clash) → pass a 16k float32 numpy array. All belong in `escdo_boot.sh`.
- _(older D-entries were never archived; treat D1–D12 as lost — do not cite them)_
<!-- /MEM:LEDGER -->

<!-- MEM:FACTS -->
- Iraqi TTS engine: Habibi-TTS **IRQ** (Apache-2.0) wired into the ComfyUI node as model `F5-IRQ` (model_type F5TTS_v1_Base). Weights cached permanently at `/workspace/hf_home`; `/tmp/hf` and `/tmp/ttsvenv` were wiped by the reset.
- ComfyUI env: 1017 nodes, 5 F5-TTS nodes, agent `crdt=on`, public gateway live. Long findings live in `detail/` and the daily logs (`2026-10-23.md`).

- AGENT-PLAN AUDIT 2026-10-24 -> `detail/agent-audit-2026-10-24.md`: a "5 brains + router" plan contained FABRICATED items (OptiRouter, VeloceGrid, ProjectOS 2026, NVIDIA 57,600/day) and wrong limits (Groq, Cerebras, Gemini Pro). **MEASURED: MoA/council LOST to a single model** (84s, empty answer, vs 25s and 10/10 alone) — reasoning models burn max_tokens on `reasoning` then return empty `content` (use max_tokens>=4000, fall back to the `reasoning` field). **OpenHands is ALREADY the router: it runs on LiteLLM**, so a LangGraph layer is redundant. Only 14/20 Kilo models are free; the strong ones are 401/PAID.

- FREE-QUOTA EVIDENCE 2026-10-24 -> `detail/free-quota-evidence-2026-10-24.md`: **Kilo gateway is the only keyless free LLM API that actually works** — official limit 200 req/HOUR per IP (measured: 30 OK then 100 consecutive 429). That is 3.33 req/min, so "5-10 req/min sustained" is IMPOSSIBLE on one IP. Exhaustion body: PROMOTION_MODEL_LIMIT_REACHED, X-Should-Retry:false. **LESSON: /v1/models listing is public on every provider and proves NOTHING about free inference — HF router, ModelScope, NVIDIA NIM, OpenRouter all list models then 401 on /chat/completions.** Only Pollinations works keyless (but ~1 req/4s then 402). Groq Whisper is genuinely free (20 RPM, 2000 audio req/day, no card) for STT. Free hosting: HF Spaces free tier NO LONGER allows Docker Spaces (needs PRO); GitHub Codespaces 120 core-hrs/mo is the strongest real free browser option; Render spins down + ephemeral disk + 30-day Postgres; Railway needs a card; Fly.io has no free tier. **OpenHands is built on LiteLLM (proof: TIKTOKEN_CACHE_DIR -> /litellm/) so it IS already a model router — a LangGraph routing layer is redundant.** Audit of alternatives: no project combines web UI + real code execution + live event stream + MCP + skills + persistent memory + resume except OpenHands. Open WebUI is NOT OSI open source since v0.6.6 (branding clause). DEAD/ARCHIVED: Flowise, Roo Code, opencode-go.
<!-- /MEM:FACTS -->
- EMOTION-TAG ROOT CAUSE 2026-10-23: full proof in `detail/emotion-tag-rootcause-2026-10-23.md`. Short: Lahgtna's `iraqiTextUtil.clean_tags()` rewrites `[tag]`->`<tag>`; the vocab has NO `<tag>`, so every emotion was shredded to single chars. FIX: `_itu.clean_tags = lambda t: t` after the sys.path insert. Valid tokens: [whisper] [shhh] [laughter] [giggle] [cry] [sigh] [gasp] [groan] [mumble] [sniff] [cough] [clear_throat] [inhale] [UH]. Invalid: laugh scream shout sob moan pause breath stutter hes weep throat_clear.
- TELEGRAM BOT 2026-10-23: @escaidoaiopd_bot, chat_id 5399915161. Token ONLY in `~/.escdo/telegram.json` (600). Never hardcode in repo: cell templates keep `__TOKEN__`/`__CHAT__`/`__REF_IDS__`, rendered by `tools/make_cell.py`. Details: `detail/telegram-bot-2026-10-23.md`.
- PLAN AUDIT 2026-10-23 (user's "final TTS plan", verified): full list in `detail/plan-audit-2026-10-23.md`. Short — FOUR errors: (1) it presents the `[tag]`->`<tag>` rewrite as a FEATURE, it is the bug (0/18 tags usable); (2) `Conditionals.save(model.prepare_conditionals(wav), p)` is wrong — that method returns nothing, use `model.prepare_conditionals(wav,exaggeration=.5); Conditionals.save(model.conds,p)`; (3) `pip install perth==1.0.0` is a TRAP — PyPI `perth` is unrelated and shadows the needed `resemble-perth`; (4) "generate child/old/woman from scratch with a seed" is impossible — generate() asserts conds is not None, every voice needs a reference. Correct: repo/model ids, 7.47GB, iq->"no", MuseTalk MIT, MOSS-TTSD Apache-2.0.
- PLAN v3 (docs/plan-tts-v3.md) is the CURRENT authoritative TTS plan; docs/plan-tts-v2.md and docs/FINAL-PLAN.md are SUPERSEDED (they describe Habibi-TTS IRQ, not the live Lahgtna stack). HARD FACT 2026-10-23: the reference audio is TRUNCATED — `ENC_COND_LEN = 6*16000` = 6.0s (T3 tokens) and `DEC_COND_LEN = 10*24000` = 10.0s (vocoder) in prepare_conditionals. Cloning from 1-2 HOURS of audio is USELESS; the model README asks for a clean 3-10s mono clip at >=22kHz. Quality comes from cleanliness, not length. Also: Lahgtna has NO duration control at all (no duration/length_scale/speed in generate()) — exact scene timing MUST come from a post-processing layer (librosa.effects.time_stretch / pyrubberband, CPU); time-stretch beyond ~20% audibly distorts, so split the text instead.

