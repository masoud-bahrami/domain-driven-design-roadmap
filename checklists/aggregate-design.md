# Aggregate Design Checklist

- [ ] State the Aggregate's business purpose.
- [ ] Identify the Aggregate Root.
- [ ] Identify the Entities and Value Objects inside the boundary.
- [ ] Define the invariants that must hold at transaction completion.
- [ ] Define business operations that enforce those invariants.
- [ ] Model references to other Aggregates through their roots or identities.
- [ ] Identify Domain Events emitted by meaningful state changes.
- [ ] Check cohesion, transaction size, and contention.
- [ ] Write tests that demonstrate the invariants.
