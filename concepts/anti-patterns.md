# Anti-patterns

Domain-Driven Design provides principles and patterns for building software around a deep understanding of the business domain. However, adopting DDD does not automatically produce a good domain model, a maintainable architecture, or successful software.

It is entirely possible to use Aggregates, Repositories, Domain Events, Bounded Contexts, and Value Objects while building a system that is difficult to understand, expensive to change, and disconnected from the business.

Sometimes the problem is not the absence of DDD patterns. It is how they are applied.

An anti-pattern is a recurring approach to solving a problem that appears reasonable in the short term but creates undesirable consequences over time. It may simplify an immediate implementation while introducing hidden coupling, unnecessary complexity, conceptual confusion, or architectural constraints.

Some anti-patterns directly contradict DDD principles. Others are broader software design problems that frequently appear in DDD implementations.

The purpose of this chapter is not to establish another collection of rules that developers must follow blindly. It is to develop the ability to recognize when a modeling or architectural decision is working against the problem it was intended to solve.

A pattern is useful when it helps express and protect the domain. The same pattern can become an anti-pattern when applied without understanding its purpose.

---

## 1. Ignoring the Ubiquitous Language (Lost in Translation)

**The problem:** The development team and domain experts use different terminology, or use the same words with different meanings.

One of the fundamental ideas in DDD is that the language used by the business should influence the language used in the software model.

When developers translate business conversations into technical terminology without preserving the underlying meaning, the software gradually becomes disconnected from the domain.

Consider a logistics business.

A domain expert talks about:

- Assigning a vehicle to a shipment.
- Dispatching a shipment.
- Delivering a shipment.
- Confirming proof of delivery.

But the software uses:

```csharp
UpdateRecord();
ChangeStatus();
ProcessData();
ExecuteOperation();
```

These method names reveal almost nothing about the business.

A more expressive model might use:

```csharp
shipment.AssignVehicle(vehicleId);
shipment.Dispatch();
shipment.MarkAsDelivered();
shipment.ConfirmProofOfDelivery();
```

The difference is not cosmetic. The second model communicates business intent and makes the code easier to discuss with domain experts.

There is another, more subtle problem: the same word can mean different things to different people.

For example, what does *delivered* mean?

- The driver says the goods were delivered.
- The customer signed a delivery receipt.
- The delivery document was uploaded.
- The recipient confirmed the quantity.
- The delivery was financially settled.

These are not necessarily the same event or state.

If the team does not investigate the meaning, developers may implement a state transition that looks correct technically but represents the wrong business decision.

**Consequences:**

- Business rules are misunderstood or implemented incorrectly.
- Different parts of the system use inconsistent terminology.
- Conversations become dependent on explanations and translations.
- Code communicates technical implementation instead of business intent.
- New requirements become harder to discuss because the model lacks a shared vocabulary.

**How to address it:**

- Work directly with domain experts.
- Explore examples, exceptions, and real business scenarios.
- Identify ambiguous terms and document their different meanings.
- Use the language consistently in code, tests, documentation, and conversations.
- Treat the glossary as a living artifact rather than a document created once and forgotten.
- Revisit terminology when the business model changes.

Most importantly, do not assume that agreeing on a word means agreeing on its meaning.

A shared language is not a dictionary. It is a shared understanding of the decisions and distinctions that matter.

---

## 2. Anemic Domain Model (The Hollow Shell)

**The problem:** Domain objects contain data but have little or no behavior. Business rules are implemented elsewhere, often in application services or generic service classes.

Consider this implementation:

```csharp
public class BankAccount
{
    public decimal Balance { get; set; }
    public AccountStatus Status { get; set; }
}
```

And a separate service:

```csharp
public class BankAccountService
{
    public void Withdraw(
        BankAccount account,
        decimal amount)
    {
        if (account.Status != AccountStatus.Active)
            throw new Exception("Account is not active.");

        if (account.Balance < amount)
            throw new Exception("Insufficient funds.");

        account.Balance -= amount;
    }
}
```

The `BankAccount` object does not protect its own business rules. Any caller can modify its balance directly.

As the application grows, more services begin implementing account-related behavior:

```text
BankAccountService
 ├── Withdraw()
 ├── Deposit()

PaymentService
 ├── DeductFromAccount()

TransferService
 ├── TransferBetweenAccounts()

AdminService
 ├── AdjustBalance()
```

The business rules are scattered across classes that happen to know how to manipulate account data.

The result is procedural code disguised as an object-oriented model.

A more behavior-oriented model exposes meaningful operations:

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; }
    public AccountStatus Status { get; private set; }

    public void Withdraw(decimal amount)
    {
        if (Status != AccountStatus.Active)
            throw new DomainException(
                "Account is not active.");

        if (amount <= 0)
            throw new DomainException(
                "Withdrawal amount must be positive.");

        if (Balance < amount)
            throw new DomainException(
                "Insufficient funds.");

        Balance -= amount;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new DomainException(
                "Deposit amount must be positive.");

        if (Status != AccountStatus.Active)
            throw new DomainException(
                "Account is not active.");

        Balance += amount;
    }
}
```

Now the object controls its own state transitions.

**Consequences:**

- Business rules become scattered and duplicated.
- Different callers may implement the same operation differently.
- Invalid states become easier to create.
- The domain model communicates little about the business.
- Testing domain behavior becomes harder because behavior is distributed across services.

**How to address it:**

- Identify business operations and place behavior where it naturally belongs.
- Encapsulate state that must change according to domain rules.
- Prefer meaningful methods over unrestricted setters.
- Keep application services focused on orchestration.
- Use Domain Services when behavior genuinely belongs to the domain but does not naturally belong to one Entity or Value Object.

One important qualification: not every domain model needs to be rich and object-oriented.

Functional domain models can express behavior through functions and immutable data structures. Some domains also have relatively simple rules that do not justify elaborate objects.

The anti-pattern is not using services. It is separating data from the rules that give that data meaning without a good reason.

---

## 3. God Objects and the Big Ball of Mud (The Everything Doer)

**The problem:** One class, module, or component accumulates too many responsibilities and becomes the center of unrelated business operations.

Consider a system with an `OrderService` that handles:

- Creating orders.
- Calculating discounts.
- Reserving inventory.
- Processing payments.
- Generating invoices.
- Sending notifications.
- Arranging shipments.
- Updating customer loyalty points.
- Handling refunds.
- Producing reports.

It might initially look convenient:

```csharp
public class OrderService
{
    public void CreateOrder() { }
    public void CalculateDiscount() { }
    public void ReserveInventory() { }
    public void ProcessPayment() { }
    public void GenerateInvoice() { }
    public void SendNotification() { }
    public void ArrangeShipment() { }
    public void RefundPayment() { }
    public void UpdateLoyaltyPoints() { }
}
```

But the class is no longer responsible for one coherent capability. It has become an accumulation point for everything related to orders.

A change to the payment workflow may affect order creation. A change to notification infrastructure may require modifying the same class. Testing one operation may require setting up unrelated dependencies.

A similar problem can occur at a larger scale.

A single application module may contain customer management, inventory, billing, shipping, reporting, and authentication, all tightly coupled through shared entities and services.

This is often described as a Big Ball of Mud.

**Consequences:**

- Changes in one area have unpredictable effects elsewhere.
- Dependencies become difficult to understand.
- Business responsibilities are unclear.
- Tests require increasingly complicated setup.
- Teams cannot work independently.
- Refactoring becomes risky because everything depends on everything else.

**How to address it:**

- Identify the distinct responsibilities and business capabilities.
- Move behavior into appropriate domain objects and services.
- Introduce clear module boundaries.
- Separate independent Bounded Contexts when the domain and organizational needs justify it.
- Define explicit interfaces for communication between modules.
- Avoid splitting classes or modules based only on their size.

A class with 500 lines is not automatically a God Object. A class with 50 lines can still coordinate unrelated responsibilities.

The real problem is a lack of conceptual cohesion.

Do not decompose a God Object into dozens of tiny classes that all depend on one another. That merely replaces one kind of complexity with another.

The goal is to create meaningful boundaries, not maximize the number of files.

---

## 4. Premature Generalization (The Abstraction Trap)

**The problem:** Developers create abstractions for hypothetical future requirements instead of modeling the actual domain they currently understand.

Consider a system that initially supports two payment methods: credit card and bank transfer.

A developer decides to build a universal payment framework:

```text
PaymentStrategy
PaymentProviderFactory
PaymentCapabilityRegistry
PaymentExecutionPipeline
PaymentMetadataResolver
PaymentExtensionManager
PaymentPolicyProvider
PaymentConfigurationEngine
```

The intention is to support every possible payment method in the future.

But the business currently has only two payment methods, and their differences are relatively small.

The abstraction introduces more concepts than the domain itself.

Another example is creating a universal `Status` abstraction:

```csharp
public interface IStatus
{
    string Code { get; }
    string Name { get; }
    bool IsTerminal { get; }
    bool CanTransitionTo(IStatus next);
}
```

Then using it for orders, payments, shipments, customers, invoices, and subscriptions.

These concepts may all have states, but their transition rules and business meanings are different.

A cancelled order, a failed payment, and an expired subscription are not necessarily variations of one universal status model.

**Consequences:**

- The model becomes more abstract than the business.
- Developers must understand unnecessary infrastructure before implementing simple features.
- Real domain differences are hidden behind generic interfaces.
- Changes become difficult because abstractions attempt to support incompatible use cases.
- The system becomes harder to reason about despite appearing more reusable.

**How to address it:**

- Start with concrete business scenarios.
- Look for repeated concepts and behavior before introducing abstractions.
- Generalize when multiple real examples reveal a meaningful commonality.
- Prefer a small amount of duplication over an abstraction that hides important differences.
- Refactor when the actual requirements justify a shared concept.

This does not mean avoiding abstractions or ignoring future requirements.

It means distinguishing between a future requirement supported by evidence and a future requirement invented to justify a framework.

A good abstraction captures a real similarity. A bad abstraction forces different things to behave as though they were the same.

---

## 5. Data-Centric Design (The Database Dictates)

**The problem:** The database schema becomes the primary model of the domain.

In database-driven development, developers often begin by designing tables and then generate classes from those tables.

For example:

```sql
CREATE TABLE Orders
(
    Id UNIQUEIDENTIFIER PRIMARY KEY,
    Status INT,
    TotalAmount DECIMAL(18, 2),
    CustomerId UNIQUEIDENTIFIER
);
```

The generated class becomes the domain model:

```csharp
public class Order
{
    public Guid Id { get; set; }
    public int Status { get; set; }
    public decimal TotalAmount { get; set; }
    public Guid CustomerId { get; set; }
}
```

The class reflects how the data is stored, but says very little about what an order means or which operations are valid.

The problem becomes more visible when database structures are reused across different business contexts.

For example, an enterprise database may contain a `Customer` table shared by sales, billing, support, and subscriptions.

But each context may have a different understanding of a customer:

- Sales cares about prospects and commercial relationships.
- Billing cares about payment responsibility and financial information.
- Support cares about service history and communication.
- Subscription management cares about subscription eligibility and lifecycle.

A single database entity cannot automatically represent all these models correctly.

**Consequences:**

- Domain behavior becomes dependent on database structure.
- Database changes affect unrelated business logic.
- Shared tables create accidental coupling between contexts.
- Entities become collections of columns rather than meaningful domain concepts.
- The model reflects historical implementation decisions rather than current business understanding.

**How to address it:**

- Explore the domain before deciding how to persist it.
- Design domain concepts around business behavior and invariants.
- Treat the database as a persistence mechanism, not the definition of the domain.
- Use mapping when the persistence model and domain model have different responsibilities.
- Allow different Bounded Contexts to have different models even when they store related information.

This does not mean every system needs a separate database schema, a custom ORM mapping layer, or complete independence between domain and persistence models.

For a simple CRUD application, using persistence models directly may be perfectly reasonable.

The anti-pattern appears when database structure dictates business meaning and prevents the model from evolving independently.

---

## 6. CRUD-Based Domain Design (The Four-Operation Trap)

**The problem:** The system is modeled entirely around creating, reading, updating, and deleting records, without expressing meaningful business operations.

Consider an order-management API:

```text
POST   /orders
GET    /orders/{id}
PUT    /orders/{id}
DELETE /orders/{id}
```

This API may be sufficient for basic record management, but it can hide important domain operations.

What does `PUT /orders/{id}` actually mean?

- Add an item?
- Change the delivery address?
- Apply a discount?
- Confirm the order?
- Cancel the order?
- Reopen a cancelled order?

These operations may have different rules and consequences.

A generic update operation can become a way to bypass business invariants:

```csharp
public void UpdateOrder(OrderDto dto)
{
    var order = repository.GetById(dto.Id);

    order.Status = dto.Status;
    order.TotalAmount = dto.TotalAmount;
    order.CustomerId = dto.CustomerId;

    repository.Save(order);
}
```

This allows callers to change critical state without expressing why the change is valid.

A behavior-oriented API might instead expose:

```text
POST /orders
POST /orders/{id}/items
POST /orders/{id}/confirmation
POST /orders/{id}/cancellation
POST /orders/{id}/delivery-address
```

And the domain model might provide:

```csharp
order.AddItem(productId, quantity);
order.ApplyDiscount(discount);
order.Confirm();
order.Cancel(reason);
order.ChangeDeliveryAddress(address);
```

Each operation can enforce its own rules.

**Consequences:**

- Business operations are hidden behind generic updates.
- State transitions become difficult to control.
- Validation is scattered across API endpoints and services.
- The model communicates data manipulation rather than business intent.
- Important domain events and side effects become harder to identify.

**How to address it:**

- Identify the commands users and business processes actually perform.
- Express meaningful operations explicitly.
- Put invariants inside the relevant Aggregate or domain policy.
- Keep queries separate from commands when that improves clarity.
- Use CRUD where the domain genuinely requires simple data management.

CRUD itself is not an anti-pattern.

There are many systems where CRUD is exactly the right model. Administrative screens, configuration management, reference data, and simple catalogs may not need elaborate domain behavior.

The mistake is assuming that every business operation is just an update to a record.

---

## 7. Confusing Entities and Value Objects (The Identity Crisis)

**The problem:** Domain concepts are modeled with the wrong identity and equality semantics.

An Entity is defined by its identity and continuity over time. A Value Object is defined by its attributes and has no independent identity.

Consider a customer and an address.

A customer remains the same customer even if their name, email, or address changes.

An address, however, may be represented as a value:

```csharp
public sealed record Address(
    string Street,
    string City,
    string PostalCode,
    string Country);
```

Two addresses with the same relevant attributes may be considered equal.

Now consider a bank account.

Two accounts may have the same balance and currency, but they are not the same account. Their identities matter.

Modeling an account as a Value Object would lose that distinction.

A more subtle example is money.

Two amounts of `100` are not necessarily equal if one is in USD and the other is in EUR.

```csharp
public sealed record Money(
    decimal Amount,
    Currency Currency);
```

Equality should consider both amount and currency.

**Consequences:**

- Identity and equality become ambiguous.
- Persistence and state changes become difficult to model.
- Collections may behave incorrectly.
- Domain concepts lose important distinctions.
- Developers introduce artificial identifiers for concepts that should be values.

**How to address it:**

Ask:

- Does the concept have a lifecycle of its own?
- Must it be distinguished from other instances even when its attributes are identical?
- Can it change over time while remaining the same conceptual thing?
- Is it meaningful to compare two instances by their values alone?
- Does the business refer to it by a persistent identity?

Do not decide based only on whether a concept has an ID column in the database.

An object can have a technical identifier for persistence without that identifier being meaningful to the domain.

Similarly, a concept that looks like a value may become an Entity when its lifecycle or business rules require it.

For example, an address used as a value inside a customer profile may be a Value Object. A delivery address record that must be tracked, verified, approved, and referenced independently may need Entity identity.

The classification depends on the business meaning and lifecycle, not the shape of the data.

---

## 8. Oversized Aggregates (The Consistency Boundary That Ate the System)

**The problem:** Aggregates contain too many objects or unrelated concepts because developers confuse object relationships with consistency requirements.

Consider a Customer Aggregate:

```text
Customer
 ├── Addresses
 ├── Orders
 │    ├── OrderLines
 │    └── Payments
 ├── LoyaltyPoints
 ├── SupportTickets
 └── Notifications
```

At first glance, this may look like a complete representation of a customer.

But should changing a customer's phone number require loading all their orders and support tickets?

Should reserving inventory require locking the entire warehouse?

Should updating loyalty points compete with confirming an order?

Probably not.

An Aggregate is a consistency boundary. It is not a collection of everything that can be navigated from a root entity.

**Consequences:**

- Large object graphs must be loaded for small operations.
- Unrelated changes create concurrency conflicts.
- Persistence becomes expensive.
- Business responsibilities become coupled.
- Aggregate boundaries become difficult to reason about.

**How to address it:**

- Identify the invariants that must be protected atomically.
- Keep the Aggregate as small as those invariants allow.
- Reference other Aggregates by identity.
- Coordinate cross-Aggregate processes through application services, domain events, or process managers.
- Evaluate concurrency and loading costs.

For example:

```text
Customer (Aggregate Root)

Order (Aggregate Root)
 ├── OrderLine
 └── OrderLine

Payment (Aggregate Root)

InventoryItem (Aggregate Root)
```

These Aggregates may collaborate in a business process without becoming one consistency boundary.

The opposite mistake is also possible: splitting everything into separate Aggregates even when a rule genuinely requires atomic consistency.

The goal is not to make Aggregates as small as possible. It is to make them as small as the business invariants allow.

---

## 9. The Service Layer as a Second Domain Model

**The problem:** Instead of modeling business behavior in the domain, developers create a large collection of services that gradually becomes the actual domain model.

For example:

```text
OrderService
 ├── CreateOrder()
 ├── ConfirmOrder()
 ├── CancelOrder()
 ├── RecalculateOrder()
 ├── ValidateOrder()
 ├── ApplyDiscount()
 └── UpdateOrderStatus()

PaymentService
 ├── AuthorizePayment()
 ├── CapturePayment()
 └── RefundPayment()

InventoryService
 ├── ReserveInventory()
 ├── ReleaseInventory()
 └── AdjustInventory()
```

Services are not inherently problematic. Application Services and Domain Services are both legitimate DDD building blocks.

The problem appears when all meaningful behavior is placed in services while domain objects become passive data structures.

A service can become the place where developers put logic simply because it is easier than deciding which domain concept owns that behavior.

**Consequences:**

- Domain objects lose responsibility.
- Business behavior becomes difficult to discover.
- Services grow into procedural coordinators.
- Similar rules may be implemented in multiple services.
- The domain model becomes a data structure used by an external application model.

**How to address it:**

Distinguish between different kinds of services.

**Application Services** coordinate use cases. They load Aggregates, invoke domain behavior, interact with external systems, and manage the application workflow.

**Domain Services** express domain behavior that does not naturally belong to a single Entity or Value Object.

**Infrastructure Services** provide technical capabilities such as sending email, accessing a payment provider, or publishing messages.

For example:

```csharp
public class ConfirmOrderHandler
{
    public async Task Handle(
        ConfirmOrder command,
        CancellationToken cancellationToken)
    {
        var order = await _orders.GetById(
            command.OrderId,
            cancellationToken);

        order.Confirm();

        await _orders.Save(
            order,
            cancellationToken);
    }
}
```

The application service coordinates the operation. The Aggregate owns the confirmation rules.

A Domain Service might be appropriate for a pricing policy that depends on several domain concepts and does not naturally belong to one of them.

Do not eliminate services to make a domain model look more object-oriented. Give each service a clear reason to exist.

---

## 10. The Pattern Collector (DDD by Checklist)

**The problem:** Developers introduce DDD patterns because they have learned their names, not because the domain needs them.

A team learns about:

- Entities.
- Value Objects.
- Aggregates.
- Repositories.
- Domain Services.
- Domain Events.
- Factories.
- Specifications.
- CQRS.
- Event Sourcing.
- Sagas.

Then every project starts with all of them.

A simple reference-data module ends up with a Repository, Unit of Work, Specification, Domain Event, Factory, and several layers of abstractions, even though it only needs to retrieve and update a small set of records.

Another team introduces Event Sourcing because it sounds like a natural extension of DDD, even though it has no meaningful requirement for historical reconstruction, event-based auditability, or temporal queries.

**Consequences:**

- Architecture becomes more complicated than the problem.
- Developers spend time maintaining abstractions rather than implementing business capabilities.
- The team mistakes pattern usage for domain understanding.
- New developers must learn unnecessary architectural machinery.
- The project becomes harder to change.

**How to address it:**

- Understand the purpose of each pattern before introducing it.
- Start with the business problem, not the pattern catalogue.
- Introduce abstractions when they provide a meaningful benefit.
- Prefer the simplest design that protects the important domain rules.
- Revisit patterns when the domain or technical requirements change.

DDD is not a checklist.

A system does not become more domain-driven because it has more Aggregates, more interfaces, or more layers.

Sometimes the most domain-driven design is a straightforward class with one meaningful method.

---

## 11. Overusing Domain Events (Event Everything)

**The problem:** Developers introduce events for every state change or method invocation, regardless of whether the event represents a meaningful business fact.

For example:

```text
OrderCreated
OrderNameChanged
OrderDescriptionUpdated
OrderPropertyModified
OrderFieldChanged
OrderStatusAssigned
```

Some of these may be useful events in a particular domain. But creating an event for every technical mutation does not automatically create a meaningful event-driven model.

Consider the difference between:

```csharp
order.Status = OrderStatus.Confirmed;
```

and:

```csharp
order.Confirm();
```

A meaningful domain event might be:

```csharp
OrderConfirmed
```

The event represents something that happened in the business.

It can be used to notify other parts of the system that the order has been confirmed.

A different problem occurs when developers publish domain events simply to avoid direct method calls, even between closely related objects within the same consistency boundary.

This can obscure control flow and make the model harder to understand.

**Consequences:**

- The number of events grows without adding meaningful business information.
- Event handlers become difficult to discover.
- Event ordering and failure handling become complicated.
- Technical state changes are mistaken for business facts.
- The system becomes unnecessarily asynchronous.

**How to address it:**

- Publish events for meaningful domain occurrences.
- Name events in the past tense when they represent facts that happened.
- Distinguish domain events from integration events and persistence events.
- Use events to communicate between appropriate boundaries.
- Keep immediate invariant enforcement inside the Aggregate.
- Introduce asynchronous processing when the business and architecture benefit from it.

For example, an order may raise `OrderConfirmed` after successfully transitioning into the confirmed state.

The event does not replace the confirmation rule. It communicates the result of an operation that has already satisfied that rule.

Also remember that domain events are not automatically integration events. A local domain event may contain internal details that should never be exposed to another Bounded Context.

---

## 12. Confusing Bounded Contexts with Microservices (The Distributed Monolith)

**The problem:** Teams assume that every Bounded Context must become an independently deployed microservice.

A Bounded Context is a boundary around a particular domain model and its language. It is primarily a modeling and organizational concept.

A microservice is a deployment and operational architecture choice.

They can align, but they are not the same thing.

Consider a system with these Bounded Contexts:

```text
Sales
Inventory
Billing
Shipping
Customer Management
```

These contexts may initially exist as modules inside a modular monolith.

That can provide:

- Clear domain boundaries.
- Explicit module interfaces.
- Independent models.
- Easier local development.
- Simpler deployment and operations.

Splitting them into microservices introduces additional concerns:

- Network communication.
- Distributed failures.
- Eventual consistency.
- Deployment coordination.
- Observability.
- Service discovery.
- Message delivery guarantees.
- Data ownership and migration.

If every service makes synchronous calls to every other service, the system may become a distributed monolith.

For example:

```text
Order Service
   |
   v
Customer Service
   |
   v
Inventory Service
   |
   v
Pricing Service
   |
   v
Payment Service
```

A single business operation now depends on several network calls. A failure in one service can affect the entire workflow.

**Consequences:**

- Operational complexity grows faster than business value.
- Simple operations become distributed workflows.
- Teams become coupled through synchronous dependencies.
- Debugging and testing become harder.
- Deployment independence exists in theory but not in practice.

**How to address it:**

- Identify Bounded Contexts based on domain language and model boundaries.
- Establish modular boundaries before considering distributed deployment.
- Use a modular monolith when it provides the required independence.
- Split into services when independent deployment, scaling, ownership, or operational requirements justify the cost.
- Design service communication and failure handling explicitly.

A Bounded Context tells you where a model applies. It does not tell you how many servers you need.

---

## 13. Treating Context Maps as Architecture Diagrams

**The problem:** A Context Map is created as a diagram of services and dependencies rather than a representation of relationships between Bounded Contexts.

A Context Map is intended to make the relationships between models explicit.

It can describe relationships such as:

- Customer-Supplier.
- Conformist.
- Anti-Corruption Layer.
- Open Host Service.
- Shared Kernel.
- Separate Ways.

A diagram containing boxes labelled `Customer Service`, `Order Service`, and `Payment Service` does not automatically qualify as a Context Map.

Consider two Bounded Contexts:

```text
Sales Context
    |
    | Customer-Supplier
    v
Billing Context
```

The relationship is not merely that Sales calls Billing.

It communicates that Billing provides a model or capability that Sales consumes, and that the relationship has an influence structure and coordination implications.

Without understanding the relationship, the team may introduce an integration that is technically functional but creates unwanted model coupling.

**Consequences:**

- Teams misunderstand integration responsibilities.
- Upstream model changes unexpectedly affect downstream contexts.
- Shared language and model ownership become unclear.
- Integration decisions are made without considering organizational relationships.

**How to address it:**

- Identify the actual Bounded Contexts first.
- Describe their model relationships.
- Make ownership and influence explicit.
- Record translation and integration responsibilities.
- Revisit the map as teams, systems, and business relationships evolve.

A Context Map is a model of relationships, not just a picture of the deployment topology.

---

## 14. Treating CQRS and Event Sourcing as Mandatory DDD Architecture

**The problem:** Developers assume that DDD requires Command Query Responsibility Segregation or Event Sourcing.

These patterns can be valuable, but neither is a prerequisite for DDD.

A conventional application can use Aggregates, Value Objects, Repositories, Domain Services, and a relational database without adopting Event Sourcing or separate read models.

CQRS separates the responsibility of handling commands from the responsibility of answering queries. It can be implemented with separate models while using the same database.

Event Sourcing stores changes as a sequence of events rather than persisting only the latest state.

Both introduce design and operational consequences.

Consider a simple product catalogue.

If users mostly browse products and administrators occasionally update their descriptions, a conventional persistence model may be sufficient.

Introducing Event Sourcing would require decisions about:

- Event schema evolution.
- Rebuilding projections.
- Snapshotting.
- Event ordering.
- Event storage.
- Replay safety.
- Operational recovery.

Those costs need to solve an actual problem.

**Consequences:**

- Unnecessary architectural complexity.
- More difficult debugging and operational recovery.
- Increased testing and infrastructure requirements.
- Developers focus on persistence patterns instead of domain behavior.

**How to address it:**

- Understand the specific problems CQRS and Event Sourcing solve.
- Introduce CQRS when read and write responsibilities genuinely need different models.
- Introduce Event Sourcing when the event history is itself valuable to the business or system.
- Consider conventional persistence first when it meets the requirements.
- Evaluate the full lifecycle and operational cost of each pattern.

DDD is compatible with many persistence and architectural approaches.

The model should drive the architecture, not the other way around.

---

## 15. Treating Every Business Rule as an Aggregate Invariant

**The problem:** Developers attempt to enforce every rule synchronously inside an Aggregate, even when the rule spans independent concepts, external systems, or long-running processes.

Consider confirming an order.

The operation may involve:

- Checking customer eligibility.
- Reserving inventory.
- Authorizing payment.
- Creating a shipment.
- Sending a notification.

Putting all these operations inside `Order.Confirm()` would make the Order Aggregate responsible for concepts it does not own.

An Aggregate protects its own consistency boundary. It does not need to coordinate every business process involving that Aggregate.

A better design might be:

```text
Confirm Order
     |
     v
Order Aggregate
     |
     | OrderConfirmed
     v
Order Fulfilment Process
     |
     +---- Reserve Inventory
     |
     +---- Authorize Payment
     |
     +---- Arrange Shipment
```

The Order Aggregate protects the rules for confirmation. The application workflow coordinates other capabilities.

**Consequences:**

- Aggregates become large and dependent on external systems.
- Transactions become difficult to manage.
- Network failures become domain-model concerns.
- Long-running operations become difficult to represent.
- The system confuses immediate invariants with process-level policies.

**How to address it:**

Classify rules according to their consistency requirements:

- Rules that must hold within one operation and one Aggregate belong inside that Aggregate.
- Rules involving multiple domain concepts may require a Domain Service or policy.
- Rules coordinating several Aggregates may require an application workflow or process manager.
- Rules involving external systems may require asynchronous coordination and compensation.

Not every business rule belongs in an Aggregate. The Aggregate boundary is defined by the consistency requirement, not by the desire to put all business logic in one place.

---

## 16. Designing for Hypothetical Domains (The Imaginary Domain Expert)

**The problem:** Developers build a model based on assumptions about how a business should work rather than investigating how it actually works.

This often happens when a team has limited access to domain experts or when developers are familiar with a particular industry and assume that every organization follows the same practices.

Consider an insurance application.

A developer assumes that every policy follows this lifecycle:

```text
Draft → Active → Expired
```

But the actual business may include:

- Policies activated retroactively.
- Suspended policies.
- Renewals with overlapping periods.
- Policies cancelled with refunds.
- Policies requiring manual approval.
- Different rules for different product types.

The initial model may be elegant, but it represents an imaginary business.

The problem is not that the developers failed to anticipate every edge case. It is that they treated assumptions as established domain knowledge.

**Consequences:**

- Important business rules are omitted.
- The model requires constant exceptions.
- Domain experts lose confidence in the software.
- Developers repeatedly redesign concepts after discovering real behavior.
- The system encodes assumptions that are difficult to change.

**How to address it:**

- Investigate real business workflows.
- Use concrete examples instead of abstract descriptions.
- Ask domain experts about exceptions and failure scenarios.
- Validate assumptions through workshops and collaborative modeling.
- Treat unknowns as questions rather than silently making design decisions.
- Revisit the model when new knowledge emerges.

Techniques such as Event Storming, Domain Storytelling, Example Mapping, and collaborative domain modeling can help reveal how the business actually operates.

A domain model is a hypothesis about the business. It must be tested against reality.

---

## 17. Ignoring the Cost of Legacy Integration (The Legacy Leakage)

**The problem:** A new Bounded Context directly adopts the model, terminology, and assumptions of a legacy system.

Consider a modern logistics platform integrating with an old transportation system.

The legacy system exposes:

```csharp
public class LegacyWaybill
{
    public string WaybillNo { get; set; }
    public int StatusCode { get; set; }
    public decimal CargoWeight { get; set; }
}
```

The new system begins using `LegacyWaybill` directly in its domain.

Soon, legacy status codes appear in business rules, legacy identifiers spread through application services, and legacy naming conventions become part of the new model.

The new system is no longer independent. It has inherited the conceptual limitations of the old one.

**Consequences:**

- Legacy assumptions spread throughout the new model.
- Changes to the old system affect unrelated components.
- New domain concepts become difficult to introduce.
- Integration details become business concepts by accident.
- Legacy modernization becomes more expensive.

**How to address it:**

- Introduce an Anti-Corruption Layer where the models genuinely differ.
- Translate external terminology, identifiers, statuses, and errors.
- Keep external DTOs and protocols outside the local domain.
- Test mappings against representative legacy behavior.
- Avoid building an ACL when the external model is intentionally suitable and the translation adds no meaningful value.

The goal is not to reject every legacy concept. It is to make a conscious decision about which concepts belong in the local model.

---

## 18. Over-Modeling the Domain (The Modeling Rabbit Hole)

**The problem:** Developers spend too much time designing an ideal conceptual model and too little time validating whether that model helps solve real business problems.

DDD encourages deep modeling, but that does not mean every concept needs a class, every rule needs a specification object, and every noun in a business conversation needs an Entity.

Consider a simple pricing rule:

> Apply a 10% discount to orders above a certain amount.

A team might introduce:

```text
DiscountPolicy
DiscountStrategy
DiscountSpecification
DiscountCalculationContext
DiscountEligibilityEvaluator
DiscountRuleRegistry
DiscountRuleResolver
DiscountConfigurationProvider
```

Perhaps the business actually needs a straightforward pricing policy with two rules.

The model has become more complicated than the behavior it represents.

Another example is creating Value Objects for every primitive:

```csharp
CustomerName
CustomerFirstName
CustomerLastName
CustomerMiddleName
CustomerDisplayName
CustomerFormalName
```

Some may be meaningful domain concepts. Others may simply introduce unnecessary indirection.

**Consequences:**

- Modeling becomes an end in itself.
- Developers spend more time maintaining abstractions.
- The code becomes harder to navigate.
- Simple changes require touching many components.
- The team loses sight of the business problem.

**How to address it:**

- Model concepts when they carry meaning, rules, identity, or important constraints.
- Prefer concrete examples over speculative abstractions.
- Keep the model as simple as the business allows.
- Validate design decisions through working software and domain feedback.
- Refactor as understanding improves.

A good model is not the model with the most concepts. It is the model that makes the important distinctions visible while keeping irrelevant complexity out.

---

## 19. Treating the Domain Model as a Reflection of the Organization Chart

**The problem:** Software boundaries are created around departments and reporting structures rather than the business capabilities and models they represent.

For example, an organization has:

- Sales Department.
- Finance Department.
- Operations Department.
- Customer Support Department.

A team creates corresponding software modules:

```text
SalesModule
FinanceModule
OperationsModule
SupportModule
```

This may be appropriate, but it is not automatically a good Bounded Context design.

Departments can contain multiple business capabilities. Different departments may also use the same term with different meanings.

For example, the concept of a customer may mean:

- A sales prospect in Sales.
- A legal debtor in Finance.
- A service recipient in Operations.
- A person with an open case in Support.

Likewise, a single business capability may cross several organizational departments.

**Consequences:**

- Technical boundaries reflect management structures rather than domain boundaries.
- Modules become dependent on organizational ownership.
- The model fails to express differences in business meaning.
- Organizational changes trigger unnecessary architectural changes.

**How to address it:**

- Discover Bounded Contexts from language, responsibilities, rules, and model boundaries.
- Consider organizational structure as relevant context, not the sole source of boundaries.
- Identify where terms and business rules differ.
- Use Context Mapping to describe relationships between models.
- Align team ownership with software boundaries where practical, without assuming they must be identical.

The organization chart tells you who reports to whom. It does not necessarily tell you how the business should be modeled.

---

## 20. Ignoring Model Evolution (The Frozen Model)

**The problem:** The domain model is treated as a finished artifact rather than something that evolves as the team learns more about the business.

A team spends several months designing a model, documents its Entities and Aggregates, and then treats the result as an architectural contract that should not change.

But business understanding evolves.

A concept initially modeled as a simple status may turn out to represent several distinct business states. A process initially assumed to be synchronous may turn out to involve manual approval. A customer concept may need to be separated into different models across Bounded Contexts.

When the team refuses to change the model, new requirements are implemented as exceptions and special cases.

For example:

```csharp
if (order.Status == OrderStatus.Confirmed &&
    order.IsLegacyOrder &&
    order.HasSpecialPermission &&
    order.CreatedBeforeCutoff)
{
    // Special behavior added to preserve an old assumption.
}
```

A few such exceptions may be unavoidable. But when they accumulate, they can indicate that the model no longer represents the business accurately.

**Consequences:**

- The model accumulates special cases.
- Semantic distinctions remain hidden.
- Business changes become expensive.
- Developers avoid refactoring because the original model is considered untouchable.
- The software gradually diverges from the actual business.

**How to address it:**

- Treat the domain model as an evolving understanding.
- Revisit assumptions when new business scenarios appear.
- Refactor terminology, boundaries, and behavior when necessary.
- Use tests to protect established behavior while changing the model.
- Keep the model open to new evidence from domain experts and production behavior.

A domain model is not a monument to the team's original understanding. It is a tool for expressing the current understanding of the business.

---

# Recognizing Anti-Patterns in Practice

Anti-patterns rarely appear in isolation.

An oversized Aggregate may be a symptom of data-centric design. An anemic model may be caused by CRUD-based thinking. A God Object may emerge because the team has no clear Bounded Context boundaries. A distributed monolith may be the result of splitting services before understanding the domain.

Rather than looking for isolated symptoms, investigate the decisions that produced them.

The following questions are useful during design reviews and architectural assessments.

### Language and Understanding

- Can domain experts understand the terminology used in the code?
- Do important business terms have consistent meanings?
- Are there concepts whose meaning differs across Bounded Contexts?
- Can developers explain the business rules without translating technical implementation details?

### Domain Modeling

- Do Entities and Value Objects express meaningful domain concepts?
- Are business rules protected by the appropriate domain objects?
- Are Aggregates designed around invariants?
- Are domain services used because behavior genuinely belongs outside an Entity or Value Object?
- Does the model reflect actual business scenarios or developer assumptions?

### Architecture and Boundaries

- Are Bounded Contexts based on distinct models and responsibilities?
- Can modules evolve without unnecessary dependencies?
- Are integration relationships explicit?
- Is the system distributed because it needs to be, or because the team assumes DDD requires microservices?
- Are external models leaking into local domain concepts?

### Complexity and Evolution

- Does each abstraction solve an existing problem?
- Are patterns being used because they are useful or because they are fashionable?
- Can the team change a business rule without modifying unrelated components?
- Are tests protecting business behavior or merely mirroring implementation details?
- Is the model evolving as the team's understanding improves?

---

# Practical Exercises

## Exercise 1: Diagnose an Anemic Domain Model

Given the following model:

```csharp
public class Order
{
    public Guid Id { get; set; }
    public int Status { get; set; }
    public decimal Total { get; set; }
}

public class OrderService
{
    public void Confirm(Order order)
    {
        order.Status = 2;
    }

    public void Cancel(Order order)
    {
        order.Status = 3;
    }
}
```

Tasks:

1. Identify the anti-patterns.
2. Identify the missing business rules.
3. Redesign the model using meaningful operations.
4. Decide which behavior belongs in the Aggregate.
5. Write tests for invalid state transitions.

Consider whether confirmation and cancellation are always valid regardless of the order's current state.

## Exercise 2: Identify the God Object

Consider an application service that handles customer registration, order placement, payment authorization, inventory reservation, shipping, and notifications.

Tasks:

1. Identify its responsibilities.
2. Group them by business capability.
3. Identify possible Bounded Contexts.
4. Decide which operations belong in Aggregates.
5. Propose a modular architecture.
6. Explain whether the modules should be independently deployed.

Do not assume that every module needs to become a microservice.

## Exercise 3: Challenge a Database-Driven Model

Given a database with shared `Customer`, `Order`, `Payment`, and `Shipment` tables:

1. Identify the Bounded Contexts that may use these concepts.
2. Explore whether each context needs the same model.
3. Identify where the current schema dictates business behavior.
4. Propose context-specific models.
5. Explain how the models should communicate.

## Exercise 4: Find Premature Abstractions

Choose an existing application and identify abstractions that were introduced for hypothetical future requirements.

Ask:

- Which actual use cases depend on them?
- Do they represent genuine domain similarities?
- Do they hide important differences?
- Would a simpler implementation be easier to understand?
- What concrete evidence would justify keeping or removing them?

## Exercise 5: Review a Distributed Monolith

Consider a system containing six microservices that synchronously call one another to complete ordinary business operations.

Tasks:

1. Map the dependencies.
2. Identify the Bounded Contexts.
3. Identify the business workflows.
4. Find unnecessary synchronous dependencies.
5. Consider whether a modular monolith would provide a simpler design.
6. Propose a communication strategy that respects domain boundaries and failure handling.

## Exercise 6: Conduct an Anti-Pattern Review

Select an existing project and perform a structured review.

For each identified problem:

1. Describe the observable symptom.
2. Explain the underlying design decision.
3. Identify the business or technical consequences.
4. Propose a more appropriate design.
5. Identify the trade-offs introduced by the proposed change.
6. Define how you would verify the improvement.

Do not simply label classes or modules as anti-patterns. Explain why their current design causes problems.

---

# Completion Criteria

You should be able to:

- Recognize when the Ubiquitous Language is missing or inconsistent.
- Distinguish a rich domain model from an anemic one.
- Identify God Objects and understand the importance of conceptual cohesion.
- Recognize premature generalization and unnecessary abstractions.
- Distinguish domain modeling from database schema design.
- Understand when CRUD is appropriate and when business operations need explicit modeling.
- Correctly distinguish Entities and Value Objects.
- Identify oversized Aggregates and inappropriate consistency boundaries.
- Distinguish Application Services, Domain Services, and Infrastructure Services.
- Avoid introducing patterns simply because they are part of DDD.
- Understand the difference between Bounded Contexts and microservices.
- Recognize when CQRS and Event Sourcing introduce unnecessary complexity.
- Design appropriate relationships between Bounded Contexts.
- Identify the leakage of legacy models into new domain models.
- Understand that a domain model must evolve as business understanding improves.

The objective is not to build a system that contains no anti-patterns. Real systems have constraints, compromises, historical decisions, and imperfect information.

The objective is to recognize the consequences of those decisions and know when they are becoming obstacles.

**DDD is not about applying every pattern correctly. It is about continuously improving the relationship between the software model and the reality it represents.**