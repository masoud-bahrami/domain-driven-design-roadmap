# 04 Tactical Design

Strategic Design helps us understand the business, identify meaningful boundaries, and organize domain models. Tactical Design takes the next step: expressing those models in software.

This is where domain concepts become objects, behavior, rules, and executable decisions.

The purpose is not to reproduce the business vocabulary as a collection of classes. It is to build a model that captures the important business rules, protects consistency, and makes the domain understandable to the people who work with it.

A good tactical model does more than represent data. It expresses what the business allows, what it prevents, and what happens when meaningful business decisions are made.

Tactical Design provides a set of building blocks for doing this. Some are fundamental DDD patterns, such as Entities, Value Objects, Aggregates, and Domain Events. Others, such as CQRS, Event Sourcing, and Sagas, are complementary architectural patterns that may be useful when particular requirements justify them.

The distinction matters. DDD is not about applying every pattern. It is about choosing the patterns that help express and protect the domain.

## Goal

Learn how to implement a domain model using DDD tactical patterns.

By the end of this section, you should be able to:

- Represent domain concepts as Entities and Value Objects.
- Design Aggregates around business invariants and consistency boundaries.
- Express business behavior through domain objects and Domain Services.
- Separate domain behavior from application orchestration and infrastructure.
- Use Domain Events to represent meaningful occurrences.
- Design Repositories and Factories around domain model needs.
- Understand when CQRS, Event Sourcing, and Sagas are useful, and when their complexity is unnecessary.
- Test business behavior independently of implementation details.

## Key Concepts

### 1. Entities: Objects with Identity

An Entity is an object whose identity matters independently of its attributes.

Two Entities may have exactly the same properties and still represent different things.

For example, two customers may have the same name, email address, and postal address, but they are not necessarily the same customer.

Identity allows us to distinguish them throughout their lifecycle.

Consider a Customer:

- It has an identity.
- Its attributes may change over time.
- Its identity remains stable when those attributes change.
- It has behavior and rules that govern its lifecycle.

An Entity is therefore more than a database row with an ID. It represents something whose continuity matters to the domain.

#### Implementing Entities

When implementing an Entity, consider:

**Identity**

Choose an identity that uniquely distinguishes the Entity within its model. A `CustomerId`, `OrderId`, or `ShipmentId` is often more expressive than passing generic strings or integers throughout the domain.

**Lifecycle**

An Entity may be created, changed, activated, suspended, completed, or deleted according to domain rules. These transitions should be represented by meaningful behavior.

**Equality**

Entity equality is generally based on identity, not on all of its properties.

For example, two `Customer` objects with the same `CustomerId` represent the same logical Entity, even if one has an updated address.

Be careful when implementing equality for mutable Entities. Hash-based collections and ORM identity tracking can behave unexpectedly if equality or hash codes depend on mutable properties.

**Behavior**

Prefer methods that express meaningful domain operations.

Instead of allowing arbitrary changes to properties:

```csharp
customer.Address = newAddress;
customer.Status = CustomerStatus.Active;
```

Consider expressing the business operation:

```csharp
customer.ChangeAddress(newAddress);
customer.Activate();
```

This allows the Entity to protect its own rules.

An Entity does not need dozens of methods to be considered behavior-rich. Its behavior should reflect real domain decisions rather than artificial object-oriented ceremony.

### 2. Value Objects: Objects Defined by Their Values

A Value Object represents a concept whose identity is not important independently of its attributes.

Two Value Objects are considered equal when their values are equal according to the domain's definition of equality.

For example, an Address may consist of:

- Street
- City
- Postal code
- Country

If two Address instances contain the same meaningful values, they can represent the same address without needing separate identities.

Other examples include:

- Money
- Date ranges
- Coordinates
- Percentage
- Measurement
- Email address
- Shipping dimensions

#### Implementing Value Objects

A well-designed Value Object generally has the following characteristics:

**Immutability**

Once created, its values do not change. If a different value is needed, create a new instance.

**Value-Based Equality**

Equality is determined by the values that define the concept, rather than by a unique identifier.

**Self-Validation**

A Value Object should prevent invalid values from being created when the domain requires those constraints.

For example, a Money object should not allow an amount and currency combination that violates its defined rules.

**Explicit Meaning**

A Value Object makes domain concepts visible in code.

Compare:

```csharp
decimal price = 1500m;
```

with:

```csharp
Money price = Money.From(1500m, Currency.USD);
```

The second representation communicates more about the meaning and constraints of the value.

**Behavior**

Value Objects can contain behavior related to their values.

A Money object might implement addition and subtraction, provided that the operations respect currency rules.

#### Value Objects Are Not Just Immutable DTOs

Immutability alone does not make an object a Value Object.

A Value Object represents a meaningful concept in the domain, has value-based equality, and may enforce rules about its valid state.

Likewise, not every object without an ID needs to become a Value Object. Use the concept when it makes the model more expressive and protects meaningful constraints.

### 3. Aggregates and Aggregate Roots: Protecting Consistency

Aggregates are among the most important and most frequently misunderstood tactical patterns in DDD.

An Aggregate is a cluster of domain objects treated as a unit for consistency.

It defines a boundary within which business invariants must be maintained.

An Aggregate Root is the Entity that controls access to the Aggregate and protects its rules.

Consider an Order Aggregate:

```text
Order (Aggregate Root)
 |
 +-- OrderItem
 |
 +-- OrderItem
 |
 +-- OrderItem
 |
 +-- ShippingAddress (Value Object)
 |
 +-- Money (Value Object)
```

In this example, Order is the Aggregate Root. The other objects belong to its consistency boundary.

External code interacts with the Aggregate through the root rather than directly modifying its internal members.

For example:

```csharp
order.AddItem(productId, quantity);
order.ChangeShippingAddress(address);
order.Confirm();
```

The Order Aggregate is responsible for enforcing the rules that must hold whenever these operations occur.

#### Understanding Invariants

An invariant is a business rule that must remain true whenever the Aggregate is in a valid state.

Examples:

- An order cannot be confirmed without at least one item.
- An order cannot be modified after it has been cancelled.
- An order's total must reflect its items and applicable pricing rules.
- A shipment cannot be dispatched before it has been prepared.

The exact invariants depend on the domain. They must be discovered through domain exploration, not invented from database relationships.

#### Designing Aggregate Boundaries

A useful starting point is to ask:

- Which business rules must be enforced immediately and consistently?
- Which objects must change together to preserve those rules?
- Which decisions can be made independently?
- Can another Aggregate be referenced by identity rather than by embedding its entire object graph?
- What is the expected frequency of concurrent updates?

A common design principle is:

**Keep Aggregates as small as possible while preserving the invariants that must be enforced within their boundaries.**

This does not mean that every Aggregate must contain only one Entity. It means that objects should belong together because of consistency requirements, not merely because they are related in a database.

#### One Aggregate Root per Consistency Boundary

An Aggregate has one root through which external code accesses and modifies it.

An Aggregate may contain several Entities and Value Objects, but its internal objects should not be independently modified from outside.

For example, if Order contains OrderItems, external code should not directly load an OrderItem and change its quantity. It should ask the Order to perform the operation so that the relevant invariants are checked.

#### References Between Aggregates

A common approach is to reference another Aggregate by identity.

For example:

```text
Order
  CustomerId
  ProductId
  Items
```

The Order does not necessarily need to hold an entire Customer or Product object graph.

This reduces coupling and helps maintain clear consistency boundaries.

#### Transactions and Consistency

A fundamental Aggregate design principle is that changes within an Aggregate should be committed atomically.

A command that modifies an Aggregate should either preserve its invariants and succeed, or fail without leaving the Aggregate in an invalid state.

Changes involving multiple Aggregates generally require a different approach. They may be coordinated through application logic, Domain Events, or a distributed process when necessary.

Do not make every related Aggregate part of one large transactional boundary simply because the business operation involves them.

#### Aggregate Design Mistakes

Avoid these common mistakes:

- Designing Aggregates around database tables.
- Creating enormous Aggregates containing an entire business object graph.
- Treating every Entity relationship as a reason to include objects in the same Aggregate.
- Exposing mutable collections and internal Entities.
- Allowing application services to modify Aggregate state directly.
- Loading several Aggregates into memory just to update one small piece of information.
- Assuming that one business transaction must always be one database transaction.

An Aggregate is a consistency boundary, not a convenient container for related objects.

### 4. Domain Events: Representing What Happened

A Domain Event represents something meaningful that happened in the domain.

It is expressed in the past tense because it describes a fact, not a request.

Examples:

- `OrderPlaced`
- `PaymentReceived`
- `ShipmentDispatched`
- `CustomerAddressChanged`
- `CoursePublished`
- `StudentEnrolled`

A Domain Event captures a meaningful business occurrence and can allow other parts of the application to react to it.

Consider:

```text
Order.Confirm()
       |
       v
OrderConfirmed
       |
       +----> Update order history
       |
       +----> Initiate fulfillment
       |
       +----> Notify interested parties
```

The Order Aggregate is responsible for deciding whether confirmation is allowed. Once the business operation succeeds, it can record that the event occurred.

#### Domain Events vs. Commands

These concepts should not be confused.

| Command | Domain Event |
|---|---|
| Expresses an intention | Expresses a fact |
| Requests an operation | Records something that happened |
| May be rejected | Represents a completed occurrence |
| Usually imperative | Usually past tense |
| Example: `ConfirmOrder` | Example: `OrderConfirmed` |

A command asks the domain to do something. An event tells interested parties that something has happened.

#### Domain Events vs. Integration Events

A Domain Event belongs to a domain model and expresses an occurrence meaningful within that model.

An Integration Event is a contract used to communicate a relevant occurrence across a Bounded Context boundary.

They may represent the same business fact, but they have different responsibilities.

For example:

```text
Domain Model:
    OrderConfirmed

Application / Integration:
    OrderConfirmationPublished
```

The integration contract should be intentionally designed for consumers. It should not automatically expose internal domain classes or persistence structures.

#### Handling Domain Events

Domain Events are often collected by the Aggregate or domain model and dispatched by the application layer.

The event handler may:

- Invoke another application use case.
- Update another Aggregate.
- Initiate a business process.
- Create an integration message.
- Trigger an external notification.

The domain model should not need to know about message brokers, email providers, HTTP clients, or database infrastructure to express that an event occurred.

When event handling has external side effects, think carefully about transaction boundaries and delivery guarantees. A common solution is the Transactional Outbox pattern, which persists the business change and the message to be published in the same local database transaction.

### 5. Domain Services: Domain Behavior That Does Not Belong to One Object

Sometimes a business operation does not naturally belong to a single Entity or Value Object.

A Domain Service expresses a domain operation that involves multiple domain concepts or a business rule that does not fit naturally inside one object.

For example, consider a currency conversion operation:

```text
ConvertMoney(
    amount,
    targetCurrency,
    exchangeRate
)
```

The operation represents domain behavior, but it may not naturally belong to a particular Money instance if the conversion depends on a domain-specific exchange-rate policy.

Other possible examples include:

- Calculating an insurance premium using multiple domain concepts.
- Determining eligibility based on several business facts.
- Evaluating whether a particular allocation is valid.
- Calculating a complex pricing policy.

#### Characteristics of a Domain Service

A Domain Service:

- Expresses a meaningful domain operation.
- Uses the Ubiquitous Language.
- Encapsulates domain rules that do not naturally belong to an Entity or Value Object.
- Is generally stateless.
- Does not become a home for unrelated business logic.

A Domain Service should not simply be a class containing every operation that developers could not place elsewhere.

If a method clearly describes behavior of an Entity, put it there. If it describes a meaningful concept with its own values, consider a Value Object. Use a Domain Service when the operation genuinely belongs to neither.

### 6. Application Services: Coordinating Use Cases

Application Services coordinate the execution of use cases.

They define how the application uses the domain model to accomplish a particular task.

An Application Service may:

1. Receive a request.
2. Validate application-level input and permissions.
3. Load the required Aggregates.
4. Invoke domain behavior.
5. Persist changes.
6. Coordinate transactions.
7. Return an appropriate result.

Consider a `PlaceOrder` use case:

```text
PlaceOrder
    |
    +--> Load customer
    |
    +--> Load product information
    |
    +--> Create Order
    |
    +--> Invoke Order behavior
    |
    +--> Persist Order
    |
    +--> Commit transaction
```

The Application Service coordinates the operation. The Order Aggregate still owns the business rules that determine whether the order is valid.

#### Domain Service vs. Application Service

These two services are frequently confused.

| Domain Service | Application Service |
|---|---|
| Expresses domain behavior | Coordinates a use case |
| Contains domain rules | Coordinates execution and infrastructure |
| Belongs to the domain model | Belongs to the application layer |
| Uses domain concepts | Uses domain objects and application contracts |
| Should not depend on infrastructure | May depend on repositories, transaction abstractions, and external ports |

For example:

- `CalculateInsurancePremium` may be a Domain Service.
- `CreateInsurancePolicy` may be an Application Service.

The Application Service invokes the Domain Service when the use case requires it.

#### Keep Application Services Focused

Application Services should not become the place where all business rules live.

A common anti-pattern is a procedural service that loads database records, performs every business calculation, changes properties directly, and saves everything.

This often results in an anemic domain model: the objects contain data, while the application layer owns the actual business behavior.

Application Services should coordinate. Domain objects should protect their own rules.

### 7. Factories: Creating Valid Domain Objects

Factories encapsulate the creation of complex domain objects when construction requires more than simply calling a constructor.

They are useful when:

- An object requires several coordinated steps to be created.
- Creation involves nontrivial validation.
- The construction process should be hidden from the caller.
- A valid initial state must be guaranteed.
- The creation process depends on domain-specific policies.

For example, creating an Order might require checking that the initial items are valid, calculating an initial amount, and establishing the correct status.

A Factory can express this process:

```csharp
Order order = Order.Create(
    customerId,
    items,
    shippingAddress
);
```

The Factory may be a static creation method on the Aggregate Root, a dedicated Factory object, or an application-level component depending on the complexity.

Do not create a separate Factory class for every Entity by default. Simple objects can often be created directly through constructors or static factory methods.

The important responsibility is to ensure that an object enters the model in a valid state.

### 8. Repositories: Accessing Aggregates

A Repository provides an abstraction for retrieving and persisting domain objects, typically Aggregate Roots.

It allows the domain and application layers to work with domain concepts without being coupled directly to persistence technology.

For example:

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(
        OrderId id,
        CancellationToken cancellationToken);

    Task AddAsync(
        Order order,
        CancellationToken cancellationToken);
}
```

The implementation might use Entity Framework Core, SQL, a document database, or another persistence mechanism.

The interface describes what the application needs. The infrastructure implementation describes how persistence is performed.

#### Repository Responsibilities

A Repository commonly provides:

- Retrieval of an Aggregate by identity.
- Retrieval of Aggregates using meaningful selection criteria.
- Adding a new Aggregate.
- Persistence of changes to an existing Aggregate.

Repositories should normally be defined around Aggregate Roots, rather than providing independent repositories for every internal Entity.

For example, if `OrderItem` belongs to the Order Aggregate, external application code should not normally use an `OrderItemRepository` to modify it independently.

#### Repository vs. DAO

A Data Access Object is primarily concerned with data access and persistence operations.

A DDD Repository provides a collection-like abstraction around Aggregate persistence and retrieval.

The distinction is about responsibility, not naming. Calling a generic database wrapper `Repository` does not automatically make it a DDD Repository.

#### Repository Design Considerations

**Keep the interface focused**

Expose operations that make sense for the domain and its use cases.

Avoid creating a massive generic repository with dozens of methods that every Aggregate must inherit.

**Avoid leaking persistence concerns**

Domain objects should not need to know whether they are stored in SQL Server, MongoDB, or another persistence technology.

**Be careful with generic CRUD**

Operations such as `Update(T entity)` and `Delete(T entity)` can encourage persistence-oriented behavior that ignores the Aggregate's business rules.

**Consider optimistic concurrency**

When multiple requests may modify the same Aggregate, concurrency control is important. A version or concurrency token can help detect conflicting changes.

**Do not confuse repositories with queries**

A Repository is generally concerned with retrieving and persisting Aggregates for domain operations. Reporting and read-heavy queries may be better served by dedicated query models.

#### Specification Pattern

The Specification pattern can represent reusable selection or business criteria.

For example, a specification may express which orders are eligible for a particular business operation.

However, not every query needs a Specification. For simple retrieval requirements, explicit repository methods or dedicated query handlers may be clearer.

### 9. Modules: Organizing the Domain Model

As a domain model grows, its classes need a meaningful organization.

Modules group related concepts and responsibilities together.

They can help express the structure of the domain and reduce unnecessary dependencies between unrelated concepts.

For example:

```text
Sales
    Order
    OrderItem
    PricingPolicy
    OrderRepository

Shipping
    Shipment
    DeliveryAddress
    ShippingPolicy
    ShipmentRepository
```

The organization should reflect meaningful domain responsibilities rather than simply grouping all Entities in one folder, all Services in another, and all Repositories in a third.

Namespaces, packages, assemblies, and modules are implementation mechanisms. The important thing is that the code structure communicates the model and protects its boundaries.

A module is not automatically a Bounded Context. A Bounded Context is a semantic model boundary; modules are one way to organize implementation within it.

---

## Complementary Architectural Patterns

The following patterns are frequently used alongside DDD. They are useful in particular situations, but they are not prerequisites for having a domain-driven application.

### 10. CQRS: Separating Commands and Queries

Command Query Responsibility Segregation (CQRS) separates the responsibility for changing state from the responsibility for reading state.

The central idea is simple:

- A Command expresses an intention to change the system.
- A Query requests information without changing the system.

A command may invoke an Aggregate and enforce its business rules. A query may retrieve a DTO or read model optimized for a particular screen or report.

#### Basic CQRS

CQRS does not require separate databases, message brokers, microservices, or Event Sourcing.

A simple implementation can use separate command and query handlers while sharing the same database.

```text
                 Application
                      |
            +---------+---------+
            |                   |
         Commands             Queries
            |                   |
       Domain Model          Read Model
            |                   |
            +---------+---------+
                      |
                   Database
```

The write side focuses on business behavior and consistency. The read side focuses on retrieving information in a useful shape.

#### Advanced CQRS

In more demanding systems, the read and write models may use different data stores or schemas.

For example:

- The write model may use relational tables designed around Aggregate persistence.
- The read model may use denormalized tables designed for dashboards and reports.
- Read models may be updated asynchronously from published events.

This can improve independent scalability and query performance, but it introduces additional synchronization and operational responsibilities.

#### Benefits

- Different models can serve different read and write requirements.
- Complex write rules can remain isolated from read projections.
- Read models can be optimized for specific use cases.
- Read and write workloads can be scaled independently when necessary.

#### Trade-offs

- More code and infrastructure.
- Additional synchronization responsibilities.
- Potential eventual consistency between write and read models.
- More complicated debugging and testing.
- Additional operational and monitoring requirements.

**When to use CQRS:** When the read and write responsibilities have meaningfully different needs, such as complex business commands combined with diverse, performance-sensitive queries.

Do not introduce CQRS simply because a framework makes it easy to create command and query handlers.

### 11. Event Sourcing: Persisting the History of Changes

Event Sourcing is a persistence approach in which changes to an Entity or Aggregate are stored as an ordered sequence of events.

Instead of storing only the latest state, the system stores the events that describe how that state came to exist.

Consider a bank account.

A conventional persistence model might store:

```text
AccountId: 123
Balance: 500
```

An event-sourced model might store:

```text
AccountOpened
MoneyDeposited(1000)
MoneyWithdrawn(300)
MoneyWithdrawn(200)
```

The current balance is derived by applying the events in sequence.

#### Event Store

An Event Store stores an append-only stream of events.

A stream is commonly associated with an Aggregate identity.

When loading an Aggregate, the system reconstructs its state by applying its historical events.

This process is called rehydration.

#### Aggregate Reconstitution

Conceptually:

```text
Initial State
     |
     v
AccountOpened
     |
     v
MoneyDeposited
     |
     v
MoneyWithdrawn
     |
     v
Current Account State
```

The Aggregate's behavior still protects business invariants. Event Sourcing changes how its state is persisted, not the need for a meaningful domain model.

#### Snapshots

Replaying a very long event stream can become expensive.

A snapshot stores a previously calculated state at a particular event position. The system loads the snapshot and replays only subsequent events.

A snapshot is an optimization. The event history remains the source of truth.

#### Event Versioning

Persisted events are long-lived contracts within the system.

Their schemas may need to evolve as the domain and application change.

Common approaches include:

- Tolerant deserialization for compatible changes.
- Versioned event schemas.
- Upcasting older event representations during reading.
- Carefully managed historical migrations.

Changing an event schema is not equivalent to changing an ordinary database table. Historical data must remain interpretable.

#### Event Sourcing and CQRS

Event Sourcing is frequently combined with CQRS, but they are separate patterns.

- CQRS separates command and query responsibilities.
- Event Sourcing stores changes as an event history.

You can use CQRS without Event Sourcing. You can also use Event Sourcing without adopting a fully separate read and write infrastructure.

#### Trade-offs

Event Sourcing introduces substantial complexity:

- Event schema evolution.
- Event replay and rehydration.
- Concurrency and stream version management.
- Snapshot management.
- Projection rebuilding.
- Event delivery and duplicate handling.
- Debugging historical behavior.
- Data privacy and retention concerns.

It can be valuable when the history and intent behind changes are important business information, such as in financial ledgers, audit-intensive workflows, or systems where historical reconstruction is a major requirement.

It is not a default persistence strategy for every DDD application.

Also, a message broker such as Kafka is not automatically an Event Store. A broker distributes messages, while an Event Store must support the requirements of persistent aggregate streams, including appropriate retrieval and concurrency behavior.

### 12. Sagas: Coordinating Long-Running Business Processes

A Saga coordinates a business process that spans multiple local transactions, often across Aggregates or Bounded Contexts.

A normal database transaction can provide atomicity within its transactional boundary. But a process involving multiple independently owned contexts cannot usually rely on one shared ACID transaction.

Consider an order fulfillment process:

```text
Order Accepted
      |
      v
Reserve Inventory
      |
      v
Authorize Payment
      |
      v
Prepare Shipment
      |
      v
Dispatch Shipment
```

Each step may be a separate operation with its own persistence and failure conditions.

If a later step fails, the system needs to decide how to respond to the work that has already been completed.

A Saga defines that coordination.

#### Compensation

A compensating action is a business operation that counteracts the effect of an earlier step.

For example:

- Inventory reserved → Release reservation.
- Payment authorized → Void authorization.
- Shipment scheduled → Cancel shipment, if cancellation is still possible.

Compensation is not necessarily a literal rollback.

If a payment has already been settled, refunding it is a new business transaction, not an erasure of the original payment. If a shipment has physically left the warehouse, it may be impossible to undo dispatch.

The domain must define what recovery means for each step.

#### Orchestration

In orchestration, a coordinator controls the workflow.

It decides which operation should happen next, tracks progress, and determines what to do when a step fails.

```text
              Order Saga
                  |
          +-------+-------+
          |       |       |
       Inventory Payment Shipping
```

**Advantages:**

- The workflow is explicit.
- Complex process logic has a clear owner.
- Progress and failure handling are easier to observe centrally.

**Trade-offs:**

- The orchestrator introduces another component to maintain.
- It must persist its state and recover correctly.
- It can become overly coupled to individual participants if poorly designed.

#### Choreography

In choreography, participants react to events and publish events of their own.

```text
OrderAccepted
      |
      v
InventoryReserved
      |
      v
PaymentAuthorized
      |
      v
ShipmentPrepared
```

Each participant knows which events it should handle and what action to perform.

**Advantages:**

- No central coordinator is required.
- Participants can remain relatively independent.
- Simple workflows can be implemented with fewer coordination components.

**Trade-offs:**

- The complete workflow can become difficult to understand.
- Dependencies may become implicit.
- Cyclic event relationships can emerge.
- Monitoring and troubleshooting can become harder as the process grows.

#### When to Use a Saga

Use a Saga when a business process genuinely spans multiple local transactions and requires explicit coordination and recovery behavior.

Do not introduce a Saga for a simple operation that can be completed safely within one local transaction.

A Saga is a distributed process-management pattern, not a replacement for good Aggregate design.

---

## Testing DDD Applications

Testing should focus on the behavior and rules of the domain, not only on the structure of the implementation.

The most valuable tests should help answer:

- Does the model enforce the correct business rules?
- Can invalid operations be prevented?
- Are state transitions correct?
- Are important domain events raised?
- Do application use cases coordinate the required operations?
- Do persistence and integration mechanisms preserve the intended behavior?

### 13. Unit Testing the Domain Model

Domain objects should be testable without requiring a database, message broker, web server, or external service.

Test Entities, Value Objects, Aggregates, and Domain Services through their public behavior.

For example:

```text
Given an order with no items
When the customer attempts to confirm it
Then the order must reject the operation
```

Another example:

```text
Given an order containing two items
When the customer removes one item
Then the order total must reflect the remaining item
```

These tests should focus on business outcomes rather than internal implementation details.

### 14. Testing Aggregate Invariants

Aggregates deserve particular attention because they protect consistency boundaries.

Test:

- Valid state transitions.
- Invalid state transitions.
- Business constraints.
- Concurrent modification behavior where relevant.
- Domain Events produced by successful operations.
- Operations that should be rejected.

For example, an Order should not allow an item to be added after the order has entered a state where modification is prohibited.

### 15. Integration Testing

Unit tests cannot prove that persistence, messaging, transactions, and infrastructure work correctly together.

Integration tests should cover important boundaries, such as:

- Repository persistence and retrieval.
- ORM mapping of Aggregates and Value Objects.
- Transaction behavior.
- Optimistic concurrency.
- Domain Event dispatch.
- Integration message publication.
- Message consumer idempotency.
- Read model updates.

Avoid writing only mock-based tests for repositories and then assuming the actual database mapping works.

### 16. Behavior-Driven Development

Behavior-Driven Development (BDD) helps express expected system behavior through examples that stakeholders and developers can understand.

A scenario might look like:

```gherkin
Scenario: Confirming an order with valid items

  Given an order containing at least one valid item
  When the customer confirms the order
  Then the order should be confirmed
  And an OrderConfirmed event should be recorded
```

BDD can be used at different levels:

- Business examples and discovery.
- Acceptance criteria.
- Automated acceptance tests.
- Integration tests.

A Gherkin scenario is not automatically a valuable BDD test. The quality comes from meaningful examples, shared understanding, and a useful feedback loop.

### 17. Testing Event-Sourced Systems

Event-sourced systems benefit from a particular style of testing:

1. Given a sequence of historical events.
2. When a command is executed.
3. Then the expected new events should be produced.

For example:

```text
Given:
    AccountOpened
    MoneyDeposited(1000)

When:
    WithdrawMoney(300)

Then:
    MoneyWithdrawn(300)
```

Also test:

- Event rehydration.
- Event schema compatibility.
- Projection correctness.
- Duplicate event handling.
- Optimistic concurrency.
- Recovery after processing failures.

Testing an event-sourced system requires more than verifying the Aggregate's behavior. The event store, projections, and delivery mechanisms need their own integration tests.

---

## A Practical Implementation Structure

DDD does not mandate one architecture or folder structure. However, a layered or ports-and-adapters approach can help keep domain logic independent of infrastructure.

A possible organization for a modular application is:

```text
Sales
|
+-- Domain
|   +-- Orders
|   |   +-- Order.cs
|   |   +-- OrderItem.cs
|   |   +-- OrderId.cs
|   |   +-- OrderStatus.cs
|   |   +-- IOrderRepository.cs
|   |
|   +-- Pricing
|   |   +-- Money.cs
|   |   +-- PricingPolicy.cs
|   |
|   +-- Events
|   |   +-- OrderConfirmed.cs
|
+-- Application
|   +-- Orders
|   |   +-- PlaceOrder
|   |   +-- ConfirmOrder
|   |   +-- CancelOrder
|   |
|   +-- EventHandlers
|
+-- Infrastructure
|   +-- Persistence
|   |   +-- OrderRepository.cs
|   |   +-- SalesDbContext.cs
|   |
|   +-- Messaging
|
+-- Presentation
    +-- Controllers
    +-- Contracts
```

This is an example, not a required template.

The domain should not depend on Entity Framework, HTTP, message brokers, or other infrastructure details.

The Application layer coordinates use cases and depends on domain abstractions.

Infrastructure implements persistence and external communication mechanisms.

Presentation exposes the application through APIs, user interfaces, or other entry points.

The exact organization should reflect the needs and boundaries of the system.

---

## Practical Exercise: Model and Implement a Business Operation

Choose one important business operation from a domain you understand.

For example:

- Placing and confirming an order.
- Enrolling a student in a course.
- Issuing a shipment.
- Allocating a vehicle to a delivery.
- Approving an insurance claim.
- Reserving a financial transaction.

Avoid choosing a trivial CRUD operation. The purpose is to discover and implement meaningful business rules.

### Part A: Discover the Rules

Describe the operation in business language.

Identify:

1. The actor initiating the operation.
2. The intention behind the operation.
3. The conditions that must be satisfied.
4. The rules that must remain true.
5. The possible outcomes.
6. The important state transitions.
7. The exceptional and rejected cases.

Write concrete examples for both successful and unsuccessful scenarios.

### Part B: Design the Domain Model

Identify:

- Entities and their identities.
- Value Objects and their equality rules.
- Candidate Aggregates and their Aggregate Roots.
- Invariants that each Aggregate must protect.
- Domain Events representing important occurrences.
- Domain Services for operations that do not naturally belong to one object.
- Factories where object creation requires meaningful domain logic.

For every Aggregate, explain why its objects belong together and which consistency rules justify its boundary.

### Part C: Implement the Model

Implement the operation using your chosen programming language.

Your implementation should:

- Keep business rules inside the domain model.
- Prevent invalid states through controlled behavior.
- Use immutable Value Objects where appropriate.
- Protect Aggregate internals.
- Represent meaningful business events.
- Avoid coupling the domain to infrastructure.
- Use Repositories for Aggregate persistence where appropriate.

Do not introduce CQRS or Event Sourcing just to demonstrate that you know the patterns. Start with the simplest architecture that preserves the domain rules.

### Part D: Implement the Use Case

Create an Application Service or use-case handler that:

1. Receives the request.
2. Retrieves the required Aggregates.
3. Invokes domain behavior.
4. Persists the changes.
5. Coordinates the transaction.
6. Handles the resulting application-level responsibilities.

If the operation spans multiple Aggregates or Bounded Contexts, explain how you coordinate it and what consistency guarantees are required.

### Part E: Test the Model

Write tests covering:

- Successful execution.
- Invalid operations.
- Aggregate invariants.
- Entity and Value Object behavior.
- Expected Domain Events.
- Repository persistence.
- Application use-case behavior.

If you use asynchronous integration, add tests for duplicate delivery, retries, and failure handling where relevant.

### Questions to Challenge Your Design

- Does this object really need an identity?
- Would a Value Object express the concept more accurately?
- Which business invariant justifies this Aggregate boundary?
- Can the Aggregate be smaller without violating its invariants?
- Does the application service contain business rules that belong in the domain?
- Is this operation a Domain Service or an Application Service?
- Is the Domain Event describing a fact or merely exposing an internal implementation detail?
- Does the Repository represent an Aggregate, or is it simply a generic database wrapper?
- Do you actually need CQRS, Event Sourcing, or a Saga?
- Can the domain model be tested without starting infrastructure?

---

## Completion Criteria

You have completed this section when you can:

- Explain the difference between Entities and Value Objects, including their equality semantics.
- Implement domain concepts with meaningful behavior rather than relying exclusively on mutable data structures.
- Identify business invariants and enforce them within the appropriate Aggregate.
- Explain why an Aggregate is a consistency boundary rather than a database relationship or object graph.
- Design Aggregates around business rules and concurrency requirements.
- Explain the role of an Aggregate Root.
- Use Domain Events to represent meaningful business occurrences.
- Distinguish Domain Events, Integration Events, and Commands.
- Explain when a Domain Service is appropriate and how it differs from an Application Service.
- Design Factories and Repositories around domain responsibilities.
- Keep domain logic independent of persistence and infrastructure.
- Explain CQRS and distinguish it from Event Sourcing.
- Explain the operational and modeling trade-offs of Event Sourcing.
- Understand how Sagas coordinate long-running business processes and why compensation is not the same as rollback.
- Write behavior-focused unit tests and meaningful integration tests.
- Implement at least one business operation end to end using a coherent domain model.

## Recommended Resources

### Foundational Books

- **[Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.domainlanguage.com/ddd/)** — Eric Evans. The original reference for DDD's tactical building blocks and the reasoning behind them.
- **[Implementing Domain-Driven Design](https://www.oreilly.com/library/view/implementing-domain-driven-design/9780133039900/)** — Vaughn Vernon. A detailed, implementation-oriented guide covering Entities, Value Objects, Services, Domain Events, Aggregates, Factories, Repositories, and application design.
- **[Patterns, Principles, and Practices of Domain-Driven Design](https://www.wiley.com/en-us/Patterns%2C+Principles%2C+and+Practices+of+Domain-Driven+Design-p-9781118714706)** — Scott Millett and Nick Tune. A broad treatment of tactical patterns, modeling practices, and implementation decisions.

### Architecture and Implementation

- **[Microsoft: .NET Microservices Architecture](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/)** — Practical guidance on implementing domain models, Aggregates, Domain Events, CQRS, and application layers in .NET.
- **[Domain Events: Design and Implementation](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)** — Microsoft. Explains Domain Events, event handlers, and their relationship to the application layer.
- **[CQRS Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)** — Microsoft Azure Architecture Center. Explains the different levels of CQRS and the trade-offs involved.
- **[Event Sourcing Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)** — Microsoft Azure Architecture Center. Covers event stores, rehydration, snapshots, versioning, and operational considerations.
- **[Saga Design Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga)** — Microsoft Azure Architecture Center. Explains orchestration, choreography, compensation, and failure handling.
- **[Patterns of Enterprise Application Architecture](https://martinfowler.com/books/eaa.html)** — Martin Fowler. A valuable reference for Repository, Data Mapper, Unit of Work, and other enterprise application patterns.

### Further Reading

- **[Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html)** — A useful discussion of CQRS and why it should not be applied indiscriminately.
- **[Martin Fowler: Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)** — An influential introduction to storing changes as a sequence of events.
- **[Microsoft: Domain Model Layer](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model)** — An explanation of the role of the domain model in a layered application.

---

Tactical Design is not about making the code look like a DDD textbook. It is about ensuring that the software preserves the meaning of the business.

An Entity should have a reason to exist. A Value Object should express a meaningful concept. An Aggregate should protect a real consistency boundary. A Domain Event should represent something that matters. A Repository should support the model rather than dictate it.

The goal is not to maximize the number of patterns in the codebase. It is to make the domain's rules explicit, executable, and difficult to violate.