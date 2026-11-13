# Glossary

This glossary explains the concepts and terminology used throughout Domain-Driven Design. It is not intended to be a dictionary to memorize. These concepts are useful because they help us reason about business complexity, discover meaningful models, and make software reflect the domain it serves.

Some terms describe the domain itself. Others describe ways of organizing models, protecting consistency, or communicating across boundaries.

One concept connects almost all of them: **Ubiquitous Language**. It is the language through which a team discovers and expresses its understanding of the domain.

---

## 1. Ubiquitous Language (The Language We Cultivate)

Ubiquitous Language (UL) is the shared, evolving language used by domain experts and software developers to describe and reason about a particular domain.

It is used in conversations, workshops, examples, business rules, documentation, tests, and code. The goal is not merely to make developers understand business terminology. It is to create a common way of thinking and communicating about the domain, so that the software model and the business understanding reinforce each other.

But there is an important distinction:

**Ubiquitous Language is not a glossary that a team writes once and distributes. It is a language that a team continuously cultivates.**

People may use the same word while meaning different things. They may use different words for the same concept. Some terms may be vague, overloaded, or inherited from an old system. Even domain experts may disagree because they work with different parts of the business or have different perspectives.

These are not merely communication problems. They are opportunities to discover the domain.

### Why language matters

Consider the word `Customer`.

In one business, a customer might be someone who has registered an account. In another, it might mean someone who has completed a purchase. In a financial context, it might mean an individual or organization with an active contractual relationship.

Even within one organization, different departments may use the same word differently.

If developers assume that all these people mean the same thing, the software may end up with one `Customer` model trying to represent several different concepts.

The problem is not necessarily that the team needs a better class name. It may be that the model has not yet discovered the distinctions that matter.

The same applies to verbs.

Consider `Approve`.

- Does approval mean that someone has reviewed a request?
- Does it mean the request is eligible to proceed?
- Does it mean a financial commitment has been authorized?
- Does it mean the request has been finalized?

These distinctions may lead to different business rules, responsibilities, and state transitions.

A good Ubiquitous Language makes these distinctions explicit.

### The language-model-code feedback loop

Ubiquitous Language should not live exclusively in meetings or documentation. It should influence the software model, and the software model should make the language more precise.

A useful feedback loop looks like this:

```text
      Domain Conversations
               |
               v
       Discover Language
               |
               v
       Explore Examples
               |
               v
        Refine the Model
               |
               v
       Express It in Code
               |
               v
       Validate with Experts
               |
               +-----------> New Questions
                                |
                                v
                         Refine Language
```

For example, a conversation may initially use the term `OrderApproved`.

During exploration, the team discovers that an order first passes a commercial review and then a financial authorization. These are two different decisions, owned by different people and governed by different rules.

The original term hides this distinction.

The team may introduce:

```text
OrderCommerciallyReviewed
OrderFinanciallyAuthorized
```

The model becomes more precise because the language becomes more precise.

The reverse can also happen. Developers may introduce a technical distinction that domain experts do not recognize as meaningful. That is an opportunity to question the model rather than force the terminology into the business.

### A practical recipe for cultivating Ubiquitous Language

The following process can be used in a discovery workshop, an Event Storming session, a refinement meeting, or an ongoing collaboration between developers and domain experts.

It is deliberately iterative. You do not need to complete every step before writing code, and you should expect to revisit previous decisions.

**Step 1. Collect the words people actually use.**

Start with conversations, documents, forms, reports, existing software, support tickets, and real business examples.

Do not immediately replace unfamiliar terminology with technical terms or what you think are better names.

Capture:

- Nouns: customer, shipment, allocation, contract, claim.
- Verbs: allocate, approve, dispatch, settle, suspend.
- States: pending, confirmed, reserved, expired.
- Business expressions: available capacity, eligible customer, outstanding balance.

Preserve the original expressions, including synonyms, abbreviations, and inconsistencies.

The objective is to discover the language as it is currently used, not to impose a vocabulary.

**Step 2. Find ambiguity and disagreement.**

Ask people what they mean by each important term.

Useful questions include:

- What exactly makes someone a customer?
- When does a shipment become dispatched?
- Is an approved request necessarily fulfilled?
- Can an allocation be changed after confirmation?
- Are these two terms synonyms, or do they describe different things?
- Can this happen without that happening?

Pay particular attention to words that different people use confidently but define differently.

Do not resolve every disagreement by voting. A disagreement may reveal different business responsibilities, processes, or models.

**Step 3. Explore concrete examples.**

Abstract definitions are often misleading. Ask domain experts to walk through actual situations.

For example:

- Show me an order that is considered confirmed.
- Show me an order that was reviewed but not confirmed.
- What happens when the requested quantity is unavailable?
- Can a shipment be dispatched without an assigned vehicle?
- What happens when a payment is received after an order has expired?

Examples expose the rules and distinctions hidden behind words.

For each important concept, explore normal cases, exceptional cases, and borderline cases.

**Step 4. Define the meaning, not just the name.**

For each important term, capture:

- Its meaning in this context.
- What qualifies as an instance of the concept.
- What does not qualify.
- Its relationships with other concepts.
- Its lifecycle, if applicable.
- The rules and decisions associated with it.
- Examples and counterexamples.

Avoid definitions that merely repeat the term.

For example:

> A confirmed order is an order that has passed the required commercial and financial checks and is eligible to enter fulfillment.

This is more useful than:

> A confirmed order is an order whose status is Confirmed.

The first describes a business meaning. The second describes an implementation detail.

**Step 5. Express the language in the model and code.**

Use the discovered terms in:

- Class and type names.
- Method names.
- Domain Events.
- Commands.
- Business rules.
- Tests.
- API contracts, where appropriate.
- Documentation and diagrams.

For example:

```csharp
public class Order
{
    public void Confirm()
    {
        EnsureCommercialRequirementsAreMet();
        EnsureFinancialAuthorizationExists();

        Status = OrderStatus.Confirmed;

        RaiseDomainEvent(
            new OrderConfirmed(Id));
    }
}
```

The code should make the business operation recognizable to someone who understands the domain.

This does not mean every business phrase must become a class. The objective is to express the model clearly, not to turn the vocabulary into an object catalogue.

**Step 6. Validate the model with domain experts.**

Show people the model, tests, and examples.

Ask them:

- Does this represent what actually happens?
- Is this the right term?
- Does this operation mean what its name suggests?
- Are any important distinctions missing?
- Can you think of a real situation where this model would be wrong?

If a domain expert cannot recognize the behavior expressed in the code, investigate why.

Sometimes the code is poorly named. Sometimes the model is wrong. Sometimes the business understanding itself needs further exploration.

**Step 7. Keep the language alive.**

New requirements, exceptional cases, operational experience, and changes in the business will reveal new concepts and invalidate old assumptions.

Review the language when:

- A new business capability is introduced.
- A term becomes overloaded.
- Developers repeatedly need explanations for a concept.
- Different teams use the same word differently.
- A business rule changes.
- The model no longer reflects how the business operates.

Update the model, code, tests, and relevant documentation together.

A glossary is useful, but it is only a snapshot of the team's current understanding.

### The Ubiquitous Language Canvas

Use this lightweight canvas to explore an important domain term with the people who understand the business.

It is not a bureaucratic documentation requirement. It is a conversation aid. Some terms will need only a few fields; others will require substantial exploration.

| Canvas field | Questions to explore |
|---|---|
| Term | What word or expression do people actually use? |
| Context | In which part of the business does this meaning apply? |
| Meaning | What does the term mean in this context? |
| Examples | What are real examples of it? |
| Counterexamples | What looks similar but is not an instance of it? |
| Related terms | What other concepts are connected to it? |
| Actions | What can happen to it? What actions does it participate in? |
| Lifecycle | Does it have states? What causes transitions? |
| Rules | What must always be true? What can prevent an action? |
| Synonyms and ambiguity | Are there competing words or different interpretations? |
| Ownership | Who understands, decides, or is responsible for this concept? |
| Model expression | How should the concept appear in the domain model and code? |
| Open questions | What do we still not understand? |

For example:

**Term: Shipment Dispatched**

| Field | Example |
|---|---|
| Context | Transportation |
| Meaning | A shipment has left its origin facility and entered the transportation process. |
| Examples | A loaded truck leaves the warehouse with the shipment. |
| Counterexamples | A shipment has been assigned to a truck but has not left the facility. |
| Related terms | Shipment, Vehicle, Driver, Delivery |
| Actions | Dispatch shipment |
| Lifecycle | Prepared → Dispatched → Arrived |
| Rules | A shipment must be ready for dispatch and have the required transportation assignment. |
| Synonyms and ambiguity | "Sent" and "shipped" are used inconsistently. |
| Ownership | Transportation operations |
| Model expression | `Shipment.Dispatch()` and `ShipmentDispatched` |
| Open questions | Can a partially loaded shipment be dispatched? |

Notice how this exercise reveals a distinction between assigning a vehicle and actually dispatching a shipment. That distinction may become important in the domain model.

### Ubiquitous Language is contextual

A common mistake is to assume that an organization should have one universal vocabulary for every system and department.

DDD does not require that.

A term can legitimately have different meanings in different Bounded Contexts. The language should be consistent within its context, not artificially identical across the entire organization.

For example, `Customer` may mean:

- A person or organization with an active commercial relationship in Sales.
- A person who has submitted a support request in Customer Support.
- An account holder with an active financial relationship in Billing.

These models may be related, but they do not necessarily represent the same concept.

Trying to force them into one universal `Customer` model can create more confusion than clarity.

**The goal is not one language for everyone. It is a precise language for each model, with explicit translations and relationships where models meet.**

### Signs that your Ubiquitous Language needs attention

- Developers and domain experts use the same word but explain it differently.
- Important business rules are hidden behind generic names such as `Process`, `Handle`, `Manage`, or `Update`.
- Code contains many abbreviations that only a few developers understand.
- The same concept has multiple names across documentation, tests, and code.
- A single class represents several business concepts.
- Developers routinely translate business terminology into technical terminology before discussing requirements.
- Tests describe implementation details rather than business behavior.
- Domain experts cannot recognize important business operations in the model.
- Teams disagree about what a business state actually means.

These are signals to investigate, not automatic proof that the model is wrong.

---

## 2. Domain Model (The Model of Understanding)

A Domain Model is a conceptual and behavioral representation of the business domain. It captures the concepts, relationships, rules, decisions, and behaviors that are important to a particular model.

It is not necessarily a diagram, a collection of Entity classes, or a database schema.

A useful Domain Model expresses how the business works and helps software make decisions consistently.

For example, an order model may capture:

- What makes an order valid.
- When an order can be confirmed.
- Which conditions prevent cancellation.
- How an order's total is calculated.
- What happens when an order is dispatched.

The model is an abstraction of reality, not a complete copy of it. It should represent the aspects of the domain that matter to the software and the business capabilities it supports.

## 3. Entity (Has an Identity)

An Entity is an object whose identity remains important throughout its lifecycle.

Two Entities may have identical attributes but still represent different things because they have different identities.

For example, two customers may have the same name and address, but they are still different customers.

```csharp
public class Customer
{
    public CustomerId Id { get; private set; }

    public string Name { get; private set; }
}
```

Identity is not necessarily the same as a database primary key. It is a domain concept used to distinguish one thing from another.

## 4. Value Object (Defined by Its Attributes)

A Value Object represents a descriptive concept whose identity is determined by its values rather than a persistent identity.

Two Value Objects with equivalent values are considered equal.

Value Objects are commonly immutable and are useful for expressing domain concepts with their own validation and behavior.

Examples include:

- `Money`
- `Address`
- `Quantity`
- `DateRange`
- `Percentage`
- `EmailAddress`

For example, `Money` can encapsulate an amount and currency, and prevent meaningless operations such as adding incompatible currencies without an explicit conversion.

A Value Object is not simply a class with several properties. It should represent a meaningful concept in the domain.

## 5. Aggregate (The Consistency Boundary)

An Aggregate is a cluster of associated domain objects treated as a unit for enforcing consistency rules.

It has an Aggregate Root through which external code accesses and modifies the Aggregate.

An Aggregate is primarily a consistency boundary, not merely an object graph or a convenient way to group related Entities.

For example, an `Order` Aggregate may contain an Order Entity and its Order Items. The Order Root protects rules such as preventing changes to an order after it has been dispatched.

Not every object that is conceptually related to an order must belong to its Aggregate. Aggregate boundaries should be based on business invariants and consistency requirements, not on object relationships alone.

## 6. Aggregate Root (The Guardian)

The Aggregate Root is the Entity that represents the external entry point to an Aggregate.

It is responsible for protecting the Aggregate's invariants and controlling changes within its consistency boundary.

For example, an `Order` may be the Aggregate Root for its Order Items. Other parts of the system should use the Order's behavior rather than independently modifying its internal items.

The Aggregate Root is not simply the top-level object in a hierarchy. It is the object responsible for enforcing the consistency rules of the Aggregate.

## 7. Domain Service (A Domain Operation That Doesn't Belong to an Entity or Value Object)

A Domain Service represents domain behavior that does not naturally belong to a particular Entity or Value Object.

It is useful when an operation is genuinely part of the domain but assigning it to one particular object would make the model misleading.

Examples might include:

- A domain-specific funds transfer policy.
- A pricing calculation involving several domain concepts.
- A compatibility decision between two business entities.
- A domain policy for selecting an eligible supplier.

A Domain Service should express domain behavior, not become a general-purpose place for application orchestration, database access, or infrastructure operations.

If the operation belongs naturally to an Entity or Value Object, prefer putting it there.

## 8. Domain Event (Something That Happened)

A Domain Event represents a meaningful occurrence in the domain.

It describes a fact, usually using the past tense:

- `OrderPlaced`
- `PaymentReceived`
- `ShipmentDispatched`
- `CustomerRegistered`

A Domain Event is different from a Command, which expresses an intention or request.

For example, `PlaceOrder` is a request, while `OrderPlaced` records the successful occurrence.

Domain Events make important business occurrences explicit and allow other parts of a model to react to them.

When an event crosses a Bounded Context or process boundary, it may be translated into an Integration Event with an explicitly designed public contract.

## 9. Repository (The Aggregate Access Abstraction)

A Repository provides an abstraction for retrieving and persisting Aggregates.

It allows the domain model and application logic to work with Aggregates without depending directly on the details of the underlying persistence mechanism.

For example:

```csharp
public interface IOrderRepository
{
    Task<Order?> GetById(OrderId id);

    Task Add(Order order);
}
```

Repositories are generally designed around Aggregate Roots rather than individual database tables or every Entity.

A Repository is not necessarily a generic CRUD abstraction. Its purpose is to support the lifecycle and persistence of domain Aggregates.

## 10. Subdomain (A Part of the Business Domain)

A Subdomain is a distinct area of the overall business domain.

It represents a particular set of business capabilities, responsibilities, and knowledge.

For example, an e-commerce business may have:

- Sales
- Inventory Management
- Fulfillment
- Payments
- Customer Support

Subdomains are discovered by understanding the business, not by looking at the current application's modules, database schemas, or organizational chart.

## 11. Bounded Context (The Boundary of a Model's Meaning)

A Bounded Context is an explicit boundary within which a particular domain model and its Ubiquitous Language are valid.

The same term may have different meanings in different Bounded Contexts.

For example, `Customer` may represent a commercial buyer in Sales and a person requesting assistance in Customer Support.

Both models may be valid within their respective contexts.

A Bounded Context is a boundary of meaning and model consistency. It is not automatically a microservice, a deployment unit, or a team boundary, although it can influence those architectural decisions.

## 12. Context Map (The Map of Model Relationships)

A Context Map describes the relationships between Bounded Contexts.

It helps teams understand how models interact, where translations occur, and how dependencies and integration responsibilities are organized.

Common patterns include:

- Shared Kernel
- Customer-Supplier
- Conformist
- Anti-Corruption Layer
- Open Host Service
- Published Language
- Separate Ways

A Context Map is more than a diagram of services. It describes relationships between models and the organizational and technical realities that shape those relationships.

## 13. Core Domain (The Heart of the Business)

The Core Domain is the part of the business that provides its most important competitive advantage or strategic differentiation.

It is where an organization should focus its most valuable modeling, design, and development effort.

For example, route optimization may be the Core Domain of a logistics company whose competitive advantage comes from delivering goods more efficiently than its competitors.

The Core Domain is specific to the organization's strategy. It cannot be identified reliably from technology choices or the apparent complexity of a system.

## 14. Supporting Subdomain (Supporting the Core)

A Supporting Subdomain is an area of the business that is necessary to support the Core Domain but does not itself provide the primary competitive differentiation.

For example, a logistics company may have specialized shipment coordination capabilities that support its route optimization business.

Supporting Subdomains still matter. Their design should be appropriate to their business importance and complexity.

## 15. Generic Subdomain (Common Functionality)

A Generic Subdomain represents functionality that is not specific to the organization's competitive advantage and is often available through established solutions or common approaches.

Examples may include:

- Authentication
- Document generation
- Standardized tax calculations
- Generic notification delivery

Generic does not mean unimportant. It means that building a unique solution is unlikely to provide significant strategic advantage.

## 16. Event Sourcing (The Event Stream as the Source of Truth)

Event Sourcing is a persistence strategy in which changes to an Aggregate are stored as a sequence of events rather than only storing its latest state.

The current state can be reconstructed by replaying the events.

For example:

```text
AccountOpened
MoneyDeposited
MoneyWithdrawn
MoneyDeposited
```

These events represent the history of the account. Replaying them can reconstruct its current balance.

Event Sourcing can provide powerful capabilities for historical reconstruction, auditing, and temporal analysis, but it also introduces complexity around event evolution, projections, concurrency, and operational management.

It is not a mandatory part of Domain-Driven Design. A system can use Domain Events without using Event Sourcing.

---

## A Final Thought: Language Is Never Finished

The glossary above is a reference, not a specification that every team must adopt word for word.

A team's Ubiquitous Language should emerge from its domain and remain open to revision. Some terms will become stable. Others will be challenged, replaced, or split as the team discovers more about the business.

The important thing is to keep asking:

- Are we using this word to mean the same thing?
- Does this concept represent something real in the domain?
- Are we hiding an important distinction behind a generic term?
- Does our code express the same meaning as our conversations?
- What have we learned that should change our model?

A model is not made accurate by using the right DDD patterns. It becomes more useful as the team develops a clearer understanding of the domain and expresses that understanding consistently.

**Cultivate the language, and the model has something meaningful to grow from.**