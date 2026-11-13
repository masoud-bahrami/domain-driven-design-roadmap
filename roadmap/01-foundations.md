# 01 Foundations

This section introduces the core building blocks of DDD. You'll learn the essential concepts and terminology that form the foundation for applying DDD in practice. The focus is on understanding the "**WHAT**" and "**WHY**" behind each concept.

**Goal:** Understand core DDD concepts and establish a Ubiquitous Language.

*   **Key Concepts:**
    *   Ubiquitous Language ( Establish a shared vocabulary)
        *    "Domain-Driven Design - tackling complexity in the heart of software" by Eric Evans
        *    "Domain-Driven Design Distilled" by Vaughn Vernon
        *    Ubiquitous Language Workshop (All Roles) - *Define a glossary for a simple domain (e.g., a blog, online store).*
    *   Bounded Contexts ( Define explicit boundaries for models)
        *    "Domain-Driven Design - tackling complexity in the heart of software" by Eric Evans
        *    "Domain-Driven Design Distilled" by Vaughn Vernon
        *    Context Boundary Identification - *Identify contexts in a business scenario (e.g., e-commerce with sales, shipping, support).*
    *   Entities ( Objects with a distinct identity)
        *    Distinguish from Value Objects
        *    Entity Modeling - *Model entities for a scenario (e.g., a Customer with an ID, Orders, etc.).*
    *   Value Objects ( Immutable objects defined by attributes)
        *    Implement equality and immutability
        *    Value Object Implementation - *Implement a Value Object (e.g., Address, Money) in code.*
    *   Aggregates ( Clusters of related objects with an Aggregate Root)
        *    Enforce consistency and manage transactions
        *    Aggregate Design - *Design an Aggregate (e.g., Order with Order Items) for a scenario.*
    *   Domain Events ( Significant occurrences in the domain)
        *    Identify important events
        *    Event Identification - *Identify domain events in a business process (e.g., OrderPlaced, OrderShipped).*
    *   Repositories ( Interfaces for accessing Aggregates)
        *    Separate domain logic from data access
        *    Repository Interface Definition - *Define Repository interfaces for Aggregates.*
    *   Services (Domain/Application) ( Operations not belonging to Entities/Value Objects)
        *    Distinguish domain vs. application logic
        *    Service Identification - *Identify Domain and Application Services in a scenario.*

---


## Practical exercise

Choose a domain you know. List its important concepts, terms, rules, and decisions. Ask two people involved in the work to explain the same process independently. Compare where their language and assumptions differ.

## Completion criteria

- Explain when DDD is likely to be useful and when it may be excessive.
- Explain Ubiquitous Language, Subdomain, Bounded Context, Entity, Value Object, Aggregate, and Domain Event.
- Identify assumptions that need validation with domain experts.
