<!-- MEM:HEADER -->
# Skaido Memory — ESCaiDO
<!-- /MEM:HEADER -->

<!-- MEM:STATE -->
## الحالة الحالية
_2026-10-25 11:09_

**المرحلة 1 مكتملة: الذاكرة الدائمة.**
`scripts/mem.py` يحفظ كل شيء · الخزنة تحفظ المفاتيح · GitHub يحفظ نسخة خارج الحاوية.
**الاستعادة من الصفر: `python3 scripts/mem.py restore`**
الحد الحاسم: MEMORY.md ≤5500 حرف (النظام يحقن 6000 فقط — ما زاد يُقطع بصمت).
المستودع البعيد عام → المفتاح لا يُدفع · الخزنة المشفّرة تُدفع.
<!-- /MEM:STATE -->

<!-- MEM:FACTS -->
## حقائق دائمة
- لا تشغّل push بمجلد ذاكرة فارغ — يحمي نفسه الآن ويرفض الكتابة فوق نسخة بعيدة سليمة
- التوكن الكلاسيكي ghp_ يملك صلاحيات كاملة؛ fine-grained كان قراءة فقط. التوكن في .env + الخزنة
- حد الحقن 6000 حرف — MEMORY.md يجب أن يبقى ≤5500؛ الحقائق تُقصَّر إلى 165 حرفاً والكامل في memory.db
- الحارس: scripts/mem-daemon.sh كل 15 دقيقة — snapshot + محاولة دفع (لا crontab في البيئة، حلقة خلفية)
- الخطاف التلقائي: .agents/plugins/skaido-memory/hooks/hooks.json — UserPromptSubmit يقرأ آخر 5000 حرف في كل رسالة
- الخزنة .openhands/vault.enc مشفّرة AES-256(CTR)+HMAC-SHA256 · المفتاح .openhands/.vaultkey متتبَّع في git
- المستودع البعيد: klbha779-source/memyre — عام! لذلك vault.key.b64 لا يُدفع أبداً (يُحمى تلقائياً)
- الأمر الحاسم للاستعادة الكاملة بعد فقدان كل شيء: python3 scripts/mem.py restore (يستعيد الذاكرة + القاعدة + الخزنة + .env)
- منظومة الذاكرة الدائمة: scripts/mem.py — الأوامر: load/tail/note/fact/set/add/project/projects/compress/seal/unseal/push/pull/restore/doctor/export
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
<!-- /MEM:FACTS -->