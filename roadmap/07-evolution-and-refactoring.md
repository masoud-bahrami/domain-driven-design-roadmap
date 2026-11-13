# 07 Evolution, Scaling, and Legacy Systems

DDD is not a one-time modeling exercise. A domain model is an evolving understanding of a business, not a design artifact that we create once and preserve forever.

As businesses change, new requirements emerge, teams grow, responsibilities move, and systems accumulate new capabilities. Models that once represented the business accurately may gradually become misleading. Boundaries that made sense at one point may need to change.

The challenge is not simply to introduce DDD into a system. It is to keep the model useful as the system and organization evolve.

This section explores two important aspects of that challenge:

- **Scaling DDD:** Maintaining a coherent domain model and effective collaboration as more teams and Bounded Contexts are introduced.
- **Applying DDD to legacy systems:** Introducing better models and boundaries into existing software without relying on risky, large-scale rewrites.

## 1. Scaling DDD

Applying DDD within a small team is already challenging. Applying it across a large organization introduces additional problems.

Different teams may work with different parts of the domain, maintain separate models, and depend on capabilities owned by other teams. Without deliberate coordination, these differences can create conflicting assumptions, duplicated work, and integration problems.

Scaling DDD is therefore not just about scaling the architecture. It is also about scaling shared understanding and the organization's ability to make decisions.

### 1.1 Team Organization and Bounded Contexts

A Bounded Context provides a useful starting point for organizing responsibilities and ownership.

When a team owns a particular context, it can develop deep knowledge of the business area, maintain its model, and make decisions about its evolution.

This often leads to the idea of a team per Bounded Context.

However, this is a guideline, not a universal organizational rule. A Bounded Context does not always require a dedicated team. A small team may own several related contexts, while a particularly large or complex context may require multiple collaborating teams.

The important thing is to establish clear ownership and minimize unnecessary coordination.

#### Team Topologies

Team Topologies provides a useful framework for thinking about team organization and interactions. It identifies four fundamental team types:

**Stream-aligned teams**

These teams are aligned with a continuous flow of business value. They own a product, service, or business capability and are responsible for delivering outcomes.

A stream-aligned team may own one Bounded Context or a small group of closely related contexts.

**Enabling teams**

These teams help other teams develop capabilities they do not yet possess. They may provide guidance in areas such as DDD, testing, security, observability, or architectural design.

Their purpose is to help other teams become more capable and autonomous, not to create permanent dependencies.

**Complicated-subsystem teams**

These teams own subsystems that require specialized knowledge or expertise that would be unreasonable to expect every stream-aligned team to maintain.

Examples might include complex optimization algorithms, specialized financial calculations, or sophisticated technical engines.

**Platform teams**

These teams provide internal platforms and capabilities that help other teams deliver software more effectively.

Examples include deployment infrastructure, monitoring platforms, developer tooling, and shared runtime capabilities.

These team types can coexist with DDD. However, Team Topologies and DDD solve different problems: one helps organize teams and their interactions, while the other helps model complex domains and define boundaries.

The two can inform each other, but they should not be treated as interchangeable frameworks.

#### Team Ownership and Architectural Boundaries

When organizing teams around Bounded Contexts, consider the following:

- Teams should understand the domain they own and have the authority to evolve its model.
- Responsibilities should be explicit, including ownership of data, integration contracts, and operational concerns.
- Teams should minimize dependencies that require frequent coordination.
- Cross-context decisions should involve the people responsible for the affected models.
- Shared capabilities should have clear ownership rather than becoming everyone's responsibility and nobody's priority.

A useful principle is that team boundaries should support the flow of business value while respecting the boundaries of the domain model.

For example, a team responsible for Order Management should not need to coordinate with several other teams every time it changes an internal order policy. It should, however, coordinate when a change affects an integration contract or the meaning of information exchanged with another context.

Learning objectives:

- Understand how team organization can support Bounded Context ownership.
- Explain the four fundamental Team Topologies.
- Distinguish domain ownership from organizational reporting structures.
- Identify when one team should own multiple contexts or when a context requires multiple teams.

Further reading:

- [Team Topologies](https://teamtopologies.com/), Matthew Skelton and Manuel Pais.
- [Team Topologies: Key Concepts](https://teamtopologies.com/key-concepts).

### 1.2 Managing Dependencies Between Bounded Contexts and Teams

As the number of Bounded Contexts grows, so does the importance of understanding their relationships.

A Context Map becomes more than a modeling artifact. It can serve as a communication tool for understanding dependencies, ownership, and integration responsibilities across teams.

For each important relationship, teams should understand:

- Which context is upstream and which is downstream?
- Who owns the model and the integration contract?
- What information or capabilities are being exchanged?
- How are changes communicated?
- What happens when the upstream context changes?
- Which team is responsible for resolving integration problems?

#### Communication Patterns

Not every cross-team interaction needs a meeting, but every important dependency needs a reliable way to communicate.

Useful practices include:

- Regular cross-team domain discussions.
- Shared documentation for integration contracts.
- Explicit ownership of APIs and events.
- Architecture decision records for important cross-context decisions.
- Collaborative modeling sessions when business concepts or relationships change.
- Dedicated communication channels for operational and integration issues.

The goal is not to maximize communication. It is to make the necessary communication effective and reduce unnecessary coordination.

#### Contract and Dependency Management

Integration contracts should be treated as products that have owners and consumers.

Useful techniques include:

- API versioning and compatibility policies.
- OpenAPI specifications for HTTP APIs.
- Explicit schemas for integration events.
- Consumer-driven contract testing where appropriate.
- Automated compatibility checks.
- Clear deprecation and migration policies.

A contract should express what consumers need to know, not expose the internal implementation of its provider.

For example, an Order Management Context may publish an `OrderConfirmed` event. Consumers should depend on the documented meaning and schema of that event, not on the internal structure of the Order aggregate.

#### Shared Understanding

Teams should maintain a shared understanding of the larger business domain without forcing every team to use the same internal model.

This distinction is important.

Two contexts may need to exchange information about a customer, but they do not necessarily need identical `Customer` models. Each context should represent the concepts according to its own responsibilities.

The Context Map describes these relationships and makes the differences explicit.

Learning objectives:

- Use Context Maps to communicate dependencies and ownership.
- Establish effective cross-team communication practices.
- Understand API and event contract management.
- Recognize the importance of compatibility and consumer coordination.

### 1.3 Scaling the Ubiquitous Language

The Ubiquitous Language is fundamental to DDD because it connects domain understanding with the model and its implementation.

As an organization grows, maintaining this language becomes more challenging.

Different teams may use the same word to describe different concepts. Worse, they may use different words for the same concept without realizing it.

The solution is not necessarily to create one universal glossary for the entire organization.

A language should be consistent within its Bounded Context, where it supports a particular model and its rules.

Across contexts, the same term may legitimately have different meanings.

For example:

- In Sales, a `Customer` may be a person or organization interested in purchasing.
- In Billing, a `Customer` may represent the party responsible for financial obligations.
- In Support, a `Customer` may be the subject of a support relationship.

Trying to force all these meanings into one shared model can create unnecessary complexity.

Instead:

**Within a Bounded Context**

- Use consistent terminology in conversations, documentation, and code.
- Make important business concepts explicit.
- Update the model and language together when understanding changes.
- Challenge ambiguous names and concepts.

**Across Bounded Contexts**

- Document differences in terminology.
- Make translation responsibilities explicit.
- Avoid assuming that identical names imply identical models.
- Use integration contracts that communicate the meaning of exchanged information.

A shared glossary can still be useful for organization-wide concepts, but it should not become an excuse to create a universal domain model.

The objective is shared understanding of relationships and differences, not forced uniformity.

Learning objectives:

- Explain why the Ubiquitous Language is context-specific.
- Identify semantic inconsistencies across teams.
- Maintain useful terminology without creating a universal model.
- Recognize when translation between languages is necessary.

### 1.4 Organizational Adoption of DDD

DDD adoption is not simply a technical initiative. It affects how teams collaborate, how decisions are made, and how the organization understands its business.

There are several ways to introduce DDD.

**Top-down adoption**

Leadership sponsors DDD and supports changes in team structures, architecture, and development practices.

This can provide organizational alignment and the resources needed for broader adoption.

However, a mandate to "use DDD everywhere" does not create domain understanding. Teams still need to discover the business, develop models, and learn through practice.

**Bottom-up adoption**

Developers, architects, and domain experts introduce DDD practices within a specific team or project.

This can be effective because teams can demonstrate value through real improvements rather than abstract architectural proposals.

However, without organizational support, successful practices may remain isolated or struggle against existing structures.

**Iterative adoption**

A combination of both approaches is often effective.

Start with a business area where domain complexity and the value of improved modeling are clear. Work with a small group, develop a useful model, and demonstrate tangible results.

Use what the team learns to improve the approach before expanding it to other parts of the organization.

DDD adoption should evolve with the organization's experience. The objective is not to make every team follow the same process, but to help teams make better modeling and design decisions.

Learning objectives:

- Compare top-down and bottom-up adoption.
- Understand why DDD adoption should be iterative.
- Identify an appropriate starting point for introducing DDD.
- Explain the organizational conditions needed for sustainable adoption.

## 2. Applying DDD to Legacy Systems

Legacy systems are not necessarily bad systems. They are systems that have accumulated history, assumptions, dependencies, and business knowledge over time.

They often support critical business operations and contain functionality that has evolved through years of real-world use.

The challenge is that this knowledge may not be represented in an explicit or coherent domain model.

DDD can help us understand and improve these systems, but applying DDD to a legacy codebase requires a different mindset from starting a new project.

The objective is not to replace everything with a new architecture. It is to introduce meaningful boundaries and improve the model incrementally while preserving valuable existing behavior.

### 2.1 Challenges of Applying DDD to Legacy Systems

Legacy systems often present several difficulties.

**Lack of an explicit domain model**

Business rules may be scattered across services, database procedures, controllers, and user interfaces.

The code may contain business concepts, but they are often represented through technical structures rather than an intentional model.

**Tangled dependencies**

Different parts of the system may depend on the same tables, shared utilities, global state, or undocumented behavior.

Changing one component can unexpectedly affect several others.

**Limited test coverage**

The absence of automated tests makes it difficult to distinguish intended behavior from accidental behavior.

Refactoring becomes risky because the team cannot easily determine whether existing functionality has changed.

**Outdated technology**

The system may rely on frameworks, infrastructure, or programming languages that are difficult to maintain or integrate with newer components.

**Hidden business knowledge**

Important rules may exist only in the experience of long-time employees, developers, or operational teams.

Documentation may be incomplete or outdated.

**Resistance to change**

Stakeholders may be concerned about disrupting critical operations. Developers may be reluctant to modify areas they do not fully understand.

These are not merely technical obstacles. They are reasons to approach modernization incrementally and collaboratively.

### 2.2 Start Small and Focus on High-Value Areas

A common mistake is to treat DDD adoption as an opportunity to rewrite the entire legacy system.

Large-scale replacement projects are expensive and risky, particularly when they introduce new technologies, new processes, and new models at the same time.

Instead, identify a focused business problem where better modeling can deliver meaningful value.

Look for areas with:

- Significant business complexity.
- Frequent changes or recurring defects.
- Important strategic value.
- Clear opportunities to establish a boundary.
- A team capable of working on the area with sufficient autonomy.

The initial objective should be modest enough to deliver, but meaningful enough to demonstrate the benefits of a better model.

This also gives the organization an opportunity to learn how DDD practices work in its own environment.

### 2.3 Identify Implicit Bounded Contexts

Legacy systems often contain implicit Bounded Contexts, even when nobody has formally identified them.

Look for differences in:

- Terminology.
- Business rules.
- Data ownership.
- Workflows.
- User roles.
- Responsibilities.
- Integration patterns.

For example, a legacy application may have a single `Customer` table used by Sales, Billing, and Support.

However, the rules applied to a customer in each area may be different.

Sales may care about prospects and purchasing relationships. Billing may care about payment obligations and account status. Support may care about service history and support eligibility.

The shared table does not prove that these areas share one model.

Similarly, separate database schemas or application projects do not automatically prove that they represent different Bounded Contexts.

The boundaries should be discovered through business analysis, terminology, behavior, and the responsibilities of each area.

Learning objectives:

- Identify implicit domain models in existing software.
- Discover possible Bounded Contexts from business behavior and language.
- Distinguish technical separation from semantic separation.
- Identify a suitable area for incremental modernization.

### 2.4 Establish an Anti-Corruption Layer

An Anti-Corruption Layer (ACL) protects the model of one Bounded Context from the concepts and assumptions of another.

It is particularly useful when integrating a new model with a legacy system.

Imagine that a legacy system stores customer information using a structure that does not match the model required by a new Customer Management Context.

Instead of allowing the new domain model to depend directly on the legacy data structures, introduce an ACL that translates between them.

The translation might involve:

- Mapping legacy fields to domain concepts.
- Converting legacy status codes into meaningful domain states.
- Combining information from several legacy sources.
- Translating legacy operations into application-level capabilities.
- Handling inconsistent or incomplete data.

For example, the legacy system may represent a customer's status using numeric codes:

- `1` means active.
- `2` means suspended.
- `3` means closed.

The new model should not need to carry these numeric conventions throughout its domain logic.

The ACL can translate them into concepts meaningful to the new context.

An important design principle is that translation should not become a hidden place for implementing unrelated business rules. The ACL should protect the model and perform the necessary translation, while domain behavior belongs in the domain model itself.

An ACL is also not just a thin adapter by definition. It can involve meaningful coordination and translation logic, so its complexity and maintenance cost should be considered explicitly.

Learning objectives:

- Understand the purpose of an Anti-Corruption Layer.
- Translate external models into the language of a new context.
- Keep legacy concepts out of the new domain model.
- Distinguish translation from domain behavior.

### 2.5 Bubble Context

Eric Evans describes the Bubble Context as one strategy for introducing DDD when surrounded by legacy systems.

A Bubble Context is a small, well-defined Bounded Context established around a particular development effort.

It uses an Anti-Corruption Layer to isolate the new model from the legacy system.

The new context can develop its own domain model without requiring the legacy system to adopt that model.

An important characteristic of this strategy is that the new context initially relies on the existing system for its data. It does not necessarily introduce a new database.

For example, imagine a legacy system that manages customer accounts using a model that is unsuitable for a new business capability.

A team can create a new Customer Management Bubble Context.

Inside this context:

1. The domain model expresses the concepts needed by the new capability.
2. Repositories expose the data and operations required by that model.
3. The ACL queries the legacy system and translates its information.
4. The domain model remains isolated from legacy structures and terminology.

The new context can use its own model even though its information comes from existing systems.

This approach is useful when:

- The new capability is relatively focused.
- The required information already exists in the legacy system.
- The team wants to introduce DDD incrementally.
- The organization is not yet ready to commit to a larger modernization effort.

**The Bubble Context is intentionally limited.**

It is not necessarily intended to grow into a permanent independent system. Evans describes it as a way to gain experience and deliver a focused capability with relatively modest organizational commitment.

The bubble may eventually dissolve, with its functionality becoming part of the existing legacy system. Alternatively, a successful initiative may evolve into an Autonomous Bubble.

The important lesson is that a temporary boundary can still create meaningful value.

### 2.6 Autonomous Bubble

An Autonomous Bubble is a more independent strategy for introducing a new domain model alongside legacy systems.

Unlike the Bubble Context, it has its own data store and can continue operating without continuously depending on the legacy system for every piece of information.

The data is stored in the form native to the new context, using its own abstractions and domain model.

This is not merely a cache of legacy data.

The new context may receive information from legacy systems, but it can also own and manage information that does not exist in the legacy model.

To support this independence, an ACL can synchronize information between the contexts.

For example:

1. A new Customer Management Context is introduced.
2. It owns its customer-related data and domain model.
3. A synchronizing ACL receives relevant information from the legacy system.
4. The ACL translates that information into the new model.
5. The new context evolves independently while maintaining the required integration with legacy capabilities.

Synchronization may happen through:

- Scheduled batch jobs.
- Message queues.
- Integration events.
- Other asynchronous data exchange mechanisms.

The appropriate approach depends on freshness requirements, operational constraints, and the available integration capabilities.

The synchronization mechanism should have an explicit understanding of data ownership, conflict resolution, and acceptable data delay.

#### Bubble Context vs. Autonomous Bubble

| Dimension | Bubble Context | Autonomous Bubble |
|---|---|---|
| Data ownership | Relies on legacy data | Owns its own data |
| Legacy dependency | Relatively direct dependency through an ACL | Reduced dependency through synchronization |
| Data freshness | Often retrieved when needed | Depends on synchronization strategy |
| Independence | Limited | Greater autonomy |
| New information | Usually constrained by legacy data | Can introduce and own new information |
| Typical purpose | Focused, lower-commitment initiative | Longer-term independent development |

The key difference is not the size of the application or the technology used. It is the relationship between the new context and the legacy system.

An Autonomous Bubble can be a natural next step when a successful Bubble Context grows beyond the limitations of its original integration strategy.

### 2.7 Exposing Legacy Assets as Services

Not every capability in a legacy system needs to be replaced.

Legacy systems often contain valuable functionality, integrations, and business knowledge that would be expensive or risky to recreate.

Instead of rewriting everything, existing capabilities can be exposed through well-defined services.

Eric Evans describes this strategy as exposing legacy assets as services, using an Open Host Service and a Published Language to provide a more accessible interface.

For example, a legacy system may contain several tables and complex queries related to sales history.

Different applications need access to sales information, but each application currently implements its own queries and transformations.

A better approach is to:

1. Identify the business information that consumers actually need.
2. Define a clear model for that information.
3. Expose it through a stable service contract.
4. Translate the legacy representation into the published contract.
5. Allow consumers to integrate without understanding the legacy database structure.

This service can be implemented through an Anti-Corruption Layer that hides the complexity of the legacy implementation.

The result is a reusable interface around an existing capability, without requiring the capability itself to be rewritten.

However, exposing a legacy capability as a service does not automatically make its internal design good. The integration boundary should still account for hidden side effects, operational limitations, and the behavior of the existing implementation.

Learning objectives:

- Recognize when legacy capabilities should be reused rather than replaced.
- Understand Open Host Service and Published Language.
- Expose legacy functionality through stable contracts.
- Protect consumers from legacy implementation details.

### 2.8 Expanding a Bubble

A Bubble Context can evolve into a larger modernization effort.

As the new model becomes more valuable and the organization gains confidence in its implementation, the scope of the new context may expand.

More responsibilities can move into the new model, and the legacy system can gradually lose ownership of those capabilities.

This is closely related to the Strangler Fig migration pattern.

The general idea is to replace functionality incrementally rather than attempting a complete replacement in one step.

A typical migration may follow this sequence:

1. Identify a business capability that should move out of the legacy system.
2. Establish a clear boundary around the new implementation.
3. Introduce a routing or integration mechanism where necessary.
4. Redirect selected operations to the new implementation.
5. Move the required data and responsibilities.
6. Validate behavior and consistency.
7. Retire the corresponding legacy functionality when it is no longer needed.
8. Repeat for other capabilities.

The Strangler Fig pattern is especially useful when clients can continue using a stable entry point while functionality is gradually moved behind it.

However, incremental migration is not automatically simple.

During the transition, the new and legacy systems may both need to operate on related data. Teams must understand which system is authoritative for each piece of information and how changes are synchronized.

A migration can become more complex than the original system if ownership is unclear or if both implementations continue modifying the same data without an explicit consistency strategy.

The goal is not merely to add a new application alongside the old one. It is to transfer responsibility deliberately.

### 2.9 Event Streams and Legacy Integration

Legacy systems may not support modern integration patterns directly.

They may have no concept of Domain Events, or they may expose only database tables, stored procedures, and scheduled jobs.

It can still be possible to introduce event-based integration without rewriting the entire legacy application.

For example:

- Database Change Data Capture can identify committed changes.
- An integration component can interpret those changes.
- The component can translate them into meaningful integration events.
- Other Bounded Contexts can consume those events.

However, a database change is not automatically a domain event.

A row being updated tells us something changed in the database. It does not necessarily tell us what happened in the business.

For example, a change to an order status column may represent order confirmation, cancellation, correction, or an administrative update.

The integration layer must understand enough of the legacy behavior to translate technical changes into meaningful business information.

This is particularly important when consumers depend on the semantics of published events.

Event streams can also support the synchronization of an Autonomous Bubble, reducing direct runtime dependencies between contexts.

The same principles still apply:

- Define who owns the information.
- Translate data into the receiving context's model.
- Handle duplicate and out-of-order delivery.
- Make synchronization expectations explicit.
- Monitor processing failures and data delays.

### 2.10 Characterization Tests and Safe Refactoring

Before changing legacy behavior, understand what the system currently does.

In many legacy systems, the actual behavior is not fully documented. Some behavior may even be accidental, but other parts may be relied upon by users and downstream systems.

Characterization Tests help capture the observable behavior of existing software.

These tests are particularly useful when there are no reliable specifications or comprehensive automated tests.

A practical approach:

1. Identify a behavior that needs to be understood or changed.
2. Execute the existing system with representative inputs.
3. Record its observable outputs and relevant side effects.
4. Turn those observations into automated tests.
5. Refactor the implementation while keeping the tests passing.
6. Change the tests only when the intended business behavior is deliberately changed.

For example, imagine a legacy method that calculates a customer's available credit.

Its implementation may involve several queries, hidden status rules, and historical exceptions.

Before refactoring it, create tests for representative scenarios:

- Customer with available credit.
- Customer with outstanding charges.
- Customer with suspended status.
- Customer with an exceptional account state.
- Customer with incomplete data.

These tests provide a safety net while the implementation is improved.

Characterization tests do not prove that existing behavior is correct. They document what the system currently does.

That distinction matters. If a test captures a bug, preserving it forever may not be the desired outcome. First understand whether the behavior is intentional, accidental, or relied upon by other parts of the business.

Characterization testing gives us evidence. Domain analysis helps us decide what the behavior should become.

Learning objectives:

- Understand the role of characterization tests.
- Capture existing behavior before refactoring.
- Distinguish current behavior from intended business behavior.
- Use tests to reduce the risk of incremental modernization.

### 2.11 Embrace the Existing Data Model Initially

One of the most difficult parts of legacy modernization is dealing with existing data structures.

A new domain model may not map naturally to the legacy database.

Ideally, the domain model should remain independent of persistence concerns. In practice, introducing a completely new persistence model at the same time as introducing a new domain model may create unnecessary migration risk.

Sometimes it is reasonable to work with the existing database initially.

For example:

- Introduce a new domain model.
- Implement repositories or adapters that translate between the model and existing tables.
- Keep persistence details out of domain behavior.
- Introduce new storage structures when there is a clear business and technical reason.
- Move ownership gradually when the context becomes sufficiently independent.

This approach allows the domain model to improve without requiring an immediate database replacement.

However, sharing a database is not the same as sharing a domain model.

A new context should not become dependent on another context's internal tables simply because they are accessible.

If the new context requires data from the legacy system, use an explicit integration boundary and translation mechanism.

### 2.12 Communicate with Stakeholders

Legacy modernization often affects systems that are critical to the business.

Technical improvements alone are not enough to make a migration successful.

Stakeholders should understand:

- Why the existing system needs to evolve.
- Which business capabilities will be changed.
- What will remain in the legacy system.
- How existing functionality will be protected.
- What risks exist during the transition.
- How success will be measured.

It is important to avoid presenting modernization as a purely technical exercise or promising that a new architecture will automatically solve organizational and business problems.

Demonstrate progress through business outcomes, improved changeability, reduced operational risk, and clearer ownership.

## 3. Practical Exercise

### Scaling DDD and Modernizing a Legacy System

Choose an existing system that you know well. It can be a real system or a fictional one with realistic business complexity.

Your task has two parts.

#### Part A: Scaling DDD Across Teams

Imagine the system is growing and several teams will work on it.

1. Identify at least three Bounded Contexts.
2. Describe the responsibilities and language of each context.
3. Create a Context Map showing their relationships.
4. Propose a team organization using the Team Topologies concepts.
5. Define ownership for each context.
6. Identify the most important dependencies between teams.
7. Describe how teams will manage API and event contracts.
8. Explain how the organization will maintain shared understanding without forcing every context to use the same model.

Your proposal should make it clear how teams can evolve their contexts independently while still collaborating effectively.

#### Part B: Introducing DDD into a Legacy System

Now assume that one of the contexts is currently implemented inside a large legacy application.

Describe a gradual modernization strategy.

1. Identify the implicit domain model and possible Bounded Context.
2. Identify a high-value business capability to modernize first.
3. Explain how you would isolate it using an Anti-Corruption Layer.
4. Decide whether a Bubble Context or Autonomous Bubble is more appropriate.
5. Identify legacy capabilities that should be exposed as services rather than rewritten.
6. Define how existing behavior will be captured through characterization tests.
7. Describe how data ownership and synchronization will evolve.
8. Propose a gradual migration strategy, potentially using the Strangler Fig pattern.

Finally, document the risks and trade-offs.

Consider what happens when:

- The legacy system is unavailable.
- The new context receives outdated information.
- A synchronization process fails.
- Both systems attempt to update the same business information.
- A new business rule conflicts with an assumption in the legacy model.
- A stakeholder requests a capability that crosses several contexts.

Your solution should explain not only how the new architecture will work, but also how the organization can safely move from the current system to the intended one.

## 4. Completion Criteria

You should be able to:

- Explain why DDD requires continuous model evolution.
- Organize teams around business capabilities and meaningful ownership boundaries.
- Explain the four fundamental Team Topologies and their relationship to DDD.
- Use Context Maps to understand dependencies between teams and Bounded Contexts.
- Maintain context-specific Ubiquitous Languages without forcing a universal domain model.
- Explain top-down, bottom-up, and iterative DDD adoption.
- Identify implicit Bounded Contexts in legacy systems.
- Explain the purpose and implementation of an Anti-Corruption Layer.
- Distinguish Bubble Contexts from Autonomous Bubbles.
- Understand how legacy capabilities can be exposed through Open Host Services and Published Languages.
- Explain how event streams and synchronization can support integration with legacy systems.
- Apply characterization tests before refactoring unfamiliar behavior.
- Design an incremental migration strategy without relying on a big-bang rewrite.
- Explain the organizational, technical, and operational risks of legacy modernization.

**A model is never finished simply because its implementation is complete. It remains useful only while it continues to reflect the reality of the business and the understanding of the people who work with it.**