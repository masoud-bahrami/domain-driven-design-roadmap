# Aggregate Design

- Start from business invariants, not data relationships.
- Put the invariant enforcement behind the Aggregate Root.
- Keep the boundary as small as the consistency requirement allows.
- Reference other Aggregates by identity.
- Prefer explicit business operations over generic setters.
- Consider contention, transaction size, and loading cost.
- Use tests to demonstrate the invariants.

See [the Aggregate concept page](../concepts/aggregates.md).
