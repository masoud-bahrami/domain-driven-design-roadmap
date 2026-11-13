# Events, CQRS, and Event Sourcing

- A Domain Event is a fact that happened in the domain.
- An Integration Event is a published contract for another context or system.
- CQRS separates the model used to handle commands from the model used to answer queries. It does not require separate databases or microservices.
- Event Sourcing persists state changes as an event sequence and reconstructs state by replaying events. It introduces significant requirements around event evolution, projections, replay, and operational tooling.

Use these techniques when their specific benefits justify their costs. They are not prerequisites for DDD.
