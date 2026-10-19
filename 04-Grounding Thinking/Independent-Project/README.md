# مساعد 92 التقنية — Tech92 Knowledge-Grounded Assistant

**U4 Project — Ground Its Knowledge** · نظام ذكاء اصطناعي قائم على المعرفة مبني بـ Dify (Chatflow) لشركة 92 التقنية للحلول السحابية (Technology 92 — Cloud Solutions).

---

## Project Name
**مساعد 92 التقنية (Tech92 Assistant)** — مساعد يجيب العملاء والموظفين باللهجة السعودية استناداً إلى وثائق الشركة الرسمية فقط.

## Problem Statement
فريق المبيعات والدعم في 92 التقنية يتلقى يومياً عبر واتساب الأسئلة نفسها: كم سعر الباقة؟ ما الفرق بين Core وScale؟ كم مدة العقد؟ كيف أسجّل الحضور في تطبيق الموظف؟ كيف أحذف حسابي؟ الإجابات موجودة لكنها مبعثرة بين الموقع، عروض PDF، وسياسات قانونية تتغير (آخر تحديث 2 يوليو 2026)، فتختلف الردود من موظف لآخر وقد تُذكر أسعار قديمة أو مخمَّنة.

## Target Users
1. **العملاء المحتملون وفريق المبيعات** (السعودية أساساً) — الخدمات، الباقات، الأسعار، العقود، بدء التعاقد.
2. **الموظفون الجدد والباحثون عن عمل** — استخدام تطبيق Tech 92 (الحضور، KPIs، الطلبات)، شروط التسجيل، الخصوصية وحذف الحساب.

## Proposed Solution
تطبيق **Chatflow** في Dify: `User Input → Knowledge Retrieval → LLM (gpt-5) → Answer`، مرتبط بقاعدة معرفة من 8 وثائق رسمية، وبتعليمات نظام (بالإنجليزية) تُلزم النموذج بالإجابة من المقاطع المسترجعة فقط، وذكر المصدر، والتصريح بعدم توفر المعلومة بدل اختلاقها، والرد باللهجة السعودية. اختُير Chatflow بدل Chatbot البسيط لأن كل مرحلة عقدة مستقلة يمكن فحص مخرجاتها (LAST RUN) — وهو ما سمح بالتمييز بين أعطال الاسترجاع وأعطال التوليد.

## Knowledge Sources
| الملف | المصدر | التاريخ |
|---|---|---|
| `knowledge/01_company_profile.md` | technology92.com (من نحن، التواصل، الشركاء، العملاء) | 2026 |
| `knowledge/02_services.md` | technology92.com › خدماتنا | 2026 |
| `knowledge/03_packages_pricing.md` | technology92.com/packages | 2026 |
| `knowledge/04_faq.md` | technology92.com › الأسئلة الشائعة | 2026 |
| `knowledge/05_employee_app_guide.md` | عرضا PDF لتطبيق الموظف (أُعيدت كتابتهما يدوياً بسبب OCR رديء) | يوليو 2026 |
| `knowledge/06_privacy_policy.md` | technology92.com/privacy-policy | 2 يوليو 2026 |
| `knowledge/07_terms_conditions.md` | technology92.com/terms-and-conditions | 2 يوليو 2026 |
| `knowledge/08_ai_sales_agent.md` | technology92.com/services/ai-sales-agent | 2026 |

استُبعدت عمداً: الأكواد، ملفات `.env`، مفاتيح App Store، ومواصفات التطوير الداخلية. التفاصيل وتقييم الجودة: [`docs/01_knowledge_sources_summary.md`](docs/01_knowledge_sources_summary.md)

## System Overview
```
المستخدم ──سؤال──▶ Dify Chatflow
                     │
   [User Input] ──▶ [Knowledge Retrieval]  Tech92-KB
                     │   • 8 ملفات Markdown
                     │   • Chunking: General · Delimiter \n\n · Max 1,024 chars · Overlap 50
                     │   • Index: Economical (Inverted index — كلمات مفتاحية، قيد الحساب)
                     │   • Top-K 3 → 5
                     ▼
                  [LLM: gpt-5 · Temperature 0.1]
                     │   Context = Knowledge Retrieval/result
                     │   System prompt: answer only from <context>, refuse explicitly,
                     │   cite source, reply in Saudi dialect
                     ▼
                  [Answer] + Citations and Attributions
```

## Important Design Decisions
1. **إعادة كتابة الـ PDF بدل رفعه** — الاستخراج النصي للعروض (لقطات شاشة) أعطى OCR مشوّهاً (`של`، "بكبرة"، تواريخ 2006). دليل: `knowledge/raw_ocr_sample_DO_NOT_UPLOAD.md`.
2. **Chatflow بدل Chatbot** — لفصل مرحلة الاسترجاع عن التوليد وفحص كل منهما على حدة.
3. **فهرسة Economical بوعي بقيودها** — حساب Dify Sandbox محدود الـ credits؛ البحث حرفي بالكلمات لا بالمعنى. القيد موثّق، أثره مُقاس، وعُولج بإثراء الوثائق بمرادفات بصيغة السائل.
4. **التقسيم بفاصل `\n\n` وحد 1,024 حرفاً** — بعد أن كشف الاختبار أن الإعداد الافتراضي قطّع الملف سطراً سطراً وفصل السعر عن اسم الباقة (انظر Improvements).
5. **Temperature 0.1 + قواعد رفض صريحة + "لا تحسب ما ليس في النص"** — يحوّل عدم المعرفة إلى سلوك مقصود قابل للاختبار.
6. **اللهجة السعودية** — المكتب الرئيسي والعملاء في الرياض؛ قرار نبرة لا يغيّر الحقائق المسموح بها.
7. **تفعيل Citations** — كل إجابة تحمل اسم الملف الذي اعتمدت عليه.
الإعدادات الكاملة والـ prompt: [`docs/02_dify_setup_guide.md`](docs/02_dify_setup_guide.md)

## Testing & Evaluation
- **فحص الاسترجاع** من داخل التطبيق (LAST RUN لعقدة Knowledge Retrieval، مع حفظ JSON المقاطع): [`docs/03_retrieval_tests.md`](docs/03_retrieval_tests.md)
- **الاستناد والهلوسة** (11 حالة): [`docs/04_grounding_hallucination_tests.md`](docs/04_grounding_hallucination_tests.md)
- **مجموعة الاختبار الرئيسية** (12 سؤالاً: 3 واضحة، 4 موزعة/غير مباشرة، 2 غير موجودة، 3 فخاخ) قبل/بعد: [`docs/05_test_cases_and_results.md`](docs/05_test_cases_and_results.md)

نتائج ملاحظة حتى الآن (جولة 1 → 2):
| السؤال | قبل | بعد |
|---|---|---|
| كم سعر باقة Scale شهرياً؟ | ❌ رفض خاطئ — المقاطع المسترجعة 4 أسطر منفردة بلا سعر | ✅ 3,200 ريال + استشهاد |
| كم سعر AI Sales Agent 92؟ | ✅ رفض مستند ("لا يوجد سعر معلن") + إحالة لواتساب | — |
| كم عدد الموظفين في الشركة؟ | ✅ رفض صريح، لم يحوّل "+75 شركة" إلى موظفين | — |
| هل يراقب التطبيق مكاني طول اليوم؟ | ❌ رفض خاطئ (الإجابة موجودة في سياسة الخصوصية) | ⬜ بعد إثراء المرادفات |
| عمري 17، أقدر أسجل؟ | ❌ رفض خاطئ (الشروط تقول 18+) | ⬜ |
| الأسئلة المتبقية (7) | ⬜ | ⬜ |

## Improvements Made
1. **جودة المصدر (معالجة المستندات):** استبدال OCR الـ PDF بنص مكتوب يدوياً.
2. **التقسيم (معالجة المستندات) — أهم اكتشاف:** الإعداد الافتراضي جعل كل سطر مقطعاً (24–64 حرفاً)، فصار سطر "3,200 ريال" مقطعاً مستقلاً بلا كلمة "Scale"، والبحث الحرفي لم يربطه بالسؤال؛ النموذج رفض بصدق. التغيير: Delimiter `\n\n` + Max 1,024 حرفاً → مقطع Scale أصبح 699 حرفاً يضم العنوان والسعر والمزايا، وجدول الفروق في مقطع واحد. النتيجة: نفس السؤال أُجيب صحيحاً مع الاستشهاد.
3. **التنسيق (التوليد):** الرد الأول جاء جملة واحدة ملتصقة بالمصدر؛ أُضيفت قواعد FORMATTING للـ prompt (إجابة مباشرة في سطر، نقاط، المصدر في سطر منفصل).
4. **إثراء الوثائق بمرادفات (معالجة المستندات، بسبب قيد Economical):** أسئلة "هل يراقب التطبيق مكاني؟" و"عمري 17" فشلت لأن كلماتها غير موجودة حرفياً في النص. أُضيفت داخل الأقسام المعنية أسطر "سؤال شائع" بصيغة السائل و"كلمات مرتبطة" (مراقبة، تتبع، GPS، قاصر، تحت 18، Commercial Registration…). ⬜ النتيجة بعد إعادة الرفع.
التفاصيل: [`docs/06_improvement_summary.md`](docs/06_improvement_summary.md)

## Limitations
- **قيد الحساب (Sandbox):** فهرسة Economical بلا embeddings → لا بحث دلالي، لا Hybrid، لا Rerank. الأسئلة بكلمات مختلفة عن نص الوثيقة قد لا تُسترجع؛ الحل الحالي (المرادفات) يدوي ولا يغطي كل الصياغات. أول تحسين مستقبلي: التحويل إلى High Quality embedding.
- قاعدة المعرفة لقطة بتاريخ 18 أكتوبر 2026؛ الأسعار قابلة للتغيير ولا تُحدَّث تلقائياً.
- لا تغطي السياسات الداخلية غير المنشورة (الرواتب، الإجازات السنوية، هيكل الشركة).
- محتوى التطبيق مكتوب يدوياً من العروض، فيحتاج تحديثاً عند صدور نسخ جديدة.
- نافذة Preview في Dify تعرض النص العربي المختلط بكلمات لاتينية باتجاه LTR فتبدو بعض الأسطر مقلوبة؛ التطبيق المنشور أفضل.

## How to Try the Application
- رابط التطبيق: ⬜ (Dify → Publish → Run App)
- أسئلة مقترحة: "كم سعر باقة Scale شهرياً؟" · "ما الفرق بين Core وScale؟" · "هل يراقب التطبيق مكاني؟" · "كم سعر AI Sales Agent 92؟" (يجب أن يرفض ويحيل لواتساب).
- لإعادة البناء: ارفع ملفات `knowledge/` (عدا `raw_ocr_sample`) إلى قاعدة معرفة في Dify بإعدادات التقسيم أعلاه، ثم ابنِ الـ Chatflow كما في `docs/02_dify_setup_guide.md`.

## Repository Structure
```
├── README.md
├── knowledge/            # الملفات التي تُرفع إلى Dify (+ عيّنة OCR خام للتوثيق فقط)
├── docs/
│   ├── 01_knowledge_sources_summary.md
│   ├── 02_dify_setup_guide.md
│   ├── 03_retrieval_tests.md
│   ├── 04_grounding_hallucination_tests.md
│   ├── 05_test_cases_and_results.md
│   ├── 06_improvement_summary.md
│   └── 07_presentation.md
└── evidence/             # صور الشاشة وملفات JSON للاسترجاع (انظر evidence/README.md)
```


https://udify.app/chat/F63sGM0V8HxcN8hv
