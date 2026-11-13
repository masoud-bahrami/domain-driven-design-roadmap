# 06 Architecture and Integration

Domain-Driven Design does not prescribe a particular architectural style. It does, however, give us a way to make architectural decisions based on the structure and complexity of the business domain.

Strategic design helps us discover meaningful boundaries. Architectural design determines how those boundaries are represented in software, how they are deployed, how they communicate, and how they evolve.

A Bounded Context might be implemented as a module inside a monolith, an independently deployed service, or even a separately maintained application. These are different architectural decisions, and they should not be confused.

The goal is not to make every Bounded Context a microservice. The goal is to preserve the integrity of each model while choosing an architecture that supports the business and the people building the system.

## Key Concepts

### 1. Strategic Design and Architecture

Strategic DDD decisions have architectural consequences.

Subdomains help us understand the business capabilities and where complexity or competitive advantage lies. Bounded Contexts establish the boundaries within which particular models and languages are valid. Context Maps describe the relationships and dependencies between those contexts.

Together, these concepts provide valuable input for architectural design.

However, a Context Map is not a deployment diagram. It describes relationships between models and teams, not necessarily network connections, processes, or infrastructure.

Consider an e-commerce system with three Bounded Contexts:

- **Product Catalog:** Describes products, categories, and product information.
- **Order Management:** Handles order creation, confirmation, and fulfillment decisions.
- **Payment:** Manages payment authorization, capture, and refunds.

These contexts have distinct responsibilities and languages. But this discovery alone does not tell us whether they should run in three processes, three services, or one application.

We still need to consider:

- How independently do these capabilities need to evolve?
- Which teams own them?
- Do they require independent deployment?
- Where are strong transactional consistency guarantees necessary?
- What are their performance and availability requirements?
- How much operational complexity can the organization manage?

**The architectural boundary should follow meaningful business and organizational boundaries, but the deployment boundary must also be justified by operational needs.**

Learning objectives:

- Explain how Subdomains, Bounded Contexts, and Context Maps influence architectural decisions.
- Distinguish a business-model boundary from a code and deployment boundary.
- Propose more than one architectural design for the same domain.
- Explain the trade-offs behind each proposal.

### 2. Architectural Boundaries: Model, Code, and Deployment

Before comparing architectural styles, understand three different kinds of boundaries.

**Semantic boundary**

A semantic boundary defines the scope in which a particular domain model and language have a consistent meaning. This is the concern of a Bounded Context.

For example, a `Customer` in the Sales Context may represent a prospective buyer, while a `Customer` in the Billing Context represents an entity with financial obligations and billing details.

They may refer to the same real-world person, but they do not necessarily represent the same model.

**Code boundary**

A code boundary separates responsibilities and controls dependencies inside the software. Modules, packages, assemblies, and internal interfaces can help enforce these boundaries.

A modular monolith can contain several Bounded Contexts while keeping them in one deployable application.

**Deployment boundary**

A deployment boundary determines which parts of the system can be built, released, scaled, and operated independently.

A microservice is an independently deployable unit. A module inside a monolith generally is not.

These boundaries can align, but they do not have to.

For example, one Bounded Context may be implemented using several internal modules, or several closely related contexts may initially live in one deployable application. The important question is whether their models and dependencies remain explicit and understandable.

### 3. Architectural Styles and Their Fit with DDD

Architectural styles provide different ways to organize software. None of them automatically produces a good domain model.

#### 3.1 Layered Architecture

Layered Architecture separates an application into logical layers, each with a particular responsibility.

A common arrangement is:

- **Presentation Layer:** Handles user interaction and external requests.
- **Application Layer:** Coordinates use cases and application workflows.
- **Domain Layer:** Contains domain models, business rules, and domain behavior.
- **Infrastructure Layer:** Implements technical concerns such as persistence, messaging, and external integrations.

The Domain Layer should not depend on infrastructure implementation details. For example, a domain model should not need to know whether its data is stored in SQL Server, PostgreSQL, or another persistence technology.

This separation helps protect the domain model from technical concerns.

However, layered architecture does not automatically mean that dependencies point inward. That depends on the dependency rules chosen by the design.

#### Related approaches: Hexagonal, Onion, and Clean Architecture

These approaches share a concern for isolating important business logic from external technologies, but they are not interchangeable names for the same architecture.

**Hexagonal Architecture (Ports and Adapters)**

Introduced by Alistair Cockburn, Hexagonal Architecture organizes an application around its capabilities and defines ports through which external actors and technologies interact with it.

Adapters implement those ports for specific technologies, such as HTTP, a database, or a message broker.

**Onion Architecture**

Popularized by Jeffrey Palermo, Onion Architecture places the domain model at the center and organizes dependencies so that outer layers depend on inner layers.

**Clean Architecture**

Described by Robert C. Martin, Clean Architecture organizes software around policies and details, emphasizing the Dependency Rule: source-code dependencies should point inward, toward higher-level policies.

All three can support DDD by protecting domain logic from infrastructure concerns. They provide different ways of expressing and enforcing that separation.

**DDD fit**

These approaches are useful when an application has meaningful domain behavior and we want to keep that behavior independent of delivery mechanisms and infrastructure choices.

They can be used inside a modular monolith or within an individual microservice.

A simple CRUD application may not need all these abstractions. Adding layers and interfaces without a meaningful reason can make a straightforward application harder to understand.

Example:

An order-management module may contain its domain model, application use cases, and persistence adapters. The domain model defines what it means to confirm an order, while the infrastructure layer handles how orders are stored.

Learning objectives:

- Understand the responsibility of each layer.
- Explain the difference between logical separation and dependency direction.
- Understand Ports and Adapters, Onion Architecture, and Clean Architecture.
- Apply these approaches without introducing unnecessary abstractions.

Further reading:
- [Clean Architecture, Robert C. Martin](https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/)
- [Domain-Driven Design, Eric Evans](https://www.domainlanguage.com/ddd/)
- [Hexagonal Architecture, Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
- [The Onion Architecture, Jeffrey Palermo](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/)

#### 3.2 Modular Monolith

A modular monolith is a single deployable application composed of clearly separated modules.

Each module owns a coherent set of responsibilities and exposes an explicit interface to the rest of the application.

A modular monolith can contain multiple Bounded Contexts. Each context can have its own domain model, application logic, persistence abstractions, and internal language.

For example, an e-commerce application might have:

- Catalog Module
- Ordering Module
- Payment Module
- Shipping Module

These modules run within the same application process and may share infrastructure resources, but their internal models and responsibilities should remain separate.

A well-designed modular monolith aims to prevent modules from accessing one another's internal implementation directly. Communication should happen through explicit contracts or well-defined interfaces.

**Advantages**

- Simpler deployment and operations than a distributed system.
- Easier local debugging and testing across module boundaries.
- In-process communication without network latency.
- Potentially simpler transaction management.
- Clear module boundaries without requiring independent services.

**Trade-offs**

- Modules usually share a deployment lifecycle.
- A failure in the application process can affect every module.
- Independent scaling of individual modules is limited.
- Poor dependency management can gradually turn the application into a tightly coupled monolith.

A modular monolith is not a compromise or an inferior version of microservices. It is a valid architectural choice, especially when independent deployment is not yet a business requirement.

It can also provide a useful foundation for extracting services later, provided the module boundaries are real rather than merely folders and namespaces.

#### 3.3 Microservices Architecture

Microservices Architecture structures a system as a collection of independently deployable services.

DDD is frequently used to discover service boundaries because Bounded Contexts provide a useful starting point for identifying cohesive business capabilities.

However, a Bounded Context and a microservice are not the same thing.

A Bounded Context is a model boundary. A microservice is a deployment and operational boundary.

A Bounded Context may be implemented as one microservice, but it can also remain inside a modular monolith. Conversely, a complex context may require more than one independently deployed component.

**Bounded Contexts as service boundaries**

When a Bounded Context is implemented as a microservice, it should own its domain model and the data necessary to fulfill its responsibilities.

Other services should interact with it through published contracts rather than directly accessing its internal database or domain objects.

For example:

- Product Catalog owns product descriptions and categorization.
- Order Management owns order lifecycle and order-specific decisions.
- Payment owns payment transactions and their outcomes.

The fact that all three contexts need product or customer information does not mean they should share the same domain entities or database tables.

Each context should consume the information it needs through an appropriate integration contract and represent it in its own model.

**Inter-service communication**

Services commonly communicate through:

- Synchronous APIs such as REST or gRPC.
- Asynchronous messaging through brokers and event streams.
- A combination of both.

Synchronous communication can be appropriate when an immediate response is required. Asynchronous communication is useful when the sender should not depend on the receiver being available at that moment.

Both approaches introduce trade-offs, which we will explore later in this section.

**Data ownership and consistency**

A service should own its data and control how other services access it.

This does not necessarily require a physically separate database server for every service. It does require ownership boundaries that prevent other services from depending directly on its internal schema or manipulating its data.

Once data is distributed across services, a single ACID transaction generally cannot cover the entire business process in the same way it can inside one database transaction.

Systems must account for eventual consistency, retries, duplicate messages, partial failures, and business-level recovery.

Patterns such as Sagas can coordinate multi-step business processes across services.

**Advantages**

- Independent deployment and release cycles.
- Independent scaling where workloads differ.
- Greater autonomy for teams responsible for individual services.
- Failure isolation when services are designed with appropriate resilience.
- Freedom to evolve implementation and persistence choices within service boundaries.

**Trade-offs**

- Network communication introduces latency and failure modes.
- Distributed transactions become more difficult.
- Observability, testing, deployment, and infrastructure become more demanding.
- Data duplication and eventual consistency require deliberate design.
- Poorly defined boundaries can create a distributed monolith.

Microservices are most useful when their benefits justify these costs, particularly when independent delivery, scaling, or organizational autonomy is important.

They are not a shortcut for fixing a poorly structured codebase. Distributing a tangled system usually distributes its problems as well.

Learning objectives:

- Explain how Bounded Contexts can inform service boundaries.
- Understand service autonomy and data ownership.
- Design service contracts without exposing internal domain models.
- Identify distributed-system failure modes and operational responsibilities.
- Recognize the difference between microservices and a distributed monolith.

Further reading:
- [Building Microservices, Sam Newman](https://samnewman.io/books/building_microservices_2nd_edition/)
- [Microservices Architecture, Microsoft Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/microservices)
- [Data Considerations for Microservices, Microsoft](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations)
- [Microservice Trade-Offs, Martin Fowler](https://martinfowler.com/articles/microservice-trade-offs.html)

#### 3.4 Event-Driven Architecture

Event-Driven Architecture (EDA) organizes communication and coordination around events.

An event communicates that something has happened. Consumers can react to that occurrence without requiring the producer to know every consumer or coordinate all their internal behavior.

DDD provides useful concepts for designing meaningful events, particularly Domain Events and Integration Events.

**Domain Events**

A Domain Event represents something significant that happened within a domain model.

Examples:

- `OrderConfirmed`
- `PaymentAuthorized`
- `ShipmentDispatched`

A domain event should express business meaning, not merely report a technical operation.

For example, `OrderConfirmed` describes a meaningful business transition. `OrderTableUpdated` describes a persistence detail.

Domain Events are often used within a Bounded Context to coordinate behavior between aggregates or application components.

**Integration Events**

An Integration Event communicates a committed business fact to other Bounded Contexts or external systems.

For example, after an order has been confirmed and the relevant transaction has committed, the Order Management Context may publish an `OrderConfirmed` integration event.

The Payment Context can consume that event and initiate its own payment workflow.

Domain Events and Integration Events are related, but they serve different purposes:

- Domain Events belong to the internal model and language of a context.
- Integration Events are published contracts intended for communication across boundaries.

An internal domain event should not automatically become a public integration contract. The publishing context should decide what information other contexts need and express it in a stable, intentional format.

**Message brokers**

Message brokers and event-streaming platforms help distribute messages between producers and consumers.

Examples include:

- RabbitMQ
- Apache Kafka
- Azure Service Bus

These technologies offer different messaging and delivery models. Choosing one should depend on the communication requirements, delivery guarantees, throughput, ordering, retention, and operational constraints.

A message broker does not automatically make a system event-driven in a meaningful architectural sense. The important part is the design of the events, ownership of contracts, and behavior of their consumers.

**Reliability considerations**

Asynchronous messaging introduces its own challenges:

- Messages may be delivered more than once.
- Consumers may fail while processing messages.
- Messages may arrive out of order.
- A consumer may be unavailable for an extended period.
- A database transaction may commit while publishing the corresponding message fails.

Common techniques include:

- **Idempotent consumers:** Processing the same message more than once should not produce unintended duplicate effects.
- **Transactional Outbox:** Store the business change and the message to be published in the same local database transaction, then publish the message separately.
- **Retry policies:** Retry transient failures with appropriate limits and backoff.
- **Dead-letter handling:** Isolate messages that cannot be processed successfully.
- **Schema evolution:** Change event contracts in a way that existing consumers can tolerate.
- **Observability:** Track message correlation, processing outcomes, retries, and failures.

**Event-Driven Architecture is not Event Sourcing**

These concepts are frequently combined, but they solve different problems.

Event-Driven Architecture concerns communication and coordination through events.

Event Sourcing concerns persistence: instead of storing only the latest state, the system stores a sequence of events from which the current state can be reconstructed.

A system can use EDA without Event Sourcing. It can also use Event Sourcing without adopting a system-wide event-driven architecture.

Learning objectives:

- Distinguish Domain Events from Integration Events.
- Explain how event-driven communication supports loose coupling.
- Understand asynchronous delivery and its failure modes.
- Recognize the role of brokers, Outbox, retries, and idempotency.
- Distinguish Event-Driven Architecture from Event Sourcing.

Further reading:
- [Designing Event-Driven Systems, Ben Stopford](https://www.confluent.io/designing-event-driven-systems/)
- [What Do You Mean by Event-Driven?, Martin Fowler](https://martinfowler.com/articles/201701-event-driven.html)
- [Transactional Outbox pattern, Microsoft](https://learn.microsoft.com/en-us/azure/architecture/databases/guide/transactional-outbox-cosmos)
- [Domain Events and Integration Events, Microsoft](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)

#### 3.5 CQRS and Event Sourcing

CQRS and Event Sourcing are specialized patterns that can have a significant impact on application architecture.

They are not requirements of DDD, microservices, or Event-Driven Architecture.

**Command Query Responsibility Segregation (CQRS)**

CQRS separates operations that change state from operations that retrieve information.

A command represents an intention to change the system:

- `ConfirmOrder`
- `CancelOrder`
- `AuthorizePayment`

A query retrieves information without changing the system:

- `GetOrderDetails`
- `GetCustomerOrders`
- `GetPaymentStatus`

The command side and query side may use different models, data structures, or persistence strategies.

CQRS can be introduced at different levels:

- Within a single application, using separate command and query handlers.
- Within a Bounded Context, using distinct write and read models.
- Across separate components, potentially with different data stores and independent scaling.

CQRS does not require separate databases or microservices. A simple implementation may use one database with separate models and handlers.

The main benefit is that the write model can focus on enforcing business rules while the read model can be optimized for the queries the application needs.

The cost is additional complexity, particularly when separate read models must be updated asynchronously.

Use CQRS when the complexity of the write model, the diversity of read requirements, or the need for independent optimization justifies the separation. Do not introduce it simply because the application uses DDD.

**Event Sourcing**

Event Sourcing stores changes as a sequence of events rather than treating the latest state as the only persisted representation.

For example, an order might have the following event history:

1. `OrderCreated`
2. `OrderItemAdded`
3. `OrderItemAdded`
4. `OrderConfirmed`
5. `OrderCancelled`

The current state can be reconstructed by replaying the events in sequence.

This approach can be useful when the business needs a complete history of changes, auditability, temporal reasoning, or the ability to build new projections from historical events.

However, it introduces significant design and operational considerations:

- Event schema evolution and versioning.
- Event replay and projection rebuilding.
- Snapshot strategies for long histories.
- Event ordering and concurrency control.
- Handling external side effects during replay.
- Data retention, privacy, and correction requirements.

Event Sourcing is not simply a better way to store data. It changes how the application models state, handles corrections, and evolves its persistence strategy.

**Combining CQRS and Event Sourcing**

These patterns are often used together:

- Commands are validated and handled by the write model.
- The write model records events.
- Projections consume those events to build read models.
- Queries retrieve data from the appropriate read model.

This can be powerful, but it also means the read model may temporarily lag behind the write model.

Learning objectives:

- Explain the difference between commands and queries.
- Understand how CQRS can be applied at different architectural levels.
- Explain Event Sourcing and event replay.
- Identify when these patterns are useful and when their complexity is unjustified.

Further reading:
- [CQRS, Martin Fowler](https://martinfowler.com/bliki/CQRS.html)
- [CQRS Pattern, Microsoft Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [Event Sourcing Pattern, Microsoft Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)

### 4. Integration Between Bounded Contexts

Bounded Contexts are independent models, but they rarely exist in complete isolation. They often need to exchange information or coordinate business processes.

The way they integrate is an architectural decision with direct consequences for coupling, consistency, availability, and complexity.

#### 4.1 Synchronous Integration

In synchronous integration, one context calls another and waits for a response.

For example:

1. Order Management receives a request to confirm an order.
2. It calls Payment to authorize the payment.
3. Payment returns success or failure.
4. Order Management decides how to proceed.

This approach is straightforward when an immediate response is required.

However, the caller becomes dependent on the availability and response time of the receiver.

If the Payment service is unavailable, Order Management must decide whether to fail the operation, retry, or follow another business rule.

Excessive synchronous dependencies can create cascading failures and increase response latency.

Use synchronous integration when:

- The caller genuinely needs an immediate answer.
- The dependency is part of the operation's required decision.
- The interaction has clear timeout and failure behavior.
- The number of dependent calls is controlled.

#### 4.2 Asynchronous Integration

In asynchronous integration, a context publishes a message or event without waiting for every consumer to finish processing it.

For example:

1. Order Management confirms an order.
2. It publishes an `OrderConfirmed` integration event.
3. Payment consumes the event and begins its workflow.
4. Shipping may independently consume the same event and prepare fulfillment.

The producer does not need to know how many consumers exist or how they implement their responsibilities.

This can reduce runtime coupling and support independent processing.

However, the system must handle eventual consistency and asynchronous failure.

For example, an order may already be confirmed while its payment is still being processed. The business must explicitly define whether that state is acceptable and what happens if payment authorization fails.

Use asynchronous integration when:

- Work can happen after the initiating operation.
- Consumers do not need to respond immediately.
- Independent processing and availability are valuable.
- The business can tolerate and manage eventual consistency.

#### 4.3 Choosing Between Synchronous and Asynchronous Communication

Neither approach is universally better.

| Concern | Synchronous | Asynchronous |
|---|---|---|
| Response | Immediate response | Processed later |
| Runtime dependency | Caller depends on receiver | Producer can continue without consumers responding |
| Failure handling | Timeouts, retries, fallback | Retries, duplicate handling, dead letters |
| Consistency | Can support immediate coordination | Often eventually consistent |
| Latency | Includes downstream response time | Usually shorter producer interaction |
| Complexity | Can create call chains and cascading failures | Requires message lifecycle and processing guarantees |

A system can use both approaches. For example, a user-facing API might synchronously validate a request and persist an order, then asynchronously publish an event to initiate downstream workflows.

The choice should follow the business interaction, not a general preference for APIs or messaging.

#### 4.4 Context Mapping and Integration Patterns

Context Mapping helps us understand relationships between Bounded Contexts. Those relationships can inform how integration is implemented.

Some common patterns include:

- **Customer-Supplier:** A downstream context depends on a supplier context, with the relationship and responsibilities understood by both sides.
- **Conformist:** A downstream context adopts the upstream model because it has little influence over it.
- **Anti-Corruption Layer:** A context translates an external model into its own language and protects its internal model from external assumptions.
- **Open Host Service:** A context provides a well-defined protocol or API for other contexts to use.
- **Published Language:** Contexts communicate through an agreed, documented integration language.

These are strategic relationship patterns, not messaging technologies. An Anti-Corruption Layer, for example, can translate data received through REST, a message broker, or a legacy integration.

The important point is that integration should preserve the meaning and autonomy of each model.

### 5. Choosing an Architectural Style

Architecture should be selected according to the actual needs and constraints of the system.

Consider the following dimensions.

**Domain complexity**

A simple domain with straightforward CRUD behavior may not benefit from elaborate architectural patterns.

A domain with complex rules, evolving policies, and significant business behavior may benefit from stronger domain isolation and explicit module boundaries.

**Scale and performance**

High traffic does not automatically require microservices.

A monolith can scale vertically and horizontally. Microservices become useful when independent scaling of capabilities, deployment isolation, or other requirements justify their additional cost.

**Team structure**

Independent teams may benefit from independently owned and deployed services.

However, splitting an application into services does not automatically create autonomous teams. Ownership, responsibilities, delivery practices, and communication agreements must support that autonomy.

**Consistency requirements**

Some business operations require strong consistency within a transaction.

If several capabilities must always change atomically, separating them into independent services can make the design significantly harder.

It may be better to keep that consistency boundary inside one module or service and use explicit workflows for operations that cross it.

**Operational maturity**

Microservices require reliable deployment automation, observability, service discovery, security, monitoring, and incident response.

A team that cannot operate these capabilities effectively may gain little from independently deployable services.

**Business constraints**

Regulatory requirements, auditability, availability, time-to-market, legacy systems, and infrastructure limitations can all influence the decision.

An architectural choice is only good when its benefits are meaningful in the environment where it must operate.

#### A practical comparison

| Dimension | Modular Monolith | Microservices |
|---|---|---|
| Deployment | One deployable unit | Independent deployable units |
| Code organization | Explicit internal modules | Separate service codebases or deployable components |
| Data ownership | Module ownership can be enforced within shared infrastructure | Each service owns its data and access |
| Communication | Usually in-process | Network APIs and/or messaging |
| Transactions | Easier to coordinate locally | Cross-service workflows require explicit coordination |
| Failure modes | Process-level failures can affect all modules | Network and service failures must be handled |
| Scaling | Usually scale the application as a whole | Scale individual services independently |
| Team ownership | Teams can own modules, but release lifecycle is shared | Teams can own services end to end |
| Operations | Comparatively simpler | More infrastructure and observability work |
| Evolution | Strong internal boundaries are essential | Stable contracts and compatibility are essential |

There is no universal winner. A modular monolith can provide strong modularity without distributed-system complexity. Microservices can provide independent delivery and scaling when those capabilities are genuinely needed.

A sensible strategy is to start with clear domain and module boundaries, then introduce independent deployment when a concrete requirement justifies it.

This does not mean every system must start as a monolith. It means the decision to distribute components should be deliberate rather than automatic.

## Practical Exercise

### Modular Monolith vs. Microservices: Same Domain, Different Architecture

Choose one Bounded Context from a business domain you understand. For example, use Order Management from an e-commerce system.

Design two implementations of that same context:

1. A modular monolith implementation.
2. An independently deployed microservice implementation.

Do not change the business model just to make one architecture look better. Keep the same responsibilities and business rules, then compare how each architecture affects their implementation and operation.

Your analysis must cover:

**1. Deployment**

- How is the application built and deployed?
- Can the context be released independently?
- What happens when a new version is rolled back?

**2. Data ownership**

- Who owns the data?
- Can another context access the data directly?
- How are transactions handled?
- What happens when information needs to be shared?

**3. Communication**

- How do other contexts interact with Order Management?
- Which interactions are synchronous?
- Which interactions are asynchronous?
- What contracts are exposed?

**4. Failure modes**

- What happens if a dependency becomes unavailable?
- What happens if a message is delivered twice?
- How does the system recover from partial failures?
- What is the impact of a process failure?

**5. Team responsibilities**

- Who owns the module or service?
- Who is responsible for deployment and monitoring?
- Can teams work and release independently?
- What coordination is still required?

**6. Operational costs**

- What infrastructure is required?
- How complex are deployment, monitoring, logging, and tracing?
- What additional skills and operational responsibilities are needed?
- What is the cost of running and maintaining each option?

Finally, write a short architectural decision record (ADR) explaining which approach you would choose for the given business and organizational context.

Your decision must include the assumptions behind it, the benefits you expect, the costs you accept, and the conditions that might make you reconsider it.

## Completion Criteria

You should be able to:

- Distinguish semantic, code, and deployment boundaries.
- Explain how strategic DDD decisions influence architectural design without prescribing a specific architecture.
- Compare Layered, Hexagonal, Onion, Clean, Modular Monolith, and Microservices approaches.
- Explain the difference between Domain Events and Integration Events.
- Distinguish Event-Driven Architecture from Event Sourcing.
- Explain the consequences of synchronous and asynchronous integration.
- Describe data ownership and consistency challenges in distributed systems.
- Identify when CQRS and Event Sourcing are justified and when they introduce unnecessary complexity.
- Compare a modular monolith and microservices using concrete technical and organizational constraints.
- Defend an architectural decision with explicit trade-offs rather than architectural fashion.

**The real measure of an architecture is not how many patterns it uses, but how well its boundaries preserve the meaning of the domain while allowing the system and the organization to evolve.**