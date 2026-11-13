# Architecture Choices

| Style | Useful when | Watch for |
|---|---|---|
| Layered | Familiar separation of responsibilities is sufficient | Pass-through layers and persistence-driven domain models |
| Hexagonal | External systems and technologies should be isolated behind ports and adapters | Abstractions without meaningful variation |
| Clean Architecture | Dependency direction must protect business policy | Mistaking folder structure for a good domain model |
| Modular Monolith | Strong internal boundaries are needed without distributed deployment | Hidden cross-module coupling |
| Microservices | Independent deployment, ownership, or scaling justifies distribution | Network failure, observability, data ownership, and operational complexity |
| Event-Driven | Asynchronous reactions and decoupled integration are valuable | Duplicate delivery, ordering, retries, and schema evolution |

A Bounded Context, code module, and deployable service are different kinds of boundary.
