# DDD Cheatsheet

A practical collection of short references for applying Domain-Driven Design during domain exploration, modeling sessions, architecture discussions, and code reviews.

Each cheatsheet focuses on a specific set of concepts and provides concise definitions, design principles, decision-making questions, and common pitfalls. They are designed to help you make better decisions in practice, not to replace deeper explanations, domain conversations, or collaborative modeling.

Use the [Learning Roadmap](../roadmap/README.md) to understand the recommended learning journey. Return to these cheatsheets when you need a quick reminder, want to compare related concepts, or need to challenge a design decision.

## Core Concepts and Distinctions

[Core Distinctions](core-distinctions.md)

DDD concepts are related, but they serve different purposes. Confusing them can lead to incorrect models, unnecessary abstractions, and architectural decisions based on terminology rather than business needs.

This page compares important distinctions, including:

- **Domain vs. Subdomain:** The overall business problem space versus a specific area within it.
- **Subdomain vs. Bounded Context:** A decomposition of the business problem space versus a boundary within the solution model.
- **Ubiquitous Language vs. Glossary:** A continuously cultivated shared language versus a document that records terminology.
- **Entity vs. Value Object:** Identity and lifecycle versus value-based equality.
- **Aggregate vs. Entity:** A consistency boundary versus an individual domain object.
- **Domain Service vs. Application Service:** Domain behavior that does not naturally belong to an Entity or Value Object versus orchestration of a use case.
- **Command vs. Domain Event:** An intention to perform an action versus a fact that has already happened.
- **Domain Event vs. Integration Event:** A fact meaningful within a domain model versus a contract published across a system boundary.
- **Domain Model vs. Data Model:** A model expressing business behavior and rules versus a representation optimized for storing or transferring data.
- **CQRS vs. Event Sourcing:** Separating read and write responsibilities versus persisting state as a sequence of events.

**Use this page when:** A design discussion becomes confusing because people are using similar terms interchangeably, or when you need to determine which DDD concept actually applies to a problem.

## Strategic Design

[Strategic Design](strategic-design.md)

Strategic Design helps you understand the business domain, identify meaningful boundaries, and decide where different models should exist. It focuses on the larger picture before moving into the implementation details of individual domain objects.

This page covers:

- **Domain and Subdomain Discovery:** Identifying the business areas and capabilities that make up the problem space.
- **Core, Supporting, and Generic Subdomains:** Understanding where competitive advantage and business differentiation exist, and where standard capabilities may be sufficient.
- **Ubiquitous Language:** Establishing and continuously refining the language shared by domain experts and the software team.
- **Bounded Contexts:** Defining explicit boundaries within which a domain model and its language remain consistent.
- **Context Boundaries:** Identifying where concepts, responsibilities, and business rules belong.
- **Core Domain Distillation:** Focusing modeling and engineering effort on the parts of the business that matter most strategically.
- **Team and Organizational Alignment:** Understanding how team ownership, communication, and organizational structures affect model boundaries.

The cheatsheet helps answer practical questions:

- Which parts of the business are strategically important?
- Where do business concepts have different meanings?
- Should two capabilities share one model or have separate models?
- Which responsibilities belong to each Bounded Context?
- Where should the team invest its modeling effort?
- Are proposed boundaries based on business realities or merely technical preferences?

**Use this page when:** Starting domain exploration, decomposing a large business domain, reviewing Bounded Context boundaries, or deciding where business capabilities should be owned.

## Tactical Design

[Tactical Design](tactical-design.md)

Tactical Design focuses on expressing a domain model inside a Bounded Context. It provides building blocks for representing business concepts, behavior, identity, consistency, and domain knowledge in code.

This page provides practical reminders about:

- **Entities:** Objects with identity and a lifecycle, whose continuity matters even when their attributes change.
- **Value Objects:** Concepts defined by their attributes rather than identity, typically modeled as immutable values.
- **Aggregates:** Consistency boundaries that protect business invariants through an Aggregate Root.
- **Aggregate Roots:** The single entry point for accessing and modifying an Aggregate.
- **Domain Services:** Domain operations that do not naturally belong to a particular Entity or Value Object.
- **Application Services:** Components that coordinate use cases and invoke domain behavior.
- **Repositories:** Abstractions for retrieving and persisting Aggregates.
- **Factories:** Mechanisms for creating complex domain objects when construction involves meaningful rules.
- **Domain Events:** Immutable representations of significant facts that have occurred in the domain.

It also helps you decide where business logic belongs:

- Does this rule belong to an Entity or Value Object?
- Does the operation require a Domain Service?
- Is the Application Service coordinating a use case or making business decisions that belong in the domain model?
- Does the Aggregate protect the invariants it is responsible for?
- Is a Repository hiding persistence details or exposing database-oriented operations?
- Does an abstraction express a real domain concept, or does it simply add another layer?

**Use this page when:** Designing domain objects, reviewing business logic placement, implementing a use case, or deciding how to represent a business rule in code.

## Context Mapping

[Context Mapping](context-mapping.md)

A Context Map describes the relationships between Bounded Contexts. It helps teams understand dependencies, integration responsibilities, model translations, and the consequences of interactions between independently evolving models.

This page provides a practical reference for:

- **Upstream and Downstream:** Understanding which context influences another and how that dependency affects collaboration.
- **Partnership:** Managing contexts whose teams need to coordinate closely to achieve compatible results.
- **Shared Kernel:** Sharing a carefully controlled subset of a model between contexts.
- **Customer-Supplier:** Making the relationship between an upstream provider and a downstream consumer explicit.
- **Conformist:** Adopting an upstream model when influencing it is not practical.
- **Anti-Corruption Layer:** Translating an external model into the local model to protect its integrity.
- **Open Host Service:** Providing a well-defined interface for other contexts to consume.
- **Published Language:** Defining a shared language or contract for integration.
- **Separate Ways:** Avoiding integration when the cost of collaboration outweighs its benefits.

It helps answer:

- Who owns each model and its evolution?
- Which context depends on which other context?
- Who controls the integration contract?
- Where is translation required?
- What happens when an upstream model changes?
- Is sharing a model intentional and controlled, or does it create accidental coupling?
- Would integration provide enough value to justify its cost?

An important distinction: Context Mapping patterns describe relationships between domain models and the teams that own them. Technical integration mechanisms describe how systems communicate or exchange data. REST, messaging, RPC, and shared databases do not, by themselves, define a Context Mapping relationship.

**Use this page when:** Integrating legacy systems, defining service boundaries, coordinating teams, designing contracts, or reviewing dependencies between Bounded Contexts.

## Aggregates

[Aggregates](aggregates.md)

Aggregates define consistency boundaries within a domain model. Their purpose is to protect business invariants and ensure that important rules remain valid when business operations modify domain state.

This page provides practical design rules and questions for:

- **Aggregate Root:** Identifying the single Entity through which an Aggregate is accessed and modified.
- **Aggregate Boundaries:** Deciding which objects must participate in the same consistency boundary.
- **Invariants:** Identifying business rules that must hold when an operation completes.
- **Business Operations:** Designing meaningful operations that protect those invariants.
- **References Between Aggregates:** Referencing other Aggregates by identity rather than building large object graphs.
- **Domain Events:** Representing meaningful occurrences that other parts of the domain may need to react to.
- **Transactions and Consistency:** Understanding which changes need immediate consistency and which can be coordinated eventually.
- **Concurrency and Contention:** Evaluating how simultaneous operations affect an Aggregate.
- **Testing:** Verifying invariants through business scenarios rather than testing only individual properties or setters.

A practical Aggregate design should answer:

1. What business responsibility does this Aggregate own?
2. Which invariants must it protect?
3. Which operations can change its state?
4. What must be consistent when each operation completes?
5. Which objects genuinely need to belong inside its boundary?
6. What happens when two users modify it concurrently?
7. Can the Aggregate remain smaller without weakening its invariants?

**Use this page when:** Designing a new Aggregate, reviewing an oversized domain model, investigating concurrency problems, or deciding whether two objects need to change atomically.

## Events, CQRS, and Event Sourcing

[Events, CQRS, and Event Sourcing](events-cqrs-event-sourcing.md)

These concepts are often discussed together, but they solve different problems. This page explains their differences and provides practical guidance for deciding when each is appropriate.

It covers three main areas:

**Domain Events and Integration Events**

- Represent meaningful business facts using clear, past-tense names.
- Distinguish events raised inside a domain model from messages published across Bounded Context boundaries.
- Define event ownership, payloads, identity, and versioning.
- Handle event delivery, retries, duplicate messages, and idempotent consumers.
- Consider mechanisms such as the Transactional Outbox when database changes and message publication must be coordinated reliably.

**Command Query Responsibility Segregation (CQRS)**

- Separate operations that change business state from operations that retrieve information.
- Design command handling around business intentions and domain behavior.
- Build query models that serve actual information needs.
- Decide whether read and write models need separate representations or storage.
- Understand the trade-offs of eventual consistency and projection maintenance.

**Event Sourcing**

- Persist state changes as an ordered sequence of events rather than storing only the latest state.
- Reconstruct current state by replaying events.
- Understand event stores, snapshots, concurrency control, and event versioning.
- Design projections and read models for efficient queries.
- Consider historical reconstruction, auditability, and temporal questions.
- Evaluate the operational complexity of event evolution, replay, debugging, and migration.

The most important decision questions are:

- Do consumers need to react to meaningful business events?
- Is separating reads from writes useful for this workload?
- Do different query requirements justify dedicated read models?
- Is retaining the complete history of state changes a genuine business requirement?
- Can the team support event versioning, replay, projections, and operational recovery?

**Use this page when:** Designing asynchronous workflows, integrating Bounded Contexts, optimizing read-heavy workloads, or evaluating whether Event Sourcing is justified.

Remember: using Domain Events does not require CQRS, and CQRS does not require Event Sourcing. These are separate design decisions.

## Architecture Choices

[Architecture Choices](architecture.md)

DDD does not prescribe a single software architecture. Architecture should support the domain model, preserve meaningful boundaries, and make the system practical to develop, test, deploy, and evolve.

This page provides a decision-oriented reference for architectural approaches and their trade-offs.

It covers:

- **Layered Architecture:** Separating domain behavior, application coordination, and infrastructure concerns.
- **Hexagonal Architecture (Ports and Adapters):** Keeping domain logic independent of external technologies through explicit boundaries.
- **Clean Architecture:** Organizing dependencies so business rules do not depend on implementation details.
- **Modular Monolith:** Keeping business modules within one deployable application while preserving internal boundaries.
- **Microservices:** Deploying independently bounded capabilities when operational and organizational independence justify the additional complexity.
- **Service Boundaries:** Aligning deployment and communication boundaries with business responsibilities where appropriate.
- **Persistence Strategies:** Choosing storage and persistence models without allowing database structures to dictate the domain model.
- **Inter-Context Communication:** Selecting synchronous APIs, asynchronous messaging, or other integration mechanisms based on consistency and business requirements.
- **Testing Strategies:** Testing domain behavior, application workflows, integration contracts, and architectural boundaries.

It helps answer:

- Does the domain model depend on infrastructure details?
- Are business responsibilities separated from use-case orchestration?
- Can modules evolve independently even if they share one deployment?
- Does a Bounded Context actually need to become a separate service?
- What consistency guarantees does communication between contexts require?
- Are deployment, monitoring, failure handling, and operational ownership justified by the proposed architecture?
- Is the architecture solving a real business or engineering problem, or introducing complexity prematurely?

**Use this page when:** Choosing between a modular monolith and microservices, defining application layers, designing integration boundaries, or reviewing whether the architecture supports the domain model.

---

## How to Use the Cheatsheets

These pages are intended to be consulted when a concrete design question arises.

| Situation | Start with |
|---|---|
| The team disagrees about the meaning of a business term. | Core Distinctions |
| You are exploring a complex business domain. | Strategic Design |
| You are deciding where a business rule belongs in code. | Tactical Design |
| Two systems have different models and need to communicate. | Context Mapping |
| You are deciding what belongs inside a consistency boundary. | Aggregates |
| You need reliable event-driven communication or separate read models. | Events, CQRS, and Event Sourcing |
| You are deciding between architectural styles or deployment boundaries. | Architecture Choices |

A cheatsheet should help you ask better questions, recognize important distinctions, and understand trade-offs. When a decision has significant consequences, follow the linked concept pages and validate the decision against real business scenarios.

**The goal is not to apply every DDD pattern. It is to build a model that expresses the business accurately and an architecture that allows that model to evolve.**