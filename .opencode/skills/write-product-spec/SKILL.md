---
name: write-product-spec
description: Transforms a rough feature idea into a structured product specification with user stories, acceptance criteria, edge cases, and technical considerations. Produces a document ready for engineering handoff. Use when a user describes a feature idea and needs a formal spec.
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# Write a Product Spec

Gather the user's feature idea, context, and constraints, then produce a structured specification document that an engineering team can build from without ambiguity.

## Step 1 — Extract the Core Problem

Ask yourself these questions about what the user provided. If any are unanswerable, ask the user before proceeding:

- What specific user problem does this solve?
- Who experiences this problem? (role, segment, frequency)
- What do users do today without this feature? (workaround or nothing)
- What is the business motivation? (retention, revenue, activation, cost reduction)

Write the problem statement as: "[User segment] cannot [action] because [obstacle], resulting in [negative outcome]."

## Step 2 — Define Users and Personas

Identify every distinct user role that interacts with this feature. For each:

- Role name and description
- Primary goal when using this feature
- Technical sophistication level
- Frequency of use (daily, weekly, one-time)

Common missed personas: admin configuring the feature, support staff troubleshooting it, API consumer integrating with it, user who receives the output (e.g., the invitee, not just the inviter).

## Step 3 — Write User Stories

Format: "As a [role], I want [action], so that [outcome]."

Decision tree for granularity:
- If a story takes more than 5 days to build, split it
- If a story has more than 3 acceptance criteria, consider splitting
- If two roles do the same action for different reasons, write separate stories

Group stories by workflow phase (setup, core usage, edge handling, administration).

## Step 4 — Write Acceptance Criteria

For each user story, write acceptance criteria in Given/When/Then format:

```
Given [precondition]
When [action]
Then [observable result]
```

Quality checklist for each criterion:
- Is it testable by a QA engineer with no additional context?
- Does it specify exact behavior, not vague outcomes ("shows feedback" is bad; "displays green success banner for 5 seconds" is good)?
- Does it cover the negative case? (What happens when the action fails?)
- Does it specify data validation rules where applicable?

## Step 5 — Discover Edge Cases

Systematically apply this edge case matrix to every user story:

| Category | Questions to ask |
|---|---|
| Empty/No data | What if the user has zero items? First-time use? |
| Boundaries | What at 0, 1, max, max+1? Character limits? |
| Concurrency | Two users editing the same thing simultaneously? |
| Offline/Network | Slow connection? Connection lost mid-action? |
| Permissions | User's role changes mid-session? Token expires? |
| Duplicate action | User double-clicks submit? Refreshes during processing? |
| Scale | 10x current usage? 100x data volume? |
| Undo/Recovery | Can the user reverse this action? What if they navigate away mid-flow? |
| Integrations | External service is down? Returns unexpected format? |

For each edge case found, add it as an acceptance criterion on the relevant story or as a dedicated story if complex enough.

## Step 6 — Define Anti-Requirements

Explicitly state what this feature will NOT do. This prevents scope creep and misaligned expectations. Format:

- "This feature will NOT [capability]. Rationale: [why]."

Common anti-requirements to consider: bulk operations, API access, import/export, real-time sync, mobile-specific layouts, internationalization, admin override.

## Step 7 — Add Technical Considerations

For each area, note only what constrains the implementation or requires cross-team coordination:

- **Data model changes**: New entities, modified schemas, migrations needed
- **API surface**: New endpoints, changed contracts, versioning implications
- **Dependencies**: External services, libraries, infrastructure
- **Performance**: Expected load, latency requirements, caching needs
- **Security**: Auth changes, new permissions, data sensitivity
- **Migration**: Existing user data, backward compatibility, rollout strategy

Do NOT design the implementation. Flag decisions, not solutions.

## Step 8 — Assign Priority and Effort Signals

Tag each user story:

**Priority:**
- P0: Launch blocker. Feature is broken or unusable without this.
- P1: Must-have for initial release. Core value depends on it.
- P2: Should-have. Improves experience significantly but core works without it.
- P3: Nice-to-have. Polish, optimization, secondary workflows.

**Effort signal** (t-shirt size): S (< 1 day), M (1-3 days), L (3-5 days), XL (5+ days, should be split).

## Step 9 — List Open Questions and Metrics

**Open questions**: Anything that requires a decision from a stakeholder, designer, or other team member. Format as: "Q: [question] — Owner: [role] — Blocks: [which stories]."

**Success metrics**: 1-3 measurable outcomes that indicate this feature achieved its goal. Tie metrics to the problem statement from Step 1. Include both leading indicators (adoption, usage frequency) and lagging indicators (retention impact, revenue impact).

## Step 10 — Assemble and Output

Produce the spec as a single markdown document with these sections in order:

1. Overview (problem, users, solution summary — 3-5 sentences)
2. Users and Personas
3. User Stories (grouped by workflow phase, with acceptance criteria inline)
4. Edge Cases (those not already captured in acceptance criteria)
5. Anti-Requirements
6. Technical Considerations
7. Priority and Effort Summary (table format)
8. Open Questions
9. Success Metrics

Write to a file named after the feature (e.g., `team-invitations-spec.md`).
