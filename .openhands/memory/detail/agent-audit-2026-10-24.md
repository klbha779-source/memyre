# تدقيق خطة "الوكيل متعدد العقول" (2026-10-24) — أدلة مُتحقَّق منها

## مُختلَق (غير موجود إطلاقاً)
- **OptiRouter** → PyPI 404. لا وجود له.
- **VeloceGrid** → PyPI 404. لا وجود له.
- **ProjectOS 2026** (كنشاط قياس) → لا وجود له. بحث فقط يُرجع SPEC CPU 2026.
- **"57,600 طلب/يوم" من NVIDIA** → حساب مدوّنات. NVIDIA تنشر "40 RPM" فقط، ولا حد يومي معلن.
- **"ممر مجاني لـ GLM-5.2 على OpenRouter"** → غير موجود (pay-as-you-go فقط).

## أرقام خاطئة في الخطة
- **Groq "14,400 RPD Llama 3.1 8B / 1000 RPD Qwen3-32B"** → لا يوجد هذان الصفّان أصلاً.
  الحالي: gpt-oss-120b/20b و qwen3.8-27b = 30 RPM / 1,000 RPD / 8K TPM.
- **Cerebras "30 RPM مجاناً"** → خطأ، الفعلي **5 RPM**، ويلزم **بطاقة دفع**، ولا طبقة مجانية دائمة.
- **"Gemini Pro مجاني"** → خطأ، أُزيل من الطبقة المجانية 2026-04-01.
- **"MoA حقّق 64.7% متجاوزاً Claude Fable 5 (65.3%)"** → 64.7 < 65.3، أي أنه **لم يتجاوزه**. تناقض داخلي في الخطة.

## صحيح (مُتحقَّق)
- الأدوات: litellm (v1.106.2), langgraph (v1.2.16), mem0ai (v2.2.1), letta (v0.34.12),
  open-webui (v0.12.3) — كلها حقيقية وناضجة.
- **freelm** v0.5.2 → حقيقي لكن **10 نجوم فقط**، أُنشئ 2026-06-07.
- **zerogateway** v0.1.1 → حقيقي لكن **0 نجوم**، آخر push 2026-09-30 (مهجور عملياً).
- النماذج على HF: OpenResearcher-30B-A3B (dl=731), FINAL-Bench/Darwin-27B-ZTC-v2 (dl=134),
  XYZAILab/XYZ-Aquila-pro (dl=335), zai-org/GLM-5.2 (dl=333k), moonshotai/Kimi-K2.6 (dl=346k),
  MiniMaxAI/MiniMax-M2.5 (dl=893k), Qwen/Qwen3.8-27B (dl=7.7M).
- أنشطة القياس: BrowseComp-Plus, DeepSearchQA, GAIA, DRACO (Perplexity), AA-Briefcase,
  Intelligence Index, SWE-bench Pro, Terminal-Bench — كلها حقيقية.
  (S1MB = لوحة هواة، ليست معياراً معترفاً به.)

## 🔴 الاكتشاف الحاسم (اختبار ذاتي على البوابة المجانية)
**النماذج القوية المذكورة في الخطة ليست مجانية:**
`deepseek-ai/DeepSeek-V4.1-Flash`, `zai-org/GLM-5.2`, `moonshotai/Kimi-K2.6`,
`MiniMaxAI/MiniMax-M2.5`, `Qwen/Qwen3.8-27B`, `XYZAILab/XYZ-Aquila-pro`,
`FINAL-Bench/Darwin-27B-ZTC-v2`, `OpenResearcher/OpenResearcher-30B-A3B`
→ **كلها HTTP 401** (مدفوعة).

**المجاني فعلاً: 14/20 نموذجاً فقط:**
kilo-auto/free · kilo-auto/small · openrouter/free · reflection/beam-free ·
recursive/flint-v1.0:free · poolside/laguna-s-2.1:free · cohere/north-mini-code:free ·
dots-studio/dots-3-note-preview:free · nvidia/nemotron-3-ultra-550b-a55b:free ·
nvidia/nemotron-3.5-super-vl:free · nvidia/nemotron-3.5-lightning:free ·
nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free · nvidia/nemotron-3.5-content-safety:free ·
xingchen-agi/xing4.0-29b-a4b:free
(فاشلة: kilo-auto/{efficient,frontier,balanced} 401 · nvidia/nemotron-3-super-120b-a12b 404 ·
liquid/lfm-2.5-2.6b 429 · poolside/laguna-xs-2.1 429)
الاستجابة تحتوي `usage.market_cost: 0` = مجانية فعلاً.

## 🔴 المجلس (MoA) أسوأ من نموذج واحد — بالقياس
مهمة تشخيص حقيقية (سبب الصمت + device-side assert)، تقييم بكلمات دالة:
```
نموذج واحد:
  nvidia/nemotron-3-ultra-550b:free    25.0s  نقاط=10  ✅ الأفضل
  poolside/laguna-s-2.1:free            3.5s  نقاط=7
  reflection/beam-free                 11.9s  نقاط=0
مجلس (3 مقترحات + مُجمِّع):  84.1s → المُجمِّع نقاط=0 (إجابة فارغة!)
```
**سببان مكتشفان:**
1. **نماذج الاستدلال (reasoning) تحرق كل ميزانية التوكنات في التفكير.**
   `reflection/beam-free` استهلك `reasoning_tokens=2965` من 3000 → `finish_reason: length`
   و`content` **فارغ**. الحل: `max_tokens` ≥ 4000، وقراءة حقل `reasoning` احتياطاً.
   → أي مُجمِّع بلا ميزانية سخية يُرجع فراغاً بصمت. هذا خطر خفي في معمارية MoA.
2. **المُجمِّع يُفقد التفاصيل:** nemotron أعطى نقاط=10 وحده، والمُجمِّع لخّصه إلى 3.
   **الدمج يمحو الدقة** بدل أن يرفعها.

**الكلفة:** المجلس 84s مقابل 3.5–25s لنموذج واحد = **3–24× أبطأ**.
على طبقة مجانية بحدود 50–1000 طلب/يوم، هذا يُستنزف سريعاً.

## ✅ OpenHands هو الموجِّه أصلاً
`TIKTOKEN_CACHE_DIR` يشير إلى `/litellm/` داخلياً → **OpenHands مبني على LiteLLM**.
فلا حاجة لطبقة LangGraph خارجية للتوجيه — يمكن ضبط النموذج/المزوّد من داخله مباشرة.
