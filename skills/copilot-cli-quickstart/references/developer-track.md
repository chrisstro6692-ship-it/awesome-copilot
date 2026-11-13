# Developer Track Lessons

## 🧑‍💻 Developer Track Lessons

### 🎛️ Lesson D1: Slash Commands & Modes

**Goal:** Discover the superpowers hidden behind `/` and `Shift+Tab` 🦸‍♂️

**Teach these concepts:**

1. **Slash commands** — Type `/` and a menu appears! These are your power tools:
   > | Command | What it does | |
   > |---------|-------------|---|
   > | `/help` | Shows all available commands | 📚 |
   > | `/clear` | Fresh start — clears conversation | 🧹 |
   > | `/model` | Switch between AI models | 🧠 |
   > | `/diff` | See what Copilot changed | 🔍 |
   > | `/plan` | Create an implementation plan | 📋 |
   > | `/compact` | Shrink conversation to save context | 📦 |
   > | `/context` | See context window usage | 📊 |

2. **Three modes** — Press `Shift+Tab` to cycle:
   > 🟢 **Interactive** (default) — Copilot asks before every action
   > 📋 **Plan** — Copilot creates a plan first, then you approve
   > 💻 **Shell** — Quick shell command mode. Type `!` to jump here instantly! ⚡

3. **The `!` shortcut** — Type `!` at the start to jump to shell mode. `!ls`, `!git status`, `!npm test` — lightning fast! ⚡

**Exercise:**
```
Use ask_user:
"🏋️ Try these in Copilot CLI:
1. Type /help to see all commands
2. Press Shift+Tab to cycle through modes
3. Type !ls to run a quick shell command

Which one surprised you the most?"
choices: ["😮 So many slash commands!", "🔄 The modes — plan mode is cool!", "⚡ The ! shortcut is genius!", "🤯 All of it!"]
```

---

### 📎 Lesson D2: Mentioning Files with @

**Goal:** Point Copilot at specific files for laser-focused help 🎯

**Teach these concepts:**

1. **The `@` symbol** — Type `@` and start typing a filename. Copilot autocompletes! This puts a file front and center in context. 📂

2. **Why it matters** — It's like highlighting a page in a textbook before asking a question. 📖✨

3. **Examples:**
   > 💡 `"Explain what @package.json does"`
   > 💡 `"Find bugs in @src/app.js"`
   > 💡 `"Write tests for @utils.ts"`

4. **Multiple files:**
   > `"Compare @old.js and @new.js — what changed?"`

**Exercise:**
```
Use ask_user:
"🏋️ Navigate to a project folder and try:

   'Explain what @README.md says about this project'

Did Copilot nail it?"
choices: ["✅ Perfect explanation!", "🤷 I don't have a project handy", "❌ Something didn't work"]
```

If no project folder: suggest `mkdir ~/copilot-playground && cd ~/copilot-playground` and have Copilot create files first!

---

### 📋 Lesson D3: Planning with /plan

**Goal:** Break big tasks into steps before coding 🏗️

**Teach these concepts:**

1. **Plan mode** — Ask Copilot to think before coding. It creates a structured plan with todos. Like blueprints before building! 🏛️

2. **How to use it:**
   > - Type `/plan` followed by what you want
   > - Or `Shift+Tab` to switch to plan mode
   > - Copilot creates a plan file and tracks todos

3. **Example:**
   > ```
   > /plan Build a simple Express.js API with GET /health and POST /echo
   > ```

4. **Why plan first?** 🤔 — Catches misunderstandings before code, you can edit the plan, and you stay in control of architecture.

**Exercise:**
```
Use ask_user:
"🏋️ Try:

   /plan Create a simple calculator that adds, subtracts, multiplies, and divides

Read the plan. Does it look reasonable?"
choices: ["📋 The plan looks great!", "✏️ I want to edit it — how?", "🤔 Not sure what to do with the plan"]
```

---

### ⚙️ Lesson D4: Custom Instructions

**Goal:** Teach Copilot YOUR preferences 🎨

**Teach these concepts:**

1. **Instruction files** — Special markdown files that tell Copilot your coding style. It reads them automatically! 📜

2. **Where to put them:**
   > | File | Scope | Use for |
   > |------|-------|---------|
   > | `AGENTS.md` | Per directory | Agent-specific rules |
   > | `.github/copilot-instructions.md` | Per repo | Project-wide standards |
   > | `~/.copilot/copilot-instructions.md` | Global | Personal preferences everywhere |
   > | `.github/instructions/*.instructions.md` | Per repo | Topic-specific rules |

3. **Example content:**
   > ```markdown
   > # My Preferences
   > - Always use TypeScript, never plain JavaScript
   > - Prefer functional components in React
   > - Add error handling to every async function
   > ```

4. **`/init`** — Run in any repo to scaffold instruction files. 🪄
5. **`/instructions`** — See active instruction files and toggle them. 👀

**Exercise:**
```
Use ask_user:
"🏋️ Let's personalize! Try:

   /init

Did Copilot help set up instruction files for your project?"
choices: ["✅ It created instruction files! 🎉", "🤔 Not sure what happened", "📝 I need help"]
```

---

### 🚀 Lesson D5: Advanced — MCP, Skills & Beyond

**Goal:** Unlock the full power of Copilot CLI 🔓

**Teach these concepts:**

1. **MCP servers** — Extend Copilot with external tools and data sources:
   > - `/mcp` — manage MCP server connections
   > - Think of MCP as "plugins" for Copilot — databases, APIs, custom tools
   > - Example: connect a Postgres MCP server so Copilot can query your database! 🗄️

2. **Skills** — Custom behaviors you can add (like this tutor!):
   > - `/skills list` — see installed skills
   > - `/skills add owner/repo` — install a skill from GitHub
   > - Skills teach Copilot new tricks! 🎪

3. **Session management:**
   > - `/resume` — switch between sessions
   > - `/share` — export a session as markdown or a gist
   > - `/compact` — compress conversation when context gets full

4. **Model selection:**
   > - `/model` — switch between Claude Sonnet, GPT-5, and more
   > - Different models have different strengths!

**Exercise:**
```
Use ask_user:
"🏋️ Try:

   /model

What models are available to you?"
choices: ["🧠 I see several models!", "🤔 Not sure which to pick", "❓ What's the difference between them?"]
```

---
