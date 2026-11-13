# Subdomains and Bounded Contexts

A **Subdomain** describes a part of the business problem space. A **Bounded Context** defines the boundary within which a particular model and its language are consistent. They are related, but they are not interchangeable: one is a way of understanding the business, the other is a boundary in the solution model.

## Subdomain types

- **Core Domain:** the part of the business that provides meaningful differentiation. It deserves focused domain expertise and modeling effort.
- **Supporting Subdomain:** important to the business but not a primary source of competitive advantage.
- **Generic Subdomain:** a capability that is common enough to buy, reuse, or implement with a standard solution.

These classifications depend on business strategy and can change over time.

## Bounded Contexts

A Bounded Context is a semantic boundary. The same word may have different meanings in different contexts. A `Customer` in Sales may represent a buyer, while a `Customer` in Support may represent the person associated with a case. Forcing both models into one universal representation can create ambiguity and coupling.

## Context Maps

A Context Map makes relationships between contexts visible. It should show more than boxes and arrows: record ownership, upstream/downstream relationships, published contracts, translation responsibilities, and the reasons for choosing each integration pattern.
