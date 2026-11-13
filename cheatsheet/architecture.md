# Architecture Choices

Domain-Driven Design does not prescribe a specific software architecture. The architecture should support the domain model, protect important boundaries, and make the system easier to evolve.

The following styles address different architectural concerns. They are not mutually exclusive, and a system can combine several of them.

| Style | Useful when | Watch for |
|---|---|---|
| **Layered Architecture** | A straightforward separation between presentation, application, domain, and infrastructure responsibilities is sufficient. | Pass-through layers, business logic leaking into infrastructure, and persistence-driven domain models. |
| **Hexagonal Architecture** | The domain should remain independent of external systems, frameworks, databases, and delivery mechanisms. | Creating ports and adapters without meaningful boundaries or actual isolation needs. |
| **Clean Architecture** | Dependency direction must protect business policies from infrastructure and implementation details. | Mistaking folder structure, interfaces, or dependency injection for a well-designed domain model. |
| **Modular Monolith** | Strong internal boundaries and independent module evolution are needed without distributed deployment. | Hidden cross-module dependencies, shared mutable data, and modules that are separate only in name. |
| **Microservices** | Independent deployment, ownership, scaling, or fault isolation justifies distributing capabilities across services. | Network failures, distributed transactions, observability, data ownership, deployment complexity, and operational overhead. |
| **Event-Driven Architecture** | Asynchronous reactions, loose coupling, and communication through meaningful events are valuable. | Duplicate delivery, ordering, retries, eventual consistency, message contracts, and schema evolution. |

## How to Choose

Before selecting an architecture, understand the problem it is expected to solve.

**Layered Architecture**

Choose a layered structure when the application needs clear separation of responsibilities but does not require complex isolation between external systems and the domain.

A typical arrangement separates:

- Presentation: HTTP APIs, user interfaces, and other entry points.
- Application: Use-case orchestration and coordination.
- Domain: Business concepts, rules, and behavior.
- Infrastructure: Persistence, messaging, external services, and technical implementations.

The important consideration is dependency direction and responsibility, not the number of projects or folders.

**Hexagonal Architecture**

Choose Hexagonal Architecture when you want the domain and application behavior to remain independent of the technologies through which the system interacts with the outside world.

Ports represent capabilities the application needs or exposes. Adapters connect those ports to concrete technologies.

For example, an application might depend on a `PaymentGateway` port, while separate adapters communicate with different payment providers.

**Clean Architecture**

Choose Clean Architecture when protecting business policies from infrastructure and framework dependencies is a significant concern.

Its central principle is that dependencies should point inward, toward more stable business rules. Infrastructure implements the abstractions required by the inner layers rather than dictating their design.

It is closely related to Hexagonal Architecture and can be combined with it. Neither architecture automatically produces a rich domain model.

**Modular Monolith**

Choose a Modular Monolith when the system has multiple business capabilities that need clear boundaries but do not yet require independent deployment.

A practical implementation may include:

- Explicit modules aligned with business responsibilities.
- Module-owned domain models and application use cases.
- Controlled communication through public interfaces or messages.
- Clear ownership of data and persistence.
- Tests that prevent unwanted dependencies between modules.

A modular monolith can be an excellent long-term architecture, not merely a temporary step toward microservices.

**Microservices**

Choose Microservices when independent deployment, team ownership, scaling, or fault isolation provides enough value to justify distributed-system complexity.

A service should own a meaningful business capability and its data. Its boundaries should be based on business responsibilities and operational needs, not simply on splitting existing classes or database tables.

Before introducing a microservice, consider whether the team can operate it independently, monitor it, handle failures, manage contracts, and evolve its data without depending on coordinated deployments.

**Event-Driven Architecture**

Choose an event-driven approach when parts of the system need to react to business occurrences without requiring synchronous coordination.

For example, when an order is confirmed, separate capabilities may update inventory, initiate payment processing, and notify the customer.

Events should represent meaningful facts, and consumers should be designed to handle delivery failures, retries, and duplicates.

Event-driven communication does not automatically require Event Sourcing or CQRS.

## Important Boundary Distinctions

A Bounded Context, a code module, and a deployable service are different kinds of boundaries.

- **Bounded Context:** Defines where a particular domain model and its language are consistent.
- **Code Module:** Defines an implementation and dependency boundary within the codebase.
- **Deployable Service:** Defines an independently deployed and operated software unit.

They may align, but they do not have to.

For example, several Bounded Contexts may initially live inside one modular monolith. A Bounded Context may also contain multiple internal modules. Conversely, a deployable service may contain more than one closely related model when there is a justified reason.

Do not turn every Bounded Context into a microservice by default.

## Questions to Ask Before Choosing

- What business or engineering problem are we trying to solve?
- Which boundaries need to be explicit?
- Which components must evolve independently?
- Does independent deployment provide real value?
- What consistency guarantees do business operations require?
- How will modules or services communicate?
- Who owns each capability and its data?
- Can the team operate the resulting architecture?
- What complexity does this choice introduce?
- Is there a simpler architecture that satisfies the same requirements?

**Remember:** Architecture is a set of decisions and trade-offs, not a collection of fashionable patterns. Choose the smallest set of architectural mechanisms that adequately supports the domain, its boundaries, and its expected evolution.