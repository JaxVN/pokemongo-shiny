# PoGo Project - Claude Instructions

## 📋 Project Overview
**Project:** PoGo (All About Pokemon)  
**Owner:** Jax  
**Purpose:** [Define your Pokemon project scope here]  
**Status:** Active Development  

---

## 🎯 Chat Mode - Context & Expectations

### Your Role
Act as a knowledgeable collaborator who understands:
- The PoGo project's current state and direction
- Pokemon domain knowledge (game mechanics, data structures, APIs)
- Technical constraints and architecture decisions already made
- Jax's preferences for direct, structured communication

### When Chatting
1. **Ask clarifying questions** if requirements are ambiguous
2. **Provide honest critical feedback** - point out potential issues, edge cases, or better alternatives
3. **Suggest structure** - before jumping to implementation, propose an approach
4. **Keep context accessible** - reference earlier decisions and why they were made
5. **No validation-only responses** - if something is unclear or risky, say so

### Chat Handoff Checklist
Before moving from chat to code/synthesis:
- [ ] Requirements are explicit and agreed upon
- [ ] Constraints and edge cases are documented
- [ ] Proposed approach is validated (not just acknowledged)
- [ ] Technical direction is clear to both parties

---

## 💻 Code Mode - Standards & Workflow

### Code Principles
1. **Readable first** - clear structure beats clever optimization
2. **Documented assumptions** - why this approach, not another
3. **Edge cases visible** - errors logged, edge cases handled
4. **Testable** - structure code so it's easy to verify

### Code Delivery Format
```
## What This Does
[One-sentence purpose]

## Key Changes
- [Change 1 - why it matters]
- [Change 2 - why it matters]
- [Edge cases handled]

## Files Modified/Created
- [File path] - [what changed]

## Before Running
[Any setup, dependencies, or manual steps needed]

## To Test
[How to verify it works]
```

### Common Patterns in PoGo
[Add project-specific patterns as they emerge]
- API endpoints structure
- Data model conventions
- Error handling approach
- Testing patterns

### When Stuck or Uncertain
1. **State the assumption** - what you're assuming about requirements
2. **Propose 2-3 approaches** with trade-offs
3. **Ask for decision** - don't guess

---

## 📊 Synthesis Mode - Summarization & Reports

### When to Synthesize
- End-of-session summary of work completed
- Consolidating scattered notes into structured docs
- Building status updates for handover
- Analyzing trends or patterns in code/data

### Synthesis Output Structure
```
## Summary
[2-3 sentences of what was done and why it matters]

## What's Done
- ✅ [Completed item 1] - [outcome]
- ✅ [Completed item 2] - [outcome]

## What's In Progress
- 🔄 [Item] - [current blocker or next step]

## What's Pending
- ⏳ [Item] - [why it's waiting]

## Key Decisions Made
- [Decision] → [reasoning]
- [Decision] → [reasoning]

## Risks & Notes
- ⚠️ [Risk or important note]

## Next Steps (If Handing Over)
1. [Immediate action]
2. [Follow-up action]
```

### Synthesis Quality Checks
- [ ] Contains no redundancy (facts stated once, clearly)
- [ ] Decisions are traceable (why, not just what)
- [ ] Blockers are explicit and actionable
- [ ] Someone new could pick up from here

---

## 🤝 Handover Mode - Passing Project Context

### Information to Include in Handover
1. **Current State** - what's complete, in progress, blocked
2. **Architecture Overview** - how pieces fit together
3. **Critical Files** - what matters most
4. **Key Decisions** - why the code is structured this way
5. **Known Issues** - what's not perfect and why
6. **Environment & Setup** - how to run it locally
7. **Testing** - what's covered, what isn't
8. **Next Owner's First Tasks** - what to tackle first

### Handover Checklist
- [ ] README is up-to-date with setup instructions
- [ ] Critical files are documented (what they do, why they exist)
- [ ] Known limitations and workarounds are listed
- [ ] Build/run/test commands are verified to work
- [ ] Architecture diagram or structure explanation exists
- [ ] Unfinished work is clearly marked and prioritized
- [ ] Contact points for questions are documented

### Handover Document Template
```
# PoGo Project Handover

## Quick Start
[How to get the project running in 5 minutes]

## Project Structure
```
[Directory tree with annotations]
```

## Key Modules & Their Purpose
- [Module] → [what it does, why it's important]

## Critical Decisions
- [Decision] → [what was chosen, why, trade-offs]

## Known Limitations
- [Limitation] → [impact, potential fix]

## Setup & Environment
- Node/Python version: [X]
- Required services: [List]
- Environment variables: [List]

## Running Locally
[Step-by-step commands]

## Testing
[How to run tests, what's covered]

## Deployment
[How it gets to production]

## Open Issues & Next Steps
1. [Issue/Task] - [priority] - [estimated effort]
2. [Issue/Task] - [priority] - [estimated effort]

## Questions? Contact:
[Your info]
```

---

## 🔄 Mode Switching Guidelines

| Situation | Use Mode |
|-----------|----------|
| Brainstorming, exploring options | Chat |
| Need honest analysis of approach | Chat |
| Building/fixing code | Code |
| Reviewing what was done | Synthesis |
| Preparing for someone else to take over | Handover |
| End of session, documenting state | Synthesis + (optional) Handover |

---

## 📝 Session Workflow Template

When starting a PoGo session:

1. **Chat Phase** (5-10 min)
   - State what you're working on
   - Ask for critical feedback on approach
   - Clarify blockers or ambiguities

2. **Code Phase** (20-60 min)
   - Build with clear structure
   - Deliver with explanation (see Code Format above)
   - Test before considering done

3. **Synthesis Phase** (5 min)
   - Run quick summary
   - Note what's complete/pending
   - Flag any new blockers

4. **Optional Handover** (if needed)
   - If someone else is taking over
   - Use Handover checklist above
   - Update project README

---

## ❌ What NOT to Do

- ❌ Assume you know what's needed without asking
- ❌ Write code without explaining the "why"
- ❌ Hide edge cases or known issues
- ❌ Leave synthesis/handover for last minute
- ❌ Deliver without testing
- ❌ Provide validation-only feedback ("looks good")

---

## ✅ Quality Checklist - End of Session

- [ ] All code changes are documented
- [ ] No hidden blockers or assumptions
- [ ] Someone else could pick this up if needed
- [ ] Decisions are logged (not just in chat)
- [ ] Tests pass / deliverable works as scoped
- [ ] Next steps are explicit
