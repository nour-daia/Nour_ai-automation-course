
# PlanCraft AI

### تحويل أفكار المشاريع إلى خطط تنفيذية باستخدام الذكاء الاصطناعي

**PlanCraft AI** تطبيق ويب يستقبل فكرة مشروع من المستخدم، ويحوّلها بواسطة AI Agent إلى خطة تنفيذية منظمة، ثم يعرضها داخل التطبيق.

---

## الأدوات المستخدمة

| الأداة | الدور |
|---|---|
| **Lovable** | بناء واجهة التطبيق ووظائفه |
| **Supabase** | قاعدة البيانات (الطلبات والخطط وحالة المعالجة) |
| **n8n Cloud** | Workflow يعالج الطلب ويولّد الخطة بالذكاء الاصطناعي |
| **OpenAI** | النموذج اللغوي المستخدم داخل AI Agent |

---

## مخرجات الخطة

- `Project Summary`: ملخص المشروع
- `Project Goal`: الهدف الرئيسي
- `Implementation Phases`: مراحل التنفيذ (3–6 مراحل بأولوية لكل مرحلة)
- `Recommended Actions`: الإجراءات الموصى بها
- `Risks` و`Mitigation`: المخاطر وطرق الحد منها
- `Estimated Duration`: المدة التقديرية

---

## مسار العمل (Execution Flow)

```text
المستخدم
   ↓
تطبيق Lovable  →  حفظ الطلب في Supabase (status = pending)
   ↓
اتصال آمن من جهة الخادم
   ↓
n8n Production Webhook
   ↓
Edit Fields  (استخراج request_id)
   ├──► Respond to Webhook  (استجابة فورية)
   └──► Supabase: جلب الطلب
            ↓
        AI Agent  (OpenAI Chat Model + Structured Output Parser)
            ↓
        Supabase: حفظ الخطة في project_plans
            ↓
        Supabase: تحديث الطلب (status = completed)
   ↓
التطبيق يعرض الخطة المحفوظة
```

> يستخدم الـ Workflow **الاستجابة الفورية**: عقدة `Respond to Webhook` متصلة بالتوازي بعد `Edit Fields`، لذلك نجاح استجابة الـ Webhook لا يعني اكتمال الخطة. يتابع التطبيق حالة الطلب في Supabase حتى تصبح `completed` أو `failed`.

---

## قاعدة البيانات (Supabase)

### `project_requests`
`id` · `title` · `description` · `goal` · `additional_context` · `status` · `created_at` · `updated_at`

الحالات: `pending` ← `processing` ← `completed` / `failed`

### `project_plans`
`id` · `request_id` (UNIQUE) · `summary` · `project_goal` · `phases` (JSONB) · `recommended_actions` (JSONB) · `risks` (JSONB) · `estimated_duration` · `created_at` · `updated_at`

---

## مراحل التنفيذ

1. **بناء التطبيق على Lovable**: نموذج الإدخال، صفحة الخطط، صفحة تفاصيل الخطة.
2. **ربط Lovable مع Supabase** وإنشاء الجدولين.
3. **بناء Workflow في n8n**:
   - `Webhook` (POST، المسار `plancraft-generate`)
   - `Edit Fields` لاستخراج `request_id`
   - `Supabase` لجلب الطلب
   - `AI Agent` مع `OpenAI Chat Model` و`Structured Output Parser`
   - `Supabase` (create) لحفظ الخطة
   - `Update a row` لتحديث الحالة إلى `completed`
   - `Respond to Webhook`
4. **اختبار الـ Workflow** بطلب حقيقي والتحقق من البيانات في Supabase.
5. **نشر الـ Workflow** واستخدام `Production URL`.
6. **ربط التطبيق مع n8n** عبر طبقة خادم آمنة (مثل Supabase Edge Function) ومتغير `N8N_WEBHOOK_URL`.
7. **نشر التطبيق واختباره** من البداية إلى النهاية.

---

## الأمان

- لا تُوضع المفاتيح السرية (`N8N_WEBHOOK_URL`، `N8N_WEBHOOK_SECRET`، مفاتيح Supabase) في كود المتصفح.
- يجب حماية الـ Webhook بـ Header Authentication قبل الاستخدام العام.
- لا توجد مصادقة مستخدمين بعد؛ لذلك يجب مراجعة سياسات RLS وعدم فتح الجداول للعامة.
- لا تعرض رسائل الخطأ تفاصيل داخلية أو أسرارًا للمستخدم.

---

## نقاط تحتاج مراجعة

- عقدة Supabase الأولى مضبوطة على `getAll`؛ تأكدي من تصفيتها بـ `id = request_id` حتى تُرجع الطلب المطلوب فقط.
- قيد `UNIQUE` على `request_id` يمنع خطتين لنفس الطلب؛ حدّدي سلوك إعادة المحاولة (تحديث أو رفض).
- أضيفي مسار خطأ يحدّث الحالة إلى `failed` عند فشل AI Agent أو الحفظ.
- امنعي الضغط المتكرر على زر إنشاء الخطة.

---

## معايير الاكتمال

- [x] بناء التطبيق وربطه مع Supabase
- [x] إنشاء الجدولين
- [x] بناء الـ Workflow ونشره
- [ ] حماية الـ Webhook بالمصادقة
- [ ] اختبار الفشل وإعادة المحاولة
- [ ] مراجعة سياسات الوصول (RLS)
- [ ] اختبار شامل من التطبيق المنشور: الحفظ ← المعالجة ← `completed` ← العرض بعد إعادة التحميل

---

**الهدف النهائي:** نظام متكامل يعمل بصورة موثوقة، لا مجرد واجهة تعرض نتيجة مولّدة بالذكاء الاصطناعي.
