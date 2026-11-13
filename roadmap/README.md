# Learning Roadmap

Domain-Driven Design is best learned through the work of understanding and designing software around a business domain. It is not a collection of patterns to memorize, nor a sequence of architectural decisions to apply mechanically.

A team begins by understanding the problem it is trying to solve. It explores the domain with the people who know it, discovers meaningful boundaries, develops models, expresses those models in software, and continuously revises them as its understanding improves.

This roadmap follows that journey.

Each stage introduces a set of ideas, explains why they matter, and suggests practical work through which you can develop a deeper understanding. You do not need to master every topic before moving forward. In real projects, these activities overlap, repeat, and sometimes send a team back to earlier decisions.

The stages provide direction, not a prescribed process.

## The path at a glance

| Stage | Focus | What you should be able to do |
|---|---|---|
| 0 | Preparation by role | Understand how your expertise contributes to discovering and designing the domain |
| 1 | DDD foundations | Explain the purpose of DDD and understand its central concepts |
| 2 | Domain exploration | Discover business processes, rules, events, decisions, and uncertainties collaboratively |
| 3 | Strategic design | Identify subdomains, establish Bounded Contexts, and understand their relationships |
| 4 | Tactical design | Design models that express behavior and protect business invariants |
| 5 | Implementation and testing | Translate domain models into maintainable software and verify their behavior |
| 6 | Architecture and integration | Choose architectural and integration structures that respect domain boundaries |
| 7 | Evolution and modernization | Improve existing models and systems without losing valuable business knowledge |
| 8 | Advanced and specialized topics | Evaluate advanced techniques according to the problems they solve |

---

## Stage 0. Prepare for the journey

### DDD is collaborative work

Domain-Driven Design is not exclusively a software engineering discipline. It brings together people with different kinds of knowledge to develop a shared understanding of a business domain.

The preparation required depends on your role. A developer, domain expert, product manager, architect, and UX designer do not need identical skills or the same theoretical background. What matters is understanding how their perspectives contribute to the discovery and evolution of the model.

**For software developers**

A useful foundation includes:

- Programming fundamentals and the ability to work with complex codebases.
- Object-oriented design and, where relevant, functional programming.
- Encapsulation, abstraction, modularity, and separation of concerns.
- Automated testing, refactoring, and version control.
- Basic knowledge of databases, transactions, APIs, and distributed systems.

These skills help developers turn discovered domain knowledge into software without confusing the domain model with its technical implementation.

**For architects and technical leaders**

Useful preparation includes:

- Understanding modularity, dependencies, and architectural boundaries.
- Experience with application integration and data ownership.
- Familiarity with deployment and operational trade-offs.
- The ability to reason about coupling, cohesion, and the cost of distributed systems.

Architects should learn to recognize when a technical boundary reflects a meaningful domain boundary, and when it is merely an infrastructure decision.

**For product managers, analysts, and domain experts**

The most valuable contribution is knowledge of the business itself:

- Business processes and their variations.
- Rules, policies, decisions, and exceptions.
- The meaning of business terminology.
- The people, organizations, and systems involved.
- The constraints and trade-offs behind business decisions.

Domain experts do not need to learn programming to contribute meaningfully to DDD. Their knowledge is central to discovering a useful model.

**For UX/UI designers**

User journeys, interactions, terminology, and mental models are important sources of domain insight. Designers can help reveal distinctions that are hidden by existing interfaces and identify where the language of the software does not match the language of its users.

### The goal of preparation

The objective is not to turn every participant into a DDD specialist. It is to establish a shared understanding of how different kinds of expertise contribute to the same problem.

**Practical exercise:** Choose a business process familiar to your team. Ask each participant to describe it from their own perspective. Compare the terminology, assumptions, and questions that emerge.

**You are ready to continue when:** You understand that domain knowledge is distributed across people and that building a useful model requires collaboration.

[Explore preparation by role](../concepts/role-preparation.md)

---

## Stage 1. Understand the foundations of DDD

### Learn what you are trying to achieve before learning the patterns

Before designing Entities, Aggregates, or Repositories, understand the problem that Domain-Driven Design addresses.

Software systems often become difficult not because their technology is complicated, but because the business rules they represent are complicated, interconnected, and poorly understood.

DDD addresses this challenge by putting the domain and its knowledge at the center of software design.

Start with these fundamental ideas:

- **Domain:** The area of knowledge and activity that the software is intended to support.
- **Domain Model:** A structured representation of selected aspects of the domain, useful for reasoning about and implementing business behavior.
- **Ubiquitous Language:** A shared language developed by the people working on a model and used consistently in conversations, documentation, and code.
- **Subdomain:** A distinct part of the business domain with its own concerns and responsibilities.
- **Bounded Context:** A boundary within which a particular model and its language have a defined meaning.
- **Entity:** An object distinguished by its identity and continuity.
- **Value Object:** An object defined by its values and characteristics rather than identity.
- **Aggregate:** A consistency boundary around related domain objects and rules.
- **Domain Event:** A representation of something meaningful that has happened in the domain.

Do not rush to memorize definitions. These concepts become meaningful when you understand the problems they help address and the trade-offs they introduce.

### A useful starting question

Imagine a system that handles orders, payments, inventory, and deliveries.

A customer may place an order, but the meaning of an order in the sales process may differ from its meaning in fulfillment. A product in a catalog may not be the same concept as an item tracked in a warehouse. A payment authorization is not necessarily the same thing as a completed payment.

These differences are not simply naming problems. They may reflect different business responsibilities, rules, and models.

DDD gives a team ways to discover and represent these distinctions deliberately.

**Practical exercise:** Choose a familiar business area. Write down its important terms, decisions, rules, and unanswered questions. For each term, ask whether everyone uses it with the same meaning.

**You are ready to continue when:**

- You can explain why understanding the domain is a software design concern.
- You recognize that a model is a selective representation, not a complete copy of reality.
- You can explain why the same word may have different meanings in different contexts.
- You understand why DDD is particularly valuable when business complexity justifies the investment.

[Read DDD foundations](01-foundations.md)

---

## Stage 2. Explore the domain collaboratively

### Discover the business before deciding how to model it

A model built on assumptions will eventually encode those assumptions into software.

Domain exploration is the work of discovering how the business actually operates: what happens, who makes decisions, which rules apply, what can go wrong, and why different participants understand the same process differently.

This work requires conversations with domain experts and other stakeholders. Documentation and existing software can provide useful evidence, but they should not automatically be treated as the truth.

Learn techniques that make domain knowledge visible and discussable:

- **Event Storming:** Explore business processes through domain events, commands, policies, actors, and other relevant elements.
- **Domain Storytelling:** Describe business activities as stories involving people, activities, work objects, and their relationships.
- **Example Mapping:** Explore a specific requirement through rules, examples, questions, and acceptance criteria.
- **Process and workflow analysis:** Understand the sequence of activities, decisions, and interactions in a business process.
- **Scenario exploration:** Examine normal cases, exceptional cases, and variations to uncover hidden rules.

The purpose of these techniques is not to produce impressive diagrams. It is to improve the quality of conversations and expose knowledge that would otherwise remain implicit.

### Learn to distinguish knowledge from assumptions

During exploration, a team will encounter several kinds of information:

- Facts that participants can explain and agree upon.
- Rules that govern business behavior.
- Exceptions that reveal important distinctions.
- Assumptions that have not yet been validated.
- Questions that require further investigation.
- Disagreements that may indicate different perspectives or genuinely different models.

Do not resolve every disagreement by choosing one definition. Sometimes the disagreement is evidence that a concept has different meanings in different parts of the business.

**Practical exercise:** Facilitate a small workshop around one meaningful business process. Capture the events, commands, actors, policies, external systems, rules, and open questions. Use concrete examples to validate the conversation.

**You are ready to continue when:**

- Participants can explain the process using a shared vocabulary.
- Important business rules and exceptions are visible.
- Uncertainties and assumptions are explicitly identified.
- The team has enough domain knowledge to begin discussing meaningful boundaries.

[Read domain exploration](02-domain-exploration.md)

---

## Stage 3. Design the strategic boundaries

### Discover where models begin and end

As a team explores a business domain, it becomes clear that not every part of the business should be represented by one universal model.

Different areas may have different responsibilities, terminology, rules, priorities, and rates of change. Trying to force all of them into one model can create ambiguity and unnecessary coupling.

Strategic Design provides concepts and techniques for understanding these differences and organizing the system around them.

Start with the following:

**Subdomains**

Understand how to distinguish:

- **Core Subdomains:** Areas that provide important competitive advantage or business differentiation.
- **Supporting Subdomains:** Areas that support the business but are not its primary source of differentiation.
- **Generic Subdomains:** Areas where specialized business modeling may provide little advantage compared with an established solution.

These classifications help guide investment and design effort. They should reflect the actual business strategy rather than arbitrary technical preferences.

**Bounded Contexts**

Learn how to establish boundaries around models with consistent meanings and responsibilities.

A Bounded Context is not automatically a service, database, team, or deployment unit. It is first and foremost a boundary within which a particular model is meaningful and internally consistent.

**Context Mapping**

Explore the relationships between Bounded Contexts and the ways they collaborate or depend on one another.

Learn the patterns and relationships commonly used in Context Mapping, including:

- Partnership.
- Shared Kernel.
- Customer-Supplier.
- Conformist.
- Anti-Corruption Layer.
- Open Host Service.
- Published Language.
- Separate Ways.

Understand not only the names of these patterns but also the organizational, technical, and coordination consequences of choosing them.

### Boundaries are decisions, not discoveries carved in stone

A boundary is a hypothesis about where a model makes sense. It should be informed by domain knowledge, ownership, language, dependencies, and change patterns.

Some boundaries will become clearer through implementation. Others will need to change as the business evolves.

**Practical exercise:** Draw a Context Map for a system you know. Identify its Bounded Contexts, their responsibilities, and their relationships. For each boundary, explain why it exists and what would become difficult if two contexts shared one model.

**You are ready to continue when:**

- You can distinguish a Subdomain from a Bounded Context.
- You can explain why a model is valid within one context but not necessarily another.
- You understand how contexts depend on and communicate with one another.
- The team can discuss model ownership and integration responsibilities explicitly.

[Read strategic design](03-strategic-design.md)

---

## Stage 4. Design the tactical model

### Turn domain knowledge into a model that expresses behavior

Strategic Design helps establish the boundaries within which models operate. Tactical Design focuses on the building blocks used to express those models.

The challenge is not to create an object for every business noun. It is to represent the concepts, decisions, and rules that matter to the domain.

Explore the central tactical patterns:

**Entities and Value Objects**

Understand the distinction between identity and value. Learn when an object needs continuity across changes and when its meaning is completely described by its attributes.

**Aggregates and Aggregate Roots**

Learn how to establish consistency boundaries around related objects and protect business invariants.

An Aggregate is not simply a convenient way to group database tables or an object graph that contains everything related to a business concept. Its boundary determines which rules must be enforced consistently when an operation completes.

**Domain Services**

Use Domain Services when a meaningful domain operation does not naturally belong to one Entity or Value Object.

**Domain Events**

Represent meaningful occurrences in the domain. Understand their role in expressing business facts and communicating changes without confusing them with ordinary application messages.

**Repositories**

Learn how repositories provide access to domain objects while keeping persistence concerns from dominating the domain model.

**Factories**

Use factories when the creation of a domain object or Aggregate requires meaningful logic, validation, or coordination.

### Model rules, not database structures

A common mistake is to begin with existing tables and translate each table into an Entity. This approach may preserve the database structure while missing the business meaning entirely.

Instead, start with behavior:

- What operation is the business performing?
- What must be true before it can happen?
- What must remain true afterward?
- Which decisions belong to the model?
- Which changes must be consistent?
- Which concepts have independent identity?
- Which objects are merely values?

A useful model makes important rules difficult to violate accidentally.

**Practical exercise:** Select one business operation. Describe its behavior using a concrete scenario. Identify its invariants, relevant Entities and Value Objects, and the smallest Aggregate boundary capable of protecting those invariants.

**You are ready to continue when:**

- The model expresses meaningful business behavior rather than merely storing data.
- Important invariants are protected by the model.
- Identity, value, and consistency boundaries are explicit.
- You can explain why each tactical pattern exists in your design.

[Read tactical design](04-tactical-design.md)

---

## Stage 5. Implement and test the model

### Keep the domain meaningful in the actual software

A model can be beautifully described in workshops and diagrams yet disappear when implementation begins.

The implementation should preserve the meaning discovered during domain exploration. Technical concerns are necessary, but they should not become the primary language through which the business is expressed.

Explore how to connect the domain model to application workflows, persistence, messaging, and user-facing interfaces.

Important topics include:

- Application Services and use-case orchestration.
- Domain logic and the responsibilities of the domain layer.
- Persistence and Repository implementations.
- Transactions and consistency.
- Commands, queries, and CQRS fundamentals.
- Domain Events and integration messages.
- Unit, integration, and acceptance testing.
- Object-oriented and functional approaches to domain modeling.
- Dependency management and infrastructure isolation.

### Testing as a way to protect understanding

Tests should do more than verify that methods return expected values. Well-designed tests describe meaningful business behavior and help the team detect when implementation changes violate important rules.

Use concrete examples and domain terminology. Make important scenarios readable to both developers and people familiar with the business.

Do not confuse a rich domain model with a particular programming style. Some domains benefit from object-oriented models with explicit behavior. Others may be expressed effectively using functional techniques. The objective is to preserve domain meaning and make rules understandable, testable, and maintainable.

**Practical exercise:** Implement one use case using a behavior-rich domain model. Write tests that describe its business rules, including at least one important exceptional scenario. Keep infrastructure details outside the central business logic where practical.

**You are ready to continue when:**

- The implementation communicates the intent of the domain.
- Application workflows do not contain business rules that belong in the model.
- Important business behavior is protected by meaningful tests.
- You can change the model without requiring every caller to understand its internal rules.

[Read implementation and testing](05-implementation-and-testing.md)

---

## Stage 6. Choose architecture and integration approaches

### Let domain boundaries inform technical structure

Once the team understands the domain and its models, it must decide how to organize the software.

This is where architectural approaches become relevant. Architecture should help preserve the boundaries, responsibilities, and dependencies established through domain understanding.

Explore:

**Architectural styles and patterns**

- Layered Architecture.
- Hexagonal Architecture and Ports and Adapters.
- Clean Architecture.
- Modular Monoliths.
- Microservices.

Understand their principles, constraints, and trade-offs rather than treating them as competing recipes.

**Integration and distributed systems**

- Synchronous and asynchronous communication.
- Messaging and Event-Driven Architecture.
- Integration events and Published Languages.
- Distributed transactions and their limitations.
- Sagas and process coordination.
- Eventual consistency.
- Resilience, failure handling, and observability.

### Distributed architecture is not the definition of DDD

A Bounded Context does not have to become a microservice. A domain model does not require Event Sourcing. A Context Map does not prescribe a deployment topology.

A modular monolith may be a better choice than microservices when independent deployment, operational autonomy, or distributed ownership do not justify the additional complexity.

The important question is whether the architecture supports the domain boundaries and the way the organization needs to work.

**Practical exercise:** Choose one Bounded Context and map it to software modules, deployment units, data ownership, and integration contracts. Explain why the proposed structure fits its responsibilities and operational needs. Compare it with at least one simpler alternative.

**You are ready to continue when:**

- You can distinguish domain boundaries from deployment boundaries.
- You understand the costs of communication across process boundaries.
- Integration contracts preserve the meaning of the models involved.
- Architectural choices are supported by concrete business and operational needs.

[Read architecture and integration](06-architecture-and-integration.md)

---

## Stage 7. Evolve models and modernize existing systems

### A domain model is never finished

Understanding a business is an ongoing activity. New requirements, operational experience, changing strategies, and conversations with domain experts reveal insights that were not available when the original model was created.

DDD should therefore be understood as a continuous process of discovery and refinement, not a one-time modeling exercise.

Learn how to improve models and existing systems through:

- **Supple Design:** Develop models that are easier to understand, reason about, and change.
- **Refactoring toward deeper insight:** Use new domain knowledge to improve the structure and meaning of the model.
- **Characterization Testing:** Capture existing behavior before changing systems whose rules are not fully understood.
- **Anti-Corruption Layers:** Protect a model from concepts and structures that do not belong to it.
- **Bubble Contexts and Autonomous Bubbles:** Create boundaries that allow a team to develop a clearer model alongside an existing system.
- **Strangler Fig approaches:** Gradually replace parts of a legacy system while keeping the existing system operational.
- **Incremental modernization:** Improve architecture and domain understanding without assuming that a complete rewrite is necessary.

### Legacy systems contain knowledge, not just technical debt

An old system may have confusing code and outdated architecture, but it may also encode years of business decisions, exceptions, and operational knowledge.

Replacing it without understanding those decisions can be more dangerous than maintaining it.

Modernization should begin by discovering what the system actually does, identifying which behavior must be preserved, and deciding where a better model can be introduced safely.

**Practical exercise:** Identify one confusing concept in an existing model. Work with a domain expert to clarify its meaning. Capture current behavior with tests, introduce the improved model, and refactor in small, verifiable steps.

**You are ready to continue when:**

- You can distinguish accidental complexity from essential business complexity.
- You can improve a model without requiring a risky rewrite.
- Existing behavior is understood and protected during modernization.
- The team treats domain insight as an ongoing source of architectural improvement.

[Read evolution and refactoring](07-evolution-and-refactoring.md)

---

## Stage 8. Explore advanced and specialized topics

### Go deeper when the problem calls for it

DDD has influenced and connected with many other disciplines and technical approaches. Some extend domain modeling; others address architecture, organizational design, data ownership, or distributed computing.

These topics can be valuable, but they are not mandatory stages of DDD adoption.

Explore them when you can identify a concrete problem they help solve.

**Advanced modeling and implementation**

- Functional DDD.
- Domain-Specific Languages.
- Advanced CQRS.
- Event Sourcing.
- Advanced Aggregate design.
- Specification and policy patterns.

**Architecture and distributed systems**

- Advanced Event-Driven Architecture.
- Reactive systems.
- Serverless architectures.
- Distributed workflows and process managers.
- Advanced consistency and messaging strategies.

**Organization, strategy, and data**

- Team Topologies.
- Wardley Mapping.
- Data Mesh.
- Organizational design and Conway's Law.
- Strategic technology and domain investment decisions.

Some of these topics complement DDD, while others are adjacent disciplines with their own principles and goals. Their inclusion in this roadmap does not make them part of the essential DDD method.

### Evaluate techniques by the problems they solve

For every advanced technique, ask:

1. What specific problem does it address?
2. What assumptions does it make?
3. What new complexity does it introduce?
4. What operational and organizational capabilities does it require?
5. Is there a simpler approach that would be sufficient?

**Practical exercise:** Choose one advanced technique. Describe the problem it solves, the conditions under which it is useful, the complexity it introduces, and a simpler alternative. Apply it to a real scenario rather than adopting it because it is popular.

[Read advanced topics](08-advanced-topics.md)

---

## Measuring progress

### Measure better understanding, not more DDD artifacts

The success of DDD cannot be measured by counting Aggregates, Bounded Contexts, Domain Events, microservices, or workshop diagrams.

These are tools and design outcomes. Their existence does not prove that the team understands the business or has built a better system.

Look for evidence of improved understanding and better decisions.

**Domain understanding**

- Can the team explain important business processes and their exceptions?
- Are domain terms understood consistently within their intended contexts?
- Can domain experts validate the model and identify incorrect assumptions?
- Are important uncertainties visible rather than hidden in implementation?

**Model quality**

- Does the model express meaningful business behavior?
- Are important rules and invariants protected?
- Are boundaries understandable and justified?
- Can the model accommodate new insight without becoming increasingly difficult to change?

**Collaboration and ownership**

- Do developers and domain experts have more productive conversations?
- Are responsibilities and model ownership clearer?
- Can teams identify and resolve disagreements about terminology and behavior?
- Are dependencies between contexts understood and managed?

**Software and business outcomes**

- Is there less rework caused by misunderstood requirements?
- Can changes be made with greater confidence?
- Are integration problems and accidental coupling reduced?
- Does the software better support the actual needs of the business?

These indicators are not a universal scoring system. Their relevance depends on the domain, the organization, and the problems the team is trying to address.

[Read the guide to measuring DDD success](../concepts/measuring-success.md)

---

## How to use this roadmap

You can follow the stages sequentially if you are learning DDD for the first time. If you are already working on a system, start with the stage that addresses your current challenge.

A team struggling with inconsistent terminology may benefit from returning to domain exploration. A team with unclear service boundaries may need to revisit strategic design. A team with a large legacy codebase may begin with characterization testing and incremental modernization before attempting a broader redesign.

Use the roadmap as a reference for learning and decision-making, not as a maturity checklist.

The ultimate goal is not to build software that contains every DDD pattern. It is to build software whose design reflects a deeper understanding of the domain, whose language communicates its intent, and whose structure can evolve as that understanding changes.