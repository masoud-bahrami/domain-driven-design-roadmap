# Strategic Design

Strategic Design helps you understand the larger business domain, identify its important capabilities, and establish meaningful boundaries between different models.

The goal is not to divide a system into services or modules immediately. It is to understand how the business works, where its responsibilities differ, and where separate models are necessary.

## Strategic Design Process

### 1. Identify Business Capabilities and Subdomains

Explore what the business does and identify the major capabilities required to deliver its products or services.

- Talk to domain experts and people responsible for actual business operations.
- Identify important business activities, responsibilities, and outcomes.
- Group related activities into meaningful business capabilities.
- Explore how those capabilities depend on one another.
- Identify the business problems and complexity each capability addresses.

**Questions to ask:**
- What does the business actually do?
- Which capabilities are essential to delivering value?
- Which activities represent distinct areas of business knowledge?
- Where do responsibilities overlap?

**Outcome:** An initial decomposition of the business domain into capabilities and candidate subdomains.

### 2. Classify Subdomains

Classify subdomains according to their strategic importance and competitive impact.

- **Core:** Capabilities that differentiate the business and deserve focused investment in domain understanding and implementation.
- **Supporting:** Capabilities necessary for the business but not usually a primary source of competitive advantage.
- **Generic:** Common capabilities that can often be satisfied through existing products, platforms, or standard solutions.

Consider the business strategy rather than the technical complexity of a capability. A technically complicated subsystem is not necessarily part of the Core Domain.

**Questions to ask:**
- Which capabilities create competitive advantage?
- Where does the business have unique knowledge or processes?
- Which capabilities are important but not differentiating?
- Could a standard product or external service satisfy this need?

**Outcome:** A strategic view of where the organization should invest its modeling and engineering effort.

### 3. Discover Where Language, Rules, and Models Change Meaning

Explore how different parts of the business understand and represent similar concepts.

Pay particular attention to language because identical terms do not necessarily represent identical business concepts.

- Collect terminology used by different business roles.
- Identify ambiguous terms and competing definitions.
- Explore the same concept in different business scenarios.
- Compare business rules, responsibilities, and lifecycle expectations.
- Identify where one model's assumptions stop being valid.

For example, the term `Customer` may represent a person placing an order in one context, an account holder in another, and a contractual party in a third.

These may be related concepts, but forcing them into one universal model can create unnecessary coupling and ambiguity.

**Questions to ask:**
- Does this term mean the same thing across the business?
- Are the same business rules applied everywhere?
- Do different teams make different assumptions about the same concept?
- Would sharing one model clarify or obscure these differences?

**Outcome:** A better understanding of semantic boundaries and candidate Bounded Contexts.

### 4. Define Bounded Contexts

Define Bounded Contexts around coherent models, responsibilities, and language.

A Bounded Context establishes where a particular domain model is valid and where its terminology has consistent meaning.

- Group concepts that belong to a coherent model.
- Establish clear responsibilities and ownership.
- Define what belongs inside and outside each context.
- Identify important domain concepts and business rules.
- Establish the Ubiquitous Language for each context.
- Validate proposed boundaries with domain experts and the teams involved.

Do not assume that every Subdomain maps directly to one Bounded Context. A Subdomain belongs to the business problem space, while a Bounded Context belongs to the solution model. Their relationship must be discovered and designed.

**Questions to ask:**
- Does this group of concepts form a coherent model?
- Can the team use a consistent language within this boundary?
- Are its business rules and responsibilities clear?
- Would separating these concepts reduce ambiguity or unnecessary dependencies?

**Outcome:** Explicit model boundaries with clear responsibilities and a language that is valid within each context.

### 5. Map Relationships, Ownership, and Integration Patterns

Explore how Bounded Contexts depend on and communicate with one another.

- Identify upstream and downstream relationships.
- Establish ownership of models, data, and decisions.
- Understand the influence teams have over one another.
- Select appropriate Context Mapping patterns.
- Define integration contracts and communication mechanisms.
- Identify where translation is required to protect local models.
- Record assumptions, dependencies, and integration risks.

Use patterns such as Customer-Supplier, Shared Kernel, Conformist, Anti-Corruption Layer, Published Language, and Separate Ways when they accurately describe the relationship.

Remember that a Context Map is not just a system diagram. It communicates the relationships between models, the responsibilities of teams, and the consequences of dependencies.

**Questions to ask:**
- Who owns the upstream model?
- Who controls changes to the integration contract?
- Where does translation between models happen?
- What happens when one context changes?
- Is the integration worth its coordination and maintenance cost?

**Outcome:** A Context Map that makes dependencies, ownership, integration patterns, and risks explicit.

### 6. Revisit Boundaries as Understanding Evolves

Strategic Design is not a one-time decomposition exercise.

As the business changes and the team learns more about the domain, previously reasonable boundaries may become inadequate.

- Review terminology when new distinctions emerge.
- Reconsider boundaries when responsibilities change.
- Evaluate whether existing dependencies are creating unnecessary coupling.
- Reassess Core, Supporting, and Generic classifications as business strategy evolves.
- Update ownership and Context Maps when organizational responsibilities change.
- Refine models based on actual integration and operational experience.

**Questions to ask:**
- Does the current model still represent the business accurately?
- Have responsibilities or business capabilities changed?
- Are teams repeatedly struggling with the same boundary?
- Has a previously generic capability become strategically important?
- What new understanding justifies changing the model?

**Outcome:** A domain model and Context Map that evolve with the business rather than becoming outdated architectural documentation.

## Important Principle

Strategic Design is about understanding and organizing business knowledge, not mechanically decomposing an application.

A Subdomain is not automatically a Bounded Context. A Bounded Context is not automatically a module or microservice. And a Context Map is not merely a diagram of technical communication.

**The goal is to establish boundaries where they preserve meaning, clarify responsibility, and make change easier to manage.**