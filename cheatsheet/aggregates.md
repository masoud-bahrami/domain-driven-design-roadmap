# Aggregate Design

Use these principles when identifying, designing, or reviewing Aggregates. An Aggregate is a consistency boundary responsible for protecting business invariants, not simply a group of related objects.

- **Start from business invariants, not data relationships.** Identify which business rules must remain true when an operation completes. Do not group objects just because they have foreign keys, belong to the same database table hierarchy, or are frequently loaded together.

- **Put invariant enforcement behind the Aggregate Root.** The Root must control operations that can change the Aggregate's state. External code should not be able to bypass it and leave the Aggregate in an invalid state.

- **Keep the boundary as small as the consistency requirement allows.** Include only the objects that must participate in the same consistency boundary. A larger Aggregate can increase transaction duration, memory usage, locking, and contention.

- **Reference other Aggregates by identity.** Avoid building large object graphs across Aggregate boundaries. Load and coordinate other Aggregates explicitly when a business operation requires them.

- **Prefer explicit business operations over generic setters.** Methods such as `Confirm()`, `Cancel()`, and `Allocate()` communicate business intentions and can validate the conditions under which state changes are allowed. Generic setters often expose implementation details without protecting business rules.

- **Consider contention, transaction size, and loading cost.** Ask how frequently the Aggregate changes, whether concurrent operations are likely, how much data must be loaded, and whether its invariants require all those changes to happen atomically.

- **Use tests to demonstrate the invariants.** Write tests around business scenarios, including valid transitions, rejected operations, boundary conditions, and concurrent modifications where relevant. Verify that invalid states cannot be reached through the Aggregate's public operations.

**A useful design question:** What must be true when this business operation finishes, and what is the smallest consistency boundary capable of guaranteeing it?

See [the Aggregate concept page](../concepts/aggregates.md) for detailed explanations, examples, design guidance, and common mistakes.