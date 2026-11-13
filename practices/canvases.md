# Canvases

Visual tools are invaluable for applying DDD, especially during workshops and collaborative sessions. Canvases provide structured templates for exploring, documenting, and communicating key DDD concepts, fostering a shared understanding between technical and business stakeholders. This section introduces some useful canvases.

*   **Bounded Context Canvas  (Defining the Scope):**

    The Bounded Context Canvas helps define and document the boundaries of a specific context. It encourages teams to explicitly consider the scope, purpose, and relationships of each context, preventing misunderstandings and promoting clear communication.

    *   **Key Elements of a Bounded Context Canvas:**
        *   **Context Name:** A concise and meaningful name reflecting the context's purpose (e.g., "Sales," "Shipping," "Inventory").
        *   **Purpose/Mission:** A brief statement of the context's goals and responsibilities within the larger system.
        *   **Domain Experts:** List of individuals with deep knowledge of this specific domain.
        *   **Users/Stakeholders:** Identification of the people who interact with or are affected by this context.
        *   **Ubiquitous Language (Within this Context):** Key terms and definitions specific to this context, highlighting potential variations from other contexts.
        *   **In Scope/Out of Scope:** Clearly defines what is included and excluded within this context, preventing scope creep.
        *   **Relationships with Other Contexts:** Describes how this context interacts with other Bounded Contexts using established integration patterns (e.g., Shared Kernel, Customer-Supplier, Conformist, Anti-Corruption Layer).
        *   **Key Domain Events:** Important events that occur within this context, signaling significant occurrences within the domain.
        *   **Key Aggregates:** The core Aggregates that represent the most important business concepts within this context.
        *   **Implementation Technology/Architecture:** The technologies, architectural styles, and infrastructure used to implement this context.

    *   **How to Use the Bounded Context Canvas:**
        *   Use it during domain exploration workshops with domain experts and developers to foster collaborative understanding.
        *   Fill it out collaboratively, ensuring all stakeholders have input and agree on the content.
        *   Use it to communicate the context's boundaries and responsibilities to other teams and stakeholders.
        *   Regularly review and update the canvas as the domain and system evolve.

*   **Aggregate Canvas  (Designing Consistency Boundaries):**

    The Aggregate Canvas helps design and document individual Aggregates, ensuring they are cohesive, well-defined, and enforce consistency within their boundaries.

    *   **Key Elements of an Aggregate Canvas:**
        *   **Aggregate Name:** A descriptive name for the Aggregate reflecting its business purpose (e.g., "Order," "Customer," "Product").
        *   **Purpose/Responsibilities:** A brief description of the Aggregate's purpose and the business rules it enforces.
        *   **Aggregate Root:** The Entity that acts as the entry point and consistency boundary for the Aggregate.
        *   **Entities (within the Aggregate):** Other Entities that are part of the Aggregate and managed by the Aggregate Root.
        *   **Value Objects (within the Aggregate):** Immutable objects that describe aspects of the Entities within the Aggregate.
        *   **Invariants/Business Rules:** The rules that must always be true for the Aggregate to be in a valid state. These rules are enforced by the Aggregate Root.
        *   **Relationships to Other Aggregates:** How this Aggregate relates to other Aggregates, often through references to their Roots. Emphasize loose coupling between Aggregates.
        *   **Key Operations/Commands:** The actions that can be performed on the Aggregate, typically through methods on the Aggregate Root.
        *   **Key Domain Events (Emitted by the Aggregate):** Events that are triggered by actions performed on the Aggregate, signaling changes in its state.

    *   **How to Use the Aggregate Canvas:**
        *   Use it during domain modeling sessions to design individual Aggregates and their internal structure.
        *   Focus on defining clear boundaries and responsibilities for the Aggregate, ensuring it enforces consistency.
        *   Use it to communicate the design of the Aggregate to other developers and stakeholders.

*   **Tools for Creating Canvases :**

    *   **Online Whiteboarding Tools (Miro, Mural, FigJam, Microsoft Whiteboard, Google Jamboard, Excalidraw):** These digital tools are ideal for collaborative canvas creation, offering templates, real-time collaboration features, and easy sharing.
    *   **Physical Whiteboards and Sticky Notes:** Still a highly effective option for in-person workshops, providing a tactile and engaging experience.
    *   **Diagramming Tools (Draw.io (diagrams.net), Lucidchart, Visual Paradigm):** Suitable for creating more formal and polished canvases for documentation purposes.

*   **Resources for DDD Canvases :**

    *   "Event Storming" by Alberto Brandolini
    *   [Bounded Context Canvas](https://github.com/ddd-crew/bounded-context-canvas)
    *   [Aggregate Design Canvas](https://github.com/ddd-crew/aggregate-design-canvas)
    *   "Domain-Driven Design Distilled" by Vaughn Vernon provides clear explanations of Bounded Contexts and Aggregates, foundational for using the canvases effectively.
    *   "Implementing Domain-Driven Design" by Vaughn Vernon provides practical examples of Aggregate design and can be a valuable companion when using the Aggregate Canvas.

---
