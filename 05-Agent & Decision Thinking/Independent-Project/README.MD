# 🍽️ بوت حجز المطاعم على واتساب

بوت على واتساب بيستقبل طلب الزبون، بيحجز طاولة لشخصين بالمطعم، بيبعت دعوة لصديق الزبون، وبيضيف الموعد على Google Calendar مع تذكير.

مبني على **n8n** + **Green-API** + **OpenAI** + **Google Sheets** + **Google Calendar**.

---

## 📋 شو بيعمل البوت

1. الزبون بيراسل البوت على واتساب: "بدي احجز".
2. البوت بيسأله عن اللي ناقص، سؤال بكل رسالة:
   - اسم المطعم
   - رقم واتساب الصديق
   - تاريخ ووقت الموعد
   - الإيميل
3. بيتأكد إن المطعم موجود بقاعدة البيانات (Google Sheets).
4. بيبعت ملخص للزبون وبيستنى التأكيد.
5. بعد التأكيد:
   - 📩 بيبعت للمطعم رسالة حجز طاولة لشخصين
   - 💌 بيبعت للصديق دعوة فيها اسم المطعم والوقت
   - 📅 بينشئ حدث على Google Calendar بإيميل الزبون (مدته ساعتين، مع تذكير)
   - ✅ بيبعت للزبون تأكيد إن كل شي تم

---

## 🧩 مكونات الـ Workflow

```
Webhook → If → Booking Agent → HTTP Request (الرد على الزبون)
                    │
                    ├── OpenAI Chat Model   (العقل)
                    ├── Conversation Memory (ذاكرة المحادثة لكل زبون)
                    ├── restaurants_db      (Google Sheets: أسماء وأرقام المطاعم)
                    ├── send_whatsapp       (Green-API: إرسال للمطعم وللصديق)
                    └── create_calendar_event (Google Calendar)
```

| الـ Node | وظيفته |
|---|---|
| **Webhook** | بيستقبل الإشعارات من Green-API |
| **If** | بيمرّر بس الرسايل الواردة (`incomingMessageReceived`) وبيتجاهل باقي الإشعارات |
| **Booking Agent** | الـ AI اللي بيدير المحادثة وبيستخدم الأدوات |
| **OpenAI Chat Model** | الموديل (`gpt-5-mini`) |
| **Conversation Memory** | بيتذكر آخر 10 رسايل لكل زبون (المفتاح: رقم الزبون) |
| **restaurants_db** | بيقرأ قائمة المطاعم من Google Sheets |
| **send_whatsapp** | بيبعت رسالة واتساب لأي رقم (المطعم أو الصديق) |
| **create_calendar_event** | بينشئ حدث بالكلندر ويبعت دعوة للإيميل |
| **HTTP Request** | بيبعت رد الـ Agent للزبون |

---

## ⚙️ خطوات التشغيل

### 1. Green-API
1. اعمل حساب على [green-api.com](https://green-api.com) وأنشئ **instance** واتساب.
2. اربط رقم واتساب بمسح الـ QR (واتساب ← الأجهزة المرتبطة ← ربط جهاز).
3. انسخ: `apiUrl` و `idInstance` و `apiTokenInstance`.
4. من **Webhooks ← Change**:
   - **Webhook Url**: رابط الـ Production تبع n8n (فيه `/webhook/` مش `/webhook-test/`)
   - **Receive webhooks on incoming messages and files**: ✅ Yes
   - احفظ.

### 2. Google Sheets (قاعدة بيانات المطاعم)
اعمل شيت فيه عمودين:

| name | whatsapp_phone |
|---|---|
| مطعم تيلندي | 9705XXXXXXXX |
| مطعم ارجوان | 9705XXXXXXXX |
| مطعم سنونو | 9705XXXXXXXX |

الأرقام بصيغة دولية، أرقام فقط بدون `+` وبدون صفر بالأول.

### 3. n8n
1. اعمل Import للـ workflow.
2. اربط الـ Credentials:
   - **OpenAI**
   - **Google Sheets** ← واختار الشيت والتاب بـ `restaurants_db`
   - **Google Calendar** ← وبـ **Calendar** اختار تقويمك **From list**
3. بـ **HTTP Request** و **send_whatsapp** حط الرابط:
   ```
   {apiUrl}/waInstance{idInstance}/sendMessage/{apiTokenInstance}
   ```
4. من **Settings** تبع الـ workflow، خلي **Timezone = Asia/Hebron**.
5. احفظ وفعّل الـ workflow (**Active / Publish**).

---

## 🔑 الإعدادات المهمة

**Prompt بالـ Booking Agent** (بيقرأ نص الرسالة بكل أنواعها):
```
{{ $json.body.messageData.textMessageData?.textMessage || $json.body.messageData.extendedTextMessageData?.text || 'رسالة غير نصية' }}
```

**شرط الـ If:**
```
{{ $json.body.typeWebhook }}  is equal to  incomingMessageReceived
```

**chatId بـ send_whatsapp:**
```
{{ $fromAI('recipient_phone', 'رقم المستلم بصيغة دولية أرقام فقط بدون + ...', 'string') }}@c.us
```

**Session Key بالـ Memory:**
```
{{ $('Webhook').item.json.body.senderData.chatId.replace('@c.us', '') }}
```

---

## 🧪 التجربة

ابعت للبوت من واتساب:
> بدي احجز بمطعم سنونو بكرا الساعة 3، وبدي اعزم صديقي 9705XXXXXXXX، وإيميلي example@gmail.com

لما يبعتلك الملخص رد **"أكّد"**، وتابع النتيجة من تبويب **Executions** بـ n8n.

---

## ⚠️ ملاحظات مهمة

- **حد الخطة المجانية بـ Green-API (Developer):** بتقدر تراسل **3 أرقام بس بالشهر**، وكل رقم بينحسب حتى لو كان غلط (رقم ناقص أو جروب). للاستخدام الحقيقي لازم خطة **Business**.
- **لا تربط رقمك الشخصي:** البوت بيرد على أي حدا بيراسل الرقم المربوط. استخدم رقم مخصص للبوت، أو ضيف شرط بالـ If يحدد الأرقام المسموحة وقت التجربة.
- **الـ Pin 📌:** إذا الـ Webhook عليه Pin، التجارب داخل المحرّر بتستخدم نفس الرسالة القديمة. شيله من زر **Unpin**، ولا تستخدم **Debug in editor** إلا إذا بدك تثبّت البيانات.
- **لا تجرّب الأدوات لحالها** بزر Execute step (زي الكلندر أو send_whatsapp)، لأنها بتاخد قيمها من الـ Agent. جرّب دايماً من واتساب.
- **التوكن سري:** `apiTokenInstance` زي كلمة السر. لا تنشره ولا تحطه بملفات مشاركة. إذا انكشف، اعمل توكن جديد من Green-API وحدّثه بالـ nodes.

---

## 🛠️ حل المشاكل

| المشكلة | السبب والحل |
|---|---|
| `401 Authorization failed` | التوكن أو رقم الـ instance غلط بالرابط. انسخهم من جديد من Green-API |
| `466 quota exceeded` | وصلت لحد الـ 3 أرقام. راسل من رقم مسموح، أو رقّي الخطة |
| `No prompt specified` | الـ Prompt مش لاقي نص الرسالة. استخدم التعبير اللي بيقرأ النوعين (فوق) |
| `Calendar parameter's value is invalid` | بـ create_calendar_event غيّر Calendar لـ **From list** واختار تقويمك |
| `Referenced node doesn't exist` | في تعبير بيشير لـ node محذوف (مثلاً `WhatsApp Trigger`). استبدله بـ `Webhook` |
| الرسالة بتروح على **false** بالـ If | في مسافة مخفية بالشرط. امسحه واكتبه بإيدك |
| البوت ما بيرد على رسايل جديدة | تأكد إن الـ workflow **Active**، وإن الرابط بـ Green-API هو الـ Production |
