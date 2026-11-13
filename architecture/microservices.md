# Microservices

Microservices are an architectural style in which a system is composed of relatively small, independently deployable services. Each service owns a well-defined set of capabilities and communicates with other services through explicit boundaries, typically using network-based APIs or asynchronous messaging.

The important word is not *micro*. It is *independently*.

A service boundary should allow a team to develop, test, deploy, operate, and evolve a capability without requiring every other part of the system to change in lockstep. Achieving that independence requires more than splitting an application into multiple projects or deploying separate APIs. It requires clear ownership of business capabilities, data, contracts, and operational responsibilities.

## Microservices and Domain-Driven Design

Domain-Driven Design and microservices address related but different concerns.

DDD helps discover and model the business domain. It provides concepts such as Subdomains, Bounded Contexts, Ubiquitous Language, and Context Mapping to understand where business models and responsibilities belong.

Microservices provide one possible way to distribute the resulting software across independently deployable processes.

A Bounded Context can be a useful starting point for identifying a microservice boundary because it defines an area in which a particular model and language are consistent. However, the two concepts are not interchangeable:

- A **Bounded Context** is a semantic boundary around a model.
- A **microservice** is a deployable and operational boundary around software capabilities.
- A **team boundary** defines responsibility and ownership.
- A **database boundary** defines data ownership and persistence isolation.

These boundaries may align, but they do not have to.

For example, an e-commerce platform might identify separate Bounded Contexts for Ordering, Inventory, Payments, and Shipping. Initially, these contexts could be implemented as modules inside one application. As the system evolves, the Shipping context might become an independently deployed service because it has different operational requirements and a dedicated team. Ordering and Inventory might remain modules in the same deployment because their current collaboration and release requirements do not justify distribution.

The important point is that the model should inform the architecture, not be forced into a predetermined number of services.

## When Microservices Make Sense

Microservices become valuable when independent operation provides enough benefit to justify the additional complexity.

Consider the following situations:

**Independent deployment**

A team needs to release changes to a particular capability without coordinating a deployment of the entire system.

For example, a Payment service might need frequent changes to support new payment providers, while the rest of the platform follows a slower release cycle.

**Different scaling requirements**

Some capabilities experience substantially different workloads from others.

For example, a document-processing service might receive large bursts of uploads, while the administrative application has relatively low and predictable traffic. Scaling the processing capability independently may be economically and operationally useful.

**Independent ownership**

Different teams have clear responsibility for different business capabilities and can make decisions within their boundaries.

For example, a Shipping team owns shipment planning, dispatch, and delivery status, while a Payment team owns payment authorization, capture, and refunds.

This works best when the teams can understand and evolve their models without constantly negotiating changes to shared implementation details.

**Different operational requirements**

Some capabilities may require different availability targets, deployment schedules, security controls, or infrastructure.

For example, a notification service may tolerate temporary delivery delays and use asynchronous processing, while an authorization capability may require immediate responses.

**Technology diversity**

A capability may have a genuine technical requirement that makes a different technology appropriate.

However, using a different programming language or database is not, by itself, a sufficient reason to create a microservice. The operational cost of maintaining multiple technology stacks must be considered.

## The Costs of Distribution

A monolithic application can often make a local method call and participate in one database transaction. A distributed system cannot assume either of those properties.

Moving a boundary across a process or network introduces new failure modes and responsibilities.

### Network failures and latency

A network request can time out, fail, arrive late, or succeed on the remote side while the caller fails to receive the response.

Consider an Ordering service that requests a reservation from Inventory. The request times out. Did Inventory reserve the stock?

The caller cannot safely assume that a timeout means the operation failed. It may need to query the operation's status, retry with an idempotency key, or use another explicit recovery mechanism.

### Data ownership and consistency

Each service should own its data and expose business capabilities through contracts rather than allowing other services to manipulate its database directly.

Suppose Ordering needs to know whether a product is available. It should not query Inventory's tables directly. It should interact through an API or an appropriate integration event.

This introduces consistency considerations. A product's availability may change between the time it is displayed and the time an order is submitted. The system must define which operation makes the authoritative decision and how it handles conflicts.

Distributed transactions are not automatically available just because services share a business process. Many systems use local transactions, asynchronous communication, and explicit business workflows to manage this reality.

### Eventual consistency

When services maintain separate data, changes may not become visible everywhere immediately.

For example:

1. Ordering accepts an order.
2. Ordering publishes an `OrderPlaced` integration event.
3. Inventory receives the event and reserves the required stock.
4. Shipping receives a later event and prepares the shipment.

Between these steps, the system may temporarily contain different views of the business process.

This is not necessarily incorrect. The important requirement is to define which inconsistencies are acceptable, how long they may last, and what happens when a step fails.

### Contract evolution

Services communicate through contracts that evolve over time.

Changing an API response or event payload can break consumers that depend on the previous version. Teams therefore need deliberate compatibility strategies, such as additive changes, tolerant readers, explicit deprecation, and versioning when necessary.

An integration contract should communicate business meaning, not merely expose the internal structure of the producer's domain model.

### Observability and operations

A request may travel through several services before a business operation completes.

Without distributed tracing, correlation identifiers, structured logs, metrics, and clear operational ownership, diagnosing a failure becomes difficult.

Teams also need deployment automation, health checks, monitoring, alerting, secret management, network security, and procedures for handling unavailable dependencies.

These are not optional extras. They are part of the cost of operating independently deployed services.

## Microservices and Context Mapping

Context Mapping remains important when Bounded Contexts are implemented as microservices.

A network connection does not automatically define a healthy model boundary. Services still need to understand their relationships and integration contracts.

For example, consider Ordering and Inventory:

- Ordering is the upstream source of order-related decisions.
- Inventory owns stock availability and reservation rules.
- Ordering requests a reservation through an explicit contract.
- Inventory decides whether the reservation can be made.
- The result is communicated through a response or an integration event, depending on the workflow.

The relationship should be designed deliberately. It may involve Customer-Supplier, Published Language, an Anti-Corruption Layer, or a combination of patterns.

An Anti-Corruption Layer can be particularly useful when integrating with a legacy service whose model does not match the language and concepts of the new context.

Do not confuse the communication mechanism with the relationship pattern. REST does not automatically mean Customer-Supplier, and messaging does not automatically mean Published Language.

## Microservices Versus a Modular Monolith

A modular monolith is an application deployed as one unit but internally divided into well-defined modules.

Each module can own its domain model, application logic, persistence responsibilities, and public interfaces. Other modules interact through explicit contracts rather than accessing internal implementation details.

This provides many of the modeling and organizational benefits of clear service boundaries without introducing network communication between every module.

Consider an early-stage logistics platform with separate Cargo, Fleet, Quota, and Shipping capabilities.

A modular monolith could enforce these boundaries within one deployment:

- Cargo owns cargo-related concepts and rules.
- Fleet owns vehicle and driver management.
- Quota owns fuel allocation rules.
- Shipping coordinates shipment-related decisions.

The modules can be tested independently and have clear ownership of their models. If Shipping later needs independent scaling or deployment, it can be extracted behind a well-defined interface.

A modular monolith is not necessarily a temporary or inferior architecture. It may remain the most appropriate solution for the lifetime of a system.

| Dimension | Modular Monolith | Microservices |
|---|---|---|
| Deployment | Usually one deployment unit | Independent deployment units |
| Communication | In-process calls or internal messaging | Network APIs and messaging |
| Data ownership | Can be separated by module | Usually isolated per service |
| Transactions | Local database transactions are easier | Cross-service consistency requires explicit design |
| Failure modes | Primarily process and infrastructure failures | Includes network and partial failures |
| Scaling | Usually scales as a deployment unit, though internal workloads can use separate workers | Services can scale independently |
| Operations | Generally simpler | Requires distributed observability and operations |
| Team autonomy | Depends on module and ownership discipline | Can support stronger independent ownership |

A modular monolith can also contain separate background workers or processes without requiring every module to become a microservice.

## Common Microservices Mistakes

**One microservice per table**

Database tables are not business capabilities. Splitting services according to tables often produces excessive communication and tightly coupled workflows.

**One microservice per class or technical layer**

A service should own meaningful business responsibility. Creating separate services for repositories, validation, or generic utilities usually distributes implementation details rather than business capabilities.

**Treating every Bounded Context as a microservice**

A Bounded Context identifies a model boundary, not an automatic deployment boundary. Some contexts are too closely related operationally to justify independent deployment, while others may eventually need multiple deployable components.

**Sharing databases across services**

When multiple services directly modify the same database, data ownership becomes ambiguous. Changes to one service can unexpectedly affect another, and independent evolution becomes difficult.

**Distributed monoliths**

A system may consist of many deployed services but still require coordinated releases, synchronous calls across long dependency chains, and shared operational assumptions.

This is distribution without meaningful independence. It combines the complexity of microservices with many of the limitations of a monolith.

**Ignoring business workflows**

A business process that previously ran inside one transaction may now span multiple services. Simply replacing method calls with HTTP requests does not preserve the original consistency guarantees.

The workflow needs explicit decisions about retries, timeouts, idempotency, compensation, and failure recovery.

**Premature extraction**

Extracting services before understanding the domain and its boundaries can freeze an immature model into distributed contracts. Changing a poorly chosen boundary later is significantly more expensive than changing an internal module.

## A Practical Approach

Before creating a microservice, answer these questions:

1. **Business responsibility:** What business capability does this service own?
2. **Language and model:** Does it have a coherent model and a clear Ubiquitous Language?
3. **Data ownership:** Which data and business decisions belong exclusively to it?
4. **Contracts:** What does it expose, and what does it depend on?
5. **Consistency:** Which business rules must be enforced immediately, and which can tolerate eventual consistency?
6. **Failure handling:** What happens when this service or one of its dependencies is unavailable?
7. **Operational independence:** Is independent deployment, scaling, or operation genuinely valuable?
8. **Team ownership:** Is there a team able to take responsibility for its development and operation?
9. **Extraction cost:** What additional infrastructure, monitoring, security, and maintenance responsibilities will distribution introduce?
10. **Alternatives:** Could a well-structured module or separate worker solve the problem more simply?

If the answers do not establish a meaningful reason for independent deployment, keep the boundary inside a modular monolith.

## Exercise

Take an existing application and identify three candidate business capabilities that might become microservices.

For each candidate:

- Identify its Bounded Context and Ubiquitous Language.
- Describe the business responsibilities and decisions it owns.
- Identify the data it should own.
- Draw its relationships with other contexts.
- Define the APIs or integration events required for collaboration.
- Describe what happens when a dependency becomes unavailable.
- Identify consistency requirements and possible recovery mechanisms.
- Compare independent deployment with keeping the capability as a module.

Finally, choose one candidate and explain why extracting it would create more value than operational complexity.

## Completion Criteria

You should be able to:

- Explain the difference between a Bounded Context and a microservice.
- Recognize that DDD does not prescribe one microservice per Bounded Context.
- Identify business capabilities that may benefit from independent deployment.
- Explain the trade-offs between a modular monolith and microservices.
- Design explicit service contracts and data ownership boundaries.
- Reason about network failures, eventual consistency, retries, idempotency, and contract evolution.
- Recognize a distributed monolith and explain why it is problematic.
- Justify a microservice boundary based on business and operational needs rather than technical fashion.

**The goal is not to maximize the number of services. It is to establish boundaries that allow the business model, teams, and software to evolve with an appropriate degree of independence.**