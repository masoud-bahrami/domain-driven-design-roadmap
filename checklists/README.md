# DDD Checklists and Templates

This section provides practical checklists and reusable templates for applying Domain-Driven Design in real projects.

DDD is not a process that becomes successful by completing a predefined list of activities. These resources are intended to support conversations, expose overlooked questions, and make important design decisions explicit. Adapt them to the domain, the team, and the problem you are trying to solve.

Use them as prompts for better thinking, not as mandatory procedures.

## Checklists for Key Activities

### Ubiquitous Language Checklist

Ubiquitous Language is not a document to produce once and then forget. It is continuously developed through conversations, examples, modeling decisions, and implementation.

Use this checklist throughout the project:

- [ ] Identify the domain experts and people who understand the business processes.
- [ ] Collect the terms and expressions people actually use in their work.
- [ ] Clarify the meaning of important terms with concrete examples.
- [ ] Identify synonyms, ambiguous expressions, and terms that mean different things to different people.
- [ ] Distinguish concepts that sound similar but represent different business meanings.
- [ ] Document definitions, examples, counterexamples, and relevant business rules.
- [ ] Establish the context in which each term is valid.
- [ ] Challenge the language by exploring exceptional and alternative scenarios.
- [ ] Use the language consistently in conversations, documentation, models, code, and tests.
- [ ] Review terminology whenever business understanding or the domain model changes.
- [ ] Validate important definitions with domain experts.
- [ ] Avoid forcing identical terminology across Bounded Contexts when the underlying concepts differ.

**A useful question:** If a domain expert reads a class, method, test, or event name, would they recognize the business meaning without needing a technical translation?

### Event Storming Checklist

Event Storming is a collaborative modeling workshop for exploring business processes through meaningful events. The following checklist is particularly useful for preparing and facilitating a session.

**Before the session**

- [ ] Invite the right participants, including domain experts, developers, testers, product people, and other relevant stakeholders.
- [ ] Include people who understand the actual work, not only those who know the existing software.
- [ ] Define the scope and purpose of the workshop.
- [ ] Choose an appropriate Event Storming format, such as Big Picture, Process Modeling, or Software Design.
- [ ] Prepare a sufficiently large modeling space, physical or digital.
- [ ] Agree on a simple visual notation so participants understand the meaning of each type of sticky note.

**During the session**

- [ ] Start with events that describe things that have happened in the business.
- [ ] Write Domain Events in the past tense.
- [ ] Place events on a timeline and explore their possible order.
- [ ] Encourage participants to contribute independently before immediately debating solutions.
- [ ] Identify commands that trigger or request business actions.
- [ ] Identify actors and external systems involved in the process.
- [ ] Explore policies, business rules, and reactions to events.
- [ ] Identify important decisions, questions, and alternative paths.
- [ ] Discuss exceptional scenarios, failures, and rejected actions.
- [ ] Identify possible Aggregates and other domain concepts as hypotheses, not final design decisions.
- [ ] Explore relevant read models and information requirements when appropriate.
- [ ] Look for language inconsistencies and competing interpretations.
- [ ] Mark hotspots, disagreements, and unanswered questions.
- [ ] Explore possible Bounded Context boundaries when the workshop scope supports it.

**After the session**

- [ ] Capture the resulting model and preserve the important relationships.
- [ ] Document discoveries, assumptions, disagreements, and open questions.
- [ ] Validate important findings with domain experts.
- [ ] Follow up on unresolved business rules and terminology.
- [ ] Translate useful discoveries into the domain model, documentation, and implementation where appropriate.
- [ ] Revisit the model as understanding evolves.

The goal is not to produce a beautiful board. It is to develop a better understanding of how the business works.

### Context Mapping Checklist

A Context Map describes the relationships between Bounded Contexts and the way their models interact.

- [ ] Identify the Bounded Contexts relevant to the system.
- [ ] Clarify each context's purpose, responsibility, and ownership.
- [ ] Identify upstream and downstream relationships.
- [ ] Understand which context influences another context's model or behavior.
- [ ] Identify the integration needs between contexts.
- [ ] Select appropriate relationship patterns, such as Shared Kernel, Customer-Supplier, Conformist, or Separate Ways.
- [ ] Consider integration approaches such as Open Host Service and Published Language.
- [ ] Determine whether an Anti-Corruption Layer is needed to protect a context's model.
- [ ] Document the contracts, protocols, and data exchanged between contexts.
- [ ] Record the rationale behind important relationship and integration decisions.
- [ ] Identify organizational dependencies and ownership responsibilities.
- [ ] Highlight fragile relationships, unclear boundaries, and integration risks.
- [ ] Validate the map with the teams responsible for the participating contexts.
- [ ] Review the map when business responsibilities or integration needs change.

Remember that a Context Map is not merely a diagram of services or databases. It describes relationships between models and the teams or systems responsible for them.

### Aggregate Design Checklist

An Aggregate is a consistency boundary that protects business invariants. Its design should begin with the rules that must remain true, not with the structure of existing database tables or object relationships.

**Understand the business rules**

- [ ] Identify the business invariants that must be protected.
- [ ] Determine which operations can violate those invariants.
- [ ] Clarify when consistency must be immediate and when eventual consistency is acceptable.
- [ ] Explore concurrent operations and conflicting business actions.

**Define the boundary**

- [ ] Identify a suitable Aggregate Root.
- [ ] Determine which Entities and Value Objects belong inside the Aggregate.
- [ ] Define what is inside and outside the consistency boundary.
- [ ] Ensure that external code cannot bypass the Aggregate Root to modify its internal state.
- [ ] Reference other Aggregates through their identities rather than maintaining large object graphs.

**Design behavior**

- [ ] Express business operations as meaningful methods on the Aggregate Root.
- [ ] Ensure operations protect the Aggregate's invariants.
- [ ] Define the conditions under which operations are accepted or rejected.
- [ ] Identify meaningful Domain Events produced by state transitions.
- [ ] Avoid exposing setters or operations that allow invalid states.

**Evaluate the design**

- [ ] Keep Aggregates focused and reasonably small.
- [ ] Avoid grouping objects merely because they are related or stored together.
- [ ] Consider loading, persistence, locking, contention, and concurrency implications.
- [ ] Evaluate whether a business rule can be enforced within one Aggregate.
- [ ] Use explicit coordination between Aggregates when a business process spans multiple consistency boundaries.
- [ ] Validate the design against realistic business scenarios and concurrent operations.

A useful design question is: **What must be true immediately after this business operation completes, and which boundary is responsible for making it true?**

## Templates for Key Artifacts

Templates provide a starting structure for capturing important modeling decisions. They should remain lightweight enough to support collaboration rather than become documentation that nobody maintains.

### Bounded Context Canvas

Use this canvas to describe the purpose, boundaries, language, and relationships of a Bounded Context.

| Section | Questions to explore |
|---|---|
| Context Name | What do we call this context? |
| Purpose and Mission | Why does this context exist? What business capability does it support? |
| Responsibilities | What decisions and responsibilities belong here? |
| Domain Experts | Who understands and validates this part of the business? |
| Users and Stakeholders | Who interacts with or depends on this context? |
| Ubiquitous Language | What are its important terms, definitions, and distinctions? |
| In Scope | What concepts, rules, and processes belong here? |
| Out of Scope | What explicitly belongs elsewhere? |
| Key Domain Concepts | What are the important concepts in its model? |
| Relationships | Which other contexts does it interact with? |
| Context Map Patterns | What relationships and integration patterns are used, and why? |
| Key Events | What meaningful business events does it publish or consume? |
| Key Aggregates | Which consistency boundaries protect its important invariants? |
| Ownership | Which team or group is responsible for the context? |
| Technology and Architecture | What implementation and deployment decisions support it? |
| Open Questions | What assumptions, uncertainties, and decisions remain unresolved? |

**When to use it:** During strategic design, when defining or revisiting Bounded Context boundaries, and when discussing ownership and integration with other teams.

### Aggregate Canvas

Use this canvas to reason about an Aggregate before committing to its implementation.

| Section | Questions to explore |
|---|---|
| Aggregate Name | What is the business concept represented by this Aggregate? |
| Purpose | What business responsibility does it own? |
| Aggregate Root | Which Entity controls access to the Aggregate? |
| Entities | Which objects have identity within this boundary? |
| Value Objects | Which concepts are represented by their values? |
| Invariants | What business rules must always hold? |
| Operations | What meaningful business actions can be performed? |
| Preconditions | Under what conditions can each operation succeed? |
| State Transitions | How can the Aggregate change over its lifecycle? |
| Domain Events | What meaningful facts can it produce? |
| Other Aggregates | Which other Aggregates does it reference? |
| Consistency | Which changes must be atomic? |
| Concurrency | What happens when operations compete to modify the same Aggregate? |
| Persistence | What persistence and concurrency strategy supports the boundary? |
| Open Questions | Which design decisions still need validation? |

**When to use it:** During tactical modeling, particularly when an Aggregate has complex invariants, lifecycle rules, or concurrency requirements.

### Context Map Diagram

A Context Map visualizes the relationships between Bounded Contexts.

**Suggested format:**

- Represent each Bounded Context as a clearly named box.
- Show relationships using directional lines where appropriate.
- Label relationships with the relevant Context Mapping pattern.
- Identify upstream and downstream responsibilities.
- Show important integration contracts or Published Languages.
- Highlight Anti-Corruption Layers and other translation boundaries.
- Include team ownership when it helps explain organizational dependencies.

**Tools:**

- [diagrams.net](https://www.diagrams.net/)
- [Lucidchart](https://www.lucidchart.com/)
- [PlantUML](https://plantuml.com/)
- [Miro](https://miro.com/)
- [Mural](https://www.mural.co/)

The diagram should explain the relationships and dependencies between models, not simply show which services communicate with each other.

### Event Storming Output

Event Storming does not have one mandatory output format. The resulting board depends on the workshop's purpose and scope.

For a typical process-oriented session, the model may include:

| Element | Common notation | Meaning |
|---|---|---|
| Domain Event | Orange sticky note | Something meaningful that happened |
| Command | Blue sticky note | An intention to perform an action |
| Actor | Yellow sticky note | A person or role initiating an action |
| Aggregate | Yellow or other agreed notation | A possible consistency boundary |
| Policy | Lilac sticky note | A rule or reaction triggered by an event |
| Read Model | Green sticky note | Information needed to make a decision |
| External System | Pink sticky note | A system outside the modeled boundary |
| Hotspot | Red sticky note | A question, disagreement, or area of uncertainty |

Colors are conventions, not the essence of Event Storming. Agree on the notation before starting, and adapt it when necessary.

**A useful output should preserve:**

- The sequence and relationships of business events.
- Commands, actors, policies, and external interactions.
- Important decisions and business rules.
- Alternative paths and exceptional scenarios.
- Questions, disagreements, and assumptions requiring further investigation.
- Potential modeling boundaries and concepts that need deeper exploration.

### Ubiquitous Language Canvas

Use this template to capture and refine important terms in a Bounded Context.

| Section | Description |
|---|---|
| Term | The word or expression used by the domain |
| Context | Where and by whom the term is used |
| Definition | What the term means in this context |
| Examples | Concrete scenarios in which the term applies |
| Counterexamples | Scenarios that may look similar but do not qualify |
| Related Concepts | Other concepts connected to the term |
| Actions | What can happen to or through this concept? |
| Lifecycle | What states or transitions does it have? |
| Business Rules | What constraints govern its behavior? |
| Synonyms | What alternative expressions are used? |
| Ambiguities | What different interpretations exist? |
| Model Expression | How is the concept represented in the domain model and code? |
| Open Questions | What still needs clarification? |

Do not attempt to fill every field for every term. Capture the information that helps the team understand and use the language consistently.

## Practical Guidance

A few principles help keep these tools useful:

1. **Start with the problem, not the template.** Choose a checklist or canvas because it helps answer a real question.
2. **Make assumptions visible.** Distinguish confirmed business knowledge from hypotheses and unresolved questions.
3. **Involve domain experts.** A model created only by developers can be technically consistent and still misunderstand the business.
4. **Keep the language connected to the model.** When a concept changes, revisit its definition, examples, code, and tests.
5. **Prefer useful artifacts over complete documentation.** A small, accurate Context Map is more valuable than a large diagram nobody understands.
6. **Revisit decisions.** Domain knowledge, business capabilities, and organizational relationships evolve.
7. **Treat checklists as prompts, not gates.** Not every project needs every activity, pattern, or artifact.

The purpose of these resources is not to make DDD more bureaucratic. It is to make important conversations, assumptions, and design decisions easier to explore and communicate.

**Use the tools to improve your understanding of the domain, not to replace it.**