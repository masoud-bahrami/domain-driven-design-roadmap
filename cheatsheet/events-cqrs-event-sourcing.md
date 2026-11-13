# Events, CQRS, and Event Sourcing

These techniques are often discussed together in DDD, but they address different concerns. Understanding their responsibilities and trade-offs helps avoid introducing unnecessary architectural complexity.

## Domain Events

A **Domain Event** represents a meaningful fact that has happened within a domain.

- Express events in the past tense, such as `OrderConfirmed` or `PaymentReceived`.
- Raise events when meaningful business state changes occur.
- Let the domain model express what happened without depending directly on external messaging infrastructure.
- Include enough information for the intended consumers to understand the occurrence.
- Avoid creating events for every technical operation or insignificant state change.

**Ask:** Does this represent something meaningful that happened in the business, or merely an implementation detail?

## Integration Events

An **Integration Event** is a published contract that communicates a meaningful occurrence to another Bounded Context or external system.

- Publish events when another context needs to react to a business occurrence.
- Define explicit contracts that consumers can understand independently of the producer's internal model.
- Avoid exposing internal domain objects or persistence structures directly.
- Consider event identity, versioning, compatibility, and ownership.
- Design consumers to handle duplicate delivery, retries, and temporary failures.
- Consider the Transactional Outbox pattern when database changes and message publication must be coordinated reliably.

**Ask:** Which external consumers need to know about this occurrence, and what stable contract should they receive?

A Domain Event and an Integration Event may represent the same business occurrence, but they are not necessarily the same message or model.

## Command Query Responsibility Segregation (CQRS)

CQRS separates the responsibilities of handling commands and answering queries.

- **Commands** express intentions to change business state.
- **Command handling** validates the request and invokes the appropriate domain behavior.
- **Queries** retrieve information without changing business state.
- **Read models** can be shaped around specific information requirements rather than mirroring the write-side domain model.

CQRS can be implemented at different levels of complexity:

- Separate command and query handlers within one application.
- Different models for writing and reading data.
- Independently optimized read and write storage when justified.

CQRS does not require separate databases, microservices, message brokers, or Event Sourcing.

**Ask:** Are the requirements for changing state sufficiently different from the requirements for reading it to justify separate models?

## Event Sourcing

Event Sourcing persists state changes as an ordered sequence of events rather than storing only the latest state.

The current state is reconstructed by replaying the relevant events.

Consider its implications carefully:

- **Event Design:** Persisted events become part of the long-term historical record. Their meaning and structure deserve careful design.
- **Event Evolution:** Event schemas may need to remain readable across application versions and business changes.
- **Concurrency:** Event streams need appropriate concurrency control to prevent conflicting writes.
- **Projections:** Queries often rely on projections built from persisted events.
- **Replay:** Rebuilding state and projections requires reliable replay mechanisms.
- **Snapshots:** Large event streams may require snapshots to reduce reconstruction costs.
- **Operational Tooling:** Debugging, monitoring, backups, recovery, and event inspection become more demanding.
- **Historical Semantics:** Existing events should preserve what was true when they occurred rather than being silently reinterpreted using today's business rules.

**Ask:** Is retaining the complete history of state changes a genuine business requirement that justifies the additional complexity?

## How They Relate

| Concept | Primary responsibility | Requires the others? |
|---|---|---|
| Domain Events | Express meaningful facts inside the domain | No |
| Integration Events | Communicate facts across boundaries | No |
| CQRS | Separate command and query responsibilities | No |
| Event Sourcing | Persist state as an event sequence | No |

These techniques can complement one another, but none is a prerequisite for the others.

For example, a system can use Domain Events without a message broker, CQRS without separate databases, or Integration Events without Event Sourcing.

## Choosing the Right Technique

Before introducing any of these approaches, ask:

- What specific problem are we trying to solve?
- Is there a genuine need to communicate business occurrences asynchronously?
- Are read and write requirements different enough to justify CQRS?
- Is historical reconstruction or auditability a core business requirement?
- Can the team manage event versioning, retries, projections, and operational recovery?
- Would a simpler design provide the same business value?

**Remember:** DDD is about understanding and modeling the business, not adopting every pattern associated with it. Use these techniques when their specific benefits justify their costs. They are not prerequisites for DDD.