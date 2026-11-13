# 04 Tactical Design

This section dives into the practical implementation of DDD patterns in code. You'll learn how to translate the domain model into a working software system, using specific techniques and addressing common implementation challenges. This is where you'll put your understanding of DDD into practice.

*   **Key Concepts and Implementation Details:**

    *   **Domain Model Implementation Patterns ( Apply best practices in code):** This is about translating your domain model (Entities, Value Objects, Aggregates, Domain Events) into actual code.
        *    *Implement Entities:* Focus on identity, equality (especially for mutable Entities), and lifecycle management. Example: A `Customer` Entity with a unique ID, methods for updating address, etc.
        *    *Implement Value Objects:* Focus on immutability and equality based on value, not identity. Example: An `Address` Value Object with street, city, zip code, etc., where two addresses are equal if all their components are equal.
        *    *Implement Aggregates and Aggregate Roots:* Enforce consistency boundaries. Example: An `Order` Aggregate Root containing `OrderItems`. Only the `Order` can be directly accessed and modified from outside the Aggregate.
        *    *Implement Domain Events:* Focus on capturing significant occurrences in the domain. Example: An `OrderShipped` event with relevant data like shipment date and tracking number.
        *    "Implementing Domain-Driven Design" by Vaughn Vernon (Chapters on Tactical Design)
        *    Sample code repositories demonstrating DDD implementation in your chosen language.

    *   **Repositories ( Implement data access abstraction):** Implement interfaces for accessing Aggregates, separating domain logic from persistence concerns.
        *    *Implement Repository Interfaces:* Define interfaces that specify the operations for retrieving and persisting Aggregates. Example: An `OrderRepository` interface with methods like `GetById(Guid id)`, `Save(Order order)`.
        *    *Implement Concrete Repositories:* Implement the interfaces using your chosen persistence technology (e.g., Entity Framework, JPA, MongoDB).
        *    Understand different repository patterns (e.g., simple repository, specification pattern).
        *    "Patterns of Enterprise Application Architecture" by Martin Fowler (Chapter on Data Access Patterns)

    *   **Domain Services ( Encapsulate domain logic that doesn't belong to Entities or Value Objects):** Implement services for operations that span multiple Aggregates or don't naturally fit within a single Entity or Value Object.
        *    *Identify and Implement Domain Services:* Example: An `OrderFulfillmentService` that handles the process of shipping an order, which might involve interactions with multiple Aggregates (Order, Inventory, Shipping).
        *    Understand the difference between Domain Services and Application Services.

    *   **Application Services ( Orchestrate use cases and transactions):** Implement services that handle use cases and coordinate interactions between domain objects. These are often thin wrappers around domain logic.
        *    *Implement Use Cases as Application Services:* Example: A `PlaceOrder` Application Service that receives a request to place an order, retrieves necessary data from Repositories, invokes methods on domain objects, and persists changes.
        *    Understand how Application Services handle transactions and security.

    *   **CQRS ( Separate read and write models):** Implement Command Query Responsibility Segregation to optimize read and write operations separately.
        *    *Separate Command and Query Models:* Create separate data models for handling commands (write operations) and queries (read operations).
        *    *Implement Command Handlers and Query Handlers:* Implement handlers that process commands and queries.
        *    Understand the benefits and trade-offs of CQRS (e.g., eventual consistency).

    *   **Event Sourcing ( Persist domain events as the source of truth):** Persist domain events instead of the current state of Aggregates.
        *    *Implement Event Store:* Choose an event store or implement a custom one.
        *    *Implement Aggregate Reconstitution:* Reconstruct the current state of Aggregates from their event history.
        *    Understand eventual consistency and how to handle it.

    *   **Saga Pattern ( Manage long-running transactions across Aggregates or Bounded Contexts):** Implement Sagas to coordinate complex business processes that involve multiple steps and might span different parts of the system.
        *    *Implement Saga Orchestration or Choreography:* Choose an appropriate Saga implementation approach.
        *    *Implement Compensation Logic:* Define actions to undo changes in case of failures.
        *    Understand the different Saga implementation patterns and their trade-offs.

    *   **Testing DDD Applications ( Write behavior-focused tests):** Focus on testing the behavior of the domain model, not just implementation details.
        *    *Write Unit Tests for Domain Logic:* Test Entities, Value Objects, Aggregates, and Domain Services in isolation.
        *    *Write Integration Tests for Aggregate Interactions:* Test how Aggregates interact with each other and with Repositories.
        *    *Use Behavior-Driven Development (BDD):* Use a BDD framework (e.g., SpecFlow, Cucumber) to write acceptance tests that define the expected behavior of the system.

---


## Practical exercise

Model one important business operation. Identify invariants, Aggregate boundaries, Entities, Value Objects, Domain Events, and repository needs.

## Completion criteria

- Explain why an object is an Entity or Value Object.
- Identify consistency boundaries and invariants.
- Design an Aggregate around business rules rather than database structure.
