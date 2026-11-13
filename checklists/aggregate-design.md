### Aggregate Design Checklist

- [ ] State the Aggregate's business purpose and responsibility.
- [ ] Identify the Aggregate Root that controls access to the boundary.
- [ ] Identify the Entities and Value Objects inside the Aggregate.
- [ ] Define the business invariants that must hold when each transaction completes.
- [ ] Define business operations that enforce those invariants and protect valid state transitions.
- [ ] Model references to other Aggregates through their identities, rather than large object graphs.
- [ ] Identify Domain Events emitted by meaningful business state changes.
- [ ] Evaluate cohesion, transaction size, loading requirements, and concurrency contention.
- [ ] Write tests that demonstrate the invariants and verify important business scenarios.