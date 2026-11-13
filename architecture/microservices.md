# Microservices

Microservices are independently deployable services that communicate over process boundaries. They may align with Bounded Contexts, but DDD does not require a microservice per context.

Consider them when independent deployment, scaling, ownership, or technology needs justify distributed-system complexity. Account for network failure, observability, data ownership, versioning, security, deployment automation, and eventual consistency. A modular monolith may be a better first step.
