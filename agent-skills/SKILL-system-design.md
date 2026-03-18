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

---

## Domain-Driven Design (DDD) Tactical Patterns

Apply when the domain has complex business rules, multiple distinct sub-domains, or the
system will be maintained by multiple teams over years.

### Bounded Context
A bounded context is the boundary within which a domain model is internally consistent and
a team owns it end-to-end. The same word can mean different things in different contexts —
and that is fine.

```
Bounded Context: Order Management
  → "Customer" means: who placed the order, their shipping address
  → Owns: Order, OrderLine, ShippingAddress, Fulfilment

Bounded Context: Billing
  → "Customer" means: billing entity, payment method, invoice history
  → Owns: Invoice, PaymentMethod, Subscription
```

Rule: never share a domain model across bounded contexts. Share only IDs and events.

### Ubiquitous Language
Every term used in code must match the term used by domain experts. If the business says
"reservation", code says `Reservation` — not `Booking`, not `Slot`, not `Entry`.

Practical application:
- Hold a glossary in `docs/ubiquitous-language.md`
- Any code review that introduces a term not in the glossary requires a discussion
- Domain objects in code must be named with domain vocabulary, not technical vocabulary

### Aggregates
An aggregate is a cluster of domain objects treated as a single unit for data changes.
The aggregate root is the only entry point — no direct access to internal objects.

```python
# Correct: mutate Order through the aggregate root
order.add_item(product_id, quantity)  # validates invariants internally

# Wrong: reach into the aggregate and mutate internals directly
order.items.append(OrderItem(product_id, quantity))  # bypasses invariants
```

Rules:
- One aggregate = one DB transaction boundary
- Aggregates reference other aggregates by ID only, never by object reference
- Keep aggregates small — if an aggregate has >5 entity types, it's probably two aggregates

---

## CAP Theorem and Consistency Models

Every distributed data system must choose between guarantees when a network partition occurs.

| Model | Guarantee | Use when |
|-------|-----------|----------|
| **Strong consistency** | Every read returns the latest write | Financial transactions, inventory counts, anything where stale data causes real harm |
| **Eventual consistency** | Reads may be stale; all nodes converge eventually | Social feeds, analytics, non-critical caches, any read-heavy workload tolerating lag |
| **Causal consistency** | If A caused B, everyone sees A before B | Comment threads, activity feeds with replies, any scenario where ordering within a session matters |
| **Read-your-writes** | After a write, the same client always sees it | User profile updates, settings changes — user must see their own changes immediately |

**Document the consistency model** for every data store and every critical read path in the
architecture. "It depends" is not an acceptable answer.

Default recommendation: use strong consistency for mutations (via a single authoritative DB),
eventual consistency for derived read models (via projections/cache).

---

## Event-Driven Architecture Patterns

### Outbox Pattern (guaranteed event delivery)
The most reliable way to publish events after a DB write. Solves the dual-write problem
(write to DB + publish to queue atomically).

```sql
-- Outbox table in the same database as the domain
CREATE TABLE outbox_events (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type  VARCHAR(100) NOT NULL,
  aggregate_id VARCHAR(100) NOT NULL,
  payload     JSONB NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  published_at TIMESTAMPTZ          -- NULL = not yet published
);
```

```python
# In the same DB transaction as the domain write:
with transaction():
    order.confirm()
    db.save(order)
    db.insert(OutboxEvent(type="order.confirmed", aggregate_id=order.id, payload=order.to_event()))

# Separate outbox relay process polls and publishes:
# SELECT * FROM outbox_events WHERE published_at IS NULL ORDER BY created_at LIMIT 100
```

### Saga Pattern (distributed transactions)
For operations spanning multiple services where a 2PC transaction isn't feasible.

**Choreography saga** (simpler, less coupling):
- Each service listens for events and reacts with its own local transaction + publishes the next event
- Use when: ≤3 steps, services are loosely coupled

**Orchestration saga** (explicit coordination):
- A central saga orchestrator tells each service what to do and handles failures
- Use when: >3 steps, complex compensating transactions, need a single place to trace the saga

Compensating transactions: for every step, define the undo action (reversal, not rollback).
```
Order Saga:
  Step 1: Reserve inventory  → Compensation: Release reservation
  Step 2: Charge payment     → Compensation: Refund
  Step 3: Create shipment    → Compensation: Cancel shipment
```

### Event Sourcing
Store the full history of events (immutable log) as the source of truth; derive current state
by replaying events.

Use when: audit trail is a hard requirement, temporal queries ("what was the state on date X?"),
complex domain with many state transitions.

Do NOT use when: simple CRUD with no audit requirement (adds significant complexity).

```python
# Events are immutable facts
events = [
    OrderCreated(order_id, customer_id, items, at=t1),
    ItemAdded(order_id, item, at=t2),
    OrderConfirmed(order_id, at=t3),
]
# Current state is always derived
order = Order.from_events(events)  # replays all events to reconstruct
```

---

## Legacy Migration Patterns

### Strangler Fig
Incrementally replace a legacy system by routing new traffic to the new system,
while the old system handles the remainder. Never "big bang" replace a working legacy system.

```
Phase 1: New system handles NEW data only → legacy handles existing
Phase 2: Migrate a subset of existing data → new system handles it
Phase 3: Migrate remaining data → decommission legacy
```

Implementation: put a facade (proxy/router) in front of both systems. The facade routes
by entity age, feature flag, or user cohort. The legacy system is never called for
entities already migrated.

### Anti-Corruption Layer (ACL)
A translation layer between your clean domain model and a legacy or external system's model.
Prevents the external system's concepts from leaking into your domain.

```python
class LegacyOrderAdapter:
    """Anti-corruption layer: translates legacy Order format to domain Order."""

    def to_domain(self, legacy_order: dict) -> Order:
        return Order(
            id=legacy_order["ord_num"],          # legacy uses "ord_num"
            customer_id=legacy_order["cust_id"],
            total=Money(legacy_order["tot_amt"], "USD"),  # legacy stores as cents
        )
```

Rule: ACL code lives at the boundary, never in the domain. The domain model never knows
about the external system's field names or data formats.
