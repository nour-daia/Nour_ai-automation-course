# Nour Abu Daia | AI-Powered Professional Portfolio

> A professional portfolio built as a small application: visitors can explore my skills, projects, and experience, or simply ask an AI assistant that answers only from the information in the portfolio.

---

## 🎯 Problem

### The Context

Recruiters, employers, and clients usually have very little time. They open a portfolio, scan it quickly, and try to decide whether the person matches what they need. A traditional portfolio is a CV turned into a website, so the visitor has to do all the searching and matching alone.

### The Problem

A static portfolio does not help the visitor answer their own questions, such as: *What automation experience does this person have? Which project shows React skills? Is she a fit for a full stack role? How do I contact her?* The information exists, but finding the relevant part takes effort.

### Why It Matters

If a visitor cannot quickly find what is relevant to them, they leave. A portfolio that guides visitors to the right information, and answers their questions directly, makes my professional identity clearer and makes it easier for people to contact me.

---

## 💡 Solution

A portfolio designed as a **professional application**, not just a page. It presents my professional identity clearly and adds an **AI Assistant ("Ask me about my work")** that answers visitor questions based on the portfolio's own content.

What does the system do?

1. Introduces who I am, what I do, and the value I offer in the first screen (Full Stack Developer | Workflow Automation | AI Engineering Student).
2. Presents About, Skills (grouped in categories), Projects, and Experience & Education in an organized way.
3. Lets visitors ask the AI Assistant questions about my skills, projects, and experience, in English or Arabic, with suggested questions to start from.
4. Gives clear ways to get in touch: contact form, email, LinkedIn, and GitHub.

---

## 👤 Users

**Primary Users:**  
Recruiters and employers evaluating a candidate, and clients or collaborators looking for a developer or automation specialist.

**User Need:**  
Quickly understand who I am, what I have built, and whether my skills fit their need, then contact me.

---

## 🧠 How I Designed the Solution

I started from the visitor, not from the page: *what should they know about me, what should they be able to do, and where can AI improve their experience?*

- **Identity first:** the Home section answers "who, what, which field, what value" immediately, with direct buttons to Projects, the AI Assistant, and Contact.
- **Structure by visitor intent:** navigation follows how a visitor evaluates someone: Home → About → Skills → Projects → Experience → AI Assistant → Contact.
- **Skills grouped by category** (Frontend, Backend & Data, CMS, Automation & AI, Tools, Design & Collaboration) so they can be scanned rather than read as one long list.
- **AI where search effort is highest:** instead of adding AI for its own sake, I placed it where visitors would otherwise have to hunt for answers, matching their question to my real work.
- **Grounded AI:** the assistant must rely only on portfolio information and handle missing information openly instead of inventing answers.

---

## ⚙️ Requirements

- Clear professional identity on the first screen.
- Sections: Home, About, Skills, Projects, Experience / Education, Contact.
- At least one AI feature with real value for the visitor.
- AI answers must come only from information in the portfolio.
- Works on desktop and mobile.
- Clear way to contact me.

### Core Features

- Hero section with identity and three calls to action.
- Categorized skills.
- Project cards with status, short description, tools, and a details view.
- Experience and education timeline.
- AI Assistant with suggested questions, English and Arabic support.
- Contact form plus direct contact links.
- Light / dark mode toggle.

---

## ⚙️ How It Works

### User Flow

```text
Home → Projects → Project Details → Contact
```

```text
Home → AI Assistant → Ask Question → AI Response → Contact
```

### AI Assistant Flow

```text
Visitor question
  ↓
Assistant receives question + portfolio knowledge
  ↓
AI answers using ONLY portfolio information
  ↓
If information is missing → says so clearly
  ↓
Answer shown to the visitor
```

---

## 🤖 Role of AI

What does AI do in this solution?

- Answers visitor questions about my skills, projects, and experience (for example: *What automation experience does Nour have?* or *Which project shows her React skills?*).
- Helps a visitor judge fit for a role (for example: *Is she a fit for a full stack role?*) by connecting the question to evidence in my projects and experience.
- Points visitors to contact options when they ask how to reach me.
- Supports both English and Arabic questions.

### What AI Should NOT Do

- Invent skills, projects, clients, achievements, or experience that are not in the portfolio.
- Guess when information is missing. It should say the information is not available in the portfolio.
- Present itself as a final judgment of my ability or promise hiring outcomes.

---

## 🔄 Automation

What parts of the process are automated?

- Answering common visitor questions at any time without me being involved.
- Matching a visitor's question to the relevant skills and projects.
- Suggested questions help the visitor start the conversation in one click.

> ⚠️ **[Edit]** If the contact form sends messages to your email or stores them, add it here. If it is not connected yet, say so honestly.

---

## 🗂️ Data

### Input

Portfolio data that the app and the AI Assistant rely on:

- **Profile:** name, headline, location, short introduction, languages (Arabic native, English upper intermediate).
- **Skills:** Frontend, Backend & Data, CMS, Automation & AI, Tools, Design & Collaboration.
- **Projects:** Internal Employee Profile System, Automated Blogging System, AI-Powered Customer Support Chatbots, Company Websites (WordPress), Gaza Strip Web Projects.
- **Experience:** two Web Developer roles (Saudi Arabia remote collaboration, Gaza Strip).
- **Education:** B.Sc. Artificial Intelligence Engineering (UCAS, current third-year student), Diploma in Software Development and Databases (GTC), secondary school scientific stream.
- **AI Knowledge:** the same portfolio information, used as the only source for assistant answers.
- **Visitor input:** a question for the assistant, or a name, email, and message in the contact form.

### Output

- A structured portfolio the visitor can navigate.
- An AI answer grounded in portfolio data, in the visitor's language.
- A submitted contact message.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Lovable | Building the portfolio application from structured requirements and iterating on it |
| AI model | Powering the "Ask me about my work" assistant |
| GitHub | Repository and version control |

> ⚠️ **[Edit]** Add the exact stack Lovable generated for your project (for example React, TypeScript, Tailwind CSS) and the backend or service used for the AI call, if you confirmed them in the code.

---

## 🔑 Key Design Decisions

### Decision 1

**Decision:**  
Treat the portfolio as a professional application with one clear AI feature, instead of adding many features.

**Why:**  
A simple, clear system is more useful to a visitor than a crowded one. The AI assistant targets the biggest friction: finding relevant information quickly.

### Decision 2

**Decision:**  
Restrict the AI Assistant to portfolio information only.

**Why:**  
A portfolio represents my real professional identity. An assistant that invents details would damage trust, so missing information must be handled openly.

### Decision 3

**Decision:**  
Provide suggested questions and bilingual (English/Arabic) support.

**Why:**  
Suggested questions lower the effort to start, and bilingual support fits both local and international visitors.

### Decision 4

**Decision:**  
Write detailed requirements for Lovable (goal, user, pages, data, AI feature, UX, constraints) instead of a generic "build me a portfolio" prompt.

**Why:**  
Clear decisions produce a result that matches my identity and goals, and make later iterations specific.

---

## 🧪 Testing

> ⚠️ **[Fill in Actual Result and Status after testing the live site. Do not assume a test passed.]**

### Test 1: Question covered by the portfolio

**Input:**  
"What automation experience does Nour have?"

**Expected Result:**  
Answer mentions n8n workflow automation, the Automated Blogging System, and AI chatbots, with no invented details.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 2: Question with information not in the portfolio

**Input:**  
A question the portfolio does not cover (for example about a skill, employer, or salary expectation not listed).

**Expected Result:**  
The assistant says the information is not available in the portfolio instead of inventing an answer.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 3: Fit question

**Input:**  
"Is she a fit for a full stack role?"

**Expected Result:**  
Answer relies on real skills and projects (Next.js, TypeScript, React, PHP, MySQL, REST APIs) and does not guarantee outcomes.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 4: Arabic question

**Input:**  
A question in Arabic about my projects.

**Expected Result:**  
Answer in Arabic, based on the same portfolio information.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 5: User flow and contact

**Input:**  
Home → Projects → Project Details → Contact, then submit the contact form.

**Expected Result:**  
Navigation works without errors and the form gives clear feedback.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

### Test 6: Desktop and mobile

**Input:**  
Open the live site on desktop and on a phone.

**Expected Result:**  
Layout, navigation, and the AI Assistant are usable on both.

**Actual Result:**  
...

**Status:** ✅ / ❌

---

## 📸 Evidence

### Final Result

![Final Result](./screenshots/final-result.png)

### AI Assistant

![AI Assistant](./screenshots/ai-assistant.png)

> ⚠️ **[Add screenshots]** Home, Projects, the AI Assistant answering a question, and the mobile view.

---

## ⚠️ Challenges & Failures

What didn't work as expected?

- ...
- ...

How did I solve it?

- ...
- ...

> ⚠️ **[Fill in from your real iterations.]** Ideas to consider: first Lovable version vs final, AI answers that invented or missed details, layout issues on mobile.

---

## 📚 What I Learned

### Technical Lesson

...

### Problem-Solving Lesson

...

### AI Lesson

...

> ⚠️ **[Write in your own words.]**

---

## 🪞 Reflection

1. **What problem did I design the portfolio to solve?**  
   Helping visitors quickly understand my professional identity and find what is relevant to their need, instead of searching a static page.

2. **Who is the primary user?**  
   Recruiters, employers, and clients evaluating me for work.

3. **What was the most important design decision?**  
   Making the AI Assistant rely only on portfolio information, so it supports trust instead of risking invented claims.

4. **Where did I use AI, and why there?**  
   In the "Ask me about my work" assistant, because that is where visitors face the most friction when trying to match their question to my work.

5. **What changed between the first and final version?**  
   [Fill in]

6. **What did I learn about building applications with AI?**  
   [Fill in]

---

## 🔮 Future Improvements

If I continued developing this project, I would:

- Add detailed pages for every project with the problem, solution, my role, tools, and result.
- Add real dates to the experience timeline.
- Let the assistant suggest the most relevant project for a visitor's stated interest.
- Add more languages or a downloadable CV.

---

## 🔗 Project Links

**Live Demo:** https://pixel-perfect-code-158.lovable.app/#home

**Workflow:** [Add link, if any]

**Repository:** [Add GitHub repository link]

---

## 👩‍💻 Author

**Nour Abu Daia**  
Full Stack Developer | Workflow Automation | AI Engineering Student, Gaza, Palestine

[LinkedIn](https://www.linkedin.com/in/nour-abudaia-840b4b287) · [GitHub](https://github.com/nour-daia)
