<!-- MEM:HEADER -->
# Skaido Memory — ESCaiDO
<!-- /MEM:HEADER -->

<!-- MEM:STATE -->
## الحالة الحالية
_2026-10-25 11:36_

**المنظومة كاملة ومُختبَرة.**
🎯 **الكود القصير:** `/workspace/sk` (42 سطراً) — استعادة كاملة في ثوانٍ
📝 **المسجّل:** `scripts/recorder.py` — يسجّل كل رسالة تلقائياً في TRANSCRIPT.md ويدفع كل 5 دقائق
🛡️ **الحرس:** 600ث · كاشف تكرار · reaper مستقل
♻️ **المشرف:** 5 خدمات (الحارس، المسجّل، البوت، خادمان)
🔒 **نسختان:** GitHub + git محلي · حجب 15 نوع سر
<!-- /MEM:STATE -->

<!-- MEM:FACTS -->
## حقائق دائمة
- الاستعادة تعمل: محو .openhands/memory كاملاً ثم /workspace/sk → 17 ملفاً · 272 سطر سجل · 41 حقيقة · 196KB سجل محادثة · الخزنة تفتح
- recorder.py: يقرأ llm_message من MessageEvent (لا message) · role في llm_message.role · يكتب TRANSCRIPT.md غير محدود + يحجب الأسرار
- GitHub push protection يحجب أي ملف فيه سر حقيقي: أنواع اكتُشفت فعلياً = VERCEL_PERSONAL_ACCESS_TOKEN (vcp_) و GOOGLE_GCP_API_KEY_BOUND_SERVICE_ACCOUNT (AQ.) — كشفها…
- memory.db ملف ثنائي .db لا يُدفع عبر contents API — أُعيد بناؤه من memory-export.json في restore (الذي يُصدَّر تلقائياً قبل كل دفع)
- الكارثة التي اكتشفتها: الحارس الخلفي دفع ذاكرة فارغة فوق GitHub ومسحت 41 حقيقة → الحل: _is_empty() يرفض الدفع الفارغ + _git_backup() نسخة ثانية في git المحلي
- الحارس المستقل (Reaper) ضروري: يقتل حتى المهمة التي لا تفحص نفسها — لأن الخيط المعلّق لا يستطيع أن يقتل نفسه
- الاختبارات المُثبتة: الحرس يقتل عند 6ث (6.0ث فعلياً) · الحارس المستقل قتل مهمة صمّاء عند 5ث · /restore يعمل في 5.4ث
- المشرف يفحص المنافذ (curl) لا أسماء العمليات — pgrep -f يفشل مع 'python3 server.py' الذي يعمل بـ cwd مختلف
- تطبيق workbench يحتاج: python3 -m workbench.run مع httpx+uvicorn+fastapi+python-multipart · المنفذ 12000
- البدء التلقائي: .agents/plugins/skaido-memory/hooks/startup.sh يُشغّل scripts/supervise.sh في كل جلسة (SessionStart) — idempotent
- الأمر الحاسم: python3 scripts/mem.py restore يستعيد الذاكرة + القاعدة + الخزنة + .env من GitHub
- بوت التيليجرام @escaidoaiopd_bot: /start /restore /status /progress /keys /save /projects · رمز الاستعادة SKAIDO-RESTORE · التوكن في الخزنة (TG_TOKEN)
- المشرف scripts/supervise.sh: يُبقي الحارس + البوت + الخادم حيّين للأبد · يفحص المنافذ لا الأسماء · يستعيد المفاتيح من الخزنة تلقائياً
- الواجهة agent.js: fetchGuard بمهلة 600ث + AbortController + 3 إعادات بمضاعفة (0.8/1.6/3.2ث) · window.abortAll للإلغاء اليدوي
- أسباب التكرار/الglitch التي أُصلحت: لا AbortController في الواجهة (fetch يعلّق للأبد) · إعادة المحاولة على finish_reason=length · لا سقف زمني على مسار المحادثة
- الحرس (guard.py): سقف صارم 600ث لأي عملية · كاشف تكرار حرفي · حارس مستقل يقتل حتى المهمة التي لا تفحص نفسها
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
<!-- /MEM:FACTS -->