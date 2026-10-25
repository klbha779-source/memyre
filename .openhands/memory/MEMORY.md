<!-- MEM:HEADER -->
# Skaido Memory — ESCaiDO
<!-- /MEM:HEADER -->

<!-- MEM:STATE -->
## الحالة الحالية
_2026-10-25 13:51_

**المرحلة 3: حماية آخر لحظة — مُنجزة.**
⏱️ **الخطاف:** كل رسالة ترسلها → تُدفع قبل أن أرد (UserPromptSubmit)
⚡ **سرعة الدفع:** 19ث → **0.5ث** (بصمات sha256)
🔒 **قفل:** يمنع 3 رسائل سريعة من التسجيل 3 مرات
🚪 **flush عند الإغلاق:** SIGTERM/SIGINT/atexit
✅ **مُختبَر:** رسالة جديدة → GitHub في 3 ثوان
<!-- /MEM:STATE -->

<!-- MEM:FACTS -->
## حقائق دائمة
- طلب المستخدم: تمام ابدا الان بهذه الخطه وكما قلت لك عندما تريد ان تبحث بان الان فصاعدا لا تبحث في ابحث في منظوماتنا الجديده ومنظوماتنا الجديده اجعلها من ضمن الام سي…
- طلب المستخدم: ياخي لا تشغل هذا الوكيل الفرعي وكيل فرعي: web-researcher هذا الوكيل يتاخر جدا في الرد اريدك تستبدله بخطاف سريع او ام سي بي مختلف ولكن سريع في البحث هذ…
- طلب المستخدم: اريدك تقارن المنظومه القديمه بمنظومات الان انت تبحث عنها الان ابحث عنها وشوف هل هذه منظومه القديمه الافضل ام يوجد لديك فكره ومنظومه لن تطرق على اي انس…
- طلب المستخدم: الآن بعد ما تأكدنا إن الذاكرة محفوظة، الآن نبدأ بالشيء الجدي، الأهم اللي هو هدفنا الأصلي. الآن أريد منك تعمل منظومة بحث متكاملة شاملة كاملة لأوبن هاند…
- ملف الخطافات الفعلي: .agents/plugins/skaido-memory/hooks/hooks.json (فيه UserPromptSubmit + SessionStart). الخطاف لا يعمل في الجلسة الجارية إن أُضيف بعد بدئها — يعم…
- قفل .runtime/recorder.lock (90ث) ضروري: الخطاف يُنفَّذ عند كل رسالة، و3 رسائل سريعة = 3 عمليات متزامنة تسجّل نفس الدور 3 مرات. أُضيف أيضاً منع تكرار في record() يقا…
- الأداء: push() كان 19ث → صار 0.5ث بـ .pushed.json (بصمات sha256؛ الملفات غير المتغيرة تُتجاهل). يجب استثناء .pushed.json نفسه من الدفع وإلا يُبطل بصماته
- الثغرة القاتلة التي طلب إصلاحها المستخدم: 'آخر دقائق قبل إغلاق المحادثة'. الحل 4 طبقات: (1) خطاف UserPromptSubmit يدفع فوراً عند كل رسالة قبل أن يبدأ الوكيل بالرد (…
- طلب المستخدم: اريد اخر دقائق قبل اغلاق محفوظه ايضا يعني ما اريد اخر دقائق يعني مثلا لو كنت انا معه في موضوع مهم وتم اغلاق المحادثه كيف سيعرف النظام الجديد بالمحادثه…
- طلب المستخدم: سؤال هل التقدم سيكون محفوظ الى الابد ام سيكون كل محرز جديده جديده يعاد من البدايه اجبني بسرعة نصيا
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
<!-- /MEM:FACTS -->