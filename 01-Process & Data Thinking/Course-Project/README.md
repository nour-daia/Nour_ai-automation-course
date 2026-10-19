# Google Forms to AI Analysis and Email Notification

> A Zapier automation that takes client requests submitted through a Google Form, uses AI to extract and classify the request details, saves each request to Google Sheets, and sends an email notification based on whether the request is valid or needs review.

---

## 🎯 Problem

### The Context

A business receives client requests through a Google Form. Each request is written in free text (for example: "We need a website, our budget is $5000, we need it next month"), so every submission looks different.

### The Problem

Someone has to read each request manually, figure out what service the client wants, what the timeline and budget are, whether anything is missing, and who should handle it. This is slow, inconsistent, and easy to forget.

### Why It Matters

Automating the first review gives the team structured, comparable records right away, makes sure incomplete requests are flagged instead of ignored, and notifies the right people faster.

---

## 💡 Solution

A Zap that runs every time a new form response arrives. It uses AI to turn the free-text request into structured fields, then routes the request into one of two paths (valid or fallback/needs review), records it in a Google Sheet, and sends an email.

What does the system do?

1. Detects a new response in Google Forms.
2. Uses AI by Zapier to extract the client request details (service type, summary, timeline, budget, missing information, status, routing).
3. Splits the flow into paths: **VALID** or **Fallback**.
4. Creates a row in Google Sheets for the request and sends an email through Gmail.

---

## 👤 Users

**Primary Users:**  
The team that receives and handles client requests (for example the sales or web team).

**User Need:**  
Get clean, structured, correctly routed client requests, and be told immediately when a request is incomplete and needs human review.

---

## 🧠 How I Designed the Solution

- The form collects the basic client info (Client Name, Email, Company) and the request text.
- AI is used only for the part that needs language understanding: reading the free-text request and extracting structured fields.
- Routing is done with a **Paths** step, so each outcome has its own record and email, instead of one path handling everything.
- A **Fallback** path exists so that unclear or incomplete requests are not lost; they are saved with a review status and sent to a human.

---

## ⚙️ How It Works

```text
Google Forms: New Form Response
  ↓
AI by Zapier: Extract client request details
  ↓
Paths: Split into paths
  ├── VALID    → Google Sheets: Create Spreadsheet Row → Gmail: Send Email
  └── Fallback → Google Sheets: Create Spreadsheet Row → Gmail: Send Email
```

---

## 🤖 Role of AI

What does AI do in this solution?

- Reads the free-text request and extracts structured details.
- Identifies the service type (for example: website) and writes a short request summary.
- Extracts timeline and budget when they are mentioned.
- Lists missing information when the request is incomplete.
- Produces a request status (for example `VALID` or `NEEDS_REVIEW`) and a routing value (for example `web_team`) used by the Paths step.

### What AI Should NOT Do

- Invent a timeline, budget, or service that the client did not mention.
- Make the final decision on unclear requests without a human (those should be flagged for review).

---

## 🔄 Automation

What parts of the process are automated?

- Detecting new form responses.
- Extracting and classifying the request.
- Routing into the valid or fallback path.
- Saving the record in Google Sheets.
- Sending the notification email.

---

## 🗂️ Data

### Input

Google Form fields: **Client Name**, **Email**, **Company**, and the **request text** (plus any other form fields: [add the remaining form questions here]).

### Output

A row in the Google Sheet `AI Client Intake Records` with these columns: `client_name`, `email`, `company`, `request_text`, `service_type`, `request_summary`, `timeline`, `budget`, `missing_information`, `request_status`, `requires_human`, `routing`, [add the remaining columns], and an email sent through Gmail.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Google Forms | Collect client requests |
| Zapier | Orchestrate the workflow and Paths |
| AI by Zapier | Extract and classify request details |
| Google Sheets | Store structured request records |
| Gmail | Send email notifications |

---

## 🔑 Key Design Decisions

### Decision 1

**Decision:**  
Use a Paths step with a VALID path and a Fallback path.

**Why:**  
Incomplete or unclear requests must be handled differently from complete ones, and must never be silently dropped.

### Decision 2

**Decision:**  
Save every request to Google Sheets, including the ones that need review.

**Why:**  
The team keeps a full record, and `request_status` / `requires_human` show which rows need attention.

### Decision 3

**Decision:**  
Use AI only to extract and classify, and keep routing rules in Zapier Paths conditions.

**Why:**  
Rules in Paths are predictable and easy to check, while AI handles the language understanding.

---

## 🧪 Testing

> Fill in the actual results from your runs. The statuses below are based on what is visible in the Sheet screenshot.

### Test 1 — Complete request

**Input:**  
Client "nour", company "nour", request: "We need a website... budget is $5000... next month".

**Expected Result:**  
Service type `website`, timeline `next month`, budget `$5000`, status `VALID`, routed to `web_team`, row saved, email sent.

**Actual Result:**  
Rows 2, 3 and 5 in the Sheet show `VALID`, `website`, `next month`, `$5000`, `web_team`. The Zap run shows the VALID path ran, with the row created and the email sent.

**Status:** ✅ (verify the email was received)

---

### Test 2 — Unclear / meaningless request

**Input:**  
Request text: `j`.

**Expected Result:**  
Status `NEEDS_REVIEW`, human review required, saved through the Fallback path, review email sent.

**Actual Result:**  
Row 6 shows `NEEDS_REVIEW`, summary "The client provided 'j'", and human review required.

**Status:** ✅ (verify the email was received)

---

### Test 3 — Data consistency across rows

**Input:**  
Several test submissions.

**Expected Result:**  
Each value appears in its own column, with consistent values.

**Actual Result:**  
Some rows have inconsistencies (see Challenges below).

**Status:** ❌

---

## 📸 Evidence

### Final Result

![Final Result](./screenshots/sheet-results.png)

### Workflow

![Workflow](./screenshots/zap-workflow.png)

### Google Form

![Google Form](./screenshots/google-form.png)

---

## ⚠️ Challenges & Failures

What didn't work as expected?

- In row 2, the `requires_human` column contains `VALID` (looks like a value shifted into the wrong column).
- In row 4, the `missing_information` column contains `$5000` (the budget) instead of `NONE`, and the status/routing show a review value.
- The `requires_human` column has inconsistent values: `FALSE` and `Ture` (a typo of `TRUE`).
- The Fallback path did not run in the latest run shown (expected, since the request was valid).

How did I solve it?

- [Write what you changed: for example, fixing the field mapping in the Sheets step, and forcing the AI output to use exactly `TRUE` / `FALSE`.]
- [Write how you re-tested after the fix.]

---

## 📚 What I Learned

### Technical Lesson

Field mapping between steps matters: one wrong mapping puts values in the wrong column. Test each step's output before relying on it.

### Problem-Solving Lesson

Test both the normal case and the bad-input case (like the request `j`), not only the happy path.

### AI Lesson

AI output must follow a strict format (exact allowed values) if later steps depend on it, otherwise typos and inconsistent values break routing and reporting.

---

## 🔮 Future Improvements

If I continued developing this project, I would:

- Restrict AI output to fixed allowed values for status, routing and `requires_human`.
- Send the client an automatic confirmation or a request for the missing information.
- Add a "Needs Human Review" notification to a specific person or channel.
- Add a dashboard summarizing requests by service type and status.

---

## 🔗 Project Links

**Live Demo:** [Link]

**Workflow:** [Zap link]

**Repository:** [Link]

---

## 👩‍💻 Author

**Nour**

[LinkedIn] · [GitHub]
