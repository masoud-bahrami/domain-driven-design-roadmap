# Evolution, Scaling, and Legacy Systems

DDD is not a one-time modeling exercise. Models, teams, and systems change as understanding grows.


## Scaling DDD

Applying DDD in larger organizations and projects presents unique challenges related to team organization, communication, and managing dependencies between different parts of the system. This section explores strategies for scaling DDD effectively.

*   **Team Organization and Bounded Contexts :**

    *   **Team per Bounded Context:** Organizing teams around Bounded Contexts is a common and effective approach. This allows teams to become deep experts in their respective domains and fosters ownership and autonomy.
    *   **Stream-Aligned Teams:** Teams aligned to the flow of business value, ideally owning a single Bounded Context or a small, cohesive set of related contexts.
    *   **Enabling Teams:** Teams providing services, tools, or platforms that other teams can use. They might own Generic Subdomains or provide shared infrastructure.
    *   **Complicated Subsystem Teams:** Teams responsible for complex technical subsystems that require specialized skills. They might support multiple Bounded Contexts.
    *   **Platform Teams:** Teams that build and maintain internal platforms that other teams rely on. They often provide infrastructure or shared services.
    *   **Team API Design:** When teams own separate Bounded Contexts, clear and well-defined APIs are essential for communication and integration. Consider using API-first design principles and documenting APIs using tools like OpenAPI (Swagger).

*   **Managing Dependencies Between Bounded Contexts and Teams :**

    *   **Context Mapping as a Communication Tool :** The Context Map becomes a crucial communication tool for understanding dependencies between teams and Bounded Contexts. It helps to visualize how different parts of the system interact and identify potential integration challenges.
    *   **Communication Patterns:** Establishing clear communication patterns between teams is essential. This can include regular meetings, shared documentation, and dedicated communication channels.
    *   **Dependency Management:** Implement clear dependency management practices to avoid integration issues. This can include versioning APIs, using contract testing, and establishing clear release processes.
    *   **Shared Understanding:** Foster a shared understanding of the overall domain and the relationships between different Bounded Contexts. This can be achieved through regular domain modeling workshops and cross-team communication.

*   **Scaling the Ubiquitous Language :**

    *   **Consistent Terminology Across Teams:** Ensure that the Ubiquitous Language is used consistently across all teams working on the project. This can be achieved through a shared glossary, regular communication, and domain modeling workshops involving representatives from different teams.
    *   **Context-Specific Language:** Recognize that the Ubiquitous Language might have slight variations in different Bounded Contexts. Be explicit about these variations and document them clearly.

*   **Organizational Structures and DDD Adoption :**

    *   **Top-Down vs. Bottom-Up Adoption:** DDD can be adopted in a top-down or bottom-up manner, or a combination of both. Top-down adoption requires strong leadership support, while bottom-up adoption relies on grassroots efforts within teams.
    *   **Iterative Adoption:** DDD is best adopted iteratively, starting with a small part of the system and gradually expanding its scope.


## Applying DDD to Legacy Systems

Applying DDD to existing, or "legacy," systems presents unique challenges. These systems often lack a clear domain model, have tangled dependencies, and may be difficult to change. However, DDD can be incredibly valuable in these situations, providing a framework for understanding and gradually improving these systems. This section outlines strategies and specific patterns from Eric Evans for applying DDD in a legacy context, drawing heavily from his paper "Getting Started with DDD When Surrounded by Legacy Systems."

*   **Challenges of Applying DDD to Legacy Systems :**

    *   **Lack of a Clear Domain Model:** The existing code may not reflect a well-defined domain model, making it difficult to understand the business logic and relationships between concepts.
    *   **Tangled Dependencies:** The system may have complex and intertwined dependencies, making it difficult to isolate parts of the code and introduce changes without unintended consequences.
    *   **Lack of Tests:** The absence of comprehensive tests makes refactoring risky and can lead to regressions.
    *   **Outdated Technology:** The system may be built using outdated technologies that are difficult to work with or integrate with modern systems.
    *   **Resistance to Change:** There may be resistance to adopting new practices or refactoring the existing code within the organization.

*   **Strategies and Patterns for Applying DDD to Legacy Systems :**

    *   **Start Small and Focus on High-Value Areas :** Don't attempt to rewrite the entire system at once. Identify a small, high-value area of the business that would benefit most from DDD and focus your initial efforts there. This allows for demonstrating value quickly and building momentum.
    *   **Identify Implicit Bounded Contexts :** Analyze the existing code, database schema, and documentation (if any) to identify implicit boundaries and potential Bounded Contexts. Look for areas of the system that have distinct sets of concepts, terminology, and business rules.
    *   **Create an Anti-Corruption Layer (ACL) :** An ACL acts as a translation layer between the new DDD code and the legacy system. It prevents the legacy system's data structures, terminology, and design flaws from polluting the new domain model. This allows you to develop clean DDD code within a well-defined boundary.
    *   **Strangler Fig Application (Gradual Migration) :** This pattern involves gradually replacing parts of the legacy system with new DDD-based components. The new components "strangle" the old system over time, intercepting more and more requests, until the legacy system is eventually completely replaced. This allows for a phased migration with minimal disruption to existing functionality.
    *   **Extract Services:** Identify cohesive pieces of functionality within the legacy system that can be extracted as independent services. These services can then be integrated with the new DDD code through well-defined APIs. This promotes modularity and allows for independent deployment and scaling of different parts of the system.
    *   **Characterization Tests (Golden Master Tests) :** If the legacy system lacks tests, start by writing characterization tests. These tests capture the existing behavior of the system, allowing you to refactor with confidence without changing its observable functionality. These tests act as a "golden master" against which future changes can be compared.
    *   **Embrace the Existing Data Model (Initially) :** In the initial stages, it might be necessary to adapt the new domain model to work with the existing database schema or data structures. However, as you refactor and extract services, you can gradually decouple the domain model from the legacy data storage.
    *   **Communicate with Stakeholders:** Clearly communicate the benefits of applying DDD and the planned migration strategy to stakeholders. Address any concerns, build support for the changes, and keep stakeholders informed of progress.

    *   **Specific Patterns from Eric Evans ("Getting Started with DDD When Surrounded by Legacy Systems"):**

        *   **Bubble Context :** This pattern involves creating a new Bounded Context around a portion of the legacy system that you want to modernize. The Bubble Context acts as a protective layer, isolating the new DDD code from the legacy code. Communication between the Bubble Context and the legacy system is handled through well-defined interfaces, often implemented as an Anti-Corruption Layer. The key characteristic of a Bubble Context is its *temporary* nature. It's meant to be a stepping stone, not a permanent solution.
        *   **Autonomous Bubble :** An extension of the Bubble Context where the new context is designed to be as independent as possible from the legacy system. This minimizes dependencies and allows the new context to evolve more freely. The integration with the legacy system is often delayed or minimized to specific data exchanges or asynchronous communication. The Autonomous Bubble has its own data store (not just a cache) and aims for long-term integration, not eventual absorption into the legacy.
        *   **Exposing Legacy Assets as Services :** This pattern involves wrapping existing functionality within the legacy system as services that can be consumed by the new DDD code. This allows you to reuse existing functionality without having to rewrite it immediately. These services often become part of the integration layer between the legacy system and the new Bounded Contexts. This is often implemented using an Open Host Service pattern, defining a clear protocol for interaction.
        *   **Expanding a Bubble :** This describes the strategy of gradually expanding the scope of a Bubble Context to encompass more and more of the legacy system's functionality. As the Bubble Context grows and takes over responsibilities, the legacy system shrinks until it is eventually completely replaced. This is closely related to the Strangler Fig Application pattern and represents a long-term migration strategy.

        *   **Key Differences between Bubble Context and Autonomous Bubble:** The core distinction lies in data management and longevity. A Bubble Context relies on the legacy system for its data (through an ACL) and is typically short-lived, eventually being absorbed back into the legacy. An Autonomous Bubble has its own data store, operates independently, and is intended for longer-term integration or replacement of legacy functionality.

*   **Example: Applying an ACL within a Bubble Context (and transitioning to an Autonomous Bubble):**

    Imagine a legacy system that stores customer data in a format that doesn't align with your new domain model. You can initially create a *Bubble Context* called "Customer Management" to encapsulate the new DDD implementation. Within this "Customer Management" context, you create an *Anti-Corruption Layer*. The ACL translates the legacy customer data into a `Customer` Aggregate in your new DDD code.

    Later, as the "Customer Management" context becomes more important, you might transition to an *Autonomous Bubble*. This involves creating a dedicated data store for the "Customer Management" context and implementing a *Synchronizing ACL* to keep the data in sync with the legacy system (perhaps using a nightly batch or message queue). This allows the "Customer Management" context to evolve independently and potentially replace the customer data functionality in the legacy system entirely.

*   **Benefits of Applying DDD to Legacy Systems :**

    *   **Improved Understanding of the System:** DDD provides a framework for analyzing and understanding the complex logic within legacy systems, leading to better documentation and knowledge sharing.
    *   **Reduced Complexity:** By applying Bounded Contexts and other DDD patterns, you can break down the system into smaller, more manageable parts, making it easier to understand, maintain, and change.
    *   **Increased Maintainability and Adaptability:** The new DDD code will be easier to understand, change, and maintain than the often brittle and complex legacy code.
    *   **Reduced Risk of Changes:** By using techniques like ACLs and characterization tests, you can minimize the risk of introducing errors when making changes to the system or integrating with the legacy components.
    *   **Gradual Modernization:** DDD provides a roadmap for gradually modernizing the system without requiring a risky and expensive "big bang" rewrite.

*   **Connection to Other DDD Concepts:**

    *   **Bounded Contexts:** Essential for isolating the new DDD code from the legacy system and managing integration points.
    *   **Anti-Corruption Layer:** A key integration pattern specifically designed for working with external systems, including legacy systems.
    *   **Refactoring:** A crucial practice for gradually improving the system and migrating functionality to the new DDD model.
    *   **Testing:** Provides confidence during refactoring and migration, ensuring that changes don't break existing functionality.

*   **Resources for Working with Legacy Systems :**

    *   **Working Effectively with Legacy Code by Michael Feathers :** This is an invaluable resource for anyone working with legacy systems. It provides practical strategies for making code more testable and easier to change.
    *   **Refactoring: Improving the Design of Existing Code by Martin Fowler :** While not specific to legacy systems, this book provides a catalog of refactorings that are essential for improving code quality and making it easier to understand and change.
    *   **Getting Started with DDD When Surrounded by Legacy Systems by Eric Evans © 2013, Domain Language, Inc.:** This paper is the primary source for the "Bubble Context," "Autonomous Bubble," and "Exposing Legacy Assets as Services" patterns.

---


## Related practices

- [Supple Design](../practices/supple-design.md)
- [Refactoring toward deeper insight](../practices/breakthrough-refactoring.md)
- [Anti-Corruption Layers and legacy boundaries](../concepts/anti-corruption-layer.md)
