# Core Distinctions

| Concept A | Concept B | Difference |
|---|---|---|
| Entity | Value Object | Entity has identity across changes; Value Object is defined by its attributes and is commonly immutable. |
| Subdomain | Bounded Context | Subdomain describes the business problem space; Bounded Context defines the boundary of a model. |
| Domain Event | Integration Event | Domain Event expresses an internal domain fact; Integration Event is a contract published for other systems or contexts. |
| Domain Service | Application Service | Domain Service expresses domain behavior that does not naturally belong to an Entity or Value Object; Application Service coordinates a use case. |
| Core Domain | Generic Subdomain | Core Domain differentiates the business; Generic Subdomain is common capability that may be bought or reused. |
| Model boundary | Deployment boundary | A semantic boundary need not be an independently deployed service. |

## Questions before choosing a pattern

- What business rule or source of complexity are we addressing?
- Which language and model are valid here?
- Who owns the decision and the data?
- What changes together, and what can change independently?
- What is the simplest design that preserves the required meaning and consistency?
