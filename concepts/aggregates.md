# Aggregates

An Aggregate is a cluster of domain objects treated as a consistency boundary. The Aggregate Root is the single entry point through which external code interacts with the Aggregate. The root protects invariants that must hold when a transaction completes.

Design Aggregates around business invariants, not around object graphs or database relationships. Keep them as small as the consistency rules allow. Reference other Aggregates by identity rather than building large interconnected object graphs.

Questions to ask:

- Which rules must be true immediately after this operation?
- Which concepts must change atomically to preserve those rules?
- Can this rule be enforced within one Aggregate?
- Does the proposed boundary create unnecessary contention or loading?
- Are commands expressed as meaningful business operations?

An Aggregate is not automatically a database table, a collection of all related entities, or a microservice.
