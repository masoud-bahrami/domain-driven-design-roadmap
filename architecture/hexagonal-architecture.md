# Hexagonal Architecture

Hexagonal Architecture (Ports and Adapters) keeps the application and domain logic independent from external technologies. The core defines ports; adapters connect those ports to databases, message brokers, user interfaces, and external services.

It complements DDD by protecting the domain model from infrastructure details. Use it when the system has multiple interaction mechanisms or when infrastructure needs to change independently. Avoid creating abstractions without a real boundary or a meaningful variation to protect.
