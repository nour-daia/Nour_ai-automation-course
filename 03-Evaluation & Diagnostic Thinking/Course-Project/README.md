# TaskBridge

> A responsive web app that helps freelancers and early-career professionals turn unclear work requests into clear plans with prioritized tasks, statuses, and tracked progress.

---

## 🎯 Problem

### The Context

Freelancers, recent graduates, and early-career professionals often receive work as vague requests ("I need a small online store") or set goals for themselves without a clear path. The work lives in messages, notes, and memory.

### The Problem

A person may know what they want to achieve but not how to turn it into practical steps. They struggle to answer:

- What outcome should be delivered?
- Which tasks are required, and which should start first?
- What is already done, and what is still in progress?
- What is the next step?

Without a clear plan, organizing work, setting priorities, and tracking progress becomes difficult.

### Why It Matters

A clear plan reduces missed work, makes priorities visible, and lets people see real progress. For freelancers it also makes client requests easier to scope and deliver.

---

## 💡 Solution

TaskBridge provides one organized place to manage plans and tasks. The core idea is the sequence:

**Goal → Plan → Tasks → Priorities → Status → Progress**

What does the system do?

1. Lets a user create an account and sign in, so every plan belongs to its owner.
2. Lets the user define a plan: title, description, goal / desired outcome, optional deadline, and deliverables.
3. Lets the user break the plan into tasks, each with a priority (High / Medium / Low) and a status (To Do / In Progress / Done).
4. Calculates and shows progress automatically from completed tasks, and keeps all data saved so it is still there after refresh or re-login.

---

## 👤 Users

**Primary Users:**  
Freelancers managing client requests, recent graduates organizing professional projects, early-career professionals, and individuals who want to turn goals into actionable tasks. This version is for individual use only (no team collaboration).

**User Need:**  
Turn a goal or request into a structured plan, decide what to do first, and track what has been done.

---

## 🧠 How I Designed the Solution

I treated the project as a system, not just a screen (*Build the System, Not the Screen*): understand the problem, design the data and relationships, define the behavior, then secure and test it.

- **Data model first:** the relationship is `User → Plans → Tasks`. A user owns many plans, a plan contains many tasks, and every task belongs to exactly one plan.
- **Backend with Supabase:** Supabase Auth for accounts, PostgreSQL for persistent storage, and Row Level Security for access control.
- **Security at the database level:** each user can access only their own records, enforced by RLS policies rather than by hiding things in the interface.
- **Data integrity:** allowed values for priority and status are enforced by database constraints, a task's owner must match its plan's owner, and deleting a plan deletes its tasks.
- **Small, focused MVP:** authentication, plans, tasks, priorities, statuses, progress, persistence, and a responsive interface. Everything else was deliberately left out.

---

## ⚙️ How It Works

```text
User signs up / signs in (Supabase Auth)
  ↓
Creates a plan: title, description, goal, deadline, deliverables
  ↓
Adds tasks with priority and status
  ↓
Logic: Progress = (Completed Tasks ÷ Total Tasks) × 100
  ↓
Plan saved in Supabase (protected by RLS)
  ↓
Output: plan details, task list, and live progress percentage
```

**Pages:** Landing Page, Authentication (sign in / sign up), My Plans, Create / Edit Plan, Plan Details.

**Example:** a plan with 5 tasks where 2 are Done shows 40% progress. A plan with no tasks shows 0%.

---

## 🤖 Role of AI

AI is **not a feature inside the app** in this version. It was used as a **tool to build the app**.

What does AI do in this solution?

- Lovable (AI App Builder) generated the application from structured requirements.
- AI helped speed up building the interface and connecting it to Supabase.

### What AI Should NOT Do

- Replace understanding of the system: I defined the data model, relationships, security rules, and behavior, and reviewed what was generated.
- Be trusted without checking: its output (CRUD operations, progress calculation, access control) had to be tested for real behavior.

> ⚠️ **[Edit]** Add which decisions you reviewed or changed in Lovable's output and how you verified them.

---

## 🔄 Automation

What parts of the process are automated?

- Progress percentage is recalculated automatically whenever a task's status changes.
- Data is saved automatically to the database, so it persists across refreshes and logins.
- Access control is enforced automatically by RLS policies on every database request.
- Deleting a plan automatically deletes its tasks.

---

## 🗂️ Data

### Input

- **Account:** sign-up and sign-in credentials (handled by Supabase Auth).
- **Plan:** `title`, `description`, `goal`, `deadline` (optional), `deliverables` (JSONB), `next_step` (optional).
- **Task:** `title`, `description` (optional), `priority` (High / Medium / Low), `status` (To Do / In Progress / Done), `sort_order`.

### Output

- A personal list of plans for the signed-in user.
- Plan details with all related tasks.
- Progress percentage per plan.

**Tables:** `plans` and `plan_tasks`. Both store `user_id` and have Row Level Security enabled. See `database-schema.md` for the full schema.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Lovable | Building the application with an AI App Builder |
| Supabase Auth | User authentication |
| Supabase PostgreSQL | Storing plans and tasks |
| Supabase RLS | Per-user access control at the database level |
| GitHub | Documentation, files, and version control |

---

## 🔑 Key Design Decisions

### Decision 1

**Decision:**  
Enforce data isolation with Row Level Security in the database.

**Why:**  
Hiding data in the interface does not protect it. RLS ensures one user cannot read, edit, or delete another user's plans or tasks even if the interface is bypassed.

### Decision 2

**Decision:**  
Keep `user_id` on tasks and require the task owner to match the plan owner.

**Why:**  
It prevents a user from attaching a task to a plan they do not own and keeps ownership consistent.

### Decision 3

**Decision:**  
Restrict priority and status to fixed allowed values in the database.

**Why:**  
The values used in the app must match the database constraints, which protects data integrity.

### Decision 4

**Decision:**  
Keep the first version small: no AI plan generation, notifications, payments, or team collaboration.

**Why:**  
Focusing on the core functions makes it possible to verify their quality before adding more.

### Decision 5

**Decision:**  
Never place the `service_role` key in frontend code or in a public repository.

**Why:**  
That key bypasses access control. Using the correct client key with RLS keeps access safe.

---

## 🧪 Testing

> ⚠️ **[Fill in Actual Result and Status after testing the live app. Do not assume a test passed.]**

### Test 1: Create account and sign in

**Input:**  
Sign up with a new account, sign out, then sign in again.

**Expected Result:**  
Account is created, sign-out and sign-in work, and private pages require authentication.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 2: Create a plan and add tasks

**Input:**  
Create "Launch a Small Online Store" and add tasks with different priorities and statuses.

**Expected Result:**  
Plan and tasks are saved and displayed correctly with their priority and status.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 3: Progress calculation

**Input:**  
Plan with 5 tasks, mark 2 as Done. Then mark one back to In Progress. Then test a plan with no tasks.

**Expected Result:**  
40% with 2 of 5 done, the value updates when a task status changes, and an empty plan shows 0%.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 4: Data persistence

**Input:**  
Refresh the page, then sign out and sign in again.

**Expected Result:**  
Plans and tasks are still there.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 5: User data isolation (two accounts)

**Input:**  
Using User A and User B, try to read, edit, and delete the other user's plans and tasks, and try to create a task under a plan owned by the other user.

**Expected Result:**  
All attempts are blocked. Each user sees only their own data.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 6: Delete a plan

**Input:**  
Delete a plan that has tasks.

**Expected Result:**  
The plan and all its tasks are deleted.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 7: Responsive design

**Input:**  
Open the app on desktop and mobile screen sizes.

**Expected Result:**  
Layout and all main actions work on both.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

## 📸 Evidence

### Final Result

![Final Result](./screenshots/final-result.png)

### Landing Page

![Landing Page](./screenshots/landing.png)

> ⚠️ **[Add screenshots]** Plans list, plan details with progress, the Supabase tables, RLS policies, and the two-account isolation test.

---

## ⚠️ Challenges & Failures

What didn't work as expected?

- ...
- ...

How did I solve it?

- ...
- ...

> ⚠️ **[Fill in from your real experience.]** Ideas to consider: Lovable and Supabase connection, mismatch between app values and database constraints, RLS blocking legitimate actions, progress shown incorrectly for empty plans.

---

## 📚 What I Learned

### Technical Lesson

...

### Problem-Solving Lesson

...

### AI Lesson

...

> ⚠️ **[Write in your own words.]** Hint for the AI lesson: AI builds quickly, but the engineer must understand, direct, test, and secure what it builds.

---

## 🔮 Future Improvements

If I continued developing this project, I would:

- Add AI plan generation from a natural-language goal, then evaluate the generated plans for clarity and realism.
- Integrate with external automation tools to move data or trigger actions.
- Explore a Progressive Web App (PWA) experience.

---

## 🔗 Project Links

**Live Demo:** https://precise-layout-machine.lovable.app/

**Workflow:** Not applicable

**Repository:** [Add GitHub repository link]

---

## 👩‍💻 Author

**[Your Name]**

[LinkedIn] · [GitHub]
