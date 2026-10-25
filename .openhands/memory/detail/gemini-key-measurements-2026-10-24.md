# قياسات مفتاح Gemini + مفتاح NVIDIA (2026-10-24) — أدلة مباشرة

## ما أُعطي فعلاً
- **مفتاح Gemini صالح** (53 حرفاً، يبدأ AQ.) — تحقّق: HTTP 200.
- **مفتاح NVIDIA لم تُعطَ قيمته** — أُعطيت بيانات وصفية فقط (ID/Name/Expiration)
  بينما القيمة الملصقة هي مفتاح Gemini نفسه. لذا لا يمكن اختبار NVIDIA.
  (بهذا المفتاح: سرد 85 نموذجاً ينجح، لكن الاستنتاج 401/410/404 — لا يصلح.)

## مفتاح Gemini — الوصول المجاني المُقاس
- 50 نموذجاً مسروداً، 46 يدعم generateContent.
- نقطة وصول OpenAI-compatible تعمل:
  https://generativelanguage.googleapis.com/v1beta/openai/chat/completions
  مع Authorization: Bearer <key> — مفيدة لأنها تعني تكاملاً سهلاً مع LiteLLM/OpenHands.
- استدعاء الأدوات يعمل: أُرجِع finish_reason: tool_calls و
  get_weather({"city":"Baghdad"}).
- البث: لم أختبره (الحصة اليومية نفدت).

## 🔴 الحدود الحقيقية — من رسالة الخطأ نفسها (ليست من مدوّنة)
- quotaId: GenerateRequestsPerDayPerProjectPerModel-FreeTier
  → الحد يومي، لكل نموذج على حدة.
- الحد = 20 طلباً/يوم لكل نموذج (مقيس مرتين: gemini-2.5-flash و gemini-3.5-flash).
- حد الدقيقة = 5 لـ gemini-2.5-flash (من نص الخطأ: limit: 5 ... per minute).
- gemini-3.1-pro-preview: الحد = 0 → Pro غير متاح مجاناً.
- gemini-2.5-pro: 404 — "no longer available to new users".

### قياس فعلي (طلب كل 13 ثانية لتجنّب حد الدقيقة)
```
gemini-3.5-flash: نجح 11 ثم 429 | limit=20 | per=day
gemini-2.5-flash: نجح 0  ثم 429 | limit=20 | per=day  (كانت حصته مستنفدة)
gemini-2.5-flash (20 طلباً في 4.5s): 10×200 ثم 10×429 (حد الدقيقة = 5)
```

## النماذج القابلة للاستخدام فعلاً (5 من 14 مرشحاً)
```
✅ gemini-3.5-flash-lite · gemini-3.1-flash-lite · gemini-flash-lite-latest
✅ gemma-4-26b-a4b-it · gemini-3-flash-preview
❌ gemini-3.5-flash, gemini-3.6-flash, deep-research-preview → نفدت (day)
❌ gemini-3.1-pro-preview → 0 حصة (مدفوع)
❌ gemini-2.5-flash-lite → 404 · gemini-flash-latest → 503 · gemini-3.8-flash → 503
```

## 🎯 حساب الطاقة الحقيقية
```
Gemini:  5 نماذج × 20 طلب/يوم = 100 طلب/يوم
Kilo:    200 طلب/ساعة/IP
────────────────────────────────────────
المجموع: ~100–300 طلب/يوم فعلياً
الهدف:   1000–2000 طلب/يوم
النتيجة: ❌ أقل من الهدف بـ 5–10 أضعاف
```

## ✅ اختبار حاسم: هل يكتشف المراجع الادعاء الكاذب؟
أعطيتُ النموذج تقريري الكاذب عن سبب device-side assert (ادّعاء ±32768→inf→NaN).
gemini-3.5-flash-lite اكتشف الخطأ الرياضي بدقة:
> "The evidence shows 2**15 (which equals 32768), but standard fp16 can represent
> values up to 65504. Therefore, the claim that 32768 overflows fp16 to infinity
> is mathematically incorrect."

وأضاف:
> "Applying a monkey-patch to make a method a no-op bypasses the code entirely
> rather than addressing the underlying bug. This is a workaround, not a solution."
> "There is no evidence of testing, validation metrics, or verification logs
> provided... to substantiate that the TTS is fully functional."

هذا دليل عملي على قيمة المراجع المستقل: نموذج مجاني واحد، 7.5 ثانية، اكتشف ثلاث
مشاكل في ادّعائي — أحدها تصحيح رياضي أدقّ من اختباري أنا.
