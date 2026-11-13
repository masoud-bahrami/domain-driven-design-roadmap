# Practices

Domain-Driven Design is not a process that can be followed through a fixed sequence of workshops or modeling techniques. It is an ongoing practice of understanding a domain, developing a shared language, refining models, and improving software as business knowledge evolves.

The practices in this section help teams put DDD principles into action.

Some help discover how a business works. Others clarify business rules, explore relationships between Bounded Contexts, or improve the design of the domain model.

They are complementary techniques, not mandatory steps in a prescribed methodology. Choose them according to the questions, uncertainties, and design challenges the team needs to address.

## Domain Exploration and Discovery

These practices help teams understand the business, discover its language, and make important behavior visible.

- **[Event Storming](event-storming.md)**

  A collaborative workshop for exploring a business domain through Domain Events and the commands, actors, policies, and decisions surrounding them. Useful for discovering business processes, dependencies, exceptions, and potential boundaries.

- **[Domain Storytelling](domain-storytelling.md)**

  A visual technique for exploring how people perform their work. Participants describe concrete stories using actors, work objects, and activities, helping teams understand real workflows, responsibilities, and domain terminology.

- **[Example Mapping](example-mapping.md)**

  A collaborative technique for clarifying user stories and business rules through concrete examples, rules, questions, and acceptance criteria. Particularly useful for exposing ambiguity and edge cases before implementation.

## Strategic Design and Context Relationships

These practices help teams understand model boundaries and the relationships between different parts of a domain.

- **[Context Mapping](context-mapping.md)**

  A technique for making relationships between Bounded Contexts explicit. It helps identify upstream and downstream dependencies, model ownership, integration contracts, translation responsibilities, and the patterns used to manage collaboration between contexts.

## Modeling Tools and Design Practices

These resources support the creation, evaluation, and continuous improvement of domain models.

- **[Canvases](canvases.md)**

  Practical templates for capturing and discussing domain knowledge, including Bounded Contexts, Aggregates, Context Maps, Ubiquitous Language, and Event Storming outcomes. Canvases provide structure for conversations without replacing the discovery and reasoning behind them.

- **[Supple Design](supple-design.md)**

  Design principles and patterns from Eric Evans that make domain models more expressive, understandable, and adaptable. Covers intention-revealing interfaces, side-effect-free functions, assertions, conceptual contours, standalone classes, closure of operations, declarative design, and domain-specific languages.

- **[Refactoring Toward Deeper Insight](breakthrough-refactoring.md)**

  Explores how new domain understanding can lead to fundamental improvements in a model. Rather than merely cleaning up code, breakthrough refactoring changes the model to express previously undiscovered concepts, distinctions, and business rules more accurately.

## Choosing the Right Practice

Different situations call for different techniques. The following guide can help identify a useful starting point.

| Situation | Useful practice |
|---|---|
| The team does not understand the overall business process | Event Storming |
| You need to understand how people actually perform their work | Domain Storytelling |
| A user story or business rule is ambiguous | Example Mapping |
| Relationships and dependencies between Bounded Contexts are unclear | Context Mapping |
| You need a structured way to capture modeling discoveries | Canvases |
| The domain model is difficult to understand or change | Supple Design |
| New domain knowledge challenges the existing model | Refactoring Toward Deeper Insight |

These techniques can also be combined.

For example, a team might use Domain Storytelling to understand how a dispatcher handles a shipment, Event Storming to explore the events and decisions surrounding dispatch, and Example Mapping to clarify the rules governing dispatch authorization.

The discoveries can then inform the Ubiquitous Language, Aggregate design, and Context Map. Supple Design helps ensure that the resulting implementation expresses those discoveries clearly, while refactoring allows the model to evolve as understanding deepens.

## An Important Distinction

Not every practice in this section serves the same purpose.

- **Discovery practices** help teams learn about the business.
- **Clarification practices** turn broad statements into explicit rules and examples.
- **Strategic practices** help establish and manage model boundaries and relationships.
- **Design practices** help express domain knowledge in software and keep the model adaptable.
- **Refactoring practices** help the model evolve when new insights challenge existing assumptions.

The common thread is the continuous development of domain understanding.

None of these techniques should become a ceremony performed simply because DDD recommends it. Their value comes from the insights they produce and the improvements they enable.

**Use practices to improve understanding, not to demonstrate that you are practicing DDD.**