# Anti-patterns

This section describes common anti-patterns to avoid when applying Domain-Driven Design. These are traps that can lead to ineffective or even harmful implementations of DDD.

*   **Ignoring the Ubiquitous Language  (Lost in Translation):**
    *   **Description:** Not establishing and using a common language between developers and domain experts.
    *   **Problem:** Leads to misunderstandings, miscommunication, and a model that doesn't accurately reflect the domain. This results in software that doesn't solve the right problems.
    *   **Solution:** Actively engage with domain experts, create a glossary of terms, and use the Ubiquitous Language consistently in code, documentation, and communication. Make the Ubiquitous Language a living document.

*   **Anemic Domain Model  (The Hollow Shell):**
    *   **Description:** Domain objects (Entities and Value Objects) have little or no behavior. Business logic is placed in Services or other external classes.
    *   **Problem:** Violates encapsulation, making the domain model less expressive and leading to procedural code disguised as objects. It becomes difficult to understand the business rules by looking at the domain model.
    *   **Solution:** Move business logic into the domain objects. Ensure that Entities and Value Objects have meaningful behavior. Ask: "What actions can this object perform?"

*   **God Object/Big Ball of Mud  (The Everything Doer):**
    *   **Description:** A single object or module that tries to do too much. It becomes overly complex, difficult to understand, maintain, and test.
    *   **Problem:** Violates the Single Responsibility Principle, making the system brittle and hard to change. Changes in one area can have unintended consequences in other areas.
    *   **Solution:** Decompose the God Object into smaller, more focused objects or modules. Identify clear responsibilities and boundaries. Apply Bounded Contexts if necessary.

*   **Premature Generalization  (Over-Engineering):**
    *   **Description:** Trying to create overly generic models that don't reflect the specific needs of the domain. This often leads to unnecessary complexity and makes the model harder to understand and use.
    *   **Problem:** Creates a model that is too abstract and doesn't capture the nuances of the domain. It can also lead to code that is difficult to change when new requirements arise because the abstractions are too broad.
    *   **Solution:** Focus on modeling the specific needs of the current domain. Avoid adding complexity unless it is truly necessary. Refactor and generalize *later* if needed, based on concrete examples.

*   **Data-Centric Design (Database-Driven Design)  (The Database Dictates):**
    *   **Description:** Designing the domain model based on the database schema rather than the actual domain concepts.
    *   **Problem:** Leads to a model that is tightly coupled to the database and doesn't accurately reflect the domain. It makes it difficult to change the database or the domain model independently, hindering evolution.
    *   **Solution:** Focus on understanding the domain first and then design the database to *support* the domain model, not the other way around.

*   **CRUD-Based Design (Create, Read, Update, Delete)  (Just Four Operations):**
    *   **Description:** Focusing solely on CRUD operations and neglecting the rich behavior of the domain.
    *   **Problem:** Leads to an Anemic Domain Model and doesn't capture the business rules and processes. It results in a system that is just a data access layer with a thin UI on top.
    *   **Solution:** Identify the key business operations and model them as Domain Services or methods on Aggregates. Think about the actions users perform, not just the data they manipulate.

*   **Using Value Objects as Entities or vice versa ↔ (Identity Crisis):**
    *   **Description:** Incorrectly classifying domain concepts as either Value Objects or Entities.
    *   **Problem:** Leads to incorrect modeling and can cause issues with identity, equality, and state management.
    *   **Solution:** Carefully consider the characteristics of each concept and choose the appropriate type. Remember that Entities have identity, while Value Objects are defined by their attributes. Ask: "Does this concept have a unique identity that distinguishes it from other instances?"

---
