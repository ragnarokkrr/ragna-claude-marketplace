---
name: create-adr
description: Generate Architecture Decision Record (ADR) following standardized template
allowed-tools: ["Read", "Write", "Glob", "Bash"]
---

# Create ADR Skill

This skill generates a new Architecture Decision Record (ADR) file following a standardized template. ADRs document significant architecture decisions with their context, alternatives, and consequences.

## Purpose

Create well-structured ADR documents that capture:
- The architectural problem or decision point
- Context and constraints
- Decision made
- Alternatives considered
- Consequences (positive and negative)

## Instructions

When this skill is invoked:

1. **Find the next ADR number:**
   - Check `.spec/architecture/adrs/` for existing ADRs
   - Determine the next sequential 4-digit number (e.g., ADR-0001, ADR-0002)
   - If the directory doesn't exist, create it and start with ADR-0001

2. **Gather information:**
   - If title is provided, use it
   - Otherwise, ask the user for:
     - ADR title (concise, describes the decision)
     - Context (what problem are we solving?)
     - Decision (what did we decide?)
     - Alternatives considered (what other options existed?)
     - Consequences (what are the trade-offs?)

3. **Generate the ADR file:**
   - Create `.spec/architecture/adrs/adr-XXXX-kebab-case-title.md`
   - Use the template below
   - Set status to "Proposed" by default (can be Accepted, Deprecated, Superseded)

4. **Confirm creation:**
   - Display the file path
   - Show a preview of the content
   - Remind user to reference this ADR in section 9 of arc42 documentation

## ADR Template

```markdown
# ADR-XXXX: [Decision Title]

**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-YYYY

**Date:** YYYY-MM-DD

**Deciders:** [List key decision makers/stakeholders]

## Context

What is the issue we're addressing? What factors are driving this decision?
Include:
- Business/technical problem
- Constraints and requirements
- Assumptions
- Key stakeholders affected

## Decision

What did we decide? Be specific and actionable.
State clearly what will be done and what approach will be taken.

## Alternatives Considered

### Option 1: [Name]
- **Pros:** ...
- **Cons:** ...
- **Why rejected:** ...

### Option 2: [Name]
- **Pros:** ...
- **Cons:** ...
- **Why rejected:** ...

## Consequences

### Positive
- Benefit 1
- Benefit 2

### Negative
- Trade-off 1
- Risk/limitation 1

### Neutral
- Changes required
- Migration effort
- Team training needs

## Implementation Notes

- Key steps needed
- Dependencies
- Timeline considerations
- Validation/testing approach

## References

- Links to related ADRs
- External documentation
- Discussion threads
- RFC/design docs
```

## Usage Examples

**Example 1: Quick ADR creation**
```
User: Create an ADR for choosing PostgreSQL as our primary database
Skill: [Generates ADR-0003-postgresql-as-primary-database.md with template]
```

**Example 2: Detailed ADR with context**
```
User: We decided to use event sourcing for the order service. Create an ADR.
Skill: [Asks for alternatives considered and consequences, then generates complete ADR]
```

**Example 3: ADR for pattern adoption**
```
User: Document our decision to use hexagonal architecture
Skill: [Creates ADR explaining ports/adapters pattern choice with trade-offs]
```

## Best Practices

- **Keep titles concise:** "Use GraphQL for API" not "We decided to use GraphQL instead of REST for our API layer"
- **Focus on 'why' not 'what':** Explain the reasoning, not just the decision
- **Document alternatives:** Show you considered other options
- **Be honest about trade-offs:** Every decision has consequences
- **Update status:** Mark as "Accepted" when implemented
- **Link ADRs:** Reference related decisions
- **Date accurately:** Use the decision date, not writing date

## Integration Points

- **Section 9 of arc42:** Reference ADRs in the architecture document
- **Code comments:** Link to relevant ADRs in complex code sections
- **Pull requests:** Reference ADRs that justify architectural changes
- **Design reviews:** Use ADRs as discussion starting points

## Error Handling

If `.spec/architecture/adrs/` doesn't exist:
- Create the directory structure
- Start with ADR-0001

If user doesn't provide enough context:
- Ask specific questions about context, alternatives, consequences
- Offer examples to guide the response

If ADR number collision:
- Check existing files
- Use the next available number
- Warn if there are gaps in numbering

## Output Format

Always output:
1. Full file path: `.spec/architecture/adrs/adr-XXXX-title.md`
2. Confirmation message
3. Next steps reminder: "Add reference to section 9 of arc42 documentation"
