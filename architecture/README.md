# Architecture and DDD

DDD is concerned with modeling the domain and defining boundaries. Architecture provides structures for implementing and operating those boundaries. No architectural style is inherently required by DDD. Choose based on coupling, deployment, team ownership, operational capacity, and the rate of change.

## Topics

- [Layered Architecture](layered-architecture.md)
- [Hexagonal Architecture](hexagonal-architecture.md)
- [Clean Architecture](clean-architecture.md)
- [Modular Monolith](modular-monolith.md)
- [Microservices](microservices.md)
- [Event-Driven Architecture](event-driven-architecture.md)
- [CQRS and Event Sourcing](../roadmap/08-advanced-topics.md)

## A useful distinction

A Bounded Context is a boundary around a model. A module is a code organization boundary. A deployable service is an operational boundary. They may align, but they are not the same thing and should not be assumed to map one-to-one.
