# Layered Architecture

Layered architecture separates responsibilities into horizontal areas such as presentation, application, domain, and infrastructure. It can be a straightforward way to organize a DDD implementation, provided dependencies do not cause the domain model to become a passive reflection of persistence.

Use it when the team benefits from familiar separation of concerns and the dependency direction can be kept explicit. Watch for layers that become pass-through wrappers or a domain layer that contains only data structures.
