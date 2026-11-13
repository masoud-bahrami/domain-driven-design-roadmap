# Context Mapping

A Context Map describes the relationships between Bounded Contexts. It makes dependencies, ownership, integration responsibilities, and organizational constraints explicit.

A Context Map is not simply a diagram showing which systems communicate. It explains how independently evolving domain models relate to one another and what consequences those relationships have for the teams maintaining them.

## Context Mapping Patterns

The patterns below help describe different relationships between Bounded Contexts:

- **Shared Kernel:** Two contexts intentionally share a small, mutually agreed subset of their domain model. Changes to this shared part require coordination because they affect both contexts.

- **Customer-Supplier:** An upstream context provides capabilities or information consumed by a downstream context. The downstream team communicates its requirements, while the upstream team considers them when planning its work.

- **Conformist:** A downstream context adopts the upstream model because influencing the upstream team or changing its model is not practical. This may be acceptable, but it creates a dependency on the upstream model's evolution.

- **Anti-Corruption Layer:** A downstream context translates an external model into its own domain language and concepts. This protects the local model from foreign assumptions, terminology, and structural decisions.

- **Separate Ways:** Two contexts deliberately avoid integration because the cost of coordination and shared functionality outweighs the expected benefits.

- **Published Language:** Contexts communicate through an explicitly defined language or contract that both sides understand. This can reduce ambiguity and make integration more predictable.

These patterns describe different aspects of relationships and can sometimes be combined. For example, a Customer-Supplier relationship may use a Published Language, while the downstream context implements an Anti-Corruption Layer to translate that contract into its own model.

## Relationship Design Checklist

For every relationship between Bounded Contexts, document the following:

| Concern | Questions to answer |
|---|---|
| Upstream and Downstream | Which context provides information or capabilities? Which context consumes them? |
| Ownership | Who owns the model, data, and behavior on each side? |
| Shared Assumptions | Which definitions, rules, or expectations must both sides understand? |
| Relationship Pattern | Which Context Mapping pattern best describes the actual relationship? |
| Integration Mechanism | Do they communicate through APIs, messaging, shared data, or another mechanism? |
| Translation Responsibility | Who translates between models, and where does that translation happen? |
| Change Coordination | How are contract changes communicated, negotiated, and deployed? |
| Known Risks | What coupling, failure modes, organizational dependencies, or compatibility issues exist? |

## Practical Design Questions

Before choosing a pattern, explore the actual relationship:

- Does the downstream context have any influence over the upstream context's priorities?
- Can both teams coordinate changes, or is one team dependent on the other's decisions?
- Does the downstream model genuinely share concepts with the upstream model, or do similarly named concepts have different meanings?
- Is sharing a model worth the coordination cost?
- Should the downstream context adopt the upstream model or protect its own language through translation?
- Is integration essential to the business, or would Separate Ways be a reasonable choice?
- Who is responsible when an integration contract changes or fails?

## Important Distinctions

Do not confuse Context Mapping patterns with technical integration mechanisms.

For example:

- REST does not automatically imply Customer-Supplier.
- Messaging does not automatically imply Published Language.
- Sharing a database does not automatically imply Shared Kernel.
- Implementing an Anti-Corruption Layer does not necessarily mean the relationship is Conformist.
- Deploying two contexts as separate microservices does not automatically make them independent.

The relationship between the models, their ownership, and the organizational dynamics determine the pattern. The technology is a separate implementation decision.

**Note:** A Context Map is a living architectural artifact. Update it when responsibilities, ownership, dependencies, contracts, or team relationships change. A diagram that no longer reflects reality can be more misleading than having no diagram at all.

See [Subdomains and Bounded Contexts](../concepts/subdomains-and-bounded-contexts.md) and [Anti-Corruption Layer](../concepts/anti-corruption-layer.md) for deeper explanations.