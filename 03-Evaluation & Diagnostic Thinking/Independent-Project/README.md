# 🤝 Volunteer Board

> A community web platform where local organizers publish workshops and volunteer opportunities, and neighbors browse them and sign up to help.

---

## 🎯 Problem

### The Context

In local communities, organizers (community centers, youth groups, initiatives) regularly run workshops and activities that need volunteers. Announcements are usually shared through scattered WhatsApp groups, Facebook posts, and word of mouth.

### The Problem

- Opportunities get lost in busy group chats and social media feeds.
- Organizers have no clear way to know how many people signed up or who is coming.
- People who want to volunteer don't know where to find opportunities near them.

### Why It Matters

Small hours of volunteering add up to big change in a neighborhood. A single, simple place to post and find opportunities makes it easier for organizers to get help and for neighbors to give it.

---

## 💡 Solution

**Volunteer Board** is a web app that connects organizers with volunteers in one place.

What does the system do?

1. Users create an account and log in securely.
2. Organizers publish workshops / volunteer opportunities (title, description, date, location, number of volunteers needed).
3. Volunteers browse the available opportunities and sign up for the ones they want.
4. Organizers can see who signed up for each workshop and manage their posts.

---

## 👤 Users

**Primary Users:**
- **Organizers:** community centers, initiatives, and individuals running workshops or events.
- **Volunteers:** neighbors and students who want to give their time locally.

**User Need:**
- Organizers need to post an opportunity quickly and track who is coming.
- Volunteers need to find opportunities easily and sign up in a few clicks.

---

## 🧠 How I Designed the Solution

I started from the two sides of the problem: the person who **needs help** and the person who **wants to help**. I kept the flow as short as possible: log in → post or browse → sign up.

The design uses a calm green color palette and a friendly tagline ("Small hours, big neighborhoods.") to make the platform feel welcoming and community-focused rather than corporate.

---

## ⚙️ How It Works

```text
User signs up / logs in
  ↓
Organizer creates a workshop post
  ↓
Post is saved to the database and shown on the board
  ↓
Volunteer browses opportunities and clicks "Sign up"
  ↓
Sign-up is recorded and linked to the workshop
  ↓
Organizer sees the list of registered volunteers
```

---

## 🤖 Role of AI

What does AI do in this solution?

- AI (Lovable) was used to generate the app's interface and code from my written prompts.
- AI helped me iterate quickly on the design, pages, and layout.
- AI helped me structure the data (users, workshops, sign-ups) and fix errors during development.

### What AI Should NOT Do

- AI should not decide who is allowed to volunteer or approve people on behalf of organizers.
- AI should not access or share users' personal information (emails, passwords).

---

## 🔄 Automation

What parts of the process are automated?

- User registration and login (authentication).
- Publishing a workshop instantly to the board once it is created.
- Recording volunteer sign-ups and linking them to the right workshop.

---

## 🗂️ Data

### Input

- Account details (name, email, password).
- Workshop details (title, description, date, time, location, volunteers needed).
- Volunteer sign-ups.

### Output

- A public board of available workshops and opportunities.
- A list of signed-up volunteers for each workshop.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Lovable | AI-powered building of the web app (UI + code) |
| React + Tailwind CSS | Frontend interface and styling |
| Supabase | Database and user authentication |
| Lovable Hosting | Deploying and publishing the live app |

---

## 🔑 Key Design Decisions

### Decision 1

**Decision:**
Require users to log in before posting or signing up.

**Why:**
So organizers know who is actually coming, and to prevent spam or fake posts.

### Decision 2

**Decision:**
Keep the interface minimal, with one main action per page.

**Why:**
Volunteers and organizers come from all ages and backgrounds; a simple flow means anyone can use it without explanation.

---

## 🧪 Testing

### Test 1

**Input:**
Log in with a registered email and correct password.

**Expected Result:**
User is redirected to the board of workshops.

**Actual Result:**
User logged in successfully and saw the board.

**Status:** ✅

---

### Test 2

**Input:**
Organizer creates a new workshop, then a volunteer account signs up for it.

**Expected Result:**
The workshop appears on the board and the volunteer appears in the organizer's sign-up list.

**Actual Result:**
The workshop was published and the sign-up was recorded.

**Status:** ✅

---

## 📸 Evidence

### Final Result

![Final Result](./screenshots/final-result.png)

### Workflow

![Workflow](./screenshots/workflow.png)

---

## ⚠️ Challenges & Failures

What didn't work as expected?

- Getting the AI to produce exactly the layout I had in mind took several rounds of prompts.
- Connecting the sign-ups to the correct workshop and user needed fixing.

How did I solve it?

- I wrote more specific, step-by-step prompts and changed one thing at a time.
- I tested each feature with two accounts (organizer and volunteer) to catch errors early.

---

## 📚 What I Learned

### Technical Lesson

How a web app connects a frontend to a database and authentication, and how data like users, posts, and sign-ups relate to each other.

### Problem-Solving Lesson

Starting from the real needs of the users (organizers and volunteers) leads to a simpler, more useful product than starting from features.

### AI Lesson

AI is a powerful builder, but the quality of the result depends on how clearly I describe what I want — and I still need to test and verify everything it produces.

---

## 🔮 Future Improvements

If I continued developing this project, I would:

- Add email/WhatsApp reminders for volunteers before a workshop.
- Add filters by location, date, and category.
- Add Arabic language support and a right-to-left layout.
- Track volunteer hours and give certificates.

---

## 🔗 Project Links

**Live Demo:** [Volunteer Board](https://pixel-perfect-dream-8554.lovable.app/login)

**Workflow:** [Link]

**Repository:** [Link]

---

## 👩‍💻 Author

**[Your Name]**

[LinkedIn] · [GitHub]
