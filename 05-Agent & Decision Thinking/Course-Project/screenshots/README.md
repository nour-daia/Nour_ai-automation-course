
# XYZ Real Estate AI Agent (n8n)

وكيل ذكي (AI Agent) لشركة XYZ Real Estate مبني على n8n. يستقبل رسائل العملاء عبر Telegram، ويجمع بيانات طلبهم العقاري تدريجيًا، ويعرض ملخصًا للتأكيد، ثم يحفظ البيانات في Google Sheets.

> ملاحظة: لا يحتوي هذا المستودع على أي Token أو API Key. جميع بيانات الاعتماد محفوظة داخل n8n Credentials فقط.

---

## 1. المكونات

| المكون | الدور |
|---|---|
| Telegram Bot (`XYZ Real Estate Assistant`) | واجهة العميل، أُنشئ عبر BotFather |
| Telegram Trigger | استقبال رسائل العميل |
| AI Agent | فهم الرسائل وجمع الحقول وتحديد الخطوة التالية |
| OpenRouter Chat Model | نموذج اللغة |
| Simple Memory | سياق المحادثة، بمفتاح جلسة = Chat ID |
| Google Sheets Tool (Append Row) | إضافة صف العميل إلى الجدول |
| Telegram Send Message | إرسال رد الـ Agent للعميل |

مسار الـ Workflow الرئيسي:

```text
Telegram Trigger -> AI Agent -> Telegram Send Message
```

وترتبط بالـ Agent: OpenRouter Chat Model (Chat Model)، وSimple Memory (Memory)، وGoogle Sheets Tool (Tool).

## 2. Google Sheets

- الملف: `XYZ Real Estate Leads`
- الورقة: `Leads`
- الأعمدة التسعة بالترتيب: Property type, Buy/Rent, New/Resale, First Ourcase, Timeline, Budget, Name, Email, Phone Number

## 3. System Prompt

كُتب في حقل System Message في عقدة AI Agent، ويشمل: رسالة الترحيب، الحقول التسعة وقيمها المسموحة، فئات الميزانية وحدودها، القواعد الشرطية (Rent = `Not Applicable` للحقلين New/Resale وFirst Ourcase)، التحقق من البريد، خطوة التأكيد قبل الحفظ، وعدم ادعاء نجاح الحفظ دون نتيجة من الأداة.

## 4. ما تم إنجازه

- إنشاء البوت في Telegram وربطه بـ n8n (استقبال الرسائل يعمل).
- ربط OpenRouter Chat Model بالـ Agent.
- إضافة Simple Memory وربطها بالـ Agent.
- إنشاء Google Sheet بالأعمدة التسعة وربط Google Sheets Credentials.
- تحويل عقدة Google Sheets من `Get row(s)` إلى `Append Row` وربطها بالـ Agent كأداة.

## 5. المشاكل التي ظهرت وكيف عولجت

| المشكلة | السبب | الحل |
|---|---|---|
| خطأ عند اختيار Username للبوت | الـ Username لا يقبل مسافات، وظهر خطأ مؤقت من BotFather | استخدام Username بدون مسافات وينتهي بـ `bot` |
| AI Agent باللون الأحمر | فشل عقدة OpenRouter (الذاكرة كانت سليمة) | مراجعة الـ Credentials والرصيد واسم النموذج من سجل الخطأ |
| رسالة Unpin workflow data | بيانات اختبار مثبّتة على الـ Trigger | Unpin حتى يستقبل رسائل حقيقية |
| الـ Agent لا يرحّب برسالة الترحيب | ذاكرة قديمة من اختبارات سابقة بنفس Session ID | استخدام جلسة اختبار جديدة (لاحقة مؤقتة) |
| لا يُسجَّل شيء في الجدول | العقدة كانت `Get row(s)`، ثم بقي `columns.value` فارغًا بعد التحويل إلى Append | تحويلها إلى Append وربط الأعمدة التسعة بـ `$fromAI` |

## 6. ما تبقى

- [ ] تطبيق ربط الأعمدة التسعة بـ `$fromAI` واختبار الحفظ الفعلي.
- [ ] تعديل Session Key في Simple Memory إلى `{{ $('XYZ Real Estate Assistant').item.json.message.chat.id }}`.
- [ ] إلغاء تثبيت بيانات الـ Trigger (Unpin).
- [ ] تغيير اسم أداة الحفظ إلى اسم واضح مثل `Save_Confirmed_Lead` وإضافة وصف لها.
- [ ] **ضابط تقني يمنع الحفظ قبل تأكيد العميل** (الخطوة 10 في المشروع)؛ لا يكفي الاعتماد على System Message وحده.
- [ ] آلية منع التكرار (Duplicate Prevention).
- [ ] التحقق من نتيجة الحفظ قبل إبلاغ العميل بالنجاح.
- [ ] تنفيذ الاختبارات الـ 12 وتسجيل نتائجها.
- [ ] تسجيل عرض عملي قصير (Demo).

## 7. نتائج الاختبارات

| # | السيناريو | النتيجة |
|---|---|---|
| 1 | محادثة جديدة ورسالة الترحيب | NOT TESTED (ظهر سلوك خاطئ بسبب ذاكرة قديمة، يُعاد الاختبار بجلسة نظيفة) |
| 2–7 | جمع المعلومات، الغموض، الإيجار، السياق، التصحيح | NOT TESTED |
| 8 | عدم الحفظ دون تأكيد | NOT TESTED |
| 9 | الحفظ بعد التأكيد | NOT TESTED |
| 10 | فشل Google Sheets | NOT TESTED |
| 11 | منع تكرار الصف | NOT TESTED |
| 12 | عميلان مختلفان | NOT TESTED |

> لن تُسجَّل أي نتيجة `PASS` إلا بعد فحص سجل التنفيذ في n8n ومحتوى Google Sheets فعليًا.

## 8. الأمان

- Tokens ومفاتيح API في n8n Credentials فقط.
- لا تُرفع ملفات تحتوي بيانات اعتماد أو لقطات شاشة تُظهرها إلى GitHub.
- رسائل العملاء تُعامل كمدخلات غير موثوقة، والتحقق قبل الحفظ يجب أن يُفرض في الـ Workflow وليس في الـ Prompt فقط.
