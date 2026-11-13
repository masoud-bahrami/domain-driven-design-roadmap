# Subdomains and Bounded Contexts

Understanding the business domain is not enough. We also need to understand how to divide that domain into meaningful areas of responsibility and how to design models that can evolve without creating unnecessary ambiguity and coupling.

Two fundamental concepts help us do this: **Subdomains** and **Bounded Contexts**.

They are closely related, but they describe different things:

- A **Subdomain** describes a part of the business problem space. It helps us understand what the organization does and where its complexity and strategic value lie.
- A **Bounded Context** defines a boundary within which a particular domain model and its Ubiquitous Language are consistent. It helps us organize the solution space.

This distinction is fundamental.

Subdomains help us understand the business. Bounded Contexts help us design software models that represent it.

There is no requirement for every Subdomain to become a separate Bounded Context, nor does every Bounded Context have to correspond to exactly one Subdomain.

The relationship must be discovered and designed according to the business, the models, and the constraints of the solution.

---

## 1. Understanding Subdomains

A Subdomain is a distinct part of the overall business domain. It represents a particular area of business activity, responsibility, knowledge, and rules.

Consider an e-commerce business. Its overall domain may include:

- Product discovery
- Sales
- Pricing
- Inventory management
- Payment processing
- Order fulfillment
- Customer support

These are different areas of the business, each with its own concepts, responsibilities, and challenges.

Identifying Subdomains helps us break down a complex business into more manageable areas of understanding.

Importantly, Subdomains are discovered by examining the business itself, not by looking at existing applications, database schemas, organizational charts, or technical architecture.

An organization may have a department called Operations, but that does not mean Operations is necessarily one Subdomain. It may contain several distinct areas of business responsibility.

Likewise, several departments may collaborate within a single Subdomain.

### Types of Subdomains

DDD commonly distinguishes three types of Subdomains based on their strategic importance and the nature of the business problem they solve.

### Core Domain

The Core Domain is the part of the business that provides meaningful competitive differentiation.

It represents the capabilities, knowledge, and business rules that make the organization distinct from its competitors.

For example:

- A logistics company may differentiate itself through intelligent route optimization.
- A financial institution may differentiate itself through sophisticated risk assessment.
- An online marketplace may differentiate itself through its matching and recommendation capabilities.

The Core Domain deserves focused investment in domain expertise, modeling, and software design.

This is where understanding the business deeply can provide the greatest strategic return.

The Core Domain is not necessarily the largest, most complicated, or most technically challenging part of the system. It is the part that matters most to the organization's strategy.

### Supporting Subdomain

A Supporting Subdomain is important to the business but does not provide its primary competitive advantage.

It supports the Core Domain by enabling the organization to operate effectively.

For example, a logistics company may have specialized shipment coordination processes that support its route optimization capabilities.

Supporting Subdomains still contain business rules and may require carefully designed models. However, they generally do not justify the same level of strategic investment as the Core Domain.

### Generic Subdomain

A Generic Subdomain represents functionality that is common across many businesses and does not provide meaningful competitive differentiation.

Examples may include:

- Standard authentication
- Generic document generation
- Common notification delivery
- Standardized identity verification

Organizations can often buy, reuse, or implement these capabilities using established solutions.

The objective is to avoid investing valuable domain expertise in solving problems that are already well understood and widely available.

However, generic does not mean unimportant. A generic capability may still have significant security, reliability, regulatory, or operational requirements.

### Classification Is Not Permanent

Subdomain classifications depend on the organization's strategy and competitive environment.

A Supporting Subdomain may become part of the Core Domain if the business starts differentiating itself through that capability.

Similarly, a capability that was once unique may become a commodity as the market evolves.

Therefore, classification should be revisited as the business changes.

### Practical Exercise: Identify and Classify Subdomains

Choose a business you understand and identify its major areas of responsibility.

For each candidate Subdomain, ask:

1. What business capability does it represent?
2. What business problem does it solve?
3. Does it provide competitive differentiation?
4. How much specialized domain knowledge does it require?
5. Could the capability reasonably be purchased or reused?
6. What would happen to the business if this capability were unavailable?

Then classify it as Core, Supporting, or Generic.

Do not classify Subdomains based solely on technical complexity or the number of developers working on them.

The purpose is to understand where the business creates value and where domain expertise deserves the greatest investment.

---

## 2. Understanding Bounded Contexts

A Bounded Context is an explicit boundary within which a particular domain model and its Ubiquitous Language are valid and consistent.

The same word can have different meanings in different contexts. This is not necessarily a problem. It becomes a problem when those differences are hidden or when the software tries to force incompatible meanings into one universal model.

Consider the concept of `Customer` in an e-commerce business.

In Sales, a Customer may represent a person or organization that purchases products.

In Customer Support, a Customer may represent the person associated with a support case.

In Billing, a Customer may represent the party responsible for an invoice or payment obligation.

These concepts are related, but their responsibilities and rules are different.

Trying to represent all of them through one universal Customer model can introduce unnecessary complexity:

- Sales needs purchasing history and commercial eligibility.
- Support needs contact information and case history.
- Billing needs invoicing details and payment obligations.

Each context has its own reasons to model the concept differently.

A Bounded Context makes these differences explicit.

### A Bounded Context Is a Boundary of Meaning

The most important characteristic of a Bounded Context is semantic consistency.

Within the boundary:

- Terms have explicit meanings.
- The domain model expresses a coherent set of business concepts.
- Business rules are interpreted consistently.
- The model has clear responsibilities.
- Changes can be evaluated in relation to that model.

A Bounded Context is not automatically:

- A microservice.
- A database.
- A deployment unit.
- A team.
- A namespace.
- A folder or project in the source code.

These may be implementation choices influenced by Bounded Context boundaries, but they are not what defines a Bounded Context.

A Bounded Context can initially be implemented as a module within a modular monolith. It may later become an independently deployed service if operational, organizational, or scalability requirements justify that decision.

Do not confuse the boundary of a model with the deployment architecture used to implement it.

---

## 3. The Relationship Between Subdomains and Bounded Contexts

Subdomains and Bounded Contexts are related, but they are not interchangeable.

A Subdomain describes a business problem. A Bounded Context describes the boundary of a model designed to address a problem.

The relationship is discovered through analysis and design.

There are several possible arrangements:

### One Subdomain, One Bounded Context

A Subdomain may be sufficiently cohesive to be represented by one Bounded Context.

For example, a simple document generation capability may have one consistent model and one clear responsibility.

### One Subdomain, Multiple Bounded Contexts

A Subdomain may contain multiple models with different responsibilities or interpretations.

For example, an organization's Sales Subdomain might contain separate contexts for:

- Lead Management
- Quotation Management
- Order Management

They belong to the same broader business area, but each may require a distinct model and language.

### Multiple Subdomains, One Bounded Context

In some situations, a single model may need to support closely related business capabilities across multiple Subdomains.

This can be reasonable when the concepts and rules are sufficiently cohesive and there is no meaningful benefit in separating their models.

The point is not to force a particular mapping. It is to discover boundaries that make the business and its software easier to understand and evolve.

### A Useful Way to Think About It

```text
BUSINESS PROBLEM SPACE

        Business Domain
               |
       +-------+-------+
       |       |       |
      Sales  Billing  Shipping
       |       |       |
     Core   Supporting Supporting


SOLUTION MODEL SPACE

        Software Solution
               |
       +-------+-------+
       |       |       |
      Sales   Billing  Shipping
       BC      BC       BC

       Each Bounded Context
       owns a coherent model
       and Ubiquitous Language
```

This is only an illustrative one-to-one arrangement. Real systems may have more complex relationships.

The important point is that Subdomains and Bounded Contexts answer different questions:

- What areas of the business exist, and how important are they?
- Where should a particular model and its language be consistent?

---

## 4. Discovering Bounded Context Boundaries

Identifying Bounded Contexts is a modeling activity, not a naming exercise.

Do not begin by dividing the application into services or creating a separate project for every business capability.

Instead, explore how the business works and look for meaningful differences in language, responsibility, rules, and model behavior.

The following questions can help.

### Look for Different Meanings

- Do different parts of the business use the same term differently?
- Does the concept have different attributes or behaviors depending on the situation?
- Are there terms that appear identical but represent different business concepts?
- Do teams repeatedly need to explain what a particular word means?

Different meanings may indicate that separate models are needed.

### Look for Different Business Responsibilities

- Who owns the decisions associated with this concept?
- Which business capability is responsible for changing it?
- Are different rules applied by different parts of the business?
- Can these responsibilities evolve independently?

Different responsibilities may indicate distinct model boundaries.

### Look for Different Lifecycles and Rules

- Does the concept have different states in different business processes?
- Are state transitions governed by different rules?
- Does one part of the business need information that another part does not?
- Are there different definitions of completion, approval, or validity?

A shared word does not necessarily imply a shared lifecycle.

### Look for Integration and Change Dependencies

- Which models depend on each other?
- Which changes frequently require coordination?
- Are there stable contracts between these areas?
- Is one model being forced to adopt another model's terminology or assumptions?

These questions help reveal where boundaries may reduce accidental coupling.

### Important: Do Not Split Everything

Different vocabulary does not automatically require different Bounded Contexts.

Two teams may use different expressions for the same concept without needing separate models.

Likewise, two parts of a business may have different responsibilities but still benefit from a shared model.

A boundary should solve a real modeling or organizational problem. It should not be introduced merely because a diagram looks cleaner with more boxes.

---

## 5. Context Maps

Once Bounded Contexts have been identified, we need to understand how they relate to each other.

A Context Map describes the relationships between Bounded Contexts and makes their dependencies and integration arrangements explicit.

It is not simply a diagram showing which service calls another service.

A useful Context Map should communicate:

- Which context owns each model.
- Which context is upstream and which is downstream.
- Which contracts are published and consumed.
- Where translation between models takes place.
- Which team or responsibility owns each integration.
- What assumptions or dependencies exist.
- Why a particular relationship or integration pattern was chosen.

### Common Context Mapping Patterns

The following patterns describe common relationships between Bounded Contexts.

#### Shared Kernel

Two or more Bounded Contexts share a small part of their domain model.

The shared portion requires coordinated ownership and careful change management.

A Shared Kernel should remain deliberately small. Sharing too much model implementation can create tight coupling between contexts.

#### Customer-Supplier

An upstream context provides capabilities or information that a downstream context depends on.

The downstream context has requirements that should influence the upstream context's priorities and interface design.

The relationship recognizes that the upstream context has some responsibility toward its downstream consumers.

#### Conformist

A downstream context adopts the upstream context's model and contracts without attempting to influence them.

This may be a pragmatic choice when the upstream model is externally controlled or when the cost of maintaining a translation layer is not justified.

However, it can expose the downstream model to assumptions and limitations that do not naturally belong to it.

#### Anti-Corruption Layer

An Anti-Corruption Layer (ACL) translates between an external model and the local model.

It protects the local model from foreign terminology, structures, and assumptions.

An ACL is especially useful when integrating with legacy systems, third-party services, or contexts whose models cannot or should not be adopted directly.

The translation should express the local model rather than simply passing external data through.

#### Open Host Service

An upstream context provides a well-defined service interface for other contexts to consume.

The interface is designed as a stable and explicit contract rather than exposing internal implementation details.

#### Published Language

Two or more contexts communicate through a shared, explicitly defined language or contract.

For example, an organization may define a standard event schema for communicating shipment status across several systems.

A Published Language should represent a stable integration contract, not necessarily the internal domain model of any participating context.

#### Separate Ways

Two contexts have no meaningful integration requirement and evolve independently.

Sometimes the simplest integration is no integration at all.

Not every capability needs to share data or coordinate with every other capability.

### Choosing an Integration Pattern

There is no universally best Context Mapping pattern.

Consider:

- How much control does the downstream context have over the upstream context?
- How stable is the upstream model?
- How much semantic difference exists between the models?
- How costly would translation be?
- How important is independent evolution?
- What are the organizational and operational constraints?

For example, adopting a Conformist relationship may be reasonable when consuming a stable third-party service. An ACL may be more appropriate when an external model conflicts with the language and assumptions of your own domain.

The pattern should reflect the actual relationship, not an architectural preference.

---

## 6. Creating a Context Map: A Practical Recipe

The following process can be used to create an initial Context Map and refine it over time.

### Step 1. List the Bounded Contexts

Start with the contexts identified during domain exploration and strategic design.

For each one, record:

- Name and purpose.
- Business capabilities it supports.
- Owning team or responsible group.
- Important domain concepts.
- Core responsibilities.
- Strategic importance, where relevant.

Avoid including every application or database as a separate context. The map should describe models and their relationships.

### Step 2. Identify Dependencies

For each context, ask:

- Which other contexts does it depend on?
- Which contexts consume information or capabilities it provides?
- Does it publish events, APIs, or other contracts?
- Does it rely on another context's terminology or data structures?

Record both technical and semantic dependencies.

### Step 3. Identify Upstream and Downstream Relationships

For each dependency, identify:

- The upstream context: the provider of the model or capability.
- The downstream context: the consumer that depends on it.

This distinction helps clarify who controls the contract and who must adapt when it changes.

### Step 4. Describe the Integration Contract

Record how the contexts communicate.

For example:

- Synchronous API.
- Asynchronous messages.
- Published events.
- Shared data or database.
- File exchange.
- Manual business process.

Do not assume that the integration mechanism defines the relationship. A synchronous API can still represent a Customer-Supplier relationship, and asynchronous messaging does not automatically mean the contexts are independent.

### Step 5. Identify Translation Responsibilities

Ask:

- Does the downstream context adopt the upstream model?
- Does it translate the upstream model into its own?
- Is there a shared language or contract?
- Who owns schema evolution?
- Where are errors and incompatible changes handled?

Make translation responsibilities explicit.

### Step 6. Record the Reason for Each Relationship

A Context Map should preserve design reasoning.

For example:

> Billing consumes payment information from the Payments Context through published events. Billing maintains its own financial model and does not depend on the internal Payment Aggregate.

This explains more than a simple arrow between two boxes.

### Step 7. Review and Evolve the Map

A Context Map is not a permanent architecture diagram.

Review it when:

- Business responsibilities change.
- New capabilities are introduced.
- Teams are reorganized.
- Integration contracts become unstable.
- One context starts adopting another context's internal concepts.
- Changes repeatedly require coordination between supposedly independent contexts.

Update the map as the business and software evolve.

---

## 7. Practical Exercise: Map an E-Commerce Domain

Consider an e-commerce organization with the following business capabilities:

- Product Catalog
- Sales
- Pricing
- Inventory
- Payments
- Fulfillment
- Customer Support

Start by identifying their Subdomains and classifying them according to the organization's strategy.

Then propose candidate Bounded Contexts.

An illustrative design might look like this:

```text
+--------------------+
|   Product Catalog  |
+--------------------+
          |
          | Published Product Information
          v
+--------------------+       +--------------------+
|       Sales        |------>|      Payments      |
+--------------------+       +--------------------+
          |                            |
          |                            | Payment Events
          v                            v
+--------------------+       +--------------------+
|     Inventory      |       |      Billing       |
+--------------------+       +--------------------+
          |
          | Fulfillment Request
          v
+--------------------+
|    Fulfillment     |
+--------------------+

+--------------------+
|  Customer Support  |
+--------------------+
```

This is only a starting hypothesis. It does not prescribe the final architecture or prove that every capability needs its own Bounded Context.

Explore the relationships:

- Does Sales own the commercial definition of an order?
- Does Inventory own stock availability and reservation rules?
- Does Payments own payment authorization and capture?
- Does Billing own financial obligations and invoices?
- Does Fulfillment own shipment planning and dispatch?
- Does Customer Support need its own representation of a customer?

Then decide how these contexts should communicate and where translation is necessary.

For each relationship, document the upstream and downstream context, the contract, the translation responsibility, and the reason for the chosen pattern.

Finally, challenge the design:

- Are any contexts unnecessarily fragmented?
- Are unrelated business concepts forced into one model?
- Are there hidden dependencies through shared databases?
- Are teams depending on internal models they do not own?
- Are some boundaries based on technical convenience rather than business meaning?

The objective is not to produce the largest or most sophisticated Context Map. It is to produce a map that helps the organization understand and manage the relationships between its models.

---

## 8. Common Mistakes

### Treating Subdomains and Bounded Contexts as the Same Thing

A Subdomain describes the business problem space. A Bounded Context defines the boundary of a model.

Confusing them can lead to software boundaries that do not reflect the actual complexity of the business.

### Assuming One Subdomain Must Become One Microservice

Subdomains and Bounded Contexts do not dictate deployment architecture.

Start with meaningful model boundaries. Make deployment decisions based on additional operational and organizational requirements.

### Creating Too Many Bounded Contexts

Every difference in terminology or responsibility does not justify a separate context.

Excessive fragmentation introduces integration overhead, duplicated concepts, and unnecessary coordination.

### Creating Too Few Bounded Contexts

The opposite mistake is forcing several distinct models into one universal representation.

This often results in overloaded terminology, conditional logic, and models that become increasingly difficult to understand.

### Building Context Maps That Show Only Technical Connections

A diagram of API calls and message brokers does not explain the semantic relationship between models.

A useful Context Map captures ownership, dependencies, contracts, translation, and design reasoning.

### Treating Integration Patterns as Mandatory Architecture

Context Mapping patterns describe relationships and provide design guidance. They are not a checklist that every system must implement.

Use the simplest relationship that protects the model and supports the business.

### Ignoring Organizational Boundaries

Model boundaries and team boundaries influence one another.

If a team owns a model but cannot make decisions about its language, rules, or contracts, its autonomy may be more theoretical than real.

Consider organizational responsibilities when designing and evolving Bounded Contexts.

---

## 9. Completion Criteria

You should be able to:

- Identify Subdomains by understanding business capabilities and responsibilities.
- Distinguish Core, Supporting, and Generic Subdomains.
- Explain why Subdomain classification depends on business strategy.
- Distinguish the problem space from the solution space.
- Explain what a Bounded Context protects and why it exists.
- Recognize that one Subdomain may map to multiple Bounded Contexts, and vice versa.
- Identify candidate Bounded Context boundaries using language, responsibilities, rules, and model consistency.
- Explain the difference between a Bounded Context and a microservice.
- Create a Context Map that captures semantic and technical relationships.
- Identify upstream and downstream contexts.
- Choose and justify appropriate Context Mapping patterns.
- Document ownership, contracts, translation responsibilities, and design decisions.
- Recognize when boundaries are creating unnecessary fragmentation or coupling.

---

**Subdomains help us understand where the business has complexity and creates value. Bounded Contexts help us give that complexity coherent models. Context Maps help us understand how those models relate.**

The goal is not to divide a business into as many pieces as possible. It is to discover boundaries that make meaning explicit and allow each model to evolve without losing sight of the larger system.