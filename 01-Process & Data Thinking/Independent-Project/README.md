# AI Expense Tracker: Google Forms to Google Sheets and Gmail Alerts

> A Zapier automation that takes an expense written in plain text through a Google Form, uses AI to extract the amount, currency and category, saves it in a Google Sheet, and sends a Gmail alert when the expense cannot be processed automatically.

---

## 🎯 Problem

### The Context

Personal or small-team expenses are usually written in free text, for example: "I spent 70 shekels on shopping". Each entry is written differently, in different languages, and with or without an amount.

### The Problem

Tracking expenses manually means re-typing each one into a spreadsheet: finding the amount, deciding the currency and category, and keeping everything consistent. Entries are often forgotten or recorded incompletely.

### Why It Matters

Structured expense records make it possible to see where money goes, and a simple form makes entering an expense take a few seconds instead of managing a spreadsheet by hand.

---

## 💡 Solution

A Zap that runs every time a new expense is submitted through the "Expense Entry" form. It checks whether the text contains an amount, uses AI to extract the expense details, saves a structured row in Google Sheets, and sends a Gmail alert for entries that need attention.

What does the system do?

1. Detects a new response in the Google Form "Expense Entry".
2. Splits the flow with **Paths**: an expense that contains a number goes to the main path, everything else goes to the **Fallback** path.
3. On the main path, AI by Zapier extracts the expense information (amount, currency, category, status).
4. Creates a new row in the `Expenses` Google Sheet.
5. On the Fallback path, sends an email through Gmail so the person knows the expense was not recorded automatically.

---

## 👤 Users

**Primary Users:**  
Individuals or small teams who want to track expenses quickly.

**User Need:**  
Enter an expense in one sentence and have it recorded in an organized way, and be notified when an entry was not recorded.

---

## 🧠 How I Designed the Solution

- The form is intentionally simple: one required text field (`Expense_Text`) and an optional `Receipt_Link`, so entering an expense is fast.
- AI handles the part that needs language understanding: reading free text (including Arabic) and extracting the amount, currency and category.
- A **Paths** step checks the input before AI and Sheets run, so entries without a usable amount do not create bad rows.
- A **Fallback** path with a Gmail alert makes sure a failed entry is not silently lost.

---

## ⚙️ How It Works

```text
Google Forms: New Form Response (Expense_Text, Receipt_Link)
  ↓
Paths: Split into paths
  ├── Path "has a number"
  │     ↓
  │   AI by Zapier: Extract expense information
  │     ↓
  │   Google Sheets: Create Spreadsheet Row
  │
  └── Fallback
        ↓
      Gmail: Send Email (alert)
```

---

## 🤖 Role of AI

What does AI do in this solution?

- Reads the free-text expense (for example an Arabic sentence about spending 70 shekels).
- Extracts the amount and the currency (for example `70` and `ILS`).
- Suggests a category (for example `Shopping`).
- Sets a status for the record (for example `Ready`).

### What AI Should NOT Do

- Invent an amount or currency that is not in the text.
- Decide on unclear expenses by guessing; those go to the Fallback path.

---

## 🔄 Automation

What parts of the process are automated?

- Detecting new expense submissions.
- Checking whether the text contains an amount.
- Extracting amount, currency and category with AI.
- Saving the record in Google Sheets.
- Sending an email alert for entries that were not processed.

---

## 🗂️ Data

### Input

Google Form "Expense Entry":

- `Expense_Text` (required): the expense in free text.
- `Receipt_Link` (optional): a link to the receipt.

### Output

A row in the Google Sheet `Expenses` with the columns: `Submitted_At`, `Expense_Text`, `Amount`, `Currency`, `Category`, `Status`, `Receipt_Link`. For failed entries: a Gmail alert.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Google Forms | Collect expense entries |
| Zapier | Orchestrate the workflow and Paths |
| AI by Zapier | Extract amount, currency and category |
| Google Sheets | Store structured expense records |
| Gmail | Send alerts for entries that need attention |

---

## 🔑 Key Design Decisions

### Decision 1

**Decision:**  
Check for a number with Paths before running AI and saving to Sheets.

**Why:**  
An expense without an amount is not useful as a record, and it should not create an incomplete row.

### Decision 2

**Decision:**  
Add a Fallback path with a Gmail alert.

**Why:**  
Entries that fail the check should be reported to the user instead of disappearing.

### Decision 3

**Decision:**  
Keep the form to a single free-text field.

**Why:**  
It makes entering expenses fast, and AI does the structuring work.

---

## 🧪 Testing

> Fill in or correct these results from your own runs.

### Test 1 — Expense with an amount

**Input:**  
An Arabic expense text about spending 70 shekels on shopping.

**Expected Result:**  
Row saved with `Amount = 70`, `Currency = ILS`, `Category = Shopping`, `Status = Ready`, each value in its own column.

**Actual Result:**  
The row was saved with `Amount = 70` and `Currency = ILS` in the correct columns. However, `Category` and `Status` are empty, and their values (`Shopping`, `Ready`) plus an extra value (`0.94`) appear in the `Receipt_Link` column and in two columns without headers.

**Status:** ❌ (partially working: values shifted to the wrong columns)

---

### Test 2 — Expense without a usable amount

**Input:**  
[Write the text you submitted in the run of Oct 12, 2026.]

**Expected Result:**  
The Fallback path runs, no row is created, and a Gmail alert is sent.

**Actual Result:**  
The Zap run (Oct 12, 2026, 12:11 am) was successful: the Fallback path ran and Gmail sent 1 message. The Google Sheets step did not run.

**Status:** ✅ (confirm the alert email arrived in the inbox)

---

## 📸 Evidence

### Google Form

![Expense Entry Form](./screenshots/expense-form.png)

### Final Result

![Expenses Sheet](./screenshots/expenses-sheet.png)

### Workflow

![Zap Workflow](./screenshots/zap-workflow.png)

---

## ⚠️ Challenges & Failures

What didn't work as expected?

- The Sheet columns and the Zap field mapping do not match: `Shopping` ended up under `Receipt_Link`, and `0.94` and `Ready` ended up in columns `H` and `I`, which have no headers. `Category` and `Status` stayed empty.
- The `Receipt_Link` column does not contain a receipt link in the saved row.
- The run history shows the Fallback path ran and the Sheets step did not run, while the Sheet contains a saved row from around the same time. This row probably came from another run or an earlier version of the Zap, so check which run created it.
- Only one run appears in the last 30 days, so the testing coverage is small.

How did I solve it?

- [Write what you changed, for example: align the Sheet headers with the Zap's mapped fields, then re-map each field in the "Create Spreadsheet Row" step.]
- [Write how you re-tested after the fix.]

---

## 📚 What I Learned

### Technical Lesson

The order and names of the Sheet columns must match the field mapping in the Zap. A mismatch does not cause an error, it silently puts data in the wrong place.

### Problem-Solving Lesson

Checking the final Sheet is as important as checking that the Zap run is green. A successful run does not prove the data is correct.

### AI Lesson

AI is useful for understanding free text (including Arabic), but the workflow needs checks before and after it (like the Paths condition) to keep the saved data reliable.

---

## 🔮 Future Improvements

If I continued developing this project, I would:

- Fix the column mapping and add headers for every saved field.
- Add a confidence field and send low-confidence entries to review.
- Support more currencies and categories with a fixed list of allowed values.
- Add a monthly summary email or dashboard by category.
- Send a confirmation message when an expense is recorded successfully.

---

## 🔗 Project Links

**Live Demo:** [Link]

**Workflow:** [Zap link]

**Repository:** [Link]

---

## 👩‍💻 Author

**nour**

[LinkedIn] · [GitHub]
