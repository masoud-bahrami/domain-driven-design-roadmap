# 01. Foundations

Domain-Driven Design begins with a deceptively simple idea: software should be designed around a deep understanding of the domain it exists to support.

The difficult part is not writing code or choosing an architectural style. It is understanding the business, identifying the concepts that matter, discovering the rules that govern them, and expressing that knowledge in a model that can evolve.

A system may have clean architecture, extensive test coverage, well-designed APIs, and a sophisticated database, yet still fail to represent the business accurately. This happens when the implementation is disconnected from the knowledge, language, and decisions of the people who understand the domain.

Domain-Driven Design (DDD) addresses this problem by connecting domain knowledge, collaborative modeling, and software design.

It provides a way to reason about complex domains, establish meaningful boundaries, and build software whose structure and behavior reflect the business it serves.

This chapter introduces the foundational concepts behind DDD. It focuses on what these concepts mean, why they exist, how they relate to one another, and when they are useful.

## Learning objectives

By the end of this chapter, you should be able to:

- Explain what a domain and a domain model are.
- Understand the difference between strategic and tactical design.
- Explain the relationship between Subdomains and Bounded Contexts.
- Understand how a Ubiquitous Language is developed and maintained.
- Distinguish Entities from Value Objects.
- Explain the purpose of Aggregates and Aggregate Roots.
- Recognize meaningful Domain Events.
- Understand the roles of Repositories, Domain Services, and Application Services.
- Recognize when DDD is valuable and when it introduces unnecessary complexity.

**The central goal is not to memorize definitions. It is to understand how these concepts work together to help a team discover, express, and evolve a useful model of its domain.**

---

## 1. What is a Domain?

Before discussing domain models, entities, or aggregates, we need to understand what a domain actually is.

A domain is a sphere of knowledge, influence, or activity. It is the subject area that a software system is intended to support.

A domain is not defined by a programming language, database, framework, or organizational chart. It exists independently of the software.

Consider a company that operates an online retail business.

Its domain includes activities and knowledge related to:

- Selling products.
- Managing customers.
- Accepting and processing payments.
- Managing inventory.
- Preparing and shipping orders.
- Handling returns and refunds.
- Supporting customers.

These activities involve people, policies, decisions, constraints, and relationships. Together, they form the business domain that the software must support.

A domain can be large and complex. It may contain many areas of expertise, each with its own terminology and rules.

### Domain versus software system

A common misconception is that the domain is simply everything implemented inside an application.

It is not.

The domain exists independently of the application. Software is one way of representing and supporting selected aspects of that domain.

For example, an online retail company may have a complex process for approving refunds. Some decisions may be made by employees, some may be automated, and others may depend on external payment providers.

The software does not necessarily represent every aspect of that process. It models the parts that are relevant to the system's responsibilities.

This distinction matters because the purpose of DDD is not to reproduce the entire business inside software. It is to create a useful model of the relevant domain knowledge.

### Why understanding the domain matters

Imagine being asked to implement a refund feature.

A technical interpretation might be:

1. Receive a refund request.
2. Find the original payment.
3. Update its status.
4. Send a notification.

But the actual business may have rules such as:

- Only captured payments can be refunded.
- Some products are not eligible for refunds.
- A refund may require approval.
- Partial refunds are allowed under certain conditions.
- A refund request may be rejected if the return period has expired.
- A payment provider may accept a refund request before the money is actually returned.

Without understanding these rules, developers may implement something that works technically but behaves incorrectly from a business perspective.

The first responsibility of DDD is therefore to improve the team's understanding of the problem before committing that understanding to software.

---

## 2. What is a Domain Model?

A model is a simplified representation of selected aspects of reality.

A domain model represents the concepts, relationships, rules, and behavior that matter in a particular area of the business.

It is not a complete copy of the business. It deliberately selects what is relevant to the problem being solved.

A useful model helps people reason about the domain, discuss business decisions, and implement software behavior.

For example, a simplified sales model might contain:

- Customer.
- Shopping Cart.
- Product.
- Order.
- Order Item.
- Discount.
- Payment.

However, simply creating these objects does not mean that a useful domain model exists.

The important questions are:

- What does each concept mean?
- What responsibilities does it have?
- How does it relate to other concepts?
- What rules govern its behavior?
- Which changes are allowed?
- What must always remain true?

Consider an Order.

A weak model might represent it as a collection of properties:

```csharp
public class Order
{
    public Guid Id { get; set; }
    public decimal Total { get; set; }
    public string Status { get; set; }
}
```

This structure stores information, but it does not explain what an order means or how it behaves.

A more expressive model might expose business operations:

```csharp
public class Order
{
    public OrderId Id { get; }
    public OrderStatus Status { get; private set; }
    public Money Total { get; private set; }

    public void Confirm()
    {
        // Enforce the rules for confirming an order.
    }

    public void Cancel(CancellationReason reason)
    {
        // Enforce the rules for cancellation.
    }
}
```

This is still only a sketch. A real implementation would need to define the relevant business rules and state transitions.

But the distinction is important: the second design gives the concept a place to express its behavior.

### A model is more than a diagram

Domain models are often represented through diagrams, workshop outputs, documents, and code. These representations help people communicate, but none of them should be confused with the model itself.

A model becomes especially valuable when it informs the software's actual behavior.

If a diagram describes one set of business rules while the implementation follows another, the team has two competing models.

DDD encourages a continuous relationship between understanding, language, design, and implementation. The model should be refined as new domain knowledge emerges.

### Models are selective and contextual

There is no requirement to represent every business concept in one model.

A concept may be important in one area of the business and irrelevant in another. The same term may represent different things depending on the context in which it is used.

This leads to two of the most important concepts in DDD: Subdomains and Bounded Contexts.

---

## 3. Strategic Design and Tactical Design

DDD is commonly discussed through two complementary areas of design.

**Strategic Design** addresses the large-scale structure of the domain and the relationships between different models.

It helps answer questions such as:

- What are the important areas of the business?
- Which areas provide competitive advantage?
- Where should models have explicit boundaries?
- Which teams own which models?
- How should different models communicate?
- Where should the organization invest its modeling effort?

Its central concepts include:

- Subdomains.
- Core, Supporting, and Generic Subdomains.
- Bounded Contexts.
- Ubiquitous Language.
- Context Maps and relationships between contexts.

**Tactical Design** addresses the internal structure of a domain model within a Bounded Context.

It helps answer questions such as:

- What concepts have identity?
- Which concepts are values?
- Where should business behavior live?
- Which rules must be protected together?
- How should meaningful business changes be represented?
- How should the model be accessed and persisted?

Its central concepts include:

- Entities.
- Value Objects.
- Aggregates and Aggregate Roots.
- Domain Events.
- Domain Services.
- Repositories.
- Factories.

These two areas complement each other.

Strategic Design establishes where a model belongs and what it is responsible for. Tactical Design provides patterns for expressing that model in detail.

A team that focuses only on tactical patterns may create technically sophisticated models inside poorly defined boundaries. A team that focuses only on strategic boundaries may identify useful contexts but still struggle to express their internal business behavior.

Effective DDD requires both.

---

## 4. Subdomains

A Subdomain is a distinct area of the overall business domain.

Subdomains describe the problem space: the different areas of business knowledge and responsibility that make up the organization.

Consider an online retail company. Its domain may contain these Subdomains:

- Sales.
- Product Catalog.
- Inventory Management.
- Payment Processing.
- Order Fulfillment.
- Customer Support.

Each represents a different area of business activity and knowledge.

Subdomains help teams understand the business before deciding how to structure the software.

### Types of Subdomains

DDD commonly distinguishes three types.

#### 4.1 Core Subdomain

The Core Subdomain contains the business capabilities that provide the organization's most important competitive advantage or strategic differentiation.

For an online retailer, this might be a sophisticated pricing and promotion strategy, a unique marketplace matching algorithm, or a specialized fulfillment capability.

The exact Core Subdomain depends on the business strategy.

Core does not mean technically complicated. A relatively small set of business rules may be strategically important even if its implementation is simple.

**Design implication:** Invest in understanding, modeling, and continuously improving the Core Subdomain. This is where specialized domain knowledge and software design can create significant business value.

#### 4.2 Supporting Subdomain

A Supporting Subdomain is necessary for the business to operate but does not provide its primary competitive differentiation.

For example, an online retailer may need internal processes for managing supplier information or coordinating warehouse operations.

These capabilities matter, but they may not be where the company needs to develop its most distinctive software.

**Design implication:** Build or acquire an appropriate solution according to the business requirements. Avoid spending disproportionate modeling effort where it provides little strategic benefit.

#### 4.3 Generic Subdomain

A Generic Subdomain represents a problem that many organizations face and for which established, reusable solutions may already exist.

Examples might include:

- Authentication.
- General-purpose document generation.
- Standard notification delivery.
- Common identity verification capabilities.

**Design implication:** Consider existing products, libraries, or external services when they satisfy the requirements. Building a specialized solution from scratch may not be a worthwhile investment.

### Important distinction

Core, Supporting, and Generic are business classifications, not technical classifications.

A Core Subdomain is not automatically a microservice. A Generic Subdomain is not automatically an external service. A Supporting Subdomain does not necessarily belong in a separate module.

These classifications help guide strategic investment and modeling effort. They do not prescribe a deployment architecture.

**Practical example**

| Business capability | Possible classification |
|---|---|
| Unique promotion and pricing strategy | Core |
| Internal supplier coordination | Supporting |
| Standard email delivery | Generic |
| Customer authentication | Generic or Supporting, depending on the business |
| Specialized fulfillment optimization | Core, if it differentiates the business |

The classification must be based on the actual business strategy, not the examples above.

**Learn more:**
- [Domain-Driven Design Distilled: Strategic Design with Subdomains](https://www.informit.com/store/domain-driven-design-distilled-9780134434421)
- [Domain analysis - Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/domain-analysis)

---

## 5. Bounded Contexts

A Bounded Context is an explicit boundary within which a particular domain model is defined and applicable.

This is one of the most important ideas in DDD because it addresses a fundamental problem: a single model and vocabulary cannot always represent every part of a complex business accurately.

Consider the word "Product" in an online retail business.

In the Catalog context, a Product may represent:

- Product name.
- Description.
- Images.
- Categories.
- Marketing information.

In the Inventory context, the same product may be represented through:

- Stock Keeping Unit (SKU).
- Warehouse location.
- Available quantity.
- Reserved quantity.
- Replenishment information.

In the Sales context, the relevant concept may be an item that can be purchased at a particular price and under specific commercial conditions.

These concepts are related, but they do not necessarily have identical responsibilities or rules.

Trying to represent all of them through one universal Product model may create unnecessary dependencies and ambiguous behavior.

A Bounded Context gives each model a defined scope.

### A Bounded Context defines meaning

Within a Bounded Context:

- Terms have specific meanings.
- The model has defined responsibilities.
- Business rules are interpreted consistently.
- The implementation reflects that model.
- The team can reason about the model without assuming it applies everywhere.

A Bounded Context is not merely a folder, namespace, database, or microservice.

It is a boundary around a model. It may be implemented as a module in a modular monolith, an independently deployed service, or another suitable software structure.

The deployment decision is separate.

### Subdomain versus Bounded Context

These concepts are related but not interchangeable.

A Subdomain describes an area of the business problem.

A Bounded Context describes a boundary around a model in the solution.

For example:

- Inventory Management is a Subdomain.
- An Inventory Management Bounded Context is a model boundary designed to support that area.

A Subdomain may be represented by one Bounded Context, but the relationship is not necessarily one-to-one. A Subdomain may require multiple models, and a software model may sometimes support concerns from more than one business area.

The distinction is important because business boundaries and software-model boundaries are related decisions, not identical concepts.

### Why explicit boundaries matter

Without explicit boundaries, teams may gradually create a shared model that tries to satisfy every consumer.

This often leads to:

- Ambiguous terminology.
- Conditional logic for different business cases.
- Unclear ownership of business rules.
- Changes in one area breaking another.
- Excessive dependencies between modules.
- Conflicting interpretations of the same data.

Bounded Contexts make these differences explicit and give teams a basis for managing them.

**Practical exercise:** Identify the different meanings of "Customer", "Product", or "Order" in several areas of a business. Describe the responsibilities and rules associated with each meaning. Decide whether these differences indicate separate models and where their boundaries might belong.

**Learn more:**
- [Bounded Context - Martin Fowler](https://martinfowler.com/bliki/BoundedContext.html)
- [Domain-Driven Design Distilled](https://www.informit.com/store/domain-driven-design-distilled-9780134434421)
- [Why the language is per bounded context](https://www.ubiquitouslanguage.org/understand/language-is-per-context/)

---

## 6. Ubiquitous Language

Ubiquitous Language is a shared language developed by the people working on a domain model and used consistently within its Bounded Context.

It connects business conversations, modeling discussions, documentation, and software implementation.

The term "ubiquitous" is important. The language should not exist only in a glossary or a requirements document. It should be present in everyday conversations and reflected in the software that implements the model.

### Why language matters

Software development involves continuous translation.

A domain expert explains a business process. An analyst writes requirements. A developer interprets those requirements. A designer creates an interface. A tester writes scenarios.

At every stage, meaning can be lost.

Consider a business where people use the terms:

- Payment Authorized.
- Payment Captured.
- Payment Settled.
- Payment Refunded.

If the development team represents all of these states with a generic `Paid` flag, the implementation may erase important business distinctions.

The problem is not simply that the naming is poor. The model has lost information about how the business works.

Ubiquitous Language reduces this gap by making the domain's concepts and distinctions explicit.

### Language must be connected to the model

A glossary is useful, but it is not sufficient.

Suppose a team defines "Order" as:

"An agreement to purchase one or more products under specified commercial conditions."

If the code then uses `Order` to represent a warehouse picking request, the shared definition is not reflected in the implementation.

The language and the model have become disconnected.

A healthy Ubiquitous Language is continuously validated through examples, conversations, and software behavior.

### Language is contextual

There is no requirement that every Bounded Context use identical terminology.

The word "Customer" may mean a person who places an order in Sales, an account holder in Customer Management, and a billing party in Finance.

These differences should not automatically be eliminated.

A shared language is rigorous within its context. It does not require a universal dictionary for the entire organization.

### How to develop a Ubiquitous Language

1. Bring developers and domain experts into the same conversations.
2. Explore concrete business scenarios.
3. Identify important terms and their meanings.
4. Challenge ambiguous or overloaded words.
5. Discuss differences in terminology between business areas.
6. Reflect the agreed model in code and documentation.
7. Revisit the language when new knowledge emerges.

Do not treat the glossary as a document that is completed once and then forgotten.

It is a living representation of the team's current understanding.

### Example

Consider an order fulfillment process.

A team might initially use the word "Completed" to describe several different outcomes:

- The order has been confirmed.
- The warehouse has finished preparing the package.
- The carrier has collected the package.
- The customer has received the package.

These are different business events.

A more precise language might distinguish:

- `OrderConfirmed`
- `ShipmentPrepared`
- `ShipmentDispatched`
- `DeliveryCompleted`

The result is not merely more descriptive naming. It allows the model to represent the actual sequence and meaning of business activities.

**Practical exercise: Ubiquitous Language Workshop**

Choose a small domain, such as an online store or library management system.

Bring together at least one domain expert and one developer. Discuss a concrete process and create a glossary containing:

| Term | Definition | Example | Open questions |
|---|---|---|---|
| Order | A customer's request to purchase products under agreed conditions | A customer confirms a basket | When does a basket become an order? |
| Payment Authorization | Approval to proceed with a payment under the provider's rules | The provider authorizes a card payment | Does authorization guarantee settlement? |
| Shipment | A fulfillment unit prepared for delivery | An order is split into two shipments | Can one order have multiple shipments? |

Ask participants to challenge the definitions. Record disagreements rather than hiding them.

**Learn more:**
- [Ubiquitous Language - Martin Fowler](https://martinfowler.com/bliki/UbiquitousLanguage.html)
- [Domain-Driven Design Reference - Eric Evans](https://www.domainlanguage.com/ddd/reference/)
- [Domain-Driven Design Distilled](https://www.informit.com/store/domain-driven-design-distilled-9780134434421)

---

## 7. Entities

An Entity is a domain object whose identity distinguishes it from other objects, even when its attributes change.

An Entity has continuity over time. Its identity remains meaningful while its state changes.

For example, consider a Customer.

A customer may change:

- Name.
- Email address.
- Phone number.
- Delivery preferences.

Despite these changes, the business may still recognize the same customer.

The customer is therefore modeled as an Entity when its identity and continuity matter to the domain.

### Identity is more important than attributes

Consider two customers:

| Customer | Name | Email |
|---|---|---|
| A | Alex Smith | alex@example.com |
| B | Alex Smith | alex@example.com |

Their attributes are identical, but they may represent two distinct customers.

They are not the same Entity simply because their properties match.

Identity may be represented by a database-generated identifier, a business identifier, or another stable identity mechanism.

A database primary key is an implementation choice. It is not the definition of an Entity.

### Entities have behavior

An Entity should not necessarily be a passive data structure.

It can encapsulate behavior and protect the rules governing its state.

For example, an Order may have operations such as:

- Confirm.
- Cancel.
- Add Item.
- Remove Item.
- Mark as Ready for Fulfillment.

Not every operation is valid in every state.

The Entity can provide a place to express those rules.

### Entity identity is contextual

A concept should not automatically be classified as an Entity because it has an ID in a database.

The question is whether the business needs to distinguish one particular instance from another over time.

A customer address, for example, is usually a Value Object when the business cares about the address's values. It may be an Entity if the business needs to track a specific address record and its history independently.

The classification depends on the model and its requirements.

**Practical exercise:** Identify the important concepts in a business scenario. For each one, ask:

- Does the business need to distinguish this instance from another?
- Does it have continuity over time?
- Can its attributes change while it remains the same concept?
- Does its history matter?

If identity is central to its meaning, it may be an Entity.

**Learn more:**
- [Designing a microservice domain model - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model)

---

## 8. Value Objects

A Value Object is a domain object whose meaning is determined by its values rather than by an independent identity.

Two Value Objects with the same relevant values are considered equal in the domain.

Common examples include:

- Money.
- Address.
- Weight.
- Temperature.
- Date Range.
- Geographic Coordinates.
- Product Dimensions.

Consider an amount of money.

A value of 100 USD is not necessarily equivalent to 100 EUR.

The amount and currency together define its meaning.

A `Money` Value Object can encapsulate these values and relevant operations, such as addition, subtraction, or comparison, while protecting domain constraints.

### Characteristics of Value Objects

#### 8.1 No independent identity

A Value Object is not distinguished by a unique identifier.

Two Money objects with the same amount and currency represent the same value.

#### 8.2 Equality by value

Equality is based on the attributes that define the concept.

For example:

```csharp
var first = new Money(100, "USD");
var second = new Money(100, "USD");
```

In the domain, these values should be considered equal even though they may be different object instances.

#### 8.3 Immutability

Value Objects are generally immutable.

Once created, their values do not change. An operation that produces a different value returns a new object.

For example:

```csharp
var original = new Money(100, "USD");
var increased = original.Add(new Money(25, "USD"));
```

The original amount remains unchanged.

Immutability makes Value Objects easier to reason about, share, test, and use safely.

#### 8.4 Domain behavior

Value Objects can contain behavior.

For example, a `Money` object may know how to add another amount, compare values of compatible currencies, or apply a percentage calculation.

A Value Object is not simply a group of properties.

### Entity versus Value Object

| Question | Entity | Value Object |
|---|---|---|
| What defines its identity? | A distinct identity | Its values |
| Can its attributes change? | Yes, while identity remains | Preferably no |
| How is equality determined? | Identity | Relevant values |
| Does it have independent continuity? | Yes | No |
| Example | Customer, Order | Money, Address |

Do not classify concepts based on their names or database representation.

The same real-world concept may be modeled differently in different Bounded Contexts.

**Practical exercise:** Implement a `Money` Value Object.

It should:

- Contain an amount and currency.
- Validate its construction.
- Support equality by value.
- Prevent uncontrolled mutation.
- Define how compatible amounts can be added.
- Reject invalid operations, such as adding incompatible currencies without an explicit conversion rule.

**Learn more:**
- [Value Object - Martin Fowler](https://martinfowler.com/bliki/ValueObject.html)
- [Implementing Value Objects - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects)

---

## 9. Aggregates and Aggregate Roots

Aggregates are among the most important and most frequently misunderstood tactical patterns in DDD.

An Aggregate is a consistency boundary around a group of related domain objects.

It defines which objects must remain consistent together when a business operation is performed.

An Aggregate contains one or more Entities and may also contain Value Objects. Exactly one Entity serves as the Aggregate Root.

The Aggregate Root is the externally accessible entry point for modifying the Aggregate.

### Why do Aggregates exist?

Consider an Order containing multiple Order Items.

An order may have rules such as:

- It must contain at least one item before confirmation.
- A confirmed order cannot be edited freely.
- The total must reflect its items and applicable adjustments.
- A canceled order cannot be shipped.
- The quantity of an item must be positive.

These rules involve multiple parts of the Order.

If any caller can directly modify the Order Items, it becomes difficult to guarantee that the Order remains valid.

An Aggregate creates a boundary around the state and behavior that must be protected together.

### The Aggregate Root

In this example, `Order` is the Aggregate Root.

`OrderItem` is an internal Entity within the Aggregate.

The root provides operations that control changes to its internal members.

Conceptually:

```text
Order (Aggregate Root)
|
+-- OrderItem
|   +-- ProductId
|   +-- Quantity
|   +-- UnitPrice
|
+-- OrderItem
|   +-- ProductId
|   +-- Quantity
|   +-- UnitPrice
|
+-- Money (Total)
+-- OrderStatus
```

External parts of the system should interact with the Aggregate through its root rather than directly modifying its internal objects.

For example:

```csharp
order.AddItem(productId, quantity, unitPrice);
order.RemoveItem(itemId);
order.Confirm();
order.Cancel(reason);
```

The root can validate each operation and ensure that the Aggregate's invariants remain true.

### Aggregate invariants

An invariant is a rule that must remain true whenever the Aggregate is in a valid state.

Examples:

- An order cannot be confirmed without at least one item.
- An order item must have a positive quantity.
- A payment cannot be captured beyond the amount authorized, subject to the payment provider's rules.
- A shipment cannot be dispatched before it has been prepared.

The Aggregate should protect the invariants that must be maintained consistently within its boundary.

Not every business rule belongs inside an Aggregate. Some rules involve multiple Aggregates or processes and may require coordination outside a single transactional boundary.

### Aggregate boundaries are consistency boundaries

This is the critical point.

An Aggregate is not simply a collection of related objects. Two concepts may be strongly related in the business while still belonging to different Aggregates.

For example, an Order may refer to a Customer, but that does not mean the Customer should be included inside the Order Aggregate.

The Order and Customer may have separate identities, responsibilities, and consistency requirements.

A common design approach is to reference another Aggregate by its identity rather than holding a direct object reference.

### Transaction boundaries

A useful rule of thumb is that an operation should maintain the consistency of one Aggregate within a transaction.

When a business process requires changes across multiple Aggregates, the system may need coordination, domain events, or eventual consistency.

This is not an absolute statement that every system must use one database transaction per Aggregate. Technical implementation varies. The important modeling principle is to avoid creating large Aggregate boundaries simply because several objects are related.

Large Aggregates can introduce:

- Unnecessary locking.
- Contention.
- Difficult concurrency management.
- Excessive dependencies.
- Unrelated changes becoming coupled.
- Reduced scalability.

### How to identify an Aggregate

Start with business operations and consistency requirements, not with database relationships.

Ask:

1. Which business rules must be protected together?
2. Which changes must be immediately consistent?
3. Which objects need to participate in the same operation?
4. Which concepts have independent lifecycles?
5. Can some rules be handled through eventual consistency?
6. Is the proposed boundary too large for the operations it protects?

Do not create an Aggregate merely because objects have a one-to-many relationship.

**Practical exercise:** Model an Order with multiple Order Items. Identify its invariants and define the operations that the root should expose. Then consider whether Customer, Product, and Payment should be inside the same Aggregate. Explain your decisions using consistency requirements rather than database relationships.

**Learn more:**
- [DDD Aggregate - Martin Fowler](https://martinfowler.com/bliki/DDD_Aggregate.html)
- [Use Tactical DDD to Design Microservices - Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-ddd)
- [Designing a microservice domain model - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model)

---

## 10. Domain Events

A Domain Event represents something meaningful that has happened in the domain.

An event is expressed as a fact in the past tense.

Examples include:

- OrderPlaced.
- PaymentAuthorized.
- PaymentCaptured.
- ShipmentDispatched.
- DeliveryCompleted.
- RefundApproved.

A Domain Event is not simply a notification that a database record changed. It represents a business occurrence that matters to the model.

### Why Domain Events matter

Consider what happens when an order is placed.

Several things may need to happen:

- The order must be confirmed.
- Inventory may need to be reserved.
- A customer may need to receive a confirmation.
- A fulfillment process may need to begin.
- Analytics may need to record the transaction.

If all of these responsibilities are tightly coupled to the same operation, adding new behavior can become difficult.

A Domain Event provides a way to express the occurrence explicitly so that other parts of the system can react to it.

### Events describe facts, not requests

Compare:

- `PlaceOrder`
- `OrderPlaced`

The first expresses an intention or command: something should happen.

The second expresses a fact: something has happened.

This distinction is important.

A command may be rejected. An event represents a meaningful occurrence that has already taken place.

### Domain Events should carry relevant meaning

A useful event contains the information required to understand the occurrence.

For example, `OrderPlaced` might include:

- OrderId.
- CustomerId.
- OccurredAt.
- Relevant order details.

The exact contents depend on the model and the intended consumers.

Avoid treating events as arbitrary dumps of an entire database record.

### Domain Events and integration events

These concepts are related but not identical.

A **Domain Event** represents an occurrence within the domain model. It may be handled inside the same Bounded Context and application process.

An **Integration Event** communicates a committed business occurrence to another system or Bounded Context.

For example:

1. The Sales context confirms an order.
2. Its model raises an `OrderPlaced` Domain Event.
3. The application handles the event and completes the relevant local work.
4. After the originating transaction commits, an integration message may be published for the Fulfillment context.

The distinction matters because integration introduces concerns such as message delivery, retries, duplicate processing, schema evolution, and eventual consistency.

Publishing a message directly from a domain object is not required by DDD. The model should express the business fact; application and infrastructure components handle delivery.

### Domain Events are not Event Sourcing

Domain Events can be used in systems that do not use Event Sourcing.

Event Sourcing is a separate architectural approach in which changes to an entity or Aggregate are stored as a sequence of events from which state can be reconstructed.

Using Domain Events does not automatically mean using Event Sourcing.

**Practical exercise:** Take an order fulfillment process and identify the important things that happen. Distinguish commands from events. For every event, explain why it matters to the business and which parts of the system may need to react to it.

**Learn more:**
- [Domain Events: Design and Implementation - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)
- [Use Tactical DDD to Design Microservices - Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-ddd)

---

## 11. Repositories

A Repository provides an abstraction for accessing and persisting domain objects, typically Aggregates.

It allows the domain model or application layer to work with domain objects without depending directly on the details of a particular persistence technology.

A Repository provides a collection-like abstraction over the persistence of domain objects.

For example, an application may need to retrieve an Order, modify it, and save the result.

Conceptually:

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

The implementation might use SQL Server, another relational database, a document database, or another persistence mechanism.

The interface expresses what the application needs. The implementation handles how data is stored and retrieved.

### Repositories and Aggregate Roots

A common DDD practice is to define Repositories around Aggregate Roots rather than creating one Repository for every database table or Entity.

For example:

- `OrderRepository` retrieves and persists Order Aggregates.
- `CustomerRepository` retrieves and persists Customer Aggregates.

An Order Item that belongs to the Order Aggregate does not necessarily need its own Repository.

The Aggregate Root controls access to its internal members.

### Repositories are not generic database wrappers

A Repository should not become a collection of arbitrary SQL operations or a generic interface that hides the entire persistence mechanism without meaningful domain boundaries.

Avoid designing a Repository for every Entity simply because every Entity is stored in a database.

Also distinguish command-side persistence from read-side querying. Applications using CQRS may use specialized query models and direct query abstractions rather than forcing every read through an Aggregate Repository.

### Where should Repository interfaces live?

This depends on the architecture and dependency direction.

In many DDD-oriented applications, the Repository abstraction is defined in the domain or application core, while its implementation belongs to the infrastructure layer.

The important principle is that domain behavior should not depend directly on a specific database technology.

**Practical exercise:** Define an `IOrderRepository` interface for the Order Aggregate. Decide which operations are needed by the application and which operations belong to read-specific queries instead.

**Learn more:**
- [Designing a microservice domain model - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model)
- [Domain-Driven Design Reference - Eric Evans](https://www.domainlanguage.com/ddd/reference/)

---

## 12. Services: Domain Services and Application Services

Not every operation naturally belongs to an Entity or Value Object.

Sometimes a business operation involves multiple domain concepts and does not logically belong to any one of them.

DDD uses Services to represent such operations.

However, the word "service" has many meanings in software engineering. A Domain Service, an Application Service, and a Microservice are different concepts.

### 12.1 Domain Services

A Domain Service represents domain behavior that does not naturally belong to a particular Entity or Value Object.

It expresses a business operation that is meaningful in the domain.

For example, imagine a pricing rule that determines a final price using:

- Customer eligibility.
- Product category.
- Promotion rules.
- Purchase quantity.

If this behavior does not naturally belong to one Entity or Value Object, a Domain Service may be appropriate.

A Domain Service should:

- Express meaningful domain behavior.
- Use domain concepts and language.
- Remain focused on a coherent business responsibility.
- Avoid becoming a general-purpose container for unrelated logic.
- Avoid infrastructure responsibilities such as sending emails or accessing HTTP endpoints.

A Domain Service is often stateless, but statelessness alone does not make a class a Domain Service.

Its responsibility and meaning in the model are what matter.

### 12.2 Application Services

An Application Service coordinates a use case.

It orchestrates the steps required to execute an application operation without becoming the owner of the underlying business rules.

For example, an application service responsible for confirming an order might:

1. Retrieve the Order Aggregate.
2. Call the appropriate domain operation.
3. Persist the changed Aggregate.
4. Coordinate transaction handling.
5. Return an appropriate result.

It may also coordinate authentication, authorization, external communication, and other application-level concerns according to the architecture.

The important distinction is that the application service coordinates the work, while the domain model expresses the business rules.

### Domain Service versus Application Service

| Concern | Domain Service | Application Service |
|---|---|---|
| Primary responsibility | Express domain behavior | Orchestrate a use case |
| Business rules | Yes, when they do not belong naturally to an Entity or Value Object | Should delegate to the domain model |
| Domain knowledge | Uses domain concepts | Coordinates domain operations |
| Infrastructure | Should not depend on infrastructure details | May coordinate infrastructure through abstractions |
| Example | Calculate an eligible discount | Confirm an order |

### Avoid the anemic domain model

A common design problem is moving all business logic into application services while Entities become passive structures containing only properties and getters.

This is often called an Anemic Domain Model.

It can be appropriate to use data-oriented models in some architectures, but when applying a behavior-rich DDD model, business rules should be expressed in the domain model rather than scattered across application services.

The goal is not to place every line of logic inside an Entity. It is to assign behavior to the concept or service that best represents its meaning.

**Practical exercise:** Consider a pricing operation involving a customer, product, and promotion. Decide which parts belong to an Entity, which might belong to a Domain Service, and which belong to an Application Service. Explain why.

**Learn more:**
- [Use Tactical DDD to Design Microservices - Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-ddd)

---

## 13. How the concepts work together

The individual concepts are useful, but DDD becomes much easier to understand when you see how they relate to one another.

Let's return to the online retail example.

Imagine a customer placing an order.

### Step 1. Understand the domain

The team explores the sales and fulfillment processes with domain experts.

It discovers that an order has several possible states and that payment authorization, payment capture, and fulfillment are distinct activities.

The team records the relevant rules, decisions, and exceptions.

### Step 2. Establish the strategic boundaries

The team identifies Sales, Inventory, Payments, and Fulfillment as distinct areas of business responsibility.

It explores their models and determines which Bounded Contexts are appropriate.

Each context establishes its own Ubiquitous Language.

### Step 3. Develop the Sales model

Within the Sales Bounded Context, the team identifies concepts such as:

- Customer.
- Order.
- Order Item.
- Money.
- Discount.

It determines that Order has an identity and that Money is a Value Object.

### Step 4. Define the Aggregate

The team identifies the Order Aggregate and its consistency requirements.

The Order Root controls the operations that modify its internal Order Items and protect its invariants.

### Step 5. Implement domain behavior

The model exposes meaningful operations such as `AddItem`, `Confirm`, and `Cancel`.

A Domain Service may handle a pricing calculation if it does not naturally belong to one Entity or Value Object.

### Step 6. Coordinate the use case

An Application Service retrieves the Order, invokes the appropriate domain operation, and coordinates persistence.

A Repository abstracts access to the Order Aggregate.

### Step 7. Communicate meaningful changes

When the order is successfully confirmed, the model raises an `OrderPlaced` Domain Event.

The application handles the event and, where necessary, publishes an Integration Event for the Fulfillment context.

The Fulfillment context interprets that message according to its own model and language.

This example illustrates how the concepts complement one another:

- Strategic Design establishes the boundaries.
- Ubiquitous Language establishes shared meaning.
- Tactical Design expresses domain behavior.
- Repositories support persistence.
- Application Services coordinate use cases.
- Domain Events communicate meaningful changes.

These are not independent building blocks to apply mechanically. They are parts of a coherent design approach.

---

## 14. When should you use DDD?

DDD is particularly valuable when the business domain contains significant complexity that must be represented accurately in software.

It is not automatically the best choice for every application.

### DDD is likely to be valuable when:

- Business rules are complex or frequently changing.
- Different areas of the business use the same terms with different meanings.
- Business decisions involve many exceptions and constraints.
- Misunderstanding the domain creates expensive defects or rework.
- The software provides important competitive capabilities.
- Multiple teams need to collaborate around distinct models.
- The organization needs to evolve its software alongside changing business knowledge.
- Existing systems contain years of accumulated business behavior that must be understood and preserved.

### DDD may be excessive when:

- The application primarily performs straightforward CRUD operations.
- The domain is simple and stable.
- Business rules are minimal.
- There is little need for a rich domain model.
- The cost of collaborative modeling exceeds the expected benefit.
- A simple existing product or framework already solves the problem adequately.

For example, a basic internal application for maintaining a list of office locations may not need extensive strategic modeling, multiple Bounded Contexts, or elaborate Aggregates.

A complex financial platform with sophisticated settlement rules, regulatory constraints, and multiple business processes may benefit substantially from DDD.

### DDD is an investment

DDD requires time and collaboration.

It requires conversations with domain experts, exploration of business rules, refinement of language, and continuous attention to model quality.

The investment is justified when better domain understanding leads to better software and fewer costly misunderstandings.

The objective is not to maximize the amount of DDD used. It is to apply the ideas that help solve the actual problem.

---

## 15. Common misconceptions

Before moving forward, it is worth addressing several mistakes that frequently appear in discussions and implementations of DDD.

| Misconception | A more accurate understanding |
|---|---|
| DDD means using Entities, Value Objects, and Repositories | These are tactical patterns within a broader approach to domain understanding and design |
| Every business noun should become a class | A model should represent meaningful concepts and behavior, not reproduce every noun |
| Every Subdomain must become a Bounded Context | They are different concepts, and their relationship is not necessarily one-to-one |
| Every Bounded Context must become a microservice | A Bounded Context is a model boundary, not a deployment requirement |
| Every Entity should have its own Repository | Repositories are commonly designed around Aggregate Roots |
| Aggregates are groups of related database tables | Aggregates are consistency boundaries based on domain invariants |
| All business logic belongs in Domain Services | Behavior should be assigned to the concept or service that best represents its meaning |
| Domain Events require Event Sourcing | Domain Events can be used without storing the system's state as an event stream |
| Ubiquitous Language means one glossary for the entire organization | Language is defined and maintained within its Bounded Context |
| DDD is a one-time modeling activity | The model evolves as the team's domain knowledge improves |

A particularly important warning: adopting DDD terminology without changing the way the team discovers, discusses, and models the domain rarely produces the intended benefits.

---

## 16. Practical exercises

The best way to understand the foundations is to apply them to a domain you know.

Choose a domain such as:

- Online retail.
- Banking.
- Logistics.
- Healthcare administration.
- Library management.
- Subscription management.
- Transportation.
- Food distribution.

Avoid choosing an example solely because it is easy to implement. Choose one with enough business rules and meaningful decisions to make modeling interesting.

### Exercise 1. Discover the domain

Write a short description of the business.

Identify:

- Its purpose.
- Its main activities.
- The people involved.
- Important business decisions.
- Important constraints.
- Areas of uncertainty.

Interview at least two people involved in the process. Ask them to explain the same scenario independently.

Compare their descriptions. Record differences in terminology and assumptions.

**Deliverable:** A domain overview with questions and uncertainties.

### Exercise 2. Identify Subdomains

Divide the domain into meaningful areas of business responsibility.

Classify each area as Core, Supporting, or Generic.

Explain why you chose each classification.

**Deliverable:** A Subdomain map with a short rationale for each area.

### Exercise 3. Identify Bounded Contexts

Examine terms that appear in multiple areas of the business.

For example, investigate whether "Customer", "Product", or "Order" has the same meaning everywhere.

Identify possible model boundaries and describe the responsibilities of each context.

**Deliverable:** An initial Context Map with context responsibilities and important relationships.

### Exercise 4. Develop the Ubiquitous Language

Create a glossary for one Bounded Context.

For each important term, document:

- Its definition.
- An example.
- Related terms.
- Ambiguities.
- Questions requiring validation.

Use the language in discussions and proposed model names.

**Deliverable:** A context-specific glossary.

### Exercise 5. Model Entities and Value Objects

Identify important concepts and determine which need identity and which are defined by their values.

For each Entity, explain why identity matters.

For each Value Object, define its values, equality, and constraints.

**Deliverable:** A small domain model containing Entities and Value Objects.

### Exercise 6. Design an Aggregate

Choose a meaningful business operation.

Identify the invariants that must be protected and define the smallest useful consistency boundary.

Identify the Aggregate Root and its public operations.

Explain which related concepts should remain outside the Aggregate.

**Deliverable:** An Aggregate design with its invariants and operations.

### Exercise 7. Identify Domain Events

Explore a business process and identify meaningful events.

For each event, explain:

- What happened?
- Why does it matter?
- Which context owns the meaning?
- Which parts of the system may need to react?
- Does it remain internal or need to be communicated externally?

**Deliverable:** A list of Domain Events with descriptions and examples.

### Exercise 8. Identify Services and Repositories

Select one use case and determine:

- Which operations belong to Entities or Value Objects?
- Whether a Domain Service is needed.
- What the Application Service must coordinate.
- Which Aggregate needs a Repository.
- Which operations are better served by read-specific queries?

**Deliverable:** A simple use-case design showing the responsibilities of each component.

---

## 17. Completion criteria

You have a solid understanding of the foundations when you can explain these ideas in your own words and apply them to a real business scenario.

**Understanding the domain**

- [ ] Explain the difference between a domain and a software system.
- [ ] Explain what a domain model represents.
- [ ] Explain why a model is selective and contextual.
- [ ] Identify the difference between Strategic Design and Tactical Design.

**Strategic Design**

- [ ] Explain Subdomains and their Core, Supporting, and Generic classifications.
- [ ] Explain what a Bounded Context is.
- [ ] Distinguish a Subdomain from a Bounded Context.
- [ ] Explain why models need explicit boundaries.
- [ ] Explain how a Ubiquitous Language is developed and maintained.

**Tactical Design**

- [ ] Distinguish Entities from Value Objects.
- [ ] Explain identity and equality.
- [ ] Explain Aggregate boundaries and invariants.
- [ ] Identify an Aggregate Root and its responsibilities.
- [ ] Explain what a Domain Event represents.
- [ ] Distinguish Domain Events from Integration Events.
- [ ] Distinguish Domain Services from Application Services.
- [ ] Explain the purpose of a Repository.

**Applying the ideas**

- [ ] Identify assumptions that need validation with domain experts.
- [ ] Model a small business scenario using the concepts introduced.
- [ ] Explain why each design decision is appropriate.
- [ ] Recognize when DDD may introduce more complexity than value.

You do not need to have mastered every implementation detail at this stage. The goal is to understand the concepts well enough to participate in domain exploration and strategic design.

---

## 18. Further reading and references

The following resources provide a deeper understanding of the concepts introduced in this chapter.

### Essential books

**[Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.domainlanguage.com/ddd/)**

Eric Evans.

The foundational book that introduced the terminology and patterns that shaped modern DDD. It provides a comprehensive treatment of strategic and tactical design, modeling, language, and the relationship between software and domain knowledge.

It is a substantial book. Read it to develop a deeper understanding, not as a prerequisite for beginning practical DDD work.

**[Domain-Driven Design Distilled](https://www.informit.com/store/domain-driven-design-distilled-9780134434421)**

Vaughn Vernon.

A more concise introduction to DDD. Particularly useful for understanding Bounded Contexts, Ubiquitous Language, Subdomains, Context Mapping, Aggregates, and Domain Events.

A good starting point for readers who want an accessible introduction before studying Evans in depth.

**[Learning Domain-Driven Design](https://www.oreilly.com/library/view/learning-domain-driven-design/9781098100124/)**

Vlad Khononov.

A modern and practical treatment of DDD, with useful explanations of strategic design, modeling, and architectural boundaries.

### Reference materials

- [DDD Reference - Eric Evans](https://www.domainlanguage.com/ddd/reference/): A concise reference to the definitions and patterns from the original DDD book.
- [Martin Fowler's DDD articles](https://martinfowler.com/tags/domain%20driven%20design.html): Articles on DDD concepts and related design ideas.
- [Microsoft Learn: Domain-Driven Design](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/domain-analysis): Practical guidance on domain analysis and strategic modeling.
- [Microsoft Learn: Tactical DDD](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-ddd): A practical introduction to Entities, Value Objects, Aggregates, Services, and Domain Events.

### A suggested reading order

1. Start with *Domain-Driven Design Distilled* to understand the overall approach.
2. Read the DDD Reference to become familiar with the terminology.
3. Study the strategic design concepts before concentrating on tactical patterns.
4. Read selected chapters of Eric Evans' book to deepen your understanding.
5. Use *Learning Domain-Driven Design* to connect the concepts with practical architectural decisions.
6. Apply the ideas to a real domain and revisit the references as questions emerge.

Remember that reading about DDD is not a substitute for working with domain experts. The concepts become meaningful when they help you understand a real problem more accurately.

---

## What comes next?

Understanding the foundations gives you a vocabulary and a conceptual framework. But a vocabulary alone does not produce a useful domain model.

The next step is to learn how to discover the domain collaboratively.

You need to observe business processes, explore decisions and exceptions, identify events, challenge assumptions, and develop a shared understanding with the people who know the business.

That is the focus of the next chapter:

**[02. Domain Exploration](02-domain-exploration.md)**

The objective is to move from knowing the terminology of DDD to discovering the language and knowledge of an actual domain.