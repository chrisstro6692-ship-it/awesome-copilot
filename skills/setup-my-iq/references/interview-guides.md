# 4–9. Interview for each context file

### 4. Interview for identity.md

Load the template from `assets/templates/identity.md` in this skill bundle for
the file structure.

**First: apply step 3 (pre-fill).** Query available sources for name, role,
org, team, and manager. Present any findings for confirmation. Then proceed
to the questions below for fields not covered by pre-fill.

Open-ended questions to ask (these always need the user):

- If you had to explain what you actually do to someone at a dinner party, what would you say?
- What do people come to you for?

Factual questions to ask only if pre-fill didn't cover them:

- What's your name and current role?
- What organization or company are you with?
- What team are you on?
- Who's your manager?

Once you have enough to draft (don't keep asking past that point), draft the
file. Present it with the reaction pass (see Interview Rules below). Revise,
then write to the context directory.

Transition: "That's identity done. Next is role-and-responsibilities, which
captures what your weeks actually look like. Ready?"

### 5. Interview for role-and-responsibilities.md

Load the template from `assets/templates/role-and-responsibilities.md`.

**First: apply step 3 (pre-fill).** Query available sources for teams
supported, recurring calendar cadences, and reporting line. Present any
findings for confirmation before asking questions.

Key questions:

- Walk me through a typical week. What are the recurring things?
- What teams or groups do you support?
- For each team: what are you accountable for? What cadences do you run?
- What decisions do you make regularly? Not big strategic ones, the routine ones.
- What do you produce? Reports, emails, dashboards?
- Who do you report to?
- Are there monthly or quarterly rhythms that shape your work?

Once you have enough to draft, draft. Organize by team: each team gets its
own section with responsibilities, cadences, and deliverables. Present,
revise, write.

Transition: "That's role-and-responsibilities done. Next is team, which
covers the key people in your work life. Ready?"

### 6. Interview for team.md

Load the template from `assets/templates/team.md`.

**First: apply step 3 (pre-fill).** Query available sources for direct
reports, org chart, frequent collaborators (from calendar/chat). Present any
findings for confirmation before asking questions.

Key approach:

- Who are the people you interact with most? Start with your own team.
- Ask for team rosters (the user may paste email lists)
- For each team, identify the lead and hierarchy
- For key people: what do they need from you, what do you need from them?
- Capture focus areas or pillars per person where known
- Note interaction patterns (1:1s, scrums, syncs)
- Is there anything an agent should know about working with or preparing for
  interactions with specific people?

Use information from earlier files. If they mentioned collaborators during the
roles interview, reference them here rather than re-asking. Draft, present,
revise, write.

Transition: "That's team done. Last one: tools, systems, and configuration.
This is where we capture how everything is set up. Ready?"

### 7. Interview for tools-systems-and-config.md

Load the template from `assets/templates/tools-systems-and-config.md`.

**First: apply step 3 (pre-fill).** Query available sources for ADO
org/project/team/area path, repos, and tool configuration. Present any
findings for confirmation before asking questions.

Key questions:

- What tools and platforms do you use daily?
- How do you organize meeting notes? Where do they live?
- What tags or patterns classify your meetings by type and by team?

Then for each workstream/team the user supports (use information from
the role-and-responsibilities interview):

If the user supports more than three workstreams, ask first whether they
want to configure all of them now or start with the most important one or
two and add the rest later. For similar workstreams, ask if the same config
applies and only interview for differences.

- Does this workstream use Azure DevOps? If yes: what org, project, team,
  area path? If no, skip the ADO section for this workstream.
- (ADO workstreams only) Are there work item title patterns to exclude from
  sprint scorecards? (e.g., estimation or review placeholder items)
- What are the strategic pillars or sections for this workstream? These are
  used as grouping headers in weekly updates and sprint reports. Define them
  even if the workstream doesn't use ADO.
- For ADO workstreams: which ADO Epics map to each pillar?
- Where do sprint updates get saved? What file prefix?
- Where do weekly updates get saved? What file prefix?
- Is there a roadmap file for this workstream?

Organize with common settings at top, workstream-specific config in
sections below. Draft, present, revise, write.

Transition: "That's tools and config done. Next is communication style, so
any agent writing on your behalf can sound like you. Ready?"

### 8. Interview for communication-style.md

Load the template from `assets/templates/communication-style.md`.

This section is harder than the others. People describe their own voice in
clichés. Two stronger paths: ask about negatives, and offer to analyze real
writing samples. Use both.

Start by offering the samples path:

> "Want to share a few samples of your real writing? Paste 2 to 5 things
> you've sent: weekly status updates, leadership emails, Loop updates, a
> review entry, a Teams message you're proud of. I'll read them, extract
> the voice traits I see, and propose what should go in your file. You
> approve before anything gets written. I won't copy the raw content,
> only the style patterns."

If the user shares samples:

- Read each sample for style only: register, sentence length, openings,
  transitions, specific words used or avoided, framing patterns, how they
  handle credit, how they handle bad news.
- Do not copy raw sample text into the file. Extract patterns and present
  them back as proposed bullets in the file's structure.
- Present the proposed draft: "Here's what I extracted. Tell me what's wrong
  or missing."
- Apply the reaction pass as normal.

If the user doesn't share samples (or in addition to them), ask these
questions one at a time:

1. "If a colleague described how you write in one sentence, what would they
   say?"
2. "Read this sentence: *'I'd encourage you to lean into the opportunity to
   be a genuine force multiplier for the team.'* What's wrong with it for
   your voice?" (Surfaces the avoid-list faster than positive questions.)
3. "When you write a weekly status to leadership, what do you wish you could
   automate, and what do you always end up rewriting?"
4. "Who's the hardest audience for you to write to? What changes when you
   write to them?"
5. "When you write about yourself in a Connect or performance review,
   what's a phrase you've caught yourself writing and immediately deleted?"

Draft, present, revise, write.

After writing, tell the user:

> "I won't automatically pick up on patterns over time. If you notice me
> repeating a mistake, swapping phrases, or fighting your edits, ask me to
> update this file and I'll fold the correction in. This file evolves when
> you tell it to."

Transition: "Last one: preferences and constraints. This is how you want AI
to behave around you. Ready?"

### 9. Interview for preferences-and-constraints.md

Load the template from `assets/templates/preferences-and-constraints.md`.

Most rules here are about what an agent shouldn't do unilaterally. Surface
them by recalling concrete frustrations rather than asking for abstract
rules. Ask one question at a time:

1. "When was the last time an AI tool did something you didn't want? What
   did it do, and what should it have done instead?" (One answer often
   produces 3 boundary rules.)
2. "What's safe for an agent to do without asking you? What absolutely
   requires your approval?"
3. "Speed or thoroughness. If you had to pick one, which?"
4. "Are there any decisions you want AI to make for you, or do you always
   want options surfaced?"
5. "Where do you not want an AI speaking on your behalf?" (Teams, email,
   meetings, public chats.)
6. "Do you use any tool with persistent memory (VS Code session memory,
   an agent memory feature, etc.)? If yes, anything you want stored there,
   or specifically not stored there?" Skip the memory section if the
   answer is no.

Draft, present, revise, write. Apply the reaction pass.

Transition: "That's the interview done. Now I'll wire the files into your
custom instructions so every agent can find them."
