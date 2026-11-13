# Tactical Design

- **Entity:** identity matters over time.
- **Value Object:** attributes define equality; prefer immutability.
- **Aggregate:** consistency boundary around invariants.
- **Aggregate Root:** controls access and protects Aggregate invariants.
- **Domain Event:** immutable fact that something happened.
- **Domain Service:** domain operation that does not naturally belong to one Entity or Value Object.
- **Repository:** collection-like abstraction for retrieving and persisting Aggregates.
- **Factory:** encapsulates creation when construction is complex or invariant-sensitive.

Start with behavior and rules. Do not mechanically turn every noun into an Entity or every table into an Aggregate.
