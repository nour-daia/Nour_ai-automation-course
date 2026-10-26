
# مساعد ذكي بتقنية RAG باستخدام n8n و Supabase

مشروع يبني **مساعد دردشة ذكي** يجيب على الأسئلة اعتماداً على قاعدة معرفة خاصة (ملفات PDF) بدل الاعتماد على معلومات النموذج العامة فقط. تم بناؤه باستخدام **n8n** لأتمتة سير العمل، و**OpenAI** للنموذج اللغوي والـ Embeddings، و**Supabase** (PostgreSQL + pgvector) كقاعدة بيانات متجهات.

---

## نظرة عامة على المعمارية

المشروع مكوّن من Workflow اثنين في n8n:

| Workflow | الاسم | الوظيفة |
|---|---|---|
| 1 | `Knolgme_base` | بناء قاعدة المعرفة: استيعاب ملفات PDF من Google Drive وتخزينها كمتجهات في Supabase |
| 2 | `RAG AI` | المساعد: يستقبل رسائل الدردشة ويبحث في قاعدة المعرفة ثم يجيب |

```
┌───────────────────────── Workflow 1: Knolgme_base (الاستيعاب) ─────────────────────────┐
│                                                                                        │
│  Google Drive Trigger → file_id → Download file → Extract from File → Supabase Vector  │
│     (fileCreated)       (Edit Fields)             (PDF)                Store (Insert)  │
│                                                                          ▲      ▲      │
│                                              Embeddings OpenAI ──────────┘      │      │
│                                              Default Data Loader ◄── Text Splitter     │
└────────────────────────────────────────────────────────────────────────────────────────┘

┌───────────────────────────── Workflow 2: RAG AI (المساعد) ─────────────────────────────┐
│                                                                                        │
│  When chat message received → AI Agent → الرد                                           │
│                                  ├── Chat Model: OpenAI Chat Model                     │
│                                  ├── Memory:     Simple Memory                         │
│                                  └── Tool:       Supabase Vector Store                 │
│                                                    └── Embeddings OpenAI               │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## المتطلبات

- حساب **n8n** (Cloud أو Self-hosted)
- حساب **Supabase** (المشروع الحالي: `levora-python-service` / `AI-Powered Workflow`)
- مفتاح **OpenAI API**
- حساب **Google Drive** مع صلاحية الوصول لمجلد الملفات

---

## الإعداد

### 1) تجهيز قاعدة البيانات في Supabase

شغّل الأوامر التالية في **SQL Editor**:

```sql
-- تفعيل امتداد المتجهات
create extension if not exists vector;

-- جدول المستندات
create table documents (
  id bigserial primary key,
  content text,
  metadata jsonb,
  embedding vector(1536)  -- 1536 يناسب text-embedding-3-small و ada-002
);

-- دالة البحث بالتشابه (تستخدمها عقدة Supabase Vector Store)
create function match_documents (
  query_embedding vector(1536),
  match_count int default null,
  filter jsonb default '{}'
) returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

> ملاحظة: يجب أن يطابق بُعد المتجه (`1536`) نموذج الـ Embeddings المستخدم في n8n.

### 2) إعداد الاعتمادات (Credentials) في n8n

| الاعتماد | الاستخدام |
|---|---|
| **OpenAI API** | Chat Model + Embeddings |
| **Supabase API** | `Project URL` و `service_role key` من Project Settings ← API |
| **Google Drive OAuth2** | Trigger + Download file |

> ⚠️ لا تشارك مفتاح `service_role` ولا مفاتيح OpenAI علناً، ولا ترفعها إلى GitHub.

### 3) Workflow 1: بناء قاعدة المعرفة (`Knolgme_base`)

1. **Google Drive Trigger**: حدث `fileCreated` على المجلد المخصص.
2. **file_id** (Edit Fields): استخراج معرّف الملف.
3. **Download file**: تنزيل الملف من Google Drive.
4. **Extract from File**: استخراج النص من PDF.
5. **Supabase Vector Store** (Mode: *Insert Documents*): الجدول `documents`، ومعه:
   - **Embeddings OpenAI**: لتحويل النص إلى متجهات.
   - **Default Data Loader**: تحميل المحتوى مع الـ metadata.
   - **Text Splitter**: تقسيم النص إلى أجزاء (Chunks).

بعد التنفيذ تظهر الأجزاء في جدول `documents` (حالياً **58 سجل**).

### 4) Workflow 2: المساعد (`RAG AI`)

1. **When chat message received**: نقطة دخول الدردشة.
2. **AI Agent**: العقل المدبّر، ويستخدم:
   - **OpenAI Chat Model** كنموذج لغوي.
   - **Simple Memory** لحفظ سياق المحادثة.
   - **Supabase Vector Store** كأداة (Tool) للبحث في قاعدة المعرفة، مع **Embeddings OpenAI**.
3. اكتب في **System Message** للـ Agent تعليمات مثل:

```
أنت مساعد ذكي. عند أي سؤال عن المحتوى، استخدم أداة البحث في قاعدة المعرفة
وأجب اعتماداً على النتائج فقط. إذا لم تجد المعلومة قل ذلك بوضوح.
أجب بنفس لغة المستخدم.
```

---

## طريقة الاستخدام

1. فعّل/شغّل **Workflow 1** وأضف ملف PDF جديد إلى مجلد Google Drive.
2. تأكد من ظهور السجلات الجديدة في جدول `documents` في Supabase.
3. افتح **Workflow 2** واضغط **Open chat**، ثم اسأل عن محتوى الملفات.

مثال:

> **المستخدم:** ما هي أنواع الأتمتة المذكورة في الدليل؟
> **المساعد:** يجيب بناءً على المقاطع المسترجعة من قاعدة المعرفة.

---

## هيكل جدول `documents`

| العمود | النوع | الوصف |
|---|---|---|
| `id` | int8 | المعرّف |
| `content` | text | نص الجزء (Chunk) |
| `metadata` | jsonb | معلومات المصدر (مثل `loc.lines`) |
| `embedding` | vector | المتجه الرقمي للنص |

---

## نصائح لتحسين الجودة

- **حجم الـ Chunk**: جرّب 500–1000 حرف مع Overlap بين 50–100.
- **تنظيف النص** بعد استخراجه من PDF (إزالة الترويسات والأسطر الفارغة) لأن بعض السجلات الحالية تحتوي نصاً قصيراً مثل `1.6` و `en`.
- **تجنب التكرار**: احذف السجلات القديمة للملف قبل إعادة إدراجه.
- **عدد النتائج (Limit)** في أداة Vector Store: ابدأ بـ 4–5.
- فعّل **RLS** في Supabase إذا كان الجدول سيُستخدم من جهات أخرى.

---

## استكشاف الأخطاء

| المشكلة | الحل المحتمل |
|---|---|
| `match_documents` غير موجودة | نفّذ سكربت SQL أعلاه وتأكد من الاسم في عقدة Supabase |
| خطأ في بُعد المتجه | طابق `vector(N)` مع نموذج الـ Embeddings |
| الـ Agent لا يستخدم الأداة | وضّح في System Message ووصف الأداة متى يستخدمها |
| لا تظهر بيانات بعد الرفع | تحقق من تفعيل الـ Trigger ومن صلاحيات Google Drive |

---

## التقنيات المستخدمة

- [n8n](https://n8n.io) — أتمتة سير العمل
- [OpenAI](https://platform.openai.com) — النموذج اللغوي والـ Embeddings
- [Supabase](https://supabase.com) — PostgreSQL مع pgvector
- Google Drive — مصدر الملفات

---

## الخطوات القادمة

- [ ] دعم صيغ أخرى (DOCX، TXT)
- [ ] تنظيف وتوحيد الـ metadata وإضافة اسم الملف كمصدر
- [ ] ربط المساعد بقناة (واجهة ويب / WhatsApp / Telegram)
- [ ] إضافة عرض المصادر في إجابات المساعد
