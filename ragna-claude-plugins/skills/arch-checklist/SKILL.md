---
name: arch-checklist
description: Review architecture documentation completeness and quality using comprehensive checklist
allowed-tools: ["Read", "Glob", "Grep"]
---

# Architecture Checklist Skill

This skill provides a comprehensive checklist for reviewing and validating architecture documentation quality, completeness, and alignment with best practices.

## Purpose

Evaluate architecture documentation to ensure:
- **Completeness:** All necessary sections and artifacts present
- **Quality:** Clear, actionable, and well-structured content
- **Traceability:** Requirements mapped to design decisions
- **Consistency:** Aligned diagrams, descriptions, and code
- **Actionability:** Sufficient detail for implementation

## Instructions

When this skill is invoked:

1. **Scan documentation:**
   - Read `.spec/architecture/application-architecture.md`
   - Check for ADR files in `.spec/architecture/adrs/`
   - Identify existing diagrams and sections

2. **Evaluate against checklist:**
   - Mark items as ✓ (complete), □ (missing), or ⚠ (incomplete/weak)
   - Note specific gaps or issues
   - Highlight areas needing improvement

3. **Generate status report:**
   - Summary of completion percentage
   - Critical gaps (missing sections, diagrams)
   - Quality concerns (vague descriptions, missing details)
   - Recommendations for improvement

4. **Prioritize refinements:**
   - Must-have: Critical gaps blocking implementation
   - Should-have: Important for quality and maintainability
   - Nice-to-have: Enhancements for clarity and completeness

5. **Provide actionable feedback:**
   - Specific items to add/improve
   - Suggestions for using other skills
   - Next steps for refinement

## Architecture Quality Checklist

### Requirements & Context (Section 1-3)

**Section 1: Introduction & Goals**
- □ Business goals clearly stated with measurable KPIs
- □ Primary use cases and user stories documented
- □ Quality goals ranked with acceptance criteria (latency, throughput, availability)
- □ Stakeholders identified with concerns and decision authority

**Section 2: Constraints**
- □ Technical constraints documented (legacy systems, mandated tech, infrastructure limits)
- □ Organizational constraints captured (team structure, compliance, budget)
- □ Conventions and standards defined (coding, documentation, governance)

**Section 3: Context & Scope**
- □ Business context with actors and value streams
- □ Technical context with external systems and protocols
- □ C4 Context diagram showing system boundaries
- □ External interfaces documented with SLAs

### Architecture Design (Section 4-5)

**Section 4: Solution Strategy**
- □ Architecture drivers explained (why this style fits goals)
- □ Key decisions documented with rationale
- □ Quality tactics specified (scalability, security, observability)
- □ Trade-offs explicitly stated

**Section 5: Building Block View**
- □ C4 Container diagram showing services/components
- □ C4 Component diagram for key services (internal structure)
- □ Modules diagram: 3-4 level package hierarchy with lowercase names, no arrows
- □ Modules table describing each package/sub-package
- □ API Draft tables for all services (HTTP Method, Endpoint, Description)
- □ Async API Draft table (Channel, Producer, Message Types, Payload, Purpose, Consumers)
- □ Component responsibilities clearly defined

### Runtime & Behavior (Section 6)

**Section 6: Runtime View**
- □ UML Sequence diagram for happy-path scenario
- □ UML Sequence diagram for failure/degraded scenario
- □ UML State diagram for lifecycle/stateful behavior
- □ Background/batch processing flows documented
- □ Error handling and retry strategies shown

### Infrastructure & Data (Section 7-8)

**Section 7: Deployment View**
- □ Environment topology (dev, stage, prod) documented
- □ Infrastructure map with nodes, clusters, networking
- □ Deployment topology diagram (Mermaid or C4 Deployment)
- □ CI/CD pipeline and rollout strategy
- □ Scaling strategy and observability hooks

**Section 8: Crosscutting Concepts**
- □ Domain model and ubiquitous language defined
- □ Data model diagrams (ER for SQL or JSON schema for MongoDB)
- □ Data consistency strategy explained
- □ Security: AuthN/AuthZ, encryption, threat model
- □ Resilience: Circuit breakers, retries, timeouts, fallbacks
- □ Observability: Logging, metrics, tracing strategy
- □ Performance: Caching, async processing, scaling approach

### Decisions & Quality (Section 9-11)

**Section 9: Architecture Decisions**
- □ ADR files created for major decisions (`.spec/architecture/adrs/adr-XXXX.md`)
- □ ADRs follow template (Status, Context, Decision, Consequences)
- □ ADRs referenced in section 9 (table with ID, title, status)
- □ Alternatives considered and documented
- □ Trade-offs explicitly stated

**Section 10: Quality Requirements**
- □ Quality tree with attribute breakdown and priorities
- □ Quality scenarios (stimulus → environment → response with metrics)
- □ NFRs quantified with acceptance criteria

**Section 11: Risks & Technical Debt**
- □ Risk register with probability, impact, mitigation
- □ Technical debt backlog with remediation plan
- □ Validation tasks (POCs, benchmarks, audits)

**Section 12: Glossary**
- □ Domain terms defined with context
- □ Acronyms and abbreviations expanded
- □ Ubiquitous language alignment documented

### API & Integration Quality

**API Documentation**
- □ All endpoints documented with request/response examples
- □ Status codes and error responses specified
- □ Versioning and compatibility strategy defined
- □ Pagination, filtering, sorting patterns documented
- □ Authentication and authorization requirements clear

**Async API Documentation**
- □ Event channels documented with producers and consumers
- □ Event schemas with versioning strategy
- □ Idempotency and retry patterns specified
- □ Dead letter handling documented
- □ Schema evolution strategy defined

### Diagram Quality

**C4 Diagrams**
- □ Context: System boundaries and external actors clear
- □ Container: Services/components with technologies specified
- □ Component: Internal structure for key services detailed
- □ Consistent naming across all diagrams
- □ Relationships labeled with protocols

**UML Diagrams**
- □ Sequence diagrams show realistic scenarios
- □ State diagrams model complete lifecycles
- □ Activations and lifelines used correctly
- □ Error paths and edge cases covered

**Module/Package Diagrams**
- □ 3-4 levels of nesting
- □ Lowercase dot notation (e.g., `orders.domain.model`)
- □ No arrows (structure only, not dependencies)
- □ Description table matches diagram identifiers

**Data Model Diagrams**
- □ ER diagram for relational databases (Mermaid)
- □ JSON schema for MongoDB with relationships
- □ Cardinality and constraints shown
- □ Indexes and validation rules documented

### Implementation Readiness

**Code Organization**
- □ Package/module structure defined
- □ Layer boundaries specified (domain, application, infrastructure)
- □ Dependency direction rules stated
- □ Shared code strategy documented

**Testing Strategy**
- □ Unit testing approach defined
- □ Integration testing strategy specified
- □ Contract testing for APIs
- □ End-to-end test scenarios identified

**Operations Readiness**
- □ Deployment strategy documented
- □ Monitoring and alerting defined
- □ Incident response procedures outlined
- □ Backup and disaster recovery planned

## Checklist Categories and Scoring

### Critical (Must-Have) - Blocks Implementation
- Business goals and quality goals (Section 1)
- C4 Container diagram (Section 5)
- API Draft tables (Section 5)
- Data model (Section 8)
- Deployment topology (Section 7)
- At least 1 sequence diagram (Section 6)

**Scoring:** 0-59% = Red, 60-79% = Yellow, 80-100% = Green

### Important (Should-Have) - Impacts Quality
- ADRs for major decisions (Section 9)
- C4 Component diagram (Section 5)
- Modules hierarchy with description table (Section 5)
- Async API documentation (Section 5)
- State diagram for key entities (Section 6)
- Security and resilience strategies (Section 8)
- Risk register (Section 11)

**Scoring:** 0-69% = Yellow, 70-100% = Green

### Enhanced (Nice-to-Have) - Improves Clarity
- Stakeholder mapping (Section 1)
- Glossary with ubiquitous language (Section 12)
- Multiple sequence diagrams (happy/error paths)
- Quality scenarios with metrics (Section 10)
- Technical debt backlog (Section 11)
- Background processing flows (Section 6)

**Scoring:** Advisory only, no color coding

## Usage Examples

**Example 1: Review existing documentation**
```
User: Check my architecture doc for completeness
Skill: [Scans application-architecture.md, evaluates against checklist, reports missing items]
```

**Example 2: Pre-implementation validation**
```
User: Is the architecture ready for development to start?
Skill: [Reviews critical items, identifies blockers, recommends additions]
```

**Example 3: Quality audit**
```
User: Audit the architecture for quality and best practices
Skill: [Comprehensive review against all checklist categories, prioritized recommendations]
```

**Example 4: Gap analysis**
```
User: What's missing from my architecture docs?
Skill: [Identifies gaps in sections, diagrams, ADRs, suggests next steps]
```

## Output Format

### Status Report Template

```
# Architecture Quality Report

**Overall Completion:** X% (Y/Z items complete)

**Status:** 🟢 Green | 🟡 Yellow | 🔴 Red

---

## Critical Items (Must-Have)

✓ Business goals clearly stated
✓ C4 Container diagram present
□ API Draft tables missing for 2 services
✓ Data model documented (MongoDB JSON schema)
⚠ Deployment topology present but lacks scaling strategy
✓ Sequence diagram shows order creation flow

**Critical Score:** 75% (6/8) - 🟡 Yellow

---

## Important Items (Should-Have)

✓ 3 ADRs created for major decisions
✓ C4 Component diagram for order-service
□ Modules hierarchy diagram missing
⚠ Async API table present but lacks schema details
□ State diagram missing for order lifecycle
✓ Security strategy documented
⚠ Risk register incomplete (no mitigation owners)

**Important Score:** 57% (4/7) - 🟡 Yellow

---

## Enhanced Items (Nice-to-Have)

✓ Stakeholders identified
□ Glossary not present
✓ Error scenario sequence diagram
□ Quality scenarios not quantified
✓ Technical debt noted

**Enhanced Score:** 60% (3/5)

---

## Recommendations (Prioritized)

### Must Address Before Implementation
1. **Add API Draft tables** for payment-service and notification-service
   - Use `api-draft` skill to generate
   - Include request/response examples
   - Document error codes

2. **Enhance deployment topology**
   - Add horizontal scaling strategy (replicas, auto-scaling)
   - Document load balancer configuration
   - Specify resource limits (CPU, memory)

### Should Address for Quality
3. **Create modules hierarchy diagram**
   - Use `modules-diagram` skill
   - Show 3-4 level package structure for order-service
   - Include description table

4. **Add state diagram for order lifecycle**
   - Use `state-diagram` skill
   - Show states: PENDING → CONFIRMED → SHIPPED → DELIVERED
   - Include cancellation and failure paths

5. **Complete Async API documentation**
   - Expand with event schemas
   - Add versioning strategy
   - Document idempotency patterns

### Nice to Have for Completeness
6. **Create glossary**
   - Define domain terms (order, fulfillment, payment)
   - Document ubiquitous language
   - Add acronyms

7. **Quantify quality scenarios**
   - Response time: p99 < 200ms
   - Availability: 99.9% uptime
   - Throughput: 1000 orders/sec

---

## Next Steps

1. Use `/skill api-draft` to document payment and notification APIs
2. Update deployment section with scaling details
3. Use `/skill modules-diagram` for order-service structure
4. Use `/skill state-diagram` for order lifecycle
5. Re-run checklist to validate improvements
```

## Status Indicators

- ✓ **Complete:** Item fully addressed with quality
- □ **Missing:** Item not present in documentation
- ⚠ **Incomplete/Weak:** Item present but needs improvement

## Refinement Workflow

After presenting the checklist:

1. **User prioritizes:** "Focus on critical items" or "Work on diagrams"
2. **Agent refines:** Address specific gaps using relevant skills
3. **Re-evaluate:** Run checklist again to show progress
4. **Iterate:** Repeat until quality bar met

## Integration with Other Skills

| Gap Identified | Skill to Use | Purpose |
|----------------|--------------|---------|
| Missing ADR | `create-adr` | Generate Architecture Decision Record |
| No C4 diagrams | `c4-diagram` | Create Context/Container/Component diagrams |
| Missing data model | `mongodb-schema` | Document MongoDB collections |
| API docs incomplete | `api-draft` | Generate API endpoint tables |
| No sequence diagram | `sequence-diagram` | Show runtime interaction flows |
| State lifecycle missing | `state-diagram` | Model entity state machines |
| No modules structure | `modules-diagram` | Document package hierarchy |

## Best Practices

### When to Run
- **Initial delivery:** After first architecture draft
- **Refinement cycles:** After each improvement iteration
- **Pre-implementation:** Before development starts
- **Quality gates:** Before architecture review/approval

### How to Use
- **Be objective:** Use checklist criteria, not opinions
- **Provide examples:** Show what good looks like
- **Prioritize:** Focus on critical gaps first
- **Be actionable:** Suggest specific skills or tasks
- **Track progress:** Compare checklist results over time

### What to Avoid
- **Perfectionism:** Don't require 100% for implementation to start
- **Overload:** Don't demand everything at once
- **Vagueness:** Don't just say "add more detail"
- **Inconsistency:** Use same criteria across projects

## Error Handling

- If documentation not found: Provide checklist as template
- If partial documentation: Evaluate available sections only
- If format non-standard: Adapt checklist to actual structure
- If user unclear on priority: Recommend critical items first

## Output Format

Always provide:
1. Overall completion percentage and status (Red/Yellow/Green)
2. Category-wise breakdown (Critical, Important, Enhanced)
3. Specific items checked (✓/□/⚠)
4. Prioritized recommendations with skill suggestions
5. Next steps for refinement
6. Option to focus on specific areas

## Cross-References

This skill orchestrates other skills:
- `create-adr` - For missing decision records
- `c4-diagram` - For architecture diagrams
- `mongodb-schema` - For data model documentation
- `api-draft` - For API contracts
- `sequence-diagram` - For runtime flows
- `state-diagram` - For lifecycle modeling
- `modules-diagram` - For code organization
