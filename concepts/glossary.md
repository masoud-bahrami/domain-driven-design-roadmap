# Glossary

This glossary explains key terms used in Domain-Driven Design, helping bridge the gap between business experts and developers.

*   **Ubiquitous Language  (The Common Tongue):** A common language shared by developers and domain experts to ensure clear communication and understanding of the domain. It's used consistently in code, documentation, and discussions.

*   **Aggregate  (The Consistency Boundary):** A cluster of associated objects treated as a single unit for data changes. It's like a family: you interact with the family as a whole, not individual members in isolation. An Aggregate has a single root Entity, and all access to the Aggregate is through that root. *Example:* An `Order` is an Aggregate. It contains `OrderItems`, `ShippingAddress`, etc. You don't update an `OrderItem` directly; you go through the `Order`.

*   **Aggregate Root  (The Guardian):** The Entity at the top of an Aggregate. It's the only object in the Aggregate that can be accessed directly from outside. It's responsible for enforcing business rules and maintaining the consistency of the entire Aggregate. *Example:* In the `Order` Aggregate, the `Order` Entity itself is the Aggregate Root.

*   **Entity  (Has an Identity):** An object with a unique identity that persists over time. This identity distinguishes it from other objects, even if they have the same attributes. *Example:* A `Customer` has a unique ID, even if two customers have the same name and address.

*   **Value Object  (Defined by its Attributes):** An immutable object that represents a descriptive aspect of the domain. It is defined by its attributes rather than its identity. If all attributes are the same, two Value Objects are considered equal. *Example:* `Address`, `Money`, `Date`.

*   **Domain Service  (An Operation That Doesn't Belong):** An operation that doesn't naturally belong to an Entity or Value Object. It often involves coordinating multiple domain objects or interacting with external systems. *Example:* `TransferFunds`, `CalculateShippingCost`.

*   **Domain Event  (Something That Happened):** A significant occurrence within the domain that the business stakeholders care about. It represents something that has happened in the past. Events are immutable and are named in the past tense. *Example:* `OrderShipped`, `PaymentReceived`, `ProductAddedToCart`.

*   **Repository  (The Data Gateway):** An interface or class that provides access to Aggregates. It abstracts away the details of data storage, allowing the domain model to remain independent of the persistence mechanism.

*   **Subdomain  (A Part of the Whole):** A part of the overall business domain. It represents a specific area of expertise or responsibility within the business. *Example:* "Sales," "Shipping," "Inventory Management."

*   **Bounded Context  (The World of Meaning):** A semantic boundary around a specific domain model. It defines the context in which a particular model is valid and consistent. The same term can have different meanings in different Bounded Contexts. *Example:* "Customer" might mean a paying customer in the "Sales" context but a support ticket requester in the "Support" context.

*   **Context Map  (The Territory Map):** A diagram that visualizes the relationships between different Bounded Contexts. It shows how they interact and what integration patterns are used. It's like a map of different territories and their borders.

*   **Core Domain  (The Heart of the Business):** The most important and differentiating part of the business. It provides a competitive advantage. This is where you should focus your DDD efforts. *Example:* For a shipping company, "Route Optimization" is a Core Domain.

*   **Generic Subdomain  (Common Functionality):** A common functionality that is not specific to the business. These are often off-the-shelf solutions or third-party services. *Example:* User authentication, logging.

*   **Supporting Subdomain  (Supporting the Core):** An important part of the business that supports the Core Domain but does not provide a significant competitive advantage.

*   **Domain Model  (The Mental Model):** A conceptual model of the domain that captures the key concepts, relationships, and rules. It's a shared understanding between developers and domain experts.

*   **Event Sourcing  (The History Book):** A persistence strategy that stores all changes to an Aggregate as a sequence of events. This provides a complete audit trail and allows you to recreate the state of an Aggregate at any point in time.
