---
name: gtm-ai-gtm
description: Go-to-market strategy for AI products. Use when positioning AI products, handling "who is responsible when it breaks" objections, pricing variable-cost AI, choosing between copilot/agent/teammate framing, or selling autonomous tools into enterprises.
license: MIT
metadata:
  author: Smit Patel (https://linkedin.com/in/smitkpatel)
  source: https://github.com/beingsmit/technical-product-gtm
---

# AI Product GTM

Go-to-market strategy for AI products. These aren't generic AI principles — they're patterns from selling autonomous AI agents into enterprises where "autonomous" scared buyers and "teammate" converted them.

## When to Use

**Triggers:**
- "How do we position this AI product?"
- "Buyers say they're worried about AI breaking production"
- "Should we call it autonomous or copilot?"
- "How do we price AI when usage varies 10x by customer?"
- "Enterprise security passed but ops rejected us — why?"

**Context:**
- AI agent platforms (coding, support, ops)
- LLM-based applications
- Autonomous tools that *do* things (not just suggest)
- AI infrastructure
- Anything where the AI makes decisions

---

## Core Frameworks

### 1. The Real Enterprise AI Objection (It's Not What You Think)

**What I Learned Selling Autonomous AI Agents:**

Three months in, enterprise security reviews were passing fast. Good sign, right? Then the pattern emerged: security approved, but **operations rejected us**.

The objection wasn't "will the AI break production?" — they *assumed* it would break production eventually. The real question was:

**"Who's responsible when the agent does something wrong?"**

Not "do we trust the agent?" — "do we trust our *team* to handle this?"

**Why This Matters:**

Autonomous agents create a new operational burden. You're not selling AI capability, you're selling organizational readiness. When your agent halts production at 2am, who gets paged? Who fixes it? Who explains it to the VP?

**Framework: The Accountability Cascade**

Before deploying AI agents, enterprises need clear answers:

1. **L1 Response**: Who monitors the agent? (24/7 ops team, or dev team on-call?)
2. **L2 Escalation**: When agent action fails, who debugs? (Agent team, or product team?)
3. **L3 Ownership**: When something breaks badly, who owns customer communication?

If you can't answer all three, **they won't buy**. Doesn't matter how good your AI is.

**How This Changes Your Sales Process:**

**Old approach:**
- Demo the AI
- Show accuracy metrics
- Talk about ROI

**New approach:**
- Demo the AI
- Show the *failure modes* explicitly
- Ask: "Who on your team would handle this scenario?"
- Walk through their incident response process
- Map AI failures to their existing runbooks

**The Qualification Question:**

"Walk me through what happens when the agent takes an action that breaks a workflow. Who gets alerted? Who investigates? Who decides whether to roll back or fix forward?"

If they can't answer, they're not ready. Pause the deal and help them build the process first.

**Common Mistake:**

Treating this as a *product* objection ("we'll make the AI more accurate"). It's an *organizational* objection. More accuracy doesn't solve "who owns this at 2am?"

**Pattern I've Seen Work:**

Companies that succeed with AI agents already have:
- On-call rotations for production systems
- Incident response playbooks
- Blameless postmortem culture
- Clear escalation paths

Companies that struggle:
- Manual deployment processes
- Hero culture ("Steve fixes everything")
- No formal incident response
- Blame-focused culture

**Decision Criteria:**

Before demoing autonomous AI to enterprises, ask yourself: "If this breaks their production, who on *their* team owns the fix?" If you can't answer, they can't buy.

---

### 2. Copilot vs Agent vs Teammate (Three Different GTM Motions)

**The Positioning Trap:**

Early enterprise conversations, we positioned as "autonomous AI agent." Buyers flinched. One word change — "autonomous" → "AI teammate" — and deal progression improved measurably.

Why? **Word choice shapes buyer psychology.**

**The Three Framings:**

**1. Copilot (Safest, Lowest Value)**
- **What it means**: AI suggests, human decides every time
- **Buyer psychology**: Feels safe, non-threatening
- **GTM motion**: Developer adoption, bottoms-up
- **Use case**: Code completion, writing assistance, search
- **Objection**: "Is this worth paying for?" (low perceived value)

**2. Agent (Scariest, Highest Value)**
- **What it means**: AI acts autonomously, human reviews periodically
- **Buyer psychology**: Scary, implies replacing humans
- **GTM motion**: Enterprise sales, top-down
- **Use case**: Batch processing, automated workflows, ops
- **Objection**: "What if it breaks production?" (accountability fear)

**3. Teammate (Sweet Spot)**
- **What it means**: AI and human collaborate, split the work
- **Buyer psychology**: Partnership, not replacement
- **GTM motion**: Hybrid (dev adoption + manager approval)
- **Use case**: Most AI agent platforms
- **Objection**: "How do we integrate this into our workflow?" (process question)

**The Positioning Shift:**

**Before:** "Autonomous AI agent that handles complex workflows end-to-end"
- Developers: "Cool, but scary"
- Managers: "Will this replace our team?"
- Deal progression: Slow

**After:** "AI teammate that pairs with your engineers on complex tasks"
- Developers: "This helps me"
- Managers: "This makes my team more productive"
- Deal progression: Three enterprise deals that had stalled 4+ months closed within 8 weeks of the shift

**Specific Language Choices That Mattered:**

❌ **Don't say:**
- "Autonomous" (scary)
- "Replaces" (threatening)
- "Fully automated" (no control)
- "AI-first" (what does that even mean?)

✅ **Do say:**
- "Teammate" (collaborative)
- "Augments" or "Accelerates" (helping, not replacing)
- "You stay in control" (reassuring)
- "Handles the repetitive work" (specific value)

**How to Choose Your Framing:**

```
Does your AI make decisions without human approval?
├─ Yes → Are you selling to developers or enterprises?
│   ├─ Developers → "Agent" framing (they want autonomous)
│   └─ Enterprises → "Teammate" framing (they want control)
└─ No → "Copilot" framing (augmentation, not automation)
```

**The Hard Truth:**

You can build an agent but position it as a copilot. You can't build a copilot and position it as an agent. **Product capabilities set a ceiling, positioning chooses where you land below it.**

**Common Mistake:**

Using "autonomous" because it sounds impressive. Impressive ≠ trusted. If buyers flinch at your positioning, you've lost them.

---

### 3. The AI Pricing Problem (When Usage Varies 10x)

**The Pattern:**

Every AI company I've worked with faces this: Customer A uses 1,000 API calls/month. Customer B uses 10,000. Do you charge Customer B 10x more? If yes, they churn. If no, your margins collapse.

**The Three Models:**

**1. Seat-Based ($X per user/month)**
- **When it works**: AI augments human work predictably
- **Example**: Code completion, writing assistant
- **Problem**: Doesn't capture AI value scaling
- **Real risk**: High-usage customers are your best customers, but they subsidize low-usage ones

**2. Usage-Based ($X per API call / prediction / hour)**
- **When it works**: AI does variable work, customers understand the unit
- **Example**: Image generation, transcription, batch ML
- **Problem**: Sticker shock for high-usage customers
- **Real risk**: Customers optimize to use *less* of your product

**3. Outcome-Based ($X per outcome achieved)**
- **When it works**: You can measure outcomes reliably
- **Example**: "Pay per bug fixed" or "Pay per support ticket resolved"
- **Problem**: Hard to measure, easy to game
- **Real risk**: You bear all the risk if AI doesn't perform

**What Actually Works (Hybrid):**

Base fee (covers fixed costs) + variable fee (scales with value).

**Example structure:**
- Base: $X/month per team (access to platform)
- Variable: $Y per successful action/outcome
- Why this works:
  - Base covers infra/support costs
  - Variable aligns with customer value
  - High-usage customers aren't punished (they're getting more value)

**The Pricing Conversation I Wish I'd Had Earlier:**

When pricing usage-based AI:

**Ask the customer:**
"How much would it cost you to do this manually?"

If it's $0.10 per API call but saves them $2 in labor, you're underpriced. If it costs $0.50 per call but saves them $0.40, they won't use it enough to matter.

**Pricing Rule:**

Your variable cost should be **20-30% of customer's alternative cost**. High enough to capture value, low enough that they'll use it liberally.

**Common Mistake:**

Copying OpenAI's pricing ($0.01 per 1K tokens) because "that's what everyone does." Your cost structure isn't OpenAI's cost structure. Your value isn't OpenAI's value. Price for *your* business.

---

### 4. The AI Trust Ladder (From Someone Who Climbed It)

**The Pattern:**

You can't sell AI by saying "trust us, it works." You build trust in stages.

**First: Transparency (Before First Demo)**

Send these three docs before they ask:
- Model card (what model, trained on what, accuracy on what benchmarks)
- Security whitepaper (where data goes, how it's processed)
- Explainability doc (how to interpret AI decisions)

**Why this works:**
Buyers expect to do diligence. If you send docs *before they ask*, you look confident and credible.

**Second: Control (In the Demo)**

Show them the safety mechanisms:
- How users approve/reject AI suggestions
- Kill switches and rollback mechanisms
- Confidence scores and when AI says "I'm not sure"

**Why this works:**
Fear of "runaway AI" is real. Showing control mechanisms proves you thought about failure modes.

**Third: Performance (Week 4-8)**

Prove it works:
- Benchmarks vs baseline (human or previous tool)
- Case study from similar company
- Live demo on their data (if possible)

**Why this works:**
Proof beats promises. One customer saying "we saved X hours/week" is worth 100 marketing claims.

**Fourth: Scale (When They're Serious)**

Show enterprise readiness:
- Enterprise deployment examples
- Performance at scale (latency, throughput, error rates)
- Compliance docs (SOC 2, GDPR, etc.)

**Why this works:**
Enterprises don't deploy MVPs. They need proof you won't fall over at 1000 users.

**The Mistake I Made:**

Trying to prove performance before explaining how the AI worked. Buyers didn't trust the benchmarks because they didn't understand the system. **Order matters.**

**Decision Criteria:**

If buyers ask "how does this work?" before you've demoed, you skipped transparency. Back up and send the docs.

---

### Frameworks 5–8: Demo, Ownership, Positioning, Qualification

The enterprise AI demo, the "who owns this?" objection handler, the positioning trap, and ceiling-moment qualification. See [references/frameworks-5-8.md](references/frameworks-5-8.md) for:

- 5. The Enterprise AI Demo (Show Failure, Not Just Success)
- 6. The "Who Owns This?" Objection Handler
- 7. The AI Positioning Trap (Fighting Asymmetric Wars)
- 8. Ceiling Moment Qualification (Finding High-Intent AI Buyers)

## Decision Trees

### Which Positioning Should I Use?

```
Does your AI act autonomously (no approval per action)?
├─ Yes → Who are you selling to?
│   ├─ Developers → "Agent" framing
│   └─ Enterprises → "Teammate" framing
└─ No → "Copilot" framing
```

### Which Pricing Model Should I Use?

```
Can you measure customer outcomes reliably?
├─ Yes → Outcome-based (or hybrid with outcome component)
└─ No → Continue...
    │
    Does usage vary 5x+ by customer?
    ├─ Yes → Hybrid (base + usage)
    └─ No → Seat-based
```

### Is This Buyer Ready for AI Agents?

```
Do they have incident response processes for tool failures?
├─ Yes → Continue...
│   │
│   Do they have on-call rotations for production systems?
│   ├─ Yes → Qualified buyer
│   └─ No → Help them build it first
└─ No → Not ready (come back in 6 months)
```

---

## Common Mistakes

**1. Using "autonomous" because it sounds impressive**
   - I've watched this slow deals. "Autonomous" scares enterprises. "Teammate" progresses faster.

**2. Hiding AI failure modes**
   - Buyers know real-world data is messy. If you don't show failures, they assume you're hiding them.

**3. Treating "will it break production?" as the objection**
   - Real objection: "who's responsible when it does?" Organizational readiness, not accuracy.

**4. Pricing usage-based AI like OpenAI**
   - Your cost structure isn't theirs. Price for 20-30% of customer's alternative cost.

**5. Skipping transparency docs before demo**
   - Order matters. Transparency → Control → Performance → Scale. Don't skip steps.

**6. Demoing perfect AI**
   - Show mistakes + recovery. Builds more trust than fake perfection.

**7. Selling to buyers who demand 100% accuracy**
   - They're not ready. Filter for mature buyers with incident response processes.

---

## Quick Reference

**Enterprise objection checklist:**
- [ ] "Who gets paged when AI breaks production?" → Map to their on-call rotation
- [ ] "Who debugs AI failures?" → Map to their incident response
- [ ] "Who owns customer communication?" → Map to their escalation path

**Positioning word choices:**
- ✅ Teammate, augments, accelerates, you stay in control
- ❌ Autonomous, replaces, fully automated, AI-first

**Demo structure:**
1. Problem with quantified cost (30s)
2. AI attempt including failure/uncertainty (60s)
3. Human review and override (30s)
4. Outcome with ROI (30s)

**Trust ladder:**
1. Transparency (model card, security, explainability)
2. Control (approval workflows, kill switches, confidence scores)
3. Performance (benchmarks, case studies, live demo)
4. Scale (enterprise deployments, compliance, SLAs)

**Pricing hybrid formula:**
- Base: $X/month (covers fixed costs)
- Variable: $Y per unit (20-30% of customer's alternative cost)

---

## Related Skills

- **positioning-strategy**: General positioning frameworks and testing
- **technical-product-pricing**: Pricing models including AI-specific patterns
- **enterprise-account-planning**: Enterprise AI deal management

---

*Based on enterprise AI agent GTM across developer tools and infrastructure. Patterns drawn from working enterprise deal cycles selling autonomous AI products — some carried directly, others supported alongside sales leadership — including the positioning trap diagnosis that shifted from feature competition to structural differentiation, the ceiling-moment qualification that improved outbound conversion significantly, and frameworks tested across security, operations, and engineering buyer personas. Not theory — lessons from deals where "autonomous" killed conversations and "teammate" converted.*
