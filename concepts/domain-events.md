# Domain Events

A Domain Event represents something meaningful that has happened in the domain.

It captures a fact about the business, not an instruction to perform an operation. Domain Events are usually named in the past tense because they describe something that has already occurred:

- `OrderPlaced`
- `PaymentReceived`
- `ShipmentDispatched`
- `CustomerRegistered`
- `SubscriptionCancelled`
- `CreditLimitExceeded`

The distinction between an event and a command is fundamental.

A command expresses an intention or request:

```text
PlaceOrder
ReceivePayment
DispatchShipment
CancelSubscription
```

An event expresses a fact:

```text
OrderPlaced
PaymentReceived
ShipmentDispatched
SubscriptionCancelled
```

A command can be rejected. An event describes something that has happened and cannot simply be rejected after the fact.

For example, `PlaceOrder` asks the system to place an order. The operation may fail because the customer is blocked, the product is unavailable, or the order violates a business rule.

`OrderPlaced`, on the other hand, communicates that the order was successfully placed.

This distinction is more than a naming convention. It separates intention from outcome, and decisions from facts.

Domain Events help make business behavior explicit. They allow a model to communicate significant changes without requiring every interested component to know the internal implementation of the component that produced them.

They are particularly useful when a business occurrence has consequences in multiple parts of the system.

However, not every method call, property update, or database modification deserves an event. An event should represent something meaningful enough that the domain would recognize and care about it.

---

## 1. Events as Business Facts

The first question when designing a Domain Event is not how to publish it. It is whether the occurrence deserves to be represented as a domain fact.

Consider an online retail system.

When an order is placed, several things may follow:

- Inventory needs to be reserved.
- Payment needs to be authorized.
- A confirmation notification needs to be sent.
- A fulfillment process needs to begin.
- Sales reporting may need to be updated.

The order context does not necessarily need to know how every one of these operations is implemented.

It can record the business occurrence:

```text
OrderPlaced
```

Other components can react to that fact.

This gives us a model such as:

```text
                 PlaceOrder
                      |
                      v
                Order Aggregate
                      |
                      v
                  OrderPlaced
                      |
           +----------+----------+
           |          |          |
           v          v          v
       Inventory    Payment   Notification
        Process     Process      Process
```

The important point is that the event communicates a business occurrence, not a list of technical instructions.

Compare these examples:

| Technical change | Potential domain event |
|---|---|
| `Status = 2` | `OrderConfirmed` |
| `IsPaid = true` | `PaymentReceived` |
| `DriverId = 42` | `ShipmentAssignedToDriver` |
| `Quantity = 0` | `InventoryDepleted` |
| `Enabled = false` | `SubscriptionSuspended` |

The left column describes changes to data. The right column describes business meaning.

There are situations where changing a field is itself meaningful. For example, changing a customer's registered legal name may be an important business occurrence. But the event should still describe why that change matters, rather than merely report that a database column changed.

### Events should represent meaningful distinctions

Consider an order that moves through the following lifecycle:

```text
Draft → Placed → Confirmed → Dispatched → Delivered
```

It may be tempting to publish one generic event:

```text
OrderStatusChanged
```

But this loses important business distinctions.

An order being confirmed may trigger inventory allocation. An order being dispatched may trigger customer notifications and transportation coordination. An order being delivered may initiate invoicing or settlement.

These are different occurrences with different meanings and consequences.

Prefer:

```text
OrderConfirmed
OrderDispatched
OrderDelivered
```

when those distinctions matter to the domain.

A generic event is not always wrong. It may be appropriate for auditing arbitrary status changes or supporting a generic monitoring mechanism. But it should not replace meaningful business events when downstream behavior depends on those distinctions.

---

## 2. Commands, Events, and Queries

Domain Events become easier to understand when compared with Commands and Queries.

These are different kinds of messages with different responsibilities.

| Message | Meaning | Example |
|---|---|---|
| Command | Requests an action | `CancelOrder` |
| Event | Describes an occurrence | `OrderCancelled` |
| Query | Requests information | `GetOrderDetails` |

Consider a cancellation workflow.

```text
CancelOrder
     |
     v
Order Aggregate
     |
     +---- Validate cancellation rules
     |
     +---- Change order state
     |
     v
OrderCancelled
```

The command expresses an intention. The Aggregate decides whether that intention is valid. If the operation succeeds, the Aggregate records the resulting domain event.

For example:

```csharp
public void Cancel(CancellationReason reason)
{
    if (Status == OrderStatus.Delivered)
        throw new DomainException(
            "A delivered order cannot be cancelled.");

    if (Status == OrderStatus.Cancelled)
        throw new DomainException(
            "The order has already been cancelled.");

    Status = OrderStatus.Cancelled;

    AddDomainEvent(
        new OrderCancelled(Id, reason));
}
```

The command may fail. If it fails, `OrderCancelled` must not be raised.

This gives us an important distinction:

**Commands express what someone wants to happen. Events express what the domain has accepted as having happened.**

A Query, in contrast, asks for information without requesting a state change.

For example:

```text
GetOrderDetails
GetAvailableInventory
GetCustomerCreditLimit
```

A query should not secretly perform a business operation. Likewise, an event should not be treated as a command that asks its consumer to decide whether the original occurrence should have happened.

### A useful mental model

Imagine a conversation between people:

- "Please place this order." This is a command.
- "The order has been placed." This is an event.
- "What is the order's current status?" This is a query.

The difference is simple, but many architectural problems begin when these message types are mixed.

---

## 3. Where Should Domain Events Be Raised?

A Domain Event should normally be raised by the domain model that owns the business occurrence.

For example, if an Order Aggregate owns the rules for confirming an order, it should be responsible for recording `OrderConfirmed`.

```csharp
public class Order
{
    private readonly List<IDomainEvent> _domainEvents = [];

    public OrderId Id { get; private set; }

    public OrderStatus Status { get; private set; }

    public IReadOnlyCollection<IDomainEvent> DomainEvents
        => _domainEvents.AsReadOnly();

    public void Confirm()
    {
        if (Status != OrderStatus.Placed)
            throw new DomainException(
                "Only placed orders can be confirmed.");

        Status = OrderStatus.Confirmed;

        _domainEvents.Add(
            new OrderConfirmed(Id));
    }
}
```

The Aggregate knows that the order has been confirmed. It does not need to know who will react to that occurrence.

This keeps business behavior and event ownership together.

### Avoid raising domain events from arbitrary infrastructure code

Consider this implementation:

```csharp
public async Task ConfirmOrder(Guid orderId)
{
    var order = await _db.Orders.FindAsync(orderId);

    order.Status = OrderStatus.Confirmed;

    await _db.SaveChangesAsync();

    await _eventBus.Publish(
        new OrderConfirmed(orderId));
}
```

This code may work, but the business transition is implemented outside the domain model.

There is no guarantee that every caller will apply the same rules. Another part of the application could change the status without raising the event.

A better approach is to let the Aggregate enforce the transition and record the event. The application layer then collects and dispatches the resulting events.

### Domain events and Aggregate boundaries

An Aggregate protects its own invariants.

Suppose confirming an order requires that it has at least one item and that its customer is eligible.

Those rules belong within the Order Aggregate if the necessary information is part of its consistency boundary.

But confirming an order may also require reserving inventory or authorizing payment. Those are separate business responsibilities and may belong to other Aggregates or Bounded Contexts.

The Order Aggregate should not directly manipulate those other Aggregates merely to raise an event.

Instead, it can record `OrderConfirmed`, allowing an application workflow or another domain component to coordinate the subsequent operations.

An event communicates a fact. It does not eliminate the need to decide which component owns each business rule.

---

## 4. What Should a Domain Event Contain?

A Domain Event should contain enough information to describe the occurrence and allow interested components to react appropriately.

Consider:

```csharp
public sealed record OrderPlaced(
    OrderId OrderId,
    CustomerId CustomerId,
    DateTimeOffset OccurredAt);
```

This event communicates:

- Which order was placed.
- Which customer placed it.
- When the occurrence happened.

Depending on the business, it may also need to contain information such as the order's total amount, currency, or relevant delivery details.

The correct payload depends on the meaning of the event and the needs of its consumers.

### Prefer explicit, meaningful properties

Compare:

```csharp
public record OrderPlaced(
    Guid Id,
    string Data,
    int Type);
```

with:

```csharp
public sealed record OrderPlaced(
    OrderId OrderId,
    CustomerId CustomerId,
    Money TotalAmount,
    DateTimeOffset OccurredAt);
```

The second version communicates the business meaning directly.

A consumer should not need to reverse-engineer an opaque payload or query several unrelated tables just to understand what happened.

### Avoid exposing the entire Aggregate

A tempting shortcut is to include the entire Order object:

```csharp
public record OrderPlaced(Order Order);
```

This creates several problems:

- Consumers become coupled to the internal domain model.
- Serialization may expose internal implementation details.
- Navigational properties can create unexpectedly large payloads.
- Future model changes may break consumers.
- The event becomes difficult to evolve independently.

Prefer an explicit event contract containing the information that is relevant to the occurrence.

### Immutable facts

An event represents something that happened. Its meaning should not change after it has been created.

Immutable event types help preserve this property:

```csharp
public sealed record PaymentReceived(
    PaymentId PaymentId,
    OrderId OrderId,
    Money Amount,
    DateTimeOffset ReceivedAt);
```

Avoid modifying an event after publication or reusing an event instance for multiple occurrences.

If a payment is later refunded, do not modify `PaymentReceived` to indicate that it is no longer valid. Record another fact:

```text
PaymentReceived
PaymentRefunded
```

The original occurrence remains true. The later event adds new information about the business history.

### Identity and timestamps

Depending on the use case, an event may include:

- An event identifier.
- The identifier of the Aggregate that raised it.
- An occurrence timestamp.
- A correlation identifier.
- A causation identifier.
- A version or sequence number.

For example:

```csharp
public sealed record OrderPlaced(
    Guid EventId,
    Guid OrderId,
    Guid CustomerId,
    DateTimeOffset OccurredAt,
    Guid CorrelationId,
    Guid? CausationId);
```

These fields serve different purposes:

**Event ID** identifies the particular event occurrence. It is useful for deduplication and tracing.

**Aggregate ID** identifies the domain object associated with the event.

**OccurredAt** records when the business occurrence happened. It should not be confused with the time a message was published, delivered, or processed.

**Correlation ID** connects messages and operations belonging to the same business workflow.

**Causation ID** identifies the message or event that directly caused this event.

**Version or sequence number** can help establish ordering within a stream or Aggregate.

Not every event needs every field. Include metadata when it supports a real requirement rather than treating it as mandatory decoration.

---

## 5. Domain Events and Integration Events

A Domain Event and an Integration Event are related concepts, but they are not interchangeable.

A Domain Event communicates something meaningful within a domain model or Bounded Context.

An Integration Event communicates an occurrence to another system or Bounded Context through an explicit integration contract.

For example, an Order Context may raise:

```csharp
public sealed record OrderConfirmed(
    OrderId OrderId,
    CustomerId CustomerId,
    IReadOnlyList<OrderLine> Lines);
```

This may be appropriate inside the Order Context.

But exposing this exact type to an external Inventory Context could create unwanted coupling to the Order Context's internal model.

Instead, the Order Context may publish an integration contract:

```csharp
public sealed record OrderConfirmationPublished(
    Guid EventId,
    Guid OrderReference,
    IReadOnlyList<ConfirmedOrderItem> Items,
    DateTimeOffset OccurredAt);

public sealed record ConfirmedOrderItem(
    string ProductCode,
    decimal Quantity);
```

The integration event expresses the information the receiving context needs, using a deliberately designed contract.

The two events may represent the same business occurrence, but they serve different boundaries and may have different representations.

A possible flow:

```text
             Order Aggregate
                   |
                   v
             OrderConfirmed
              (Domain Event)
                   |
                   v
           Order Context Handler
                   |
                   v
       OrderConfirmationPublished
            (Integration Event)
                   |
                   v
             Message Broker
                   |
                   v
          Inventory Bounded Context
```

The translation between the two contracts is intentional.

### Why not publish every Domain Event?

Internal domain events may contain:

- Internal identifiers.
- Domain-specific Value Objects.
- Details that are irrelevant to external consumers.
- Concepts that may change during refactoring.
- Information that should not be exposed outside the context.

Publishing all of them creates accidental coupling.

An Integration Event should be designed as a public contract. Its name, payload, compatibility requirements, and ownership should be considered from the perspective of its consumers.

### Integration events should represent committed facts

An external system should not receive an event announcing that an order was confirmed if the transaction that confirms the order ultimately rolls back.

This is where transaction boundaries and reliable publication become important.

The next sections explore this problem.

---

## 6. Dispatching Domain Events

Raising an event and dispatching an event are two different operations.

Raising an event means the domain model has recorded that a meaningful occurrence took place.

Dispatching an event means delivering that occurrence to one or more interested handlers.

A common design is to collect events inside Aggregates and dispatch them after the relevant persistence operation.

```text
Application Command
       |
       v
Load Aggregate
       |
       v
Execute Domain Operation
       |
       v
Record Domain Event
       |
       v
Persist Changes
       |
       v
Dispatch Domain Events
       |
       v
Execute Event Handlers
```

A simplified implementation might look like:

```csharp
public interface IDomainEvent
{
    DateTimeOffset OccurredAt { get; }
}
```

```csharp
public abstract class AggregateRoot
{
    private readonly List<IDomainEvent> _domainEvents = [];

    public IReadOnlyCollection<IDomainEvent> DomainEvents
        => _domainEvents.AsReadOnly();

    protected void RaiseDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }

    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}
```

An Aggregate can raise events without depending on a message broker, mediator, or external infrastructure.

The application layer is responsible for dispatching them.

For example:

```csharp
public async Task Handle(
    ConfirmOrder command,
    CancellationToken cancellationToken)
{
    var order = await _orders.GetById(
        command.OrderId,
        cancellationToken);

    order.Confirm();

    await _unitOfWork.SaveChangesAsync(
        cancellationToken);

    await _domainEventDispatcher.DispatchAsync(
        order.DomainEvents,
        cancellationToken);

    order.ClearDomainEvents();
}
```

This example illustrates the responsibilities, but it is not a complete reliability strategy.

There is an important failure scenario:

1. The order is successfully committed to the database.
2. The application attempts to dispatch `OrderConfirmed`.
3. The process crashes before the event is delivered.
4. The order is confirmed, but the interested component never receives the event.

There is also the opposite danger if events are dispatched before the transaction commits: a consumer may act on an occurrence whose transaction is later rolled back.

This leads to an important distinction between in-process domain event dispatch and reliable integration event publication.

### Synchronous versus asynchronous handlers

Not every domain event handler needs to be asynchronous.

A synchronous handler may be useful when the reaction is part of the same local operation and must complete before that operation is considered successful.

For example, a local policy may update another part of the same domain model when a particular event occurs.

However, synchronous event handlers can make control flow difficult to follow, particularly when handlers trigger more events or perform external operations.

Asynchronous handlers are more suitable when the reaction can happen later, especially across process boundaries.

The choice should follow the consistency requirement:

- If the rule must hold immediately within the same consistency boundary, enforce it directly in the domain model.
- If the operation requires coordination within one local transaction, use an explicit local workflow where appropriate.
- If another component can react eventually, asynchronous processing may be appropriate.
- If an external system must receive the fact reliably, use a reliable publication mechanism.

Events should not be used to hide a dependency that actually requires immediate consistency.

---

## 7. The Transactional Outbox Pattern

One of the most important challenges with Integration Events is ensuring that database changes and event publication do not become inconsistent.

Consider this implementation:

```csharp
await _db.SaveChangesAsync();

await _messageBus.Publish(
    new OrderConfirmed(...));
```

There are two independent operations:

1. Commit the order change to the database.
2. Publish the integration event to a message broker.

What happens if the database commits successfully but the message broker is unavailable?

The order is confirmed, but the event is lost.

What happens if the event is published successfully but the database transaction fails?

Another system may react to an order confirmation that never actually happened.

The Transactional Outbox Pattern addresses this problem by storing the event in the same database transaction as the business change.

```text
            Confirm Order
                  |
                  v
             Order Aggregate
                  |
                  v
          Database Transaction
                  |
          +-------+-------+
          |               |
          v               v
     Update Order    Insert Outbox
          |             Message
          +-------+-------+
                  |
                  v
               COMMIT
                  |
                  v
          Outbox Background Worker
                  |
                  v
             Message Broker
                  |
                  v
             Consumers
```

A simplified outbox table might look like:

```sql
CREATE TABLE OutboxMessages
(
    Id UNIQUEIDENTIFIER NOT NULL PRIMARY KEY,
    Type NVARCHAR(500) NOT NULL,
    Payload NVARCHAR(MAX) NOT NULL,
    OccurredAt DATETIMEOFFSET NOT NULL,
    ProcessedAt DATETIMEOFFSET NULL,
    RetryCount INT NOT NULL DEFAULT 0
);
```

The application saves both the business change and the outgoing message in one database transaction.

A background worker reads unpublished messages and delivers them to the message broker.

If the process crashes before publication, the message remains in the outbox and can be retried.

The Outbox Pattern does not magically guarantee exactly-once delivery. A worker may publish a message successfully and crash before recording that it was published. The message may then be delivered again.

This is why consumers must also be designed to handle duplicates.

### When is an Outbox necessary?

The Outbox Pattern is especially valuable when:

- Database changes must reliably result in integration events.
- Events are published to a message broker.
- Losing an event would leave another context inconsistent.
- The system must recover from process or broker failures.

For simple in-process domain event handling within one transaction, a distributed messaging outbox may be unnecessary.

Choose the reliability mechanism based on the consequences of losing or duplicating the event.

---

## 8. Handling Duplicate Delivery and Idempotency

Message delivery systems commonly provide at-least-once delivery.

This means a message may be delivered more than once.

Consider:

```text
OrderConfirmed
```

An Inventory consumer receives the event and reserves inventory.

The consumer successfully reserves inventory, but the process crashes before acknowledging the message.

The broker delivers the same event again.

If the consumer reserves inventory a second time, the system may produce an incorrect result.

The consumer needs to recognize that it has already processed the event.

One common approach is to maintain an inbox or processed-message table.

```sql
CREATE TABLE ProcessedMessages
(
    ConsumerName NVARCHAR(200) NOT NULL,
    MessageId UNIQUEIDENTIFIER NOT NULL,
    ProcessedAt DATETIMEOFFSET NOT NULL,

    CONSTRAINT PK_ProcessedMessages
        PRIMARY KEY (ConsumerName, MessageId)
);
```

A consumer can check the message identifier before applying its business operation.

Conceptually:

```csharp
public async Task Handle(
    OrderConfirmedIntegrationEvent message,
    CancellationToken cancellationToken)
{
    if (await _inbox.HasProcessed(
        message.EventId,
        cancellationToken))
    {
        return;
    }

    await using var transaction =
        await _unitOfWork.BeginTransactionAsync(
            cancellationToken);

    await _inventory.Reserve(
        message.Items,
        cancellationToken);

    await _inbox.MarkProcessed(
        message.EventId,
        cancellationToken);

    await transaction.CommitAsync(cancellationToken);
}
```

The business effect and the processed-message record must be committed atomically when they share a transactional store.

Otherwise, a crash between the two operations can still create duplicate effects.

Idempotency can also be achieved through domain design.

For example, instead of blindly adding a reservation, a consumer can create a reservation with a unique business reference tied to the order. Repeating the operation then returns or recognizes the existing reservation.

The important idea is:

**Do not assume that a message is delivered only once. Design consumers so that processing the same event again does not produce an unintended business effect.**

---

## 9. Event Ordering and Concurrency

Not all events can be processed in arbitrary order.

Consider the lifecycle:

```text
OrderPlaced
OrderConfirmed
OrderDispatched
OrderDelivered
```

If `OrderDelivered` arrives before `OrderDispatched`, a consumer may not be able to process the message correctly.

However, ordering is not always globally required.

A system with thousands of orders does not necessarily need every event to be processed in one global sequence. It may only need ordering within a particular Aggregate or business process.

### Ordering by Aggregate

A useful approach is to associate events with an Aggregate identifier and a sequence number:

```csharp
public sealed record OrderDispatched(
    Guid EventId,
    Guid OrderId,
    long AggregateVersion,
    DateTimeOffset OccurredAt);
```

For example:

```text
OrderId: 123

Version 1: OrderPlaced
Version 2: OrderConfirmed
Version 3: OrderDispatched
Version 4: OrderDelivered
```

A consumer can use the version to detect duplicates, gaps, or out-of-order messages.

This does not automatically solve ordering. The message broker, partitioning strategy, producer, and consumer must all support the required ordering guarantees.

### Concurrency and competing updates

Suppose two operations attempt to confirm the same order simultaneously.

Both may read the order in the `Placed` state.

Without concurrency protection, both may attempt to perform the transition and raise `OrderConfirmed`.

Optimistic concurrency can prevent both updates from committing successfully.

The same principle applies to event streams: event sequence numbers or expected versions can help ensure that concurrent writers do not silently overwrite each other's work.

### Do not introduce global ordering unnecessarily

Global event ordering can severely limit throughput and create contention.

Start by identifying the actual business requirement:

- Must events for one order be ordered?
- Must events for one customer be ordered?
- Is ordering needed only within a particular workflow?
- Can consumers tolerate delayed or out-of-order events?
- Can they detect missing information and retry later?

Ordering is a business and technical requirement, not a default property of event-driven architecture.

---

## 10. Event Evolution and Schema Versioning

Events are contracts between producers and consumers.

Once an Integration Event is consumed by another system, changing its structure can affect independently deployed applications.

Consider the original contract:

```csharp
public sealed record OrderPlaced(
    Guid OrderId,
    decimal TotalAmount);
```

Later, the business introduces multiple currencies.

The event changes:

```csharp
public sealed record OrderPlaced(
    Guid OrderId,
    decimal TotalAmount,
    string Currency);
```

An older consumer may not understand the new field or may interpret the amount incorrectly.

A more explicit contract might use:

```csharp
public sealed record OrderPlacedV2(
    Guid OrderId,
    decimal TotalAmount,
    string Currency,
    DateTimeOffset OccurredAt);
```

But versioning every event for every change can also create unnecessary complexity.

Consider the following practices:

- Prefer additive changes when they preserve compatibility.
- Avoid changing the meaning of an existing field.
- Do not silently reinterpret an existing event.
- Define how consumers handle unknown fields.
- Use explicit versions when compatibility requires them.
- Consider translation or upcasting when historical events must be read using a newer model.
- Maintain compatibility for consumers that cannot be upgraded simultaneously.

### Event names and semantics are part of the contract

Changing `OrderPlaced` to `OrderSubmitted` may look like a simple rename.

But the two terms may describe different business occurrences.

Before changing an event name, investigate whether the domain meaning has changed or whether the original name was simply inaccurate.

Event schema evolution is not just a serialization problem. It is also a domain language problem.

---

## 11. Domain Events and Historical Truth

A Domain Event records an occurrence. That fact should remain true even when the business state changes later.

Consider:

```text
PaymentReceived
PaymentRefunded
```

The payment was received. Later, it was refunded.

The second event does not invalidate the first.

Likewise:

```text
SubscriptionActivated
SubscriptionSuspended
SubscriptionReactivated
SubscriptionCancelled
```

These events describe a sequence of business occurrences.

This is particularly important in systems that need auditability, historical reconstruction, or an explanation of how a particular state was reached.

However, recording Domain Events does not automatically make a system Event Sourced.

A conventional system may persist the current state of an Aggregate and dispatch domain events as part of its normal operation.

Event Sourcing is a separate persistence strategy in which the event stream becomes the authoritative record of state changes.

Do not confuse an event-driven design with Event Sourcing.

### Event facts and current state

Suppose an account has these events:

```text
AccountOpened
MoneyDeposited
MoneyWithdrawn
MoneyDeposited
```

The current balance may be calculated from the sequence, but a conventional application might instead persist the current balance and use the events for communication or auditing.

The important distinction is what the system treats as the authoritative source of truth.

Domain Events can exist without event-sourced persistence.

---

## 12. Domain Events and Sagas or Process Managers

A Domain Event can initiate or advance a business process that involves several Aggregates or Bounded Contexts.

Consider an order fulfillment process:

```text
OrderPlaced
     |
     v
ReserveInventory
     |
     +---- InventoryReserved
     |            |
     |            v
     |      AuthorizePayment
     |            |
     |            +---- PaymentAuthorized
     |                         |
     |                         v
     |                   ArrangeShipment
     |                         |
     |                         v
     |                   ShipmentCreated
     |
     +---- InventoryUnavailable
                  |
                  v
             Cancel Order
```

This process cannot necessarily be completed in one database transaction.

Each step may commit independently, and the overall workflow may take seconds, minutes, or even days.

A Process Manager or Saga can coordinate the sequence and handle failures.

For example, if payment authorization fails after inventory has been reserved, the process may need to release the reservation.

The important distinction is:

- Domain Events describe business occurrences.
- Commands request subsequent operations.
- A Process Manager or Saga coordinates a multi-step workflow.

Events do not automatically provide workflow orchestration.

A system that reacts to events without explicitly modeling process state may become difficult to understand when retries, timeouts, and compensating actions are introduced.

---

## 13. Common Domain Event Anti-Patterns

### 13.1 Event Everything

**Problem:** Every property update and method call generates an event.

```text
OrderNameChanged
OrderDescriptionChanged
OrderFieldUpdated
OrderPropertyModified
```

This creates noise rather than useful domain communication.

**Better approach:** Publish events for meaningful business occurrences. Use ordinary method calls or persistence mechanisms for changes that do not represent significant domain facts.

### 13.2 Events That Describe Commands

**Problem:** Events are named as requests.

```text
OrderShouldBePlaced
PaymentMustBeReceived
ShipmentNeedsToBeCreated
```

These are intentions, not facts.

**Better approach:** Use commands for requests and events for completed occurrences.

### 13.3 Publishing Before Commit

**Problem:** An event is published before the transaction that records the business change has committed.

A consumer may act on a fact that is subsequently rolled back.

**Better approach:** Use post-commit dispatch for appropriate local handlers and a Transactional Outbox for reliable integration publication.

### 13.4 Publishing Entire Domain Objects

**Problem:** External consumers receive internal Aggregates or Entities.

This exposes implementation details and creates tight coupling.

**Better approach:** Define explicit Integration Event contracts for cross-context communication.

### 13.5 Assuming Exactly-Once Delivery

**Problem:** Consumers assume each event will arrive only once.

Retries and failures can cause duplicate delivery.

**Better approach:** Use event identifiers, idempotent business operations, and inbox or deduplication mechanisms where needed.

### 13.6 Using Events to Hide Direct Dependencies

**Problem:** Developers replace every method call with an event, even when one component needs an immediate result from another.

This can obscure control flow and make failures difficult to reason about.

**Better approach:** Use direct calls for immediate collaboration within an appropriate boundary. Use events when communicating facts or coordinating appropriately asynchronous behavior.

### 13.7 Treating Event Handlers as Guaranteed

**Problem:** Developers assume that once an event is raised, every handler will eventually complete successfully.

Handlers can fail because of network outages, database errors, invalid messages, or deployment problems.

**Better approach:** Define retry, dead-letter, monitoring, and recovery strategies for asynchronous processing.

### 13.8 Reusing Events as Integration Contracts Without Review

**Problem:** Internal domain events are published externally without considering consumer needs or compatibility.

**Better approach:** Design Integration Events deliberately, and translate internal facts into stable external contracts.

---

## 14. Practical Examples

### Example 1: E-commerce Order

A customer places an order containing three products.

The Order Aggregate validates the operation and records:

```csharp
public sealed record OrderPlaced(
    Guid OrderId,
    Guid CustomerId,
    Money TotalAmount,
    DateTimeOffset OccurredAt);
```

The event may trigger:

- An inventory reservation process.
- A payment authorization process.
- A customer notification.
- An analytics update.

The Order Context does not need to know the internal implementation of these consumers.

### Example 2: Banking Transfer

A customer transfers money from one account to another.

Possible events:

```text
MoneyTransferRequested
MoneyDebited
MoneyCredited
MoneyTransferCompleted
MoneyTransferFailed
```

These events should not be treated as interchangeable.

A debit is not the same as a completed transfer. The receiving account may not have been credited yet.

The domain must distinguish the business states and failure scenarios.

A long-running transfer process may need a Process Manager to coordinate the debit, credit, and any compensating action.

### Example 3: Logistics and Shipment

A shipment moves through a delivery lifecycle:

```text
ShipmentCreated
VehicleAssigned
ShipmentDispatched
ShipmentArrivedAtDestination
DeliveryAttempted
ShipmentDelivered
```

Different events may trigger different processes:

- `VehicleAssigned` may notify the driver.
- `ShipmentDispatched` may update tracking.
- `DeliveryAttempted` may initiate a customer notification.
- `ShipmentDelivered` may initiate settlement or invoicing.

The event names describe business occurrences rather than changes to a generic status field.

### Example 4: Subscription Management

A subscription may have the following lifecycle:

```text
SubscriptionActivated
SubscriptionRenewalRequested
SubscriptionRenewed
SubscriptionPaymentFailed
SubscriptionSuspended
SubscriptionReactivated
SubscriptionCancelled
```

A failed payment does not necessarily mean that the subscription is cancelled.

The business may allow a grace period, retry payments, or suspend access temporarily.

Events help make these distinctions explicit and allow different parts of the system to react to the relevant facts.

### Example 5: Inventory Management

Consider a warehouse where stock is reserved for orders.

Possible events:

```text
StockReceived
StockReserved
StockReservationReleased
StockAdjusted
InventoryDepleted
```

A `StockReserved` event should communicate a completed reservation, not merely a request to reserve stock.

A request to reserve stock is a command. The Inventory Aggregate decides whether the reservation can be made.

---

## 15. Practical Exercises

### Exercise 1: Identify Commands and Events

Given the following operations, classify each as a Command, Domain Event, or Query:

```text
PlaceOrder
OrderPlaced
GetOrderDetails
CancelSubscription
SubscriptionCancelled
CalculateShippingCost
PaymentReceived
ReceivePayment
DispatchShipment
ShipmentDispatched
```

For each item, explain why it belongs to that category.

### Exercise 2: Design Domain Events for an Order Lifecycle

Consider this lifecycle:

```text
Draft → Placed → Confirmed → Dispatched → Delivered
```

Tasks:

1. Identify meaningful business occurrences.
2. Define a Domain Event for each relevant occurrence.
3. Decide which Aggregate owns each event.
4. Define the event properties.
5. Identify which occurrences should be exposed as Integration Events.
6. Explain which consumers may react to each event.

Do not assume that every state transition requires a separate event.

### Exercise 3: Separate Domain Events from Integration Events

Imagine an Order Bounded Context that communicates with Inventory and Payment contexts.

Tasks:

1. Define internal Domain Events.
2. Design public Integration Event contracts.
3. Identify which internal details should not be exposed.
4. Define how the contracts can evolve.
5. Explain how the receiving contexts handle unknown or duplicate events.

### Exercise 4: Implement an Outbox

Implement a simplified Transactional Outbox using a relational database.

Tasks:

1. Create an Outbox table.
2. Save the business change and outbox message in one transaction.
3. Implement a background publisher.
4. Add retry handling.
5. Track successful publication.
6. Simulate a process failure during publication.
7. Verify that unpublished messages can be recovered.

### Exercise 5: Build an Idempotent Consumer

Implement a consumer that handles `OrderConfirmed`.

Tasks:

1. Add an event identifier.
2. Store processed message identifiers.
3. Ensure the business effect and processed-message record are atomic.
4. Deliver the same message multiple times.
5. Verify that the business effect occurs only once.
6. Test what happens when processing fails before the transaction commits.

### Exercise 6: Model a Long-Running Business Process

Consider an order fulfillment workflow involving inventory, payment, and shipment.

Tasks:

1. Identify the business events.
2. Identify the commands that follow those events.
3. Define the process states.
4. Identify failure scenarios.
5. Design retry and compensation behavior.
6. Decide whether choreography or orchestration is more appropriate.

### Exercise 7: Review Event Quality

Choose an existing system and review its event contracts.

For each event, ask:

- Does it represent a meaningful business fact?
- Is its name clear and expressed in the past tense?
- Is its ownership clear?
- Does its payload contain meaningful information?
- Is it immutable?
- Does it expose internal implementation details?
- Is its delivery reliable enough for its purpose?
- Can consumers safely process it more than once?
- Are ordering requirements explicit?
- Can the contract evolve without breaking consumers?

---

## 16. Completion Criteria

You should be able to:

- Explain what makes an occurrence a Domain Event.
- Distinguish Commands, Queries, and Events.
- Identify meaningful business occurrences.
- Name events using clear domain language.
- Raise events from the domain model that owns the occurrence.
- Understand the relationship between Aggregates and Domain Events.
- Design immutable event payloads.
- Decide which metadata is useful for event identity, tracing, and ordering.
- Distinguish Domain Events from Integration Events.
- Understand synchronous and asynchronous event handling.
- Explain the purpose and limitations of the Transactional Outbox Pattern.
- Design consumers that handle duplicate delivery safely.
- Understand event ordering and concurrency requirements.
- Evolve event contracts without silently changing their meaning.
- Distinguish event-driven architecture from Event Sourcing.
- Understand how events participate in Sagas and Process Managers.
- Recognize common event-related anti-patterns.

The objective is not to turn every interaction into a message or to make every component asynchronous.

The objective is to make important business occurrences explicit, preserve their meaning, and communicate them across the right boundaries with an appropriate level of reliability.

**An event is not just a message that something changed. It is a statement about something that happened in the business. Make sure the system knows the difference.**