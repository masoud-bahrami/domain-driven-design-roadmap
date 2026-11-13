# 03 Strategic Design

This section focuses on applying DDD at a higher level, dealing with the overall architecture and organization of complex systems. You'll learn how to analyze the business domain, identify strategic subdomains, and map the relationships between different parts of the system.

**Goal:** Apply DDD to larger systems and understand context mapping and integration.

*   **Key Concepts:**

    *   **Subdomains ( Classify domain parts by strategic importance) :**
        *   **Core Subdomains :** These are the most valuable and differentiating parts of the business. They provide a competitive advantage and require the most investment. Changes in the core domain have significant business impact.
            *   *Example:* For Amazon, "Order Fulfillment"  and "Recommendation Engine"  are core.
        *   **Supporting Subdomains :** Important for business function but not differentiating. They support the Core Subdomains, often implemented with less bespoke solutions.
            *   *Example:* "Inventory Management"  or "Customer Support" .
        *   **Generic Subdomains :** Common functionalities not specific to the business. Often purchased off-the-shelf or implemented using existing solutions.
            *   *Example:* "User Authentication" , "Logging" , or "Payment Processing"  (using a third-party gateway).
        *    Subdomain Analysis - *Analyze a business and categorize its subdomains. Example: In an online learning platform, "Course Creation"  might be Core, "Student Management"  Supporting, and "Payment Processing"  Generic.*

    *   **Bounded Contexts ( Define explicit boundaries for models) :**
        *   A Bounded Context defines a specific context within which a particular domain model is valid and consistent. It establishes a semantic boundary, defining the meaning of terms and concepts *within that context*. The same term can have different meanings in different Bounded Contexts.
        *   *Example:* "Product" might mean a physical item in "Inventory"  but a digital course in "Course Catalog" .
        *    Context Boundary Identification - *Identify contexts. Example: In e-commerce: "Sales" , "Shipping" , "Inventory" , "Customer Support" .*

    *   **Relationship between Subdomains and Bounded Contexts ( Understand the crucial distinction) :**
        *   **Subdomain:** A conceptual area of the business (Problem Space). 
        *   **Bounded Context:** A concrete implementation with its own model and Ubiquitous Language (Solution Space). 
        *   One Subdomain can be implemented by one or more Bounded Contexts. A single Bounded Context might encompass multiple closely related Subdomains if they share a consistent model and language.
        *   *Example:* "Order Management"  (Subdomain) might be implemented by "Sales"  and "Fulfillment"  (Bounded Contexts). "Sales" focuses on payment status, customer details, and order total; "Fulfillment" on shipping address, tracking, and delivery status.

    *   **Context Mapping and Integration ( Visualize relationships and define integration strategies between Bounded Contexts) :**
        *   **Context Map:** A diagram visualizing relationships between Bounded Contexts, showing interactions and integration patterns. 
        *   **Integration Patterns (Strategies for integrating Bounded Contexts) :**
            *   **Shared Kernel :** Sharing a subset of the domain model (code, data structures) between contexts. Useful for tightly coupled contexts with significant shared logic. *Caution:* Creates tight coupling, hindering independent evolution. Use sparingly.
                *   *Example:* Sharing `CustomerId`  between "Sales"  and "Customer Support" .
            *   **Customer-Supplier (Upstream/Downstream) /⬅:** One context (Upstream) influences another (Downstream).
                *   **Conformist :** Downstream conforms to Upstream's model. Simple but can cause issues if the Upstream model doesn't fit Downstream needs.
                *   **Anticorruption Layer (ACL) :** Downstream creates a translation layer to protect its model from Upstream changes. Crucial for maintaining Downstream model integrity.
                    *   *Example:* "Shipping"  uses an ACL to translate "Sales"  `Order` to its `Shipment` model.
                *   **Open Host Service (OHS) :** Upstream publishes a well-defined interface (API, message schema) for Downstream use. Gives Upstream more control.
                *   **Published Language (PL) :** A formal, documented language (schema, protocol, DTOs) for context communication, ensuring interoperability. Often used with OHS or ACL.
                *   **Separate Ways :** Contexts are completely independent and don't integrate. Suitable for minimal/no interaction.
            *   **Anticorruption Layer (ACL) :** As above, crucial for decoupling and protecting models.
            *   **Published Language (PL) :** As above, essential for robust communication.
        *   **Technical Implementation of Integration :**
            *   **REST APIs :** Synchronous HTTP communication. Good for request/response, especially real-time data exchange.
            *   **Message Queues (e.g., Kafka , RabbitMQ , Azure Service Bus ):** Asynchronous communication using messages (Domain Events). Essential for EDA, high event volumes, and loose coupling.
            *   **Eventual Consistency ⏳:** Data might not be immediately consistent across contexts. Requires handling synchronization, conflicts, and eventual consistency handlers (compensating transactions, retry mechanisms, idempotent message handling).

    *   **Strategic Design ( Define architecture based on domain analysis) :**
        *   Strategic Design makes high-level system structure decisions based on domain analysis. This includes identifying Bounded Contexts, defining relationships, and choosing integration strategies. The Context Map is the key artifact.

    *   **Distillation ( Focus on essential domain parts) :**
        *   Distillation focuses on the core domain, simplifying or outsourcing less important parts to reduce complexity and focus development efforts. Techniques:
            *   **Prioritization :** Focus on features delivering the most business value.
            *   **Simplification :** Streamline processes and remove unnecessary complexity.
            *   **Outsourcing :** Delegate non-core functionalities to third-party providers or generic solutions.

---


## Practical exercise

Draw a Context Map and annotate each relationship with ownership, integration pattern, and rationale.

## Completion criteria

- Explain the difference between Subdomains and Bounded Contexts.
- Identify Core, Supporting, and Generic Subdomains.
- Explain the boundaries and relationships in a Context Map.
