# Event-Driven Architecture

Event-Driven Architecture (EDA) is an architectural style in which components communicate by producing and consuming events. An event represents something that has happened, and interested components can react to it without the originating component needing to know who those consumers are.

This style can reduce temporal coupling, support asynchronous processing, and enable integration across Bounded Contexts. It is particularly useful when different parts of a system need to react to business occurrences independently.

However, event-driven architecture introduces its own complexity. Messages can be delayed, duplicated, reordered, or lost depending on the delivery infrastructure and implementation. Failures become distributed, workflows become harder to trace, and contracts must evolve without breaking existing consumers.

**Asynchronous communication does not eliminate coupling. It changes its form.**

The most important design question is not whether a system should use events, but which facts should be communicated, who owns them, what they mean, and what guarantees consumers can rely on.

## The basic idea

Consider a logistics platform in which a shipment is dispatched.

After the Shipment Management context successfully dispatches a shipment, other parts of the system may need to react:

- Fleet Management may update vehicle availability.
- Customer Notifications may send a message to the customer.
- Billing may prepare a charge.
- Analytics may update operational reporting.
- Tracking may record the shipment's new status.

A tightly coupled implementation might require Shipment Management to call each of these systems directly.

This creates several dependencies:

- Shipment Management must know which systems need to be notified.
- It must know how to communicate with each system.
- It may become unavailable when one of its downstream dependencies is unavailable.
- Adding another consumer may require changing the original component.

With event-driven communication, Shipment Management publishes a fact:

```text
ShipmentDispatched
```

Interested consumers subscribe to that event and decide what to do.

```text
                    Shipment Management
                            |
                            |
                    ShipmentDispatched
                            |
                            v
                      Message Broker
                       /    |    \
                      /     |     \
                     v      v      v
                Tracking  Billing  Notifications
```

The originating context does not need to know the identities of all consumers.

This reduces direct knowledge between components, but it does not mean that they are completely independent. Consumers still depend on the event's meaning, structure, availability, and delivery behavior.

## Events are facts, not instructions

An important distinction is the difference between an event and a command.

A **Command** expresses an intention to perform an action. It can be accepted, rejected, or fail.

An **Event** expresses that something has already happened. It describes a fact and should not be interpreted as a request to perform that fact.

For example:

| Command | Event |
|---|---|
| DispatchShipment | ShipmentDispatched |
| ReserveInventory | InventoryReserved |
| CancelOrder | OrderCancelled |
| ApproveLoan | LoanApproved |
| IssueInvoice | InvoiceIssued |

A command asks for a decision or action. An event communicates the result of a decision or action.

This distinction affects naming, ownership, error handling, and system behavior.

For example, publishing `DispatchShipment` as though it were an event can mislead consumers into thinking dispatch has already occurred. If dispatch is rejected, the message has communicated something that is not true.

An event should describe an occurrence that has actually been accepted as part of the business process.

## Domain Events and Integration Events

Not all events serve the same purpose. DDD distinguishes between events used within a domain model and events published to communicate with other systems or Bounded Contexts.

### Domain Events

A Domain Event represents something meaningful that happened within a domain model.

Examples:

- `ShipmentDispatched`
- `InventoryReserved`
- `PaymentReceived`
- `CustomerCreditLimitExceeded`
- `SubscriptionCancelled`

Domain Events help express domain behavior and communicate changes within the model.

They may be used to:

- Trigger reactions within the same Bounded Context.
- Coordinate domain behavior that should happen after an occurrence.
- Notify application-level components that a business fact has occurred.
- Initiate a process that involves other parts of the system.

A Domain Event belongs to the model that gives it meaning. It should use that model's Ubiquitous Language.

For example:

```csharp
public sealed record ShipmentDispatched(
    ShipmentId ShipmentId,
    VehicleId VehicleId,
    DateTime DispatchedAt);
```

The event communicates a domain fact. It should be raised as a consequence of valid domain behavior, not simply because an application handler wants to publish a notification.

A Domain Event does not automatically need to be sent through a message broker. It can be handled in-process, within the same transaction or application boundary, depending on the required consistency and failure semantics.

### Integration Events

An Integration Event communicates a business occurrence across a process or Bounded Context boundary.

For example, Shipment Management may publish:

```json
{
  "eventId": "d4c6f7a1-2a6b-4b0f-9d5e-12f3a4567890",
  "eventType": "ShipmentDispatched",
  "schemaVersion": 1,
  "occurredAt": "2026-11-13T10:30:00Z",
  "shipmentId": "SHP-10452",
  "vehicleId": "VEH-208"
}
```

The message is now an external contract. Its structure and meaning must be stable enough for independent consumers.

An Integration Event may be derived from a Domain Event, but the two do not have to be identical.

For example, the internal model might use a rich `ShipmentDispatched` Domain Event containing references and information useful to the domain. The published integration contract might expose only the identifiers and facts that downstream contexts are allowed to rely on.

This separation protects the internal model from becoming a public contract that is difficult to change.

### Comparing the two

| Aspect | Domain Event | Integration Event |
|---|---|---|
| Scope | Within a domain model or Bounded Context | Across a system or Bounded Context boundary |
| Ownership | The model that raises it | The publishing context that owns the external contract |
| Purpose | Express domain facts and trigger local reactions | Communicate facts to independent consumers |
| Language | Internal Ubiquitous Language | Published language agreed with consumers |
| Delivery | Often in-process | Usually asynchronous and distributed |
| Schema evolution | Can evolve with the internal model | Requires compatibility considerations |
| Infrastructure | Does not require a broker | Commonly uses a broker or messaging infrastructure |

Do not confuse an Integration Event with a Domain Event that happens to be serialized as JSON. Crossing a boundary changes the responsibilities involved.

## Events and Bounded Contexts

Event-driven architecture is particularly relevant to DDD because it can support communication between independently modeled Bounded Contexts.

Each context owns its own model and language. A context should not need to expose its internal domain objects to other contexts.

Consider the term `ShipmentDispatched`.

In Shipment Management, it might mean that all conditions for dispatch have been satisfied and the shipment has entered the dispatched state.

In Billing, the same occurrence might be relevant as the starting point for calculating a transportation charge.

In Tracking, it might mean that a new milestone should be recorded.

These consumers do not need to share their internal models. They need to agree on the meaning of the published fact and the information required to react to it.

A Published Language can provide the contract through which contexts exchange information.

The receiving context translates that contract into its own model. An Anti-Corruption Layer may be useful when the external vocabulary or structure differs significantly from the receiving context's language.

For example:

```text
Shipment Management
       |
       | ShipmentDispatched
       v
   Event Broker
       |
       +--------------------+
       |                    |
       v                    v
    Billing              Tracking
       |                    |
       v                    v
TransportationCharge   ShipmentMilestone
```

The event is shared. The interpretation and resulting models remain owned by each consumer.

This is an important distinction: **sharing a business fact does not require sharing a domain model.**

## Event Producers, Brokers, and Consumers

A typical event-driven system includes three main roles.

### Producers

A producer is responsible for publishing an event when a relevant occurrence takes place.

The producer should:

- Own the meaning of the event.
- Publish only facts that have actually occurred.
- Define the published contract.
- Ensure that event publication is consistent with the business operation.
- Provide enough information for consumers to identify and process the occurrence.

A producer should not need to know every consumer or implement their business reactions.

### Brokers

A message broker receives messages from producers and makes them available to consumers.

Examples include:

- RabbitMQ.
- Apache Kafka.
- Azure Service Bus.
- Amazon EventBridge.

These technologies have different delivery, ordering, retention, routing, and replay capabilities. Choosing a broker should follow the communication requirements, not the assumption that all messaging technologies provide the same guarantees.

A broker can help with:

- Message routing.
- Buffering and asynchronous delivery.
- Consumer isolation.
- Retention and replay, depending on the technology.
- Load distribution.
- Retry and dead-letter handling, depending on configuration.

A broker does not automatically guarantee business-level correctness. It cannot decide whether an event represents a valid business fact or whether processing it twice is safe.

### Consumers

A consumer receives events and performs work in response.

A consumer should:

- Understand the event contract.
- Translate the event into its own model.
- Process duplicate deliveries safely.
- Handle transient and permanent failures appropriately.
- Track processing outcomes.
- Avoid assuming that events arrive immediately or in a particular order unless that guarantee is explicitly established.

A consumer should not directly manipulate another context's domain model or database.

## Event Delivery Guarantees

One of the most important aspects of event-driven architecture is understanding what delivery guarantees actually mean.

Messaging systems commonly describe delivery in terms of at-most-once, at-least-once, and exactly-once semantics.

### At-most-once delivery

A message is delivered zero or one times.

The system avoids duplicates, but messages can be lost.

This may be acceptable for some non-critical notifications or disposable telemetry, but it is generally unsuitable when missing a business event would leave contexts inconsistent.

### At-least-once delivery

A message is delivered one or more times.

The system retries delivery when it cannot confirm successful processing. Consequently, consumers may receive duplicates.

This is a common and practical delivery model for business messaging.

It requires idempotent consumers.

### Exactly-once processing

The term exactly-once is often used too broadly.

Some platforms provide exactly-once guarantees within specific transactional or processing boundaries. That does not automatically mean a complete business operation involving a database, broker, external API, and several services happens exactly once.

For example:

1. A consumer receives `ShipmentDispatched`.
2. It updates its database.
3. It calls an external billing API.
4. The process crashes before acknowledging the message.
5. The broker delivers the event again.

The consumer may now perform the external operation twice unless it has designed for duplicate handling.

Exactly-once behavior across independent systems is not something to assume because a broker advertises exactly-once capabilities.

In practice, reliable systems commonly combine at-least-once delivery, idempotent processing, transactional boundaries, and explicit recovery mechanisms.

## The Transactional Outbox Pattern

A critical problem arises when a producer must update its database and publish an event.

Consider this sequence:

```text
1. Update Shipment in SQL Server
2. Publish ShipmentDispatched to RabbitMQ
```

What happens if the database transaction succeeds but publishing the event fails?

The shipment is dispatched, but downstream contexts never learn about it.

Reversing the order does not solve the problem:

```text
1. Publish ShipmentDispatched
2. Update Shipment in SQL Server
```

What if the event is published but the database transaction fails? Consumers may react to an occurrence that never actually happened.

This is the dual-write problem.

The Transactional Outbox Pattern addresses it by storing the business change and the event record in the same database transaction.

```text
             Application Use Case
                     |
                     v
               Domain Behavior
                     |
                     v
              Database Transaction
                /            \
               v              v
        Update Shipment   Insert Outbox
               \              /
                \            /
                 Transaction
                      |
                      v
                Outbox Publisher
                      |
                      v
                 Message Broker
                      |
                      v
                   Consumers
```

A simplified outbox table might contain:

| Column | Purpose |
|---|---|
| EventId | Unique event identifier |
| EventType | Event contract name |
| Payload | Serialized event data |
| OccurredAt | Time the business occurrence happened |
| CreatedAt | Time the outbox record was created |
| PublishedAt | Time publication was confirmed, if available |
| RetryCount | Number of publication attempts |

The application transaction updates the shipment and inserts the outbox record atomically.

A separate publisher reads pending records and publishes them to the broker.

After publication is confirmed according to the broker's acknowledgement semantics, the publisher marks the record as published.

The publisher can crash after the broker accepts the message but before the database records that publication succeeded. The message may then be published again.

Therefore, the Outbox Pattern prevents the database change and event record from diverging, but it does not eliminate duplicate publication. Consumers still need idempotency.

The Outbox is a reliability pattern, not a replacement for understanding delivery semantics.

## Idempotent Consumers

An idempotent operation can be performed multiple times without producing an unintended additional effect.

This matters because at-least-once delivery means that consumers must expect duplicate messages.

Suppose Billing receives `ShipmentDispatched` and creates a transportation charge.

If the event is delivered twice, Billing should not create two charges for the same business occurrence.

A common approach is to store processed event identifiers.

```text
ProcessedMessages
-----------------
ConsumerName
EventId
ProcessedAt
```

Before applying the business change, the consumer checks whether it has already processed that event.

The message record and business change should be committed atomically in the consumer's database transaction:

```text
BEGIN TRANSACTION

    Check whether EventId has been processed

    If not:
        Apply business change
        Record EventId as processed

COMMIT
```

The exact implementation depends on the database and messaging technology, but the principle is consistent.

The idempotency key should identify the event occurrence, not merely the event type or Aggregate identifier.

For example, a shipment can be dispatched, returned, and dispatched again under a new legitimate business occurrence. Deduplicating solely by shipment ID would incorrectly suppress later events.

Also distinguish message deduplication from business-level idempotency. Sometimes the same business request can generate different message IDs, or two different events can describe operations that must not produce duplicate business effects. The consumer may need a business key or explicit invariant in addition to message-level deduplication.

## Ordering and Concurrency

Events are not necessarily delivered in the order they were created.

Imagine the following occurrences:

```text
ShipmentAssigned
ShipmentDispatched
ShipmentDelivered
```

A consumer might receive:

```text
ShipmentDispatched
ShipmentAssigned
ShipmentDelivered
```

If the consumer assumes that arrival order always represents business order, it may reach an invalid state.

Ordering guarantees depend on the broker, partitioning or routing strategy, consumer concurrency, retries, and acknowledgement behavior.

Some technologies can preserve ordering within a partition or queue, but this is not the same as guaranteeing global ordering across the entire system.

Useful techniques include:

- Include an Aggregate or business entity identifier.
- Include an event sequence number or version where ordering matters.
- Partition or route messages using a meaningful key.
- Detect stale or out-of-order events.
- Make consumers resilient to delayed delivery.
- Avoid unnecessary ordering requirements across unrelated entities.

For example, a consumer might track the latest processed sequence number for a shipment and reject or defer an event that arrives with an unexpected version.

However, sequence numbers alone do not solve every ordering problem. The consumer must define what to do when an event is missing, delayed, or permanently unavailable.

Ordering should be an explicit business and technical requirement, not an accidental assumption.

## Retries, Dead Letters, and Failure Handling

A message consumer can fail for many reasons:

- A database is temporarily unavailable.
- An external API times out.
- A message contains invalid data.
- A business rule rejects the requested reaction.
- A downstream service is overloaded.
- A software defect causes repeated exceptions.

These failures require different responses.

### Retry transient failures

Temporary infrastructure failures may justify retrying a message.

For example, if the Billing database is temporarily unavailable, the consumer may retry after a delay.

Use bounded retries and backoff strategies rather than retrying continuously.

### Avoid retrying permanent failures forever

An invalid message or an unsupported schema version is unlikely to become valid simply by retrying it repeatedly.

Such messages may need to be moved to a dead-letter queue or an equivalent quarantine mechanism.

A dead-letter queue is not a solution by itself. It requires monitoring, investigation, ownership, and a controlled replay or remediation process.

### Understand poison messages

A poison message repeatedly fails processing and prevents normal progress, depending on the messaging system and consumer strategy.

A robust implementation should isolate problematic messages without silently losing them or allowing them to block unrelated work.

### Design for recovery

For business-critical events, define:

- Who owns failed messages.
- How failures are monitored and alerted.
- How operators investigate the failure.
- Whether the message can be replayed safely.
- Whether manual correction is necessary.
- How the system detects missing downstream effects.

An event-driven system is not reliable merely because messages are persisted. Reliability also depends on the ability to detect and recover from incomplete processing.

## Event Schema Evolution and Published Language

Once an event crosses a Bounded Context boundary, its structure becomes a contract.

Changing that contract can affect consumers that are developed and deployed independently.

For example, suppose Billing consumes:

```json
{
  "shipmentId": "SHP-10452",
  "dispatchedAt": "2026-11-13T10:30:00Z"
}
```

Shipment Management later changes the field name:

```json
{
  "id": "SHP-10452",
  "dispatchTime": "2026-11-13T10:30:00Z"
}
```

A consumer expecting the original field names may fail.

Schema evolution requires deliberate compatibility decisions.

Common approaches include:

- Additive changes that introduce optional fields.
- Explicit schema versions.
- Backward-compatible and forward-compatible consumer design.
- Versioned event contracts when incompatible changes are unavoidable.
- Consumer-driven contract testing.
- A migration period in which old and new consumers can coexist.

Avoid assuming that adding a field is always harmless. Strict deserializers, validation rules, generated clients, and schema registries can make apparently additive changes incompatible.

Also distinguish schema compatibility from semantic compatibility. A field can retain its name and type while its meaning changes, breaking consumers even though deserialization still works.

For example, changing `dispatchedAt` from the time a shipment physically left a facility to the time a dispatch request was approved is a semantic breaking change.

This is where Ubiquitous Language and Published Language become particularly important. The contract must communicate a stable business meaning, not merely a stable JSON shape.

## Event Sourcing Is Not Event-Driven Architecture

These concepts are related, but they are not interchangeable.

**Event-Driven Architecture** describes how components communicate and react to events.

**Event Sourcing** describes a persistence strategy in which the sequence of domain events is the authoritative record of an entity's state.

A system can use event-driven communication while persisting its state in ordinary relational tables.

A system can use Event Sourcing without publishing every stored domain event to external consumers.

Likewise, CQRS and Event Sourcing are not prerequisites for event-driven architecture.

| Concept | Main concern |
|---|---|
| Event-Driven Architecture | Communication through events |
| Domain Events | Expressing meaningful occurrences in a domain model |
| Integration Events | Communicating facts across boundaries |
| Event Sourcing | Persisting state as a sequence of events |
| CQRS | Separating command and query responsibilities or models |
| Transactional Outbox | Reliably recording messages alongside database changes |
| Saga / Process Manager | Coordinating long-running workflows across boundaries |

These patterns can complement each other, but adopting one does not automatically justify adopting the others.

## Event Choreography and Orchestration

Distributed workflows often involve several contexts reacting to events and performing subsequent operations.

There are two common approaches.

### Choreography

In choreography, each participant reacts to events and publishes new events as the process progresses.

Consider an order fulfillment process:

```text
OrderPlaced
     |
     v
Inventory Service
     |
     v
InventoryReserved
     |
     v
Payment Service
     |
     v
PaymentAuthorized
     |
     v
Shipping Service
     |
     v
ShipmentCreated
```

No central component controls the entire process. Each participant owns its local behavior and reacts to events.

Choreography can reduce central coordination and allow participants to evolve independently.

However, as the workflow grows, it can become difficult to understand the overall process. Business flow becomes distributed across consumers, and identifying the reason for a particular outcome may require tracing several events.

### Orchestration

In orchestration, a coordinator or Process Manager controls the workflow.

```text
              Order Process Manager
                 /      |      \
                v       v       v
          Inventory  Payment  Shipping
                \       |       /
                 \      |      /
                  v     v     v
               Process State
```

The coordinator tracks progress and decides which operation should happen next.

For example:

1. Request inventory reservation.
2. Wait for the result.
3. Request payment authorization.
4. If payment succeeds, request shipment creation.
5. If payment fails, initiate inventory release.

Orchestration makes the workflow explicit, but introduces a component responsible for coordinating multiple participants.

Neither approach is universally better.

Choreography can work well for simple, loosely connected reactions. Orchestration is often easier to understand when a workflow has complex branching, explicit deadlines, compensation, or long-running state.

A growing chain of event handlers that collectively implements a business process may be a signal that an explicit Process Manager is needed.

## Sagas and Distributed Transactions

When a workflow crosses multiple services or Bounded Contexts, a single local database transaction cannot normally guarantee atomicity across the entire process.

A Saga coordinates a sequence of local transactions. If a later step fails, compensating actions may be used to address the effects of earlier steps.

For example:

```text
Reserve Inventory
       |
       v
Authorize Payment
       |
       v
Create Shipment
       |
       v
       Success
```

If shipment creation fails after payment authorization, the process may need to:

1. Void or release the payment authorization.
2. Release the inventory reservation.
3. Mark the order as unable to proceed.

A compensation is not necessarily a literal rollback. If money has already been captured, the appropriate business action may be a refund rather than erasing the original payment.

This distinction matters because distributed workflows represent business reality, not simply database transaction mechanics.

A Saga must consider:

- Current process state.
- Timeouts and deadlines.
- Duplicate messages.
- Late responses.
- Compensation failures.
- Manual intervention.
- Recovery after restarts.

Do not introduce Sagas merely because two services communicate asynchronously. Use them when a business process genuinely spans multiple local transactions and needs explicit coordination.

## Observability and Operational Concerns

An event-driven system can be difficult to debug because a single user action may trigger work across several services over an extended period.

Traditional request-response tracing may not be enough.

A useful observability strategy should include:

- Unique event identifiers.
- Correlation identifiers for related events and commands.
- Causation identifiers linking a message to the event or command that caused it.
- Producer and consumer names.
- Event type and schema version.
- Processing timestamps.
- Retry counts and failure details.
- End-to-end workflow status where necessary.

For example, if `ShipmentDispatched` causes a billing operation, a correlation identifier can connect the dispatch occurrence to the resulting charge.

Monitoring should also cover operational behavior:

- Queue depth and consumer lag.
- Message processing latency.
- Failed and dead-lettered messages.
- Retry rates.
- Outbox backlog and publication delay.
- Consumer throughput.
- Repeated processing failures.

A system that successfully publishes messages but leaves them unprocessed for hours is not necessarily healthy.

Observability should make delayed and incomplete business processes visible, not just report whether individual services are running.

## Common Mistakes

### 1. Treating every internal event as an Integration Event

Not every domain occurrence needs to be published outside its Bounded Context.

Exposing every internal event creates unnecessary contracts and makes internal model changes harder.

Publish facts that other contexts genuinely need.

### 2. Publishing technical events instead of business facts

Events such as `RecordUpdated`, `EntitySaved`, and `DatabaseRowChanged` may be useful for technical synchronization, but they often fail to communicate business meaning.

Prefer events that describe meaningful occurrences in the domain when designing business integration contracts.

### 3. Assuming asynchronous means independent

Consumers depend on event meaning, availability, schema, timing, and delivery behavior.

A change to an event contract can break several downstream contexts even if they do not share source code.

Asynchronous communication reduces some forms of coupling, but creates contract and operational coupling.

### 4. Assuming the broker guarantees exactly-once business behavior

Broker-level guarantees do not automatically protect database changes or external side effects.

Consumers must handle duplicate messages and partial failures.

### 5. Ignoring event ownership

If several contexts publish similarly named events with ambiguous meanings, consumers may not know which context owns the fact.

The publishing context should own the contract and its semantics.

### 6. Creating an event for every state change

Not every property update is a meaningful business occurrence.

Publishing `ShipmentStatusChanged` for every internal status modification may expose implementation details instead of communicating useful business facts.

An event should exist because the occurrence matters to another part of the system or to the domain model.

### 7. Building workflows through accidental choreography

A long chain of consumers and events can become an implicit workflow that nobody fully understands.

When business coordination becomes complex, consider making it explicit through orchestration or a Process Manager.

### 8. Using events to avoid designing boundaries

A message broker does not automatically create well-designed Bounded Contexts.

If two services share the same model, database assumptions, and business decisions, exchanging events does not make them independent.

First clarify model ownership and integration responsibilities. Then choose the communication mechanism.

### 9. Treating dead-letter queues as a complete failure strategy

A dead-letter queue can preserve failed messages, but it does not resolve them.

Without monitoring, ownership, investigation, and safe replay procedures, it can become a place where business failures disappear from everyday visibility.

### 10. Ignoring eventual consistency

A successful operation in one context does not mean every other context has already updated its state.

The system must communicate this reality through its behavior and user experience.

For example, after dispatching a shipment, its tracking view may update immediately while its billing projection updates several seconds later. The system should not present the delayed result as though it were guaranteed to be immediately consistent.

## Practical Guidelines

When designing an event-driven system:

1. Start with business occurrences, not broker features.
2. Distinguish Commands, Domain Events, and Integration Events.
3. Define which Bounded Context owns each published event.
4. Treat external event schemas as Published Language contracts.
5. Separate internal domain events from external integration contracts when their responsibilities differ.
6. Choose delivery guarantees according to business criticality.
7. Use an Outbox when database changes and message publication must remain consistent.
8. Make consumers idempotent and design for duplicate delivery.
9. Define ordering expectations explicitly.
10. Plan retries, dead-letter handling, monitoring, and recovery.
11. Version contracts and test compatibility with consumers.
12. Use choreography for suitable independent reactions and orchestration for complex workflows.
13. Consider Sagas when a business process spans multiple local transactions.
14. Make eventual consistency visible and understandable to users and operators.
15. Avoid introducing messaging where a simpler synchronous interaction is more appropriate.

## A Practical Exercise

Design an event-driven integration for a shipment dispatch workflow.

Assume Shipment Management owns shipment dispatching, Billing calculates transportation charges, and Customer Notifications informs customers.

Identify:

**1. Domain occurrence**

What exactly does `ShipmentDispatched` mean? What conditions must be satisfied before it can be raised?

**2. Ownership**

Which Bounded Context owns this fact? Which context owns the integration contract?

**3. Integration contract**

What information should Billing and Notifications receive? Which information should remain internal to Shipment Management?

**4. Reliability**

What happens if the database transaction succeeds but publishing fails? How will duplicate deliveries be handled?

**5. Ordering**

What happens if `ShipmentDelivered` reaches a consumer before `ShipmentDispatched`?

**6. Failure handling**

What happens if Billing is unavailable for thirty minutes? What happens if the event cannot be processed because its schema is unsupported?

**7. Observability**

How would an operator trace a shipment from dispatch through billing and notification?

**8. Consistency**

What should the user see while downstream contexts are still processing the event?

Draw the architecture, define the contracts, and document the assumptions. Then review whether every asynchronous interaction is justified by an actual business or technical requirement.

## Completion Criteria

You should be able to:

- Explain Event-Driven Architecture and its relationship to DDD.
- Distinguish Commands, Domain Events, and Integration Events.
- Explain how events support communication between Bounded Contexts.
- Design an event contract with clear ownership and stable semantics.
- Explain at-most-once, at-least-once, and exactly-once delivery claims.
- Apply the Transactional Outbox Pattern and idempotent consumer strategies.
- Identify ordering, retry, duplication, and failure-handling requirements.
- Explain schema evolution and Published Language.
- Distinguish Event-Driven Architecture from Event Sourcing and CQRS.
- Choose between choreography and orchestration based on workflow complexity.
- Explain when Sagas and compensating actions are appropriate.
- Design observability and recovery strategies for asynchronous processes.

## Further Reading

- Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software*.
- Vaughn Vernon, *Implementing Domain-Driven Design*.
- Martin Fowler, [What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html).
- Martin Fowler, [Transactional Outbox](https://microservices.io/patterns/data/transactional-outbox.html).
- Chris Richardson, [Microservices Patterns](https://microservices.io/book).
- Gregor Hohpe and Bobby Woolf, *Enterprise Integration Patterns*.
- Confluent, [Event-Driven Architecture](https://www.confluent.io/learn/event-driven-architecture/).