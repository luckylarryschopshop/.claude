---
name: system-design
description: Software architecture methodology — system decomposition, ADRs, tradeoff analysis, scalability patterns. Load when acting as Architect agent.
---

# System Design Skill

## Core Methodology

### Decomposition Approach
1. **Start with capabilities, not components** — what must the system DO, not what it IS
2. **Identify bounded contexts** — where does one team/domain own a concept end-to-end?
3. **Define service contracts before service internals** — the API is more important than the implementation
4. **Separate read and write concerns** — most systems are read-heavy; optimise accordingly

### Architecture Decision Records (ADRs)
Write an ADR for every decision that:
- Affects more than one component
- Would take more than a day to reverse
- Involves a tradeoff between two reasonable options

ADR fields: Context, Decision, Alternatives Considered, Consequences

Never use a technology because it is popular — justify it against the specific problem.

### Scalability Patterns (Reference)
| Pattern | Use when | Cost |
|---------|----------|------|
| Vertical scaling | Traffic is predictable, team is small | Low complexity |
| Horizontal scaling + load balancer | Traffic spikes, need redundancy | Medium |
| Read replica | Read-heavy, tolerate eventual consistency | Medium |
| Caching layer | Repeated reads of same data, latency-sensitive | Medium |
| Async queue | Background work, burst traffic, decoupling | High |
| CQRS | Complex domain, different read vs write models | Very high |

**Default: start with vertical + read replica. Add complexity only when measured.**

### Component Diagram Rules
- Every component shows: its responsibilities (not implementation), its consumers, its dependencies
- Every inter-component call shows: protocol (REST/gRPC/queue), data ownership, auth
- Mark data stores with their ownership (only one component may write each entity)

### Service Contract Template
```
Service: [name]
Owned entities: [list — this service is the source of truth for these]

Endpoints:
  POST /[resource]
    Input: {field: type, ...}
    Output: {field: type, ...}
    Auth: [JWT / API key / none]
    Errors: 400 (validation), 401 (auth), 409 (conflict)
```

### Common Antipatterns to Avoid
- **Shared database** between services — breaks independent deployability
- **Distributed monolith** — microservices with synchronous chains that must all be up
- **Premature optimisation** — designing for 1M users when the first 100 aren't proven
- **Vague service boundaries** — if two engineers disagree on who owns something, the boundary is wrong

### Tradeoff Framework
For every significant decision, state:
- What this makes easy
- What this makes harder
- What this makes impossible (irreversible constraints)
- Under what future conditions you'd revisit this

## Quality Checklist
Before finalising architecture:
- [ ] Can each component be deployed independently?
- [ ] Is data ownership unambiguous for every entity?
- [ ] Is there a single source of truth for every piece of data?
- [ ] Are all external dependencies (third-party APIs, cloud services) replaceable?
- [ ] Is the failure mode of each component documented?
- [ ] Does the architecture survive the loss of any single component?
