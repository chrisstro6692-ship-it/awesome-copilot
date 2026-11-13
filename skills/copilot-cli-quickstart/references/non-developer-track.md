# Non-Developer Track Lessons

## 🎨 Non-Developer Track Lessons

### 📝 Lesson N1: Writing & Editing with Copilot

**Goal:** Use Copilot as your writing assistant ✍️

**Teach these concepts:**

1. **Copilot isn't just for code** — It's amazing at writing, editing, and organizing text. Think of it as a smart editor that lives in your terminal. 📝

2. **Writing tasks to try:**
   > 🟢 `"Write a project status update for my team"`
   > 🟢 `"Draft an email to schedule a meeting about the new feature"`
   > 🟢 `"Create a bullet-point summary of this document: @notes.md"`
   > 🟢 `"Proofread this text and suggest improvements: @draft.txt"`

3. **Creating documents:**
   > 🟢 `"Create a meeting-notes.md template with sections for attendees, agenda, decisions, and action items"`
   > 🟢 `"Write a FAQ document for our product based on @readme.md"`

4. **The `@` mention** — Point Copilot at a file to work with it:
   > `"Summarize @meeting-notes.md into three key takeaways"`

**Exercise:**
```
Use ask_user:
"🏋️ Try this:

   'Create a file called meeting-notes.md with a template for taking meeting notes. Include sections for date, attendees, agenda items, decisions, and action items.'

How does the template look?"
choices: ["✅ Great template! I'd actually use this!", "✏️ I want to customize it", "🤔 I want to try something different"]
```

---

### 📋 Lesson N2: Task Planning with /plan

**Goal:** Use /plan to break down projects and tasks — no coding needed! 📋

**Teach these concepts:**

1. **What is /plan?** — It's like asking a smart assistant to create a project plan for you. You describe what you want, and Copilot breaks it into clear steps. 📊

2. **Non-code examples:**
   > 🟢 `/plan Organize a team offsite for 20 people in March`
   > 🟢 `/plan Create a content calendar for Q2 social media`
   > 🟢 `/plan Write a product requirements doc for a new login feature`
   > 🟢 `/plan Prepare a presentation about our Q1 results`

3. **How to use it:**
   > - Type `/plan` followed by your request
   > - Copilot creates a structured plan with steps
   > - Review it, edit it, then ask Copilot to help with each step!

4. **Editing the plan** — The plan is just a file. You can modify it and Copilot will follow your changes.

**Exercise:**
```
Use ask_user:
"🏋️ Try this:

   /plan Create a 5-day onboarding checklist for a new team member joining our marketing department

Did Copilot create a useful plan?"
choices: ["📋 This is actually really useful!", "✏️ It's close but I'd change some things", "🤔 I want to try a different topic"]
```

---

### 🔍 Lesson N3: Understanding Code (Without Writing It)

**Goal:** Read and understand code without being a programmer 🕵️

**Teach these concepts:**

1. **You don't need to write code to understand it** — Copilot can translate code into plain English. This is huge for PMs, designers, and anyone who works with engineers! 🤝

2. **Magic prompts for non-developers:**
   > 🟢 `"Explain @src/app.js like I'm not a developer"`
   > 🟢 `"What does this project do? Look at @README.md and @package.json"`
   > 🟢 `"What would change for users if we modified @login.py?"`
   > 🟢 `"Is there anything in @config.yml that a PM should know about?"`

3. **Code review for non-devs:**
   > 🟢 `"Summarize the recent changes — /diff"`
   > 🟢 `"What user-facing changes were made? Explain without technical jargon."`

4. **Architecture questions:**
   > 🟢 `"Draw me a simple map of how the files in this project connect"`
   > 🟢 `"What are the main features of this application?"`

**Exercise:**
```
Use ask_user:
"🏋️ Navigate to any project folder and try:

   'Explain what this project does in simple, non-technical terms'

Was the explanation clear?"
choices: ["✅ Crystal clear! Now I get it!", "🤔 It was still a bit technical", "🤷 I don't have a project to look at"]
```

If too technical: "Try adding 'explain it like I'm a product manager' to your prompt!"
If no project: suggest cloning a simple open source repo to explore.

---

### 📊 Lesson N4: Getting Summaries & Explanations

**Goal:** Turn Copilot into your personal research assistant 🔬

**Teach these concepts:**

1. **Copilot reads files so you don't have to** — Point it at any document and ask for a summary, key points, or specific information. 📚

2. **Summary prompts:**
   > 🟢 `"Give me the top 5 takeaways from @report.md"`
   > 🟢 `"What are the action items in @meeting-notes.md?"`
   > 🟢 `"Create a one-paragraph executive summary of @proposal.md"`

3. **Comparison prompts:**
   > 🟢 `"Compare @v1-spec.md and @v2-spec.md — what changed?"`
   > 🟢 `"What's different between these two approaches?"`

4. **Extraction prompts:**
   > 🟢 `"List all the dates and deadlines mentioned in @project-plan.md"`
   > 🟢 `"Pull out all the stakeholder names from @kickoff-notes.md"`
   > 🟢 `"What questions are still unanswered in @requirements.md?"`

**Exercise:**
```
Use ask_user:
"🏋️ Create a test document and try it out:

   'Create a file called test-doc.md with a fake project proposal. Then summarize it in 3 bullet points.'

Did Copilot give you a good summary?"
choices: ["✅ Great summary!", "🤔 I want to try with my own files", "📝 Show me more examples"]
```

---
