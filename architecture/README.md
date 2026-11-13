# Architecture and DDD

Domain-Driven Design is concerned with understanding and modeling the business domain, defining meaningful boundaries, and designing software that reflects that understanding.

Architecture provides the structures through which those models and boundaries are implemented, connected, deployed, and operated.

The relationship is important, but DDD does not prescribe a particular architectural style. A system designed with DDD can use Layered Architecture, Hexagonal Architecture, Clean Architecture, a Modular Monolith, Microservices, or a combination of these approaches.

The choice should follow the needs of the domain and the system, not the popularity of an architectural pattern.

A well-designed domain model can exist inside a monolith. A microservices system can have a poorly designed domain model. Choosing an architectural style does not automatically produce good domain design.

## Architecture Should Support the Domain Model

Architecture decisions should help preserve the boundaries, responsibilities, and language discovered through domain exploration and strategic design.

For example, consider a logistics platform with separate Bounded Contexts for Cargo, Fleet, Quota, and Shipping.

DDD helps establish:

- What responsibilities belong to each context.
- Which business rules and decisions each context owns.
- Where the meaning of concepts changes.
- How the contexts collaborate.
- Which models should remain independent.

Architecture then addresses questions such as:

- How should each context be organized internally?
- How should dependencies between components be controlled?
- How should one context communicate with another?
- How should data access and persistence be structured?
- Should the contexts run in one application or separate deployments?
- How can the system evolve without creating unnecessary coupling?

These are related questions, but they are not interchangeable.

For example, identifying Shipping as a Bounded Context does not automatically mean it must become a microservice. It could initially be implemented as a module within a Modular Monolith, using Hexagonal Architecture internally and a layered organization for its implementation.

The architecture should make the domain boundaries easier to understand and preserve, rather than obscure them behind technical structures.

## Architectural Styles and Approaches

The following approaches address different architectural concerns. They are not all competing alternatives, and they can often be combined.

### Layered Architecture

Organizes an application into horizontal layers, typically Presentation, Application, Domain, and Infrastructure.

It provides a familiar separation of responsibilities and can work well when dependencies are controlled and each layer has a meaningful purpose.

The main risk is allowing layers to become pass-through structures or letting persistence concerns dictate the domain model.

[Explore Layered Architecture](layered-architecture.md)

### Hexagonal Architecture

Also known as Ports and Adapters, it separates application logic from external technologies through explicit interaction boundaries.

The application defines the ports through which it communicates with the outside world. Adapters connect those ports to technologies such as databases, HTTP APIs, message brokers, and user interfaces.

It is useful when protecting the application from infrastructure dependencies and making external interactions replaceable and testable.

[Explore Hexagonal Architecture](hexagonal-architecture.md)

### Clean Architecture

Organizes software around the principle that business policies should not depend on external implementation details.

Its Dependency Rule directs dependencies inward, keeping domain and application policies independent of frameworks, databases, and delivery mechanisms.

It shares ideas with Hexagonal Architecture, but places particular emphasis on dependency direction and the independence of business rules.

[Explore Clean Architecture](clean-architecture.md)

### Modular Monolith

Organizes one deployable application into modules with explicit responsibilities, interfaces, and dependency rules.

It is useful when a system needs strong internal boundaries without the operational cost of independently deployed services.

A Modular Monolith can represent multiple Bounded Contexts while keeping communication in-process and deployment relatively simple.

[Explore Modular Monolith](modular-monolith.md)

### Microservices

Organizes a system into independently deployable services that communicate across process boundaries.

Microservices can support independent deployment, scaling, and team ownership, but introduce additional complexity around network failures, data ownership, consistency, observability, security, and operations.

They may align with Bounded Contexts, but DDD does not require one microservice per context.

[Explore Microservices](microservices.md)

### Event-Driven Architecture

Uses events to communicate that something has happened and allow other components or services to react.

It can reduce direct and temporal coupling, especially across Bounded Contexts, but introduces challenges around delivery guarantees, ordering, idempotency, contract evolution, and eventual consistency.

Event-Driven Architecture can be used within a Modular Monolith or across Microservices. It is not inherently tied to either deployment style.

[Explore Event-Driven Architecture](event-driven-architecture.md)

### CQRS and Event Sourcing

CQRS separates models used to handle commands from models used to answer queries. Event Sourcing stores changes as a sequence of events from which the current state can be reconstructed.

These are complementary but independent patterns. CQRS does not require Event Sourcing, and Event Sourcing does not automatically require a distributed architecture.

They are more specialized modeling and persistence approaches than general application architectural styles, but they can have significant architectural consequences.

[Explore CQRS and Event Sourcing](../roadmap/08-advanced-topics.md)

## A Useful Distinction: Different Kinds of Boundaries

One of the most important architectural distinctions in DDD is that several boundaries may exist in the same system.

| Boundary | What it defines | Example |
|---|---|---|
| Bounded Context | Where a particular domain model and language are consistent | Shipping |
| Module | How code and implementation responsibilities are organized | `Shipping` module |
| Deployable Service | What can be deployed and operated independently | `Shipping.Api` |
| Team Boundary | Who owns and evolves a capability | Shipping team |
| Data Ownership Boundary | Which part of the system controls particular data and its business rules | Shipping owns shipment data |

These boundaries can align, but they should not be assumed to map one-to-one.

For example, a Shipping Bounded Context might initially be implemented as a module inside a Modular Monolith. Later, it might become an independently deployed service without changing its fundamental domain model.

Likewise, a single Bounded Context may contain multiple modules or deployable components when its responsibilities and operational requirements justify that structure.

The important thing is to preserve semantic ownership and control dependencies, regardless of how the software is packaged.

## How to Choose

There is no universally correct architectural style. Evaluate the actual constraints and needs of the system.

Consider:

**Domain complexity:** How much business behavior needs to be modeled and protected? Are the domain boundaries understood well enough to guide the architecture?

**Coupling:** Which parts of the system need to change together? Which dependencies prevent independent evolution?

**Deployment requirements:** Does the entire application need to be deployed together, or are there genuine reasons for independent deployment?

**Team ownership:** Can teams own capabilities independently, or would distributing the software create more coordination than it removes?

**Operational capacity:** Can the organization support distributed tracing, monitoring, deployment automation, security, incident response, and data consistency?

**Rate of change:** Which parts of the system evolve frequently, and would architectural separation make those changes easier and safer?

**Consistency requirements:** Which business rules require immediate consistency? Which workflows can tolerate asynchronous processing and eventual consistency?

A practical starting point is often a Modular Monolith with clear Bounded Contexts and appropriate internal architecture. Introduce additional distribution only when there is a concrete reason to do so.

This is a starting point, not a universal prescription. Some systems have legitimate requirements for distributed deployment from the beginning.

## Common Misconceptions

- **DDD requires Microservices:** It does not. DDD is concerned with domain modeling and boundaries, not a mandatory deployment architecture.
- **A Bounded Context is a Microservice:** A Bounded Context defines semantic consistency, not deployment.
- **Hexagonal Architecture is the same as Clean Architecture:** They overlap, but emphasize different aspects of dependency and interaction boundaries.
- **Event-Driven Architecture requires Microservices:** Events can be used within a single process or across independently deployed systems.
- **A Modular Monolith is just a monolith with folders:** Meaningful module boundaries require explicit interfaces, controlled dependencies, and data ownership.
- **More architectural patterns mean better architecture:** Additional abstractions and infrastructure can make a system harder to understand without improving its ability to evolve.

## Exercise

Take a system you know and describe its architecture at three levels:

1. **Domain boundaries:** Identify its Bounded Contexts and the models and responsibilities each one owns.
2. **Code boundaries:** Identify its modules, internal layers, dependencies, and public interfaces.
3. **Operational boundaries:** Identify its deployable units, data stores, teams, and communication mechanisms.

Compare the three views.

Where do the boundaries align? Where do they differ? Are those differences intentional, or do they create unnecessary coupling?

Finally, identify one architectural change that would make the domain model easier to understand, protect, or evolve. Explain why the change is justified and what complexity it introduces.

## Completion Criteria

You should be able to:

- Explain the relationship between DDD and software architecture.
- Understand why DDD does not prescribe a specific architectural style.
- Distinguish domain, code, deployment, team, and data ownership boundaries.
- Explain the purpose and trade-offs of the architectural approaches introduced here.
- Recognize that architectural styles can be combined rather than treated as mutually exclusive alternatives.
- Choose architectural structures based on domain needs, coupling, deployment, team ownership, and operational capacity.
- Explain why a well-structured Modular Monolith can be preferable to premature distribution.

**Architecture should make the domain model easier to understand, protect, and evolve. The purpose is not to make the system look architecturally sophisticated; it is to give the software a structure that supports the business and the people who change it.**