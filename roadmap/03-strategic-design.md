# 03 Strategic Design

Strategic Design is about understanding the structure of a business domain and making deliberate decisions about how software models should be organized around it.

As a system grows, its complexity rarely comes from the number of classes, services, or database tables alone. It comes from the different meanings, rules, responsibilities, priorities, and forces that coexist within the business.

A customer in Sales may not be the same concept as a customer in Billing. A product in a Catalog may have a different meaning from a product in Inventory. Two teams may use the same word while referring to different concepts, or different words while describing the same underlying business reality.

Strategic Design helps us recognize these differences and decide where models should begin and end, which parts of the business deserve the most attention, and how independently evolving models should collaborate.

It is not a recipe for splitting a monolith into microservices. It is a way to reason about the business, the models that represent it, and the relationships between those models.

## Goal

Learn to analyze a business domain, identify its strategic subdomains, define meaningful Bounded Contexts, and describe the relationships between them through Context Mapping.

By the end of this section, you should be able to explain not only how parts of a system are connected, but also why those connections exist, who owns them, and what their consequences are.

## Key Concepts

### 1. Subdomains: Understanding the Business Problem

A subdomain is a distinct area of the business problem that the organization needs to understand and address.

Subdomains describe the business, not the software architecture. They help us decompose a complex domain into areas with different responsibilities, knowledge, rules, and strategic importance.

For example, an online learning business might involve:

- Course Design and Creation
- Learning Delivery
- Assessment and Certification
- Student Enrollment
- Payments and Billing
- Marketing and Customer Acquisition
- Customer Support

These areas are not automatically separate applications, teams, or services. They are ways of understanding the business problem.

#### Types of Subdomains

DDD distinguishes three strategic categories of subdomains.

**Core Subdomain**

The Core Subdomain is where the organization develops its distinctive capabilities and competitive advantage.

It contains the business knowledge, decisions, and capabilities that make the organization meaningfully different from its competitors. It is usually where investment in deep domain understanding and carefully designed software models can create the greatest strategic return.

Examples:

- A learning platform's adaptive learning and assessment approach, if it is what differentiates the platform.
- A logistics company's route optimization and capacity allocation capabilities, if those capabilities provide a competitive advantage.
- A financial institution's proprietary risk assessment and pricing models.

A Core Subdomain is not necessarily the largest, most complicated, or most visible part of the system. It is core because of its strategic significance.

**Supporting Subdomain**

A Supporting Subdomain is necessary for the business to operate, but it does not provide the same distinctive competitive advantage.

It often contains business-specific rules and workflows, so it may still require a purpose-built solution. However, it generally does not deserve the same level of strategic investment as the Core Subdomain.

Examples:

- Internal instructor scheduling for a learning platform.
- Operational case management for a logistics company.
- Internal approval workflows for a financial institution.

Supporting does not mean unimportant. A poorly implemented Supporting Subdomain can still create substantial operational problems.

**Generic Subdomain**

A Generic Subdomain represents a problem that is common across many businesses and for which established solutions or well-known approaches already exist.

Organizations often have little strategic reason to develop a distinctive implementation themselves. They may use an existing product, a third-party service, or a standard implementation.

Examples:

- Standard authentication and identity management.
- General-purpose document generation.
- Common email delivery capabilities.
- Commodity accounting or payroll functionality.

Be careful with assumptions: a capability is not automatically Generic just because a vendor offers a product for it. Payment processing, for example, may be a commodity for one business but a Core Subdomain for a payment company. The classification depends on the organization's business strategy and competitive position.

#### Subdomain Analysis

Subdomain classification is a strategic decision, not a universal taxonomy.

The same business capability can be Core in one organization, Supporting in another, and Generic in a third.

To classify a subdomain, ask:

- Does this capability differentiate the business from its competitors?
- Does the organization possess specialized knowledge or rules in this area?
- Would improving this capability materially strengthen the business?
- Can an established external solution address the problem without sacrificing strategic advantage?
- What would happen if this capability were poorly implemented?

A useful analysis should also identify the business knowledge involved, the people who understand it, the major decisions it contains, and the consequences of getting those decisions wrong.

**Important:** Do not classify subdomains based on the current software structure. Existing applications, departments, database schemas, and organizational charts are evidence about how the business is currently organized, not proof of how its domain should be understood.

---

### 2. Bounded Contexts: Defining the Boundaries of Models

A Bounded Context is an explicit boundary within which a particular domain model is defined and has a consistent meaning.

Inside that boundary, concepts, terminology, rules, and relationships belong to a coherent model. Outside it, the same words may have different meanings, or the model may no longer be appropriate.

This is one of the most important ideas in DDD: **a model is valid within a context, not necessarily across an entire organization.**

Consider an online learning platform:

- In the Course Catalog context, a Course describes an offering with a title, description, curriculum, and publication status.
- In the Learning Delivery context, a Course may represent the learning experience a student is enrolled in, with progress and completion information.
- In the Billing context, the relevant concept may be a purchasable offering with a price, invoice details, and payment conditions.

These models may refer to the same real-world business offering, but they serve different purposes and express different rules.

Trying to force them into one universal `Course` model can introduce unnecessary dependencies and ambiguous responsibilities.

#### What a Bounded Context establishes

A Bounded Context gives us a place to define:

- The meaning of domain concepts and terms.
- The rules and invariants that belong to the model.
- The relationships between concepts.
- The responsibilities of the model.
- The language used by the people designing, discussing, and implementing it.
- The boundaries across which information must be translated or communicated.

The Ubiquitous Language belongs to a Bounded Context. It should be consistent within that context, but it does not need to be identical across the entire organization.

A shared vocabulary across an organization can be useful, but forcing every department and model to use exactly the same terminology can conceal meaningful differences.

#### Identifying Bounded Contexts

Boundaries should emerge from domain understanding, not from a desire to create a particular number of services.

Useful signals include:

- The same term has different meanings in different parts of the business.
- Different areas enforce different business rules around apparently similar concepts.
- A model has responsibilities that are difficult to express coherently alongside another model.
- Different groups use different language or make decisions independently.
- Changes in one area repeatedly cause unintended changes in another.
- Integrating two parts of the system requires continuous translation or coordination.

These are clues, not mechanical rules. A difference in terminology alone does not always justify a new Bounded Context. The distinction must matter to the model, its rules, its ownership, or its evolution.

#### Bounded Contexts are not automatically Microservices

A Bounded Context is a modeling boundary. A microservice is an architectural and deployment choice.

A Bounded Context may be implemented as:

- A module inside a modular monolith.
- A separately deployed application.
- Several cooperating components.
- A legacy system that already owns a particular model.

Likewise, one application can contain multiple Bounded Contexts, even if its code does not yet make those boundaries explicit.

The important thing is to protect the consistency and ownership of each model. Deployment decisions should follow from additional architectural and operational considerations.

---

### 3. Subdomains and Bounded Contexts: Problem Space vs. Solution Space

These two concepts are closely related, but they answer different questions.

| Subdomain | Bounded Context |
|---|---|
| Describes an area of the business problem | Defines a boundary for a domain model |
| Belongs to the problem space | Belongs to the solution space |
| Helps us understand business responsibilities and strategic importance | Helps us organize models, language, and implementation responsibilities |
| Classified as Core, Supporting, or Generic | Has its own model and Ubiquitous Language |
| Does not prescribe software structure | Influences software structure, but does not prescribe deployment |

A useful starting assumption is that a Subdomain and a Bounded Context often have a meaningful relationship. However, they do not have to correspond one-to-one.

For example, an Order Management Subdomain might be represented by two Bounded Contexts:

**Sales Context**

- Customer-facing order placement.
- Pricing and discounts.
- Order acceptance.
- Commercial order status.

**Fulfillment Context**

- Allocation of inventory.
- Warehouse operations.
- Shipment preparation.
- Delivery status.

The business area is related, but the models have different responsibilities and rules.

It is also possible for one Bounded Context to support more than one closely related Subdomain, provided those areas can be represented coherently by the same model and language.

There is no universal mapping rule. The relationship should be discovered and justified through domain analysis.

---

### 4. Context Mapping: Making Relationships Explicit

Identifying Bounded Contexts is only part of Strategic Design.

In a real business, contexts rarely exist in complete isolation. They exchange information, depend on one another, coordinate changes, and sometimes impose constraints on each other's evolution.

Context Mapping describes these relationships.

A Context Map is a strategic model that shows:

- The Bounded Contexts in a domain or system.
- The relationships and dependencies between them.
- Which context influences another.
- How the teams or owners collaborate.
- Which integration patterns are used.
- Where translation is required to protect model boundaries.

A Context Map is more than a diagram of APIs or data flows. It captures organizational and model relationships as well as technical integration.

#### Upstream and Downstream

Before examining the patterns, understand two important terms.

**Upstream** refers to the context whose model or decisions influence another context.

**Downstream** refers to the context that depends on the upstream context.

For example, if Shipping consumes information provided by Sales, Sales may be upstream and Shipping downstream.

The direction describes influence and dependency. It does not necessarily describe the direction in which every message or request travels.

A downstream context may call an upstream API, receive its messages, or periodically retrieve its data. The upstream/downstream relationship is about the dependency between their models and the teams responsible for them.

#### Context Mapping Patterns

The following patterns help describe different kinds of relationships. They are not technology choices. They describe how contexts and their owners relate to one another.

**Shared Kernel**

Two Bounded Contexts share a deliberately limited subset of their domain model or implementation.

The shared part is jointly agreed upon and maintained. Changes to it require coordination between the involved teams.

For example, two closely collaborating contexts might share a small set of domain concepts and code that must remain consistent.

Sharing a simple identifier such as `CustomerId` does not automatically make a Shared Kernel. A Shared Kernel involves an intentionally shared part of the model or codebase.

**Trade-off:** It avoids duplicating certain shared concepts, but introduces coordination and coupling. If the shared area grows or changes frequently, it can become a source of friction and prevent independent evolution.

Use it only when the value of sharing is greater than the cost of coordination.

**Customer-Supplier**

This is an upstream/downstream relationship in which the downstream context has a meaningful voice in the upstream team's priorities and delivery decisions.

The downstream is the customer of the upstream capability. The upstream is the supplier.

For example, a Shipping team may depend on Sales for specific order information. If the teams establish an explicit process for Shipping to communicate its requirements and have them considered in Sales planning, their relationship can be described as Customer-Supplier.

This is not simply an API relationship. It is also an organizational agreement about priorities, collaboration, and delivery commitments.

**Trade-off:** It enables coordination around the needs of dependent contexts, but requires communication, planning, and an explicit commitment from both sides.

**Conformist**

The downstream context adopts the upstream model as it is, rather than translating it into a separate model.

This can be reasonable when the upstream model is acceptable, stable enough, and the cost of maintaining a separate model is not justified.

For example, a downstream reporting context might directly adopt a well-established external provider's data representation.

**Trade-off:** Integration becomes simpler because there is little or no translation. However, the downstream gives up some control over its model and becomes more sensitive to upstream changes and design decisions.

Conformity is a conscious trade-off, not an ideal to pursue everywhere.

**Anticorruption Layer (ACL)**

An Anticorruption Layer protects one context's model from concepts and assumptions belonging to another context.

It translates information from an external model into the language and concepts of the downstream model. It can also translate outgoing information when necessary.

For example, a Shipping context may receive an `Order` representation from Sales. Instead of allowing the Sales model to spread throughout Shipping, an ACL translates the relevant information into Shipping's own concepts, such as a `ShipmentRequest`.

The downstream model remains responsible for its own rules and terminology.

An ACL can include translators, adapters, mapping logic, and carefully designed interfaces. It is not necessarily a separate application or a particular technical pattern.

**Trade-off:** It requires additional implementation and maintenance, but protects the downstream model from external assumptions and changes.

An ACL is particularly useful when integrating with legacy systems, third-party platforms, or upstream contexts whose models do not fit your own domain.

**Open Host Service (OHS)**

An upstream context provides a well-defined protocol or set of services for other contexts to consume.

Rather than creating a different integration mechanism for every consumer, the upstream offers a consistent interface that multiple downstream contexts can use.

For example, a Customer context might provide a documented API for retrieving customer information and performing supported customer-related operations.

The upstream owns and maintains the service contract.

**Trade-off:** A stable shared interface can reduce integration complexity for multiple consumers. However, the upstream must manage compatibility, versioning, and the needs of its consumers without allowing the public contract to become an incoherent collection of special cases.

**Published Language (PL)**

A Published Language is a well-documented language or format used to communicate between Bounded Contexts.

It provides an agreed representation of the information exchanged, so that each context can translate it into its own internal model.

Examples include:

- A documented JSON message schema.
- A versioned event contract.
- An industry-standard data format.
- A formally defined interchange protocol.

For example, a Shipping context might publish a `ShipmentDispatched` integration event with a documented schema. Other contexts can consume that contract without depending on Shipping's internal classes or database structure.

Published Language is often used together with Open Host Service: OHS describes the service or protocol through which consumers access a context, while PL describes the shared language used in the exchange.

**Trade-off:** A published contract makes integration more predictable, but it creates a responsibility to manage schema evolution, compatibility, and the meaning of the information being published.

**Separate Ways**

Two Bounded Contexts deliberately do not integrate because the value of integration does not justify its cost.

For example, two internal applications might maintain separate representations of information because they serve unrelated workflows and do not need synchronized behavior.

Separate Ways does not mean that the organization has failed to integrate its systems. It can be a deliberate architectural and business decision.

**Trade-off:** It preserves independence and avoids unnecessary coupling, but may introduce duplicated information or manual work if the contexts eventually need to coordinate.

#### A Note on Pattern Classification

These patterns describe different aspects of a relationship. They are not all mutually exclusive alternatives.

For example:

- Upstream/Downstream describes the direction of influence.
- Customer-Supplier describes a particular form of collaboration within that relationship.
- Conformist and ACL describe possible downstream responses.
- OHS and PL describe ways an upstream context can expose a service and a communication language.
- Shared Kernel describes a shared model or code subset.
- Separate Ways describes a deliberate absence of integration.

Partnership is another useful relationship pattern: two contexts and their teams coordinate closely and share responsibility for evolving capabilities that depend on one another. Unlike a typical upstream/downstream relationship, neither side is simply the supplier of the other.

A Context Map should use the patterns that accurately describe the actual relationship. Do not force every relationship into a pattern just to make the diagram look complete.

---

### 5. Integration Between Bounded Contexts

Context Mapping describes the relationship between contexts. Technical integration describes how they exchange information or invoke capabilities.

These are related decisions, but they are not the same thing.

A Customer-Supplier relationship, for example, could be implemented through synchronous HTTP calls, asynchronous messages, or another integration mechanism. The technology does not determine the strategic relationship.

#### Synchronous Communication

A context may expose an API that another context calls when it needs an immediate response.

Common approaches include:

- REST APIs over HTTP.
- RPC-style communication.
- Other request/response protocols.

Synchronous communication is useful when a caller needs an immediate answer or when an operation naturally requires an interaction with another context.

However, it introduces runtime dependencies. If the upstream context is unavailable or slow, the downstream operation may also be affected.

It is important to distinguish a business capability request from direct access to another context's internal model. An API should expose an intentional contract, not simply make another context's database or internal objects remotely accessible.

#### Asynchronous Communication

Contexts can also communicate through messages and events.

Common infrastructure includes:

- RabbitMQ.
- Apache Kafka.
- Azure Service Bus.
- Other message brokers and streaming platforms.

Asynchronous communication can reduce direct runtime coupling and allow consumers to process information independently.

For example, after Shipping dispatches a shipment, it may publish a `ShipmentDispatched` integration event. Billing, Customer Notifications, or Order Tracking may consume that event according to their own responsibilities.

A few distinctions matter:

- A **Domain Event** represents something meaningful that happened within a domain model.
- An **Integration Event** communicates a relevant fact across a Bounded Context boundary.
- A message broker is infrastructure; it does not make a message a domain event.
- Publishing internal domain objects directly as external contracts can expose implementation details and couple consumers to the producer's model.

Integration contracts should be designed intentionally. They should communicate facts and information that consumers need, not expose every internal state change.

#### Eventual Consistency

When contexts maintain their own models and data, their representations of related business information may not update at exactly the same time.

This is eventual consistency.

For example, after an order is accepted, the Sales context may immediately record that decision. The Fulfillment context may receive the corresponding integration event a short time later and then update its own representation.

During that interval, the contexts may temporarily disagree about the latest state.

This is not necessarily a defect. It is a consequence of allowing models and data to evolve independently.

However, eventual consistency must be designed and handled deliberately.

Important considerations include:

- **Idempotency:** Processing the same message more than once should not produce unintended duplicate effects.
- **Retries:** Temporary failures should not silently lose important business information.
- **Ordering:** Some business processes require messages to be processed in a meaningful order.
- **Duplicate delivery:** Consumers should be prepared for repeated messages.
- **Failure handling:** Poison messages and persistent failures need explicit handling and observability.
- **Reconciliation:** Some workflows need a way to detect and correct inconsistencies.
- **Business feedback:** Users and other contexts may need to understand that an operation is pending rather than complete.

A retry mechanism alone does not solve eventual consistency. The business process must define what happens when an update is delayed, rejected, or permanently fails.

For multi-step business processes, you may also need a Saga or another coordination strategy. These are related design topics, but they are not Context Mapping patterns.

---

### 6. Strategic Design and Architecture

Strategic Design connects domain understanding to architectural decisions.

Once the subdomains and Bounded Contexts are understood, we can make more deliberate decisions about:

- Which models should remain independent.
- Which teams should own which models.
- Where integration is necessary.
- Which dependencies are acceptable.
- Where translation is needed.
- Which parts deserve investment in deep domain modeling.
- Which capabilities can use existing solutions.
- How the architecture should evolve as the business changes.

The resulting architecture should reflect the meaningful boundaries of the business rather than arbitrary technical divisions.

A modular monolith can be a very good implementation of a strategically designed domain. Microservices may be appropriate when independent deployment, scaling, ownership, or operational autonomy justify their additional costs.

The important point is that the strategic model should inform the architecture, not be retrofitted to justify a technology choice.

---

### 7. Distillation: Protecting the Core Domain

Distillation is the process of identifying and emphasizing the most valuable knowledge in the domain.

The purpose is not simply to remove code or outsource functionality. It is to make the Core Domain visible, understandable, and worthy of focused design effort.

In practice, distillation involves:

**Core Domain Identification**

Identify the capabilities and business knowledge that provide the organization's competitive advantage.

Be specific. "Order Management" may be too broad to explain what makes a business distinctive. The core might instead be a particular pricing strategy, allocation algorithm, risk model, or customer experience.

**Prioritization**

Allocate experienced people and appropriate engineering effort to the Core Domain.

Not every part of the system needs the same level of sophistication. Strategic importance should influence where the organization invests its best domain knowledge and design capacity.

**Simplification**

Reduce unnecessary complexity in Supporting and Generic areas.

This does not mean ignoring their quality. It means avoiding bespoke complexity when a simpler, established solution can satisfy the business need.

**Use of Generic Solutions**

Where appropriate, adopt existing products, libraries, platforms, or third-party services instead of building a custom solution.

The decision should consider more than initial implementation cost. Integration, operational dependency, data ownership, vendor lock-in, compliance, and future flexibility also matter.

**Make the Core Domain Explicit**

Keep the Core Domain recognizable in the model and architecture. Avoid allowing generic frameworks, infrastructure concerns, and supporting workflows to dominate the design.

A useful principle is:

> Invest deeply where the business needs to be different. Reuse or simplify where it does not.

---

## Practical Exercise: Analyze a Domain and Build a Context Map

Choose a business domain you understand well. It can be an online learning platform, an e-commerce business, a logistics company, a financial service, or a real system you have worked on.

The exercise has two connected parts.

### Part A: Subdomain Analysis

1. Identify the main business capabilities and responsibilities.
2. Group related capabilities into candidate Subdomains.
3. Classify each Subdomain as Core, Supporting, or Generic.
4. Explain the reasoning behind each classification.
5. Identify the domain experts and business knowledge involved.

Do not classify a capability as Generic merely because a third-party product exists for it. Explain whether it is strategically differentiating for this particular business.

**Deliverable:** A Subdomain map with each area's strategic classification and rationale.

### Part B: Bounded Contexts and Context Mapping

1. Identify candidate Bounded Contexts based on model boundaries, language, rules, and responsibilities.
2. Give each context a short description of its model and ownership.
3. Identify the important relationships between contexts.
4. For each relationship, describe:
   - The upstream and downstream contexts, where applicable.
   - The team or group that owns each context.
   - The relationship pattern.
   - The technical integration mechanism, if known.
   - The reason for choosing that relationship.
   - The main risks and trade-offs.
5. Identify where an Anticorruption Layer or Published Language would protect a model.
6. Identify any contexts that should deliberately remain Separate Ways.

**Deliverable:** A Context Map accompanied by a short explanation of its relationships and decisions.

### Questions to Challenge Your Design

- Are you creating a Bounded Context because the model needs a boundary, or simply because you want another microservice?
- Are two contexts genuinely using different models, or have you only given the same concept different names?
- Which context owns a business decision?
- What happens when an upstream context changes its model?
- Does a downstream context need the upstream's entire model, or only a small part of its information?
- Are you introducing an integration because the business needs it, or because the technology makes it easy?
- Which relationships create the most organizational and technical coupling?
- Which Subdomains deserve the strongest investment in domain modeling?

---

## Completion Criteria

You have completed this section when you can:

- Explain the difference between a Subdomain and a Bounded Context without treating them as interchangeable.
- Identify Core, Supporting, and Generic Subdomains and justify the classification using business strategy.
- Explain why the same business concept may have different models in different Bounded Contexts.
- Identify candidate Bounded Contexts based on language, responsibilities, rules, and model consistency.
- Explain why a Bounded Context is not automatically a microservice.
- Read and create a Context Map.
- Explain upstream and downstream relationships and distinguish them from message or request direction.
- Describe the purpose and trade-offs of Shared Kernel, Customer-Supplier, Conformist, Anticorruption Layer, Open Host Service, Published Language, Partnership, and Separate Ways.
- Distinguish strategic integration relationships from technical integration mechanisms.
- Explain the consequences of asynchronous communication and eventual consistency.
- Justify architectural decisions based on domain understanding rather than technology preferences.

## Recommended Resources

### Books

- **[Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.domainlanguage.com/ddd/)** — Eric Evans. The foundational DDD book, including the strategic patterns and concepts that underpin this section.
- **[Implementing Domain-Driven Design](https://www.informit.com/store/implementing-domain-driven-design-9780321834577)** — Vaughn Vernon. A practical treatment of DDD, including strategic design, Bounded Contexts, and integration.
- **[Learning Domain-Driven Design](https://www.oreilly.com/library/view/learning-domain-driven-design/9781098100124/)** — Vlad Khononov. A more approachable path through strategic and tactical DDD, with useful discussion of domain boundaries and architecture.

### References and Tools

- **[DDD Reference](https://www.domainlanguage.com/wp-content/uploads/2016/05/DDD_Reference_2015-03.pdf)** — Eric Evans. A concise reference to DDD terminology and patterns.
- **[DDD Crew: Context Mapping](https://github.com/ddd-crew/context-mapping)** — A visual reference for Context Mapping patterns and their relationships.
- **[Context Mapper](https://contextmapper.org/)** — A modeling tool and DSL for describing Bounded Contexts, Context Maps, and DDD relationships.
- **[Domain-Driven Design Europe](https://dddeurope.com/)** — Talks and learning resources from the DDD community.

---

Strategic Design is not about finding the perfect diagram or the ideal number of services. It is about making the structure of the business visible, protecting the integrity of its models, and understanding the consequences of the relationships between them.

A good Context Map does not merely show how a system is connected. It explains why those connections exist, who is responsible for them, and how the organization can evolve without losing the meaning of its domain.