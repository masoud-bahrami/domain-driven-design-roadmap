# Learning Roadmap

The roadmap is organized around the work a team does when applying DDD: understand the problem, discover the domain, establish boundaries, model behavior, implement the model, and evolve it. The sequence is a guide, not a rigid process.

## The path at a glance

| Stage | Focus | Typical outcome |
|---|---|---|
| 0 | Preparation by role | Shared baseline across engineering, product, domain, and design |
| 1 | DDD foundations | Explain when DDD helps and establish a first shared vocabulary |
| 2 | Domain exploration | Make business processes, rules, events, and uncertainty visible |
| 3 | Strategic design | Identify subdomains, Bounded Contexts, and relationships |
| 4 | Tactical design | Model behavior, identity, invariants, and consistency boundaries |
| 5 | Implementation and testing | Keep the model expressive and protect it with tests |
| 6 | Architecture and integration | Choose deployment and integration structures that respect boundaries |
| 7 | Evolution and modernization | Improve models safely, including in legacy systems |
| 8 | Advanced and adjacent topics | Explore specialized techniques when the problem justifies them |

## Stage 0. Preparation by role

DDD is collaborative. Developers benefit from programming fundamentals, object-oriented or functional design, testing, refactoring, databases, and version control. Product managers, analysts, and domain experts contribute business knowledge, processes, rules, priorities, examples, and validation. UX/UI designers help ensure that user journeys and interface language reflect the concepts and distinctions of the domain.

This is not a prerequisite checklist that every participant must complete. The goal is to understand how different kinds of expertise contribute to a shared model. See [role preparation](../concepts/role-preparation.md).

## Stage 1. Understand DDD

Learn what a domain is, why complexity matters, and how DDD connects language, models, and software design. Understand Ubiquitous Language, Subdomains, Bounded Contexts, Entities, Value Objects, Aggregates, and Domain Events at a high level.

**Practice:** Choose a familiar business area and write down its important terms, decisions, rules, and unanswered questions.

**You are ready to continue when:** you can explain why the same word may mean different things in different contexts, and why DDD is an investment for complex domains rather than a default architecture.

[Read the foundations](01-foundations.md)

## Stage 2. Explore the domain collaboratively

Work with domain experts to discover how the business actually operates. Use Event Storming, Domain Storytelling, Example Mapping, process flows, and concrete scenarios. Distinguish observed facts from assumptions and unresolved questions.

**Practice:** Facilitate a small workshop around one business process. Capture events, commands, actors, policies, external systems, and questions.

**You are ready to continue when:** participants can explain the process using a shared vocabulary and can point to the important rules and uncertainties.

[Read domain exploration](02-domain-exploration.md)

## Stage 3. Design the strategic boundaries

Classify Subdomains as Core, Supporting, or Generic. Discover Bounded Contexts and map their relationships. Learn the integration patterns and understand the organizational consequences of context boundaries.

**Practice:** Draw a Context Map for a system you know. Record why each boundary exists and how each context depends on the others.

**You are ready to continue when:** the team can explain where a model is valid, who owns it, and how it communicates with neighboring models.

[Read strategic design](03-strategic-design.md)

## Stage 4. Design the tactical model

Learn Entities, Value Objects, Aggregates, Aggregate Roots, Domain Services, Domain Events, Repositories, and Factories. Focus on behavior, invariants, identity, and consistency rather than simply mapping database tables to objects.

**Practice:** Model a business operation and identify the rules that must hold when it completes. Design the smallest useful Aggregate boundary that can enforce those rules.

**You are ready to continue when:** the model communicates business intent and protects its invariants without relying on callers to remember every rule.

[Read tactical design](04-tactical-design.md)

## Stage 5. Implement and test the model

Connect the domain model to application workflows, persistence, messaging, and user-facing APIs without allowing infrastructure concerns to dictate domain meaning. Understand application services, command/query separation, transactions, test strategies, and the role of OOP and functional programming.

**Practice:** Implement one use case with a behavior-rich domain model and tests that express business rules in domain language.

**You are ready to continue when:** the model can be changed with confidence and the tests explain important business behavior.

[Read implementation and testing](05-implementation-and-testing.md)

## Stage 6. Choose architecture and integration approaches

Explore Layered, Hexagonal, Clean, and modular-monolith architectures, as well as microservices and event-driven integration. Learn distributed transactions, Sagas, eventual consistency, and the costs of crossing process boundaries.

**Practice:** Map one Bounded Context to modules, deployment units, and integration contracts. Explain why those boundaries are appropriate.

**You are ready to continue when:** deployment boundaries follow real ownership, change, and operational needs rather than fashion.

[Read architecture and integration](06-architecture-and-integration.md)

## Stage 7. Evolve models and modernize existing systems

Treat the model as something that evolves with new insight. Learn Supple Design, refactoring toward deeper insight, characterization tests, Anti-Corruption Layers, Bubble Contexts, Autonomous Bubbles, and gradual modernization strategies.

**Practice:** Identify one confusing concept in an existing model. Work with a domain expert to clarify it, write tests around current behavior, then refactor in small steps.

**You are ready when:** you can improve the model incrementally without assuming a risky rewrite is necessary.

[Read evolution and refactoring](07-evolution-and-refactoring.md)

## Stage 8. Advanced and specialized topics

Explore Event Sourcing, advanced CQRS, Functional DDD, Domain-Specific Languages, Team Topologies, Wardley Mapping, Data Mesh, reactive systems, and serverless architectures when they address a concrete need. These are not mandatory DDD stages.

**Practice:** For each technique, write down the problem it solves, the complexity it introduces, and a simpler alternative.

[Read advanced topics](08-advanced-topics.md)

## Measuring progress

Measure whether the team understands the domain better, whether important decisions are clearer, whether the model reflects business rules, and whether changes become safer and more focused. Useful indicators include fewer misunderstandings, faster feedback from domain experts, clearer ownership, reduced rework, and improved business outcomes. Do not treat the number of Aggregates, services, events, or diagrams as a measure of DDD success.

[Measurement guide](../concepts/measuring-success.md)
