# Canvases

Visual tools are valuable when applying Domain-Driven Design, especially during workshops, collaborative modeling, and architecture discussions.

Canvases provide structured ways to explore, record, and communicate domain knowledge. They help technical and business stakeholders make assumptions visible, discuss important decisions, and develop a shared understanding of the problem.

A canvas is not a formal specification or a document that must be completed before development begins. It is a thinking tool. Its value comes from the conversations and discoveries it enables, not from how many fields have been filled in.

This section introduces practical canvases for exploring Bounded Contexts, designing Aggregates, mapping relationships, and cultivating Ubiquitous Language.

## 1. Bounded Context Canvas

<text color="secondary" size="xs" weight="medium">DEFINING MODEL BOUNDARIES</text>

The Bounded Context Canvas helps teams define and communicate the purpose, responsibilities, language, and boundaries of a Bounded Context.

It encourages explicit discussion about what belongs inside a context, what belongs elsewhere, and how different models interact.

### Key Elements

| Element | Description |
|---|---|
| Context Name | A meaningful name that reflects the context's business responsibility. |
| Purpose / Mission | A concise statement explaining why the context exists and what it is responsible for. |
| Business Capabilities | The capabilities and responsibilities supported by the context. |
| Domain Experts | People with knowledge of the business rules and processes represented by the context. |
| Users and Stakeholders | People or groups who interact with or are affected by the context. |
| Ubiquitous Language | Important terms, definitions, examples, and distinctions used within this context. |
| In Scope / Out of Scope | Explicit boundaries defining which responsibilities belong to this context and which do not. |
| Key Business Rules | Important rules and constraints governing the context's responsibilities. |
| Key Domain Events | Significant business occurrences that the context recognizes and may communicate to others. |
| Relationships with Other Contexts | Upstream and downstream dependencies, ownership, and Context Mapping patterns. |
| Open Questions and Assumptions | Unresolved decisions, uncertain boundaries, and assumptions that require validation. |
| Implementation Notes | Relevant architectural or technical decisions, when these have already been established. |

### How to Use It

1. Start with a candidate Bounded Context identified during domain exploration.
2. Discuss its purpose and responsibilities with domain experts.
3. Identify its language and clarify terms that have different meanings elsewhere.
4. Define what belongs inside and outside the context.
5. Identify its relationships with other contexts.
6. Record assumptions and unresolved questions rather than pretending every decision is final.
7. Review the canvas as the domain model and organizational responsibilities evolve.

**Important:** Do not use this canvas to decide prematurely that every context needs its own database, deployment, or microservice. A Bounded Context defines a model boundary, not automatically an infrastructure boundary.

## 2. Aggregate Design Canvas

<text color="secondary" size="xs" weight="medium">DESIGNING CONSISTENCY BOUNDARIES</text>

The Aggregate Design Canvas helps teams explore the responsibility, invariants, operations, and consistency requirements of an individual Aggregate.

Its primary purpose is to make the reasoning behind an Aggregate boundary explicit.

### Key Elements

| Element | Description |
|---|---|
| Aggregate Name | A meaningful name representing the business concept and responsibility. |
| Purpose / Responsibility | The business responsibility owned by the Aggregate. |
| Aggregate Root | The Entity that provides the single entry point for accessing and modifying the Aggregate. |
| Internal Entities | Entities whose identity and lifecycle are managed within the Aggregate boundary. |
| Value Objects | Concepts represented through their values, constraints, and behavior. |
| Invariants | Business rules that must hold when an operation completes. |
| Business Operations | Meaningful operations that change state while protecting the Aggregate's invariants. |
| References to Other Aggregates | Relationships with other Aggregates, typically expressed through their identities. |
| Domain Events | Significant business facts that result from successful domain operations. |
| Concurrency Considerations | Potential conflicts when multiple operations attempt to modify the Aggregate. |
| Consistency Requirements | Which changes must be consistent within the boundary and which can be coordinated eventually. |
| Open Questions | Unresolved assumptions about ownership, rules, boundaries, or behavior. |

### How to Use It

1. Start with a business responsibility, not a database table or an object graph.
2. Identify the invariants that must be protected.
3. Explore which concepts must participate in the same consistency boundary.
4. Identify the Aggregate Root and its meaningful business operations.
5. Consider references to other Aggregates and avoid unnecessary object graphs.
6. Evaluate concurrency, transaction size, and loading requirements.
7. Validate the design against realistic business scenarios.
8. Use tests to demonstrate that the Aggregate protects its invariants.

**Important:** An Aggregate is not a collection of related objects. It is a consistency boundary designed around business invariants. Do not add Entities or Value Objects simply because they are related to the Aggregate's Root.

## 3. Context Map Canvas

<text color="secondary" size="xs" weight="medium">UNDERSTANDING RELATIONSHIPS BETWEEN CONTEXTS</text>

A Context Map Canvas helps teams describe and reason about relationships between Bounded Contexts.

While the Bounded Context Canvas focuses on one context, the Context Map Canvas focuses on the connections between contexts, including ownership, dependencies, integration patterns, and translation responsibilities.

### Key Elements

| Element | Description |
|---|---|
| Upstream Context | The context that provides a model, capability, or contract consumed by another context. |
| Downstream Context | The context that depends on the upstream context. |
| Relationship Pattern | The Context Mapping pattern describing the relationship. |
| Ownership | Which team or group controls each model and its evolution. |
| Integration Contract | The information and expectations exchanged between contexts. |
| Communication Mechanism | The technical mechanism used, such as REST, messaging, or file exchange. |
| Translation Responsibility | Where and how differences between models are translated. |
| Shared Assumptions | Knowledge, terminology, or constraints that both sides rely on. |
| Change Coordination | How changes to contracts and upstream models are communicated and managed. |
| Risks and Constraints | Coupling, availability, compatibility, and organizational dependencies. |

### How to Use It

1. Identify the Bounded Contexts involved.
2. Establish their ownership and responsibilities.
3. Determine upstream and downstream relationships.
4. Select the Context Mapping patterns that describe those relationships.
5. Define integration contracts and technical mechanisms separately.
6. Identify translation responsibilities and potential model leakage.
7. Document risks, assumptions, and change coordination.
8. Review the map when context boundaries or dependencies change.

**Important:** A Context Map describes relationships between domain models and their owners. A diagram showing REST endpoints or message queues alone is not a Context Map.

## 4. Ubiquitous Language Canvas

<text color="secondary" size="xs" weight="medium">CULTIVATING SHARED UNDERSTANDING</text>

The Ubiquitous Language Canvas helps domain experts and developers explore the meaning of important terms within a Bounded Context.

Its purpose is not merely to build a glossary. It helps uncover ambiguity, discover business rules, and ensure that the language used in conversations, models, code, and tests expresses the same understanding.

### Key Elements

| Element | Description |
|---|---|
| Term | The word or expression used by domain experts. |
| Context | The Bounded Context in which the term is meaningful. |
| Definition | A precise explanation of what the term means. |
| Examples | Concrete business scenarios illustrating its meaning. |
| Counterexamples | Scenarios that may look similar but do not represent the same concept. |
| Related Concepts | Other terms and concepts connected to this one. |
| Actions | Operations or behaviors associated with the concept. |
| Lifecycle | Relevant states and transitions, where applicable. |
| Business Rules | Constraints and decisions governing the concept. |
| Synonyms and Variations | Alternative expressions used by different people or teams. |
| Ambiguities | Unclear meanings, disagreements, or competing interpretations. |
| Model Expression | How the concept is represented in the domain model, code, or tests. |
| Open Questions | Things that still need clarification or validation. |

### How to Use It

1. Listen to domain experts and collect the expressions they actually use.
2. Select terms that are important, ambiguous, or frequently misunderstood.
3. Explore concrete examples and counterexamples.
4. Discover the rules, relationships, and actions associated with each term.
5. Challenge competing definitions and clarify differences.
6. Agree on language within the Bounded Context.
7. Express the concept in the model, code, and tests where appropriate.
8. Validate the interpretation with domain experts and revisit it as understanding evolves.

**Important:** Ubiquitous Language is not a one-time documentation exercise. It is continuously cultivated through conversations, modeling, implementation, and feedback.

## 5. Event Storming Output Canvas

<text color="secondary" size="xs" weight="medium">CAPTURING DOMAIN DISCOVERIES</text>

Event Storming is a collaborative exploration technique, not simply a diagramming exercise. This canvas helps capture the important discoveries made during an Event Storming session.

It is especially useful for turning workshop discussions into questions, hypotheses, and follow-up modeling activities.

### Key Elements

| Element | Description |
|---|---|
| Workshop Scope | The business process or domain area being explored. |
| Domain Events | Significant business facts discovered during the session. |
| Commands | Intentions or requests that trigger business behavior. |
| Actors | People, roles, or external systems involved. |
| Policies | Business reactions or decisions triggered by events. |
| Aggregates / Decision Points | Candidate places where business decisions and state changes occur. |
| Read Models | Information required by actors or processes to make decisions. |
| External Systems | Systems or other contexts involved in the process. |
| Hotspots | Areas of uncertainty, disagreement, complexity, or risk. |
| Exceptions | Alternative paths, failures, and unusual business scenarios. |
| Open Questions | Issues that require further investigation. |
| Validated Insights | Findings confirmed with domain experts. |

### How to Use It

1. Define the scope and purpose of the workshop.
2. Explore the business process collaboratively.
3. Capture meaningful Domain Events and arrange them in chronological order.
4. Explore the Commands, actors, policies, and decisions involved.
5. Identify exceptions, alternative paths, and hotspots.
6. Record assumptions, disagreements, and unresolved questions.
7. Validate important findings with domain experts.
8. Use the discoveries as input for further modeling and design.

**Important:** Do not treat the workshop output as a finished domain model. Candidate Aggregates, Bounded Contexts, and relationships are hypotheses to investigate, not conclusions automatically derived from sticky notes.

## Choosing the Right Canvas

Different canvases support different kinds of conversations. Choose one based on the question you are trying to answer.

| Situation | Recommended Canvas |
|---|---|
| Defining the purpose and boundaries of a domain model | Bounded Context Canvas |
| Designing a consistency boundary around business invariants | Aggregate Design Canvas |
| Understanding dependencies between domain models | Context Map Canvas |
| Clarifying terminology and discovering business meaning | Ubiquitous Language Canvas |
| Recording discoveries from collaborative domain exploration | Event Storming Output Canvas |

These canvases can complement one another. For example, an Event Storming session may reveal candidate Bounded Contexts and important business terms. Those discoveries can then be explored using the Bounded Context and Ubiquitous Language canvases.

## Tools for Creating Canvases

The format matters less than the quality of the collaboration. Choose tools that help participants contribute, revise ideas, and preserve the results.

**Online Whiteboarding Tools**

- [Miro](https://miro.com/)
- [Mural](https://www.mural.co/)
- [FigJam](https://www.figma.com/figjam/)
- [Microsoft Whiteboard](https://whiteboard.microsoft.com/)
- [Excalidraw](https://excalidraw.com/)

These tools are useful for distributed workshops, collaborative exploration, and maintaining shared visual artifacts.

**Physical Whiteboards and Sticky Notes**

Physical workshops remain effective. They encourage participation, make relationships visible, and allow teams to rearrange ideas quickly.

**Diagramming Tools**

- [diagrams.net](https://www.diagrams.net/)
- [Lucidchart](https://www.lucidchart.com/)
- [Visual Paradigm](https://www.visual-paradigm.com/)

These tools are useful when workshop discoveries need to become more formal diagrams or architectural documentation.

## Further Reading and Resources

- **[Bounded Context Canvas](https://github.com/ddd-crew/bounded-context-canvas)** by DDD Crew.
- **[Aggregate Design Canvas](https://github.com/ddd-crew/aggregate-design-canvas)** by DDD Crew.
- **[Event Storming](https://www.eventstorming.com/)** by Alberto Brandolini.
- **Domain-Driven Design Distilled**, Vaughn Vernon. A concise introduction to Bounded Contexts, Context Mapping, and Aggregates.
- **Implementing Domain-Driven Design**, Vaughn Vernon. Practical guidance on implementing domain models and designing Aggregates.

## Final Thought

A canvas is useful when it makes invisible assumptions visible, gives people a shared space to reason about the domain, and helps turn conversations into better decisions.

Do not fill canvases for the sake of documentation. Use them to ask better questions, challenge existing assumptions, and discover what the business actually means.

**The canvas is not the model. The understanding developed through using it is what matters.**