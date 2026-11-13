# Core Distinctions

Domain-Driven Design contains concepts that are closely related but serve different purposes. Understanding their differences helps prevent incorrect modeling decisions, unnecessary abstractions, and architectural complexity.

Use this reference when discussing a design, reviewing a domain model, or deciding which concept applies to a particular problem.

## Concept Comparisons

| Concept A | Concept B | Difference |
|---|---|---|
| **Entity** | **Value Object** | An Entity has identity that persists across changes to its attributes. A Value Object is defined by its values and is commonly immutable. |
| **Subdomain** | **Bounded Context** | A Subdomain describes part of the business problem space. A Bounded Context defines the boundary within which a particular model and its language are consistent. |
| **Domain Event** | **Integration Event** | A Domain Event represents a meaningful fact within a domain model. An Integration Event is a message or contract published for other systems or Bounded Contexts. |
| **Domain Service** | **Application Service** | A Domain Service expresses domain behavior that does not naturally belong to an Entity or Value Object. An Application Service coordinates a use case and its interactions with the domain and infrastructure. |
| **Core Subdomain** | **Generic Subdomain** | A Core Subdomain contains business capabilities that provide competitive differentiation. A Generic Subdomain represents capabilities that are common and may be purchased, reused, or implemented with standard solutions. |
| **Model Boundary** | **Deployment Boundary** | A model boundary defines where a particular language and model are valid. A deployment boundary defines what is independently deployed and operated. They do not need to align. |
| **Aggregate** | **Entity** | An Entity is an object with identity. An Aggregate is a consistency boundary containing one or more domain objects, with an Aggregate Root controlling access to that boundary. |
| **Aggregate Root** | **Repository** | An Aggregate Root protects the Aggregate's invariants and controls its behavior. A Repository provides a way to retrieve and persist Aggregates. |
| **Command** | **Domain Event** | A Command expresses an intention or request to perform an action. A Domain Event records something that has already happened. |
| **Domain Model** | **Data Model** | A Domain Model expresses business concepts, behavior, and rules. A Data Model represents information for storage, transfer, or querying. |
| **CQRS** | **Event Sourcing** | CQRS separates responsibilities for changing and reading state. Event Sourcing stores state changes as an event sequence. They can be used independently. |
| **Domain Event** | **Event Sourcing Event** | A Domain Event represents a meaningful occurrence in the domain. An event stored by an Event Sourcing system is part of the persisted history used to reconstruct state. They may overlap, but are not necessarily identical. |
| **Bounded Context** | **Microservice** | A Bounded Context defines a semantic model boundary. A Microservice is an independently deployable and operable software unit. One does not automatically imply the other. |
| **Shared Kernel** | **Published Language** | A Shared Kernel is a deliberately shared part of a model. A Published Language is an agreed language or contract used for communication between contexts. |
| **Consistency** | **Transaction** | Consistency describes the correctness conditions that must hold. A transaction is one mechanism for applying changes atomically. An Aggregate's consistency boundary is a domain design decision, not simply a database transaction boundary. |

## Questions Before Choosing a Pattern

Before introducing a DDD concept or architectural pattern, ask:

- **Business purpose:** What business rule, requirement, or source of complexity are we addressing?
- **Language:** Which language and model are valid in this context? Do the terms mean the same thing to everyone involved?
- **Ownership:** Who owns the business decision, domain model, and data?
- **Consistency:** What must change together, and what can change independently?
- **Boundaries:** Where does the responsibility or meaning of this concept begin and end?
- **Dependencies:** Which parts of the system depend on this decision, and who controls those dependencies?
- **Evolution:** How might this model or capability change over time?
- **Complexity:** What additional operational, organizational, or implementation costs does this choice introduce?
- **Alternatives:** Is there a simpler design that preserves the required meaning and consistency?

## Common Misconceptions

A few reminders are particularly useful during design reviews:

- Not every Entity needs to belong to a large Aggregate.
- Not every business operation needs a Domain Service.
- Not every Bounded Context needs its own database or microservice.
- Not every business event needs to be published to a message broker.
- Using Domain Events does not require CQRS or Event Sourcing.
- CQRS does not necessarily require separate databases.
- A Repository is not automatically required for every database table or Entity.
- A shared vocabulary does not mean every Bounded Context must use identical terminology.
- More patterns do not necessarily mean a better domain model.

**The guiding principle:** Choose concepts and patterns because they help express the business accurately, protect important rules, or manage real complexity. Never introduce them simply because they appear in a DDD reference.

See the [Glossary](../concepts/glossary.md) for terminology and the relevant concept pages for detailed explanations and examples.