# 05 Implementation And Testing

Domain-Driven Design is a set of principles and patterns that can be applied regardless of the specific programming paradigm you use. However, the way you implement those principles and patterns will differ depending on whether you're using Object-Oriented Programming (OOP), Functional Programming (FP), or other paradigms. This section explores how DDD concepts translate into different programming styles and provides resources for further learning.

*   **DDD with Object-Oriented Programming (OOP) :**

    *   OOP has historically been the most common paradigm used with DDD. Many of the original DDD concepts and patterns were described in an OOP context.
    *   **Key Mappings:**
        *   **Entities:** Map naturally to classes with identity and mutable state.
        *   **Value Objects:** Map to immutable classes or structs.
        *   **Aggregates:** Represented by a cluster of objects with an Aggregate Root acting as the entry point.
        *   **Domain Services:** Implemented as classes with methods that encapsulate domain logic.
        *   **Repositories:** Implemented as interfaces and concrete classes that provide access to Aggregates.
    *   **Common Practices:**
        *   Using inheritance and polymorphism to model domain concepts.
        *   Encapsulating state within objects and using methods to interact with that state.
        *   Using design patterns like Factory, Strategy, and Observer to implement DDD patterns.
    *   **Example (Conceptual C#):**

        ```csharp
        public class Order // Aggregate Root (Entity)
        {
            public Guid Id { get; }
            public List<OrderItem> Items { get; } // Collection of Value Objects
            public Address ShippingAddress { get; set; } // Value Object
            // ... other properties and methods
        }

        public record OrderItem(Guid ProductId, int Quantity, decimal Price); // Value Object

        public interface IOrderRepository // Repository Interface
        {
            Order GetById(Guid id);
            void Save(Order order);
        }
        ```

    *   **Resources for OOP and DDD:**
        *   **Patterns, Principles, and Practices of Domain-Driven Design by Scott Millett, Nick Tune:** While not exclusively OOP-focused, it heavily uses C#  examples and is very relevant for OOP implementations of DDD.
        *   **Head First Object-Oriented Analysis and Design by Brett McLaughlin, Gary Pollice, and David West:** A more approachable introduction to OOAD that can be helpful for understanding the underlying principles used in OOP DDD.
        *   **Design Patterns: Elements of Reusable Object-Oriented Software (Gang of Four) :** While not DDD-specific, understanding design patterns is crucial for writing clean and maintainable OOP code in a DDD context.

*   **DDD with Functional Programming (FP) :**

    *   FP offers a different approach to implementing DDD, emphasizing immutability, pure functions, and data transformations.
    *   **Key Mappings:**
        *   **Entities:** Often represented as immutable data structures (records, structs) with identity.
        *   **Value Objects:** Map naturally to immutable data structures.
        *   **Aggregates:** Represented as immutable data structures with functions that return new versions of the Aggregate.
        *   **Domain Services:** Implemented as pure functions that take domain data as input and return new domain data as output.
        *   **Repositories:** Implemented as functions that operate on data structures representing the domain.
    *   **Common Practices:**
        *   Using algebraic data types (ADTs) to model domain concepts.
        *   Using pattern matching to handle different data structures.
        *   Avoiding mutable state and side effects.
        *   Using functions to transform data and implement domain logic.
    *   **Example (Conceptual F#):**

        ```fsharp
        type Address = { Street: string; City: string; ZipCode: string } // Record (Value Object)

        type OrderItem = { ProductId: Guid; Quantity: int; Price: decimal } // Record (Value Object)

        type Order = { // Record (Entity/Aggregate)
            Id: Guid
            Items: OrderItem list
            ShippingAddress: Address
            // ... other properties
        }

        // Domain Service (Pure Function)
        let calculateTotal (order: Order) =
            order.Items |> List.sumBy (fun item -> item.Quantity * item.Price)

        // Repository (Conceptual function)
        let getOrderById (id: Guid) : Order option = // Option to handle absence
            // ... retrieve order from data store
            Some { Id = id; Items = []; ShippingAddress = { Street = ""; City = ""; ZipCode = "" } }
        ```

    *   **Resources for FP and DDD:**
        *   **Domain Modeling Made Functional by Scott Wlaschin:** A fantastic resource specifically focused on applying functional programming techniques to domain modeling. Uses F# as the primary language but the concepts are applicable to other functional languages.
        *   **Functional and Reactive Domain Modeling by Debasish Ghosh:** Explores both functional and reactive approaches to domain modeling, with examples in Scala and other languages.
        *   **Relevant documentation and tutorials for your chosen functional language (e.g., F#, Haskell, Scala):**

*   **Key Differences and Considerations :**

    *   **State Management:** OOP relies on mutable state within objects, while FP emphasizes immutability. This has implications for how you model Aggregates and handle updates.
    *   **Side Effects:** OOP allows side effects within methods, while FP encourages pure functions with no side effects. This affects how you implement Domain Services and interact with external systems.
    *   **Data Structures:** OOP uses classes and objects, while FP often uses algebraic data types and records.
    *   **Tooling and Libraries:** The available tooling and libraries for DDD may vary depending on the programming language and paradigm you choose.

*   **Choosing a Paradigm for DDD ( Context and team expertise are key) :**

    *   The choice of programming paradigm for DDD depends on various factors:
        *   **Team Expertise:** The team's familiarity with OOP or FP.
        *   **Project Requirements:** The complexity of the domain and the need for performance, scalability, or maintainability.
        *   **Existing Codebase:** If you're working with a legacy system, you might need to choose a paradigm that is compatible with the existing code.
        *   **Domain Characteristics:** Some domains might lend themselves better to one paradigm over another. For example, domains with complex state transitions might be easier to model with OOP, while domains with complex data transformations might be better suited for FP.

---


## Practical exercise

Implement one use case and tests around its business behavior. Keep domain decisions in the domain model and orchestration in the application layer.

## Completion criteria

- Distinguish application orchestration from domain behavior.
- Test important rules without requiring infrastructure unnecessarily.
- Explain how persistence and external integrations support rather than define the model.
