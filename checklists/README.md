# DDD Checklists and Templates

This section provides practical tools, checklists, and templates to help you apply DDD more effectively. These resources are designed to guide you through key DDD activities and ensure you don't miss important steps.

### Checklists for Key Activities 

These checklists provide a structured approach to key DDD activities, ensuring you cover all essential aspects.

#### Event Storming Checklist 

*   [ ] Gather the right people (domain experts, developers, testers, etc.).
*   [ ] Prepare the space (large wall, sticky notes, markers).
*   [ ] Define the scope of the Event Storming session.
*   [ ] Identify Domain Events (past tense, business-relevant).
*   [ ] Place Domain Events on the timeline.
*   [ ] Identify Commands (triggering events).
*   [ ] Identify Aggregates (grouping related events and commands).
*   [ ] Identify External Systems and Actors.
*   [ ] Identify Read Models (for queries).
*   [ ] Identify Bounded Contexts (grouping related parts of the domain).
*   [ ] Document key insights and open questions.

#### Context Mapping Checklist 

*   [ ] Identify all Bounded Contexts.
*   [ ] Determine the relationships between each pair of contexts (e.g., Shared Kernel, Customer-Supplier, Conformist, Anti-Corruption Layer, Separate Ways, Published Language).
*   [ ] Document the integration points and protocols between contexts.
*   [ ] Draw a Context Map diagram to visualize the relationships.
*   [ ] Document the rationale behind each integration pattern choice.
*   [ ] Identify any open issues or areas of uncertainty regarding context boundaries or integration.

#### Aggregate Design Checklist 

*   [ ] Identify the Aggregate Root (a single Entity that controls access to the Aggregate).
*   [ ] Identify the Entities and Value Objects that belong to the Aggregate.
*   [ ] Define clear boundaries for the Aggregate (what is inside and what is outside).
*   [ ] Define the consistency rules (invariants) that the Aggregate must enforce.
*   [ ] Model relationships between Aggregates using references to their Roots.
*   [ ] Design Aggregate operations (methods on the Aggregate Root) that enforce consistency.
*   [ ] Identify Domain Events that the Aggregate emits.
*   [ ] Ensure that Aggregates are small and cohesive.
*   [ ] Consider performance implications of Aggregate design.

### Templates for Key Artifacts 

#### Bounded Context Canvas Template 

*   **Structure:** A table or grid with sections for: Context Name, Purpose/Mission, Domain Experts, Users/Stakeholders, Ubiquitous Language, In Scope/Out of Scope, Relationships with Other Contexts (and the patterns used), Key Domain Events, Key Aggregates, and Implementation Technology/Architecture.
*   **Where to find templates:** Search online for "Bounded Context Canvas template" or look at tools like Miro, Mural, or XMind, which often have built-in templates.

#### Aggregate Canvas Template 

*   **Structure:** Similar to the Bounded Context Canvas, a table or grid with sections for: Aggregate Name, Purpose/Responsibilities, Aggregate Root, Entities, Value Objects, Invariants/Business Rules, Relationships to Other Aggregates, Key Operations/Commands, and Key Domain Events.
*   **Where to find templates:** Search online for "Aggregate Canvas template" or adapt existing templates for other purposes.

#### Context Map Diagram 

*   **Format:** A diagram showing Bounded Contexts as boxes and their relationships as lines with labels indicating the integration patterns used (e.g., Shared Kernel, Customer-Supplier).
*   **Tools:** Use diagramming tools like Draw.io (diagrams.net), Lucidchart, PlantUML, or even simple drawing tools.

#### Event Storming Output 

*   **Format:** A timeline of Domain Events represented by orange sticky notes, with Commands (blue), Aggregates (yellow), and External Systems/Actors (pink) also represented by different colored stickies or shapes.
*   **Tools:** Physical whiteboards and sticky notes are the most common and effective, but digital whiteboarding tools can also be used.

---
