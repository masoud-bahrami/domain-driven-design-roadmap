# Tactical Design

Tactical Design provides building blocks for expressing a domain model within a Bounded Context. It helps translate domain knowledge, business rules, and Ubiquitous Language into software structures and behavior.

The goal is not to use every DDD building block. Choose the concepts that accurately represent the business and help protect its rules.

## Tactical Building Blocks

### Entity

An Entity represents a domain concept whose identity matters over time, even when its attributes change.

- Give it a meaningful identity within its model.
- Define its lifecycle and the transitions it can undergo.
- Place behavior related to its responsibility alongside its state.
- Protect important rules when its state changes.
- Avoid treating every database record as a domain Entity.

**Ask:** If this object's attributes change, does the business still consider it the same thing?

### Value Object

A Value Object represents a concept defined by its attributes rather than identity.

- Use value-based equality.
- Prefer immutability so its meaning cannot change unexpectedly.
- Validate its constraints when it is created.
- Encapsulate related values and behavior into meaningful concepts.
- Avoid using primitive types when a richer domain concept would clarify meaning.

For example, `Money`, `Address`, and `DateRange` can express domain concepts more accurately than unrelated primitive values.

**Ask:** Does the business care about the identity of this concept, or only what it represents?

### Aggregate

An Aggregate defines a consistency boundary around a group of domain objects.

- Start with business invariants.
- Identify which objects must participate in the same consistency boundary.
- Keep the boundary as small as the consistency requirements allow.
- Define operations that preserve valid state transitions.
- Reference other Aggregates by identity.
- Consider concurrency, transaction size, and loading requirements.

An Aggregate is not simply a collection of related Entities or a representation of a database relationship.

**Ask:** What must be true when this business operation completes, and which boundary is responsible for enforcing it?

### Aggregate Root

The Aggregate Root is the single entry point for accessing and modifying an Aggregate.

- Protect the invariants of the entire Aggregate.
- Expose meaningful business operations rather than unrestricted setters.
- Prevent external code from bypassing the boundary.
- Control changes to internal Entities and Value Objects.
- Coordinate meaningful state transitions and Domain Events.

Every Aggregate has one Root, but the Root does not need to contain all the business behavior itself. It may delegate responsibilities to internal objects while preserving control over the boundary.

**Ask:** Can any external operation modify the Aggregate without passing through its Root?

### Domain Event

A Domain Event represents an immutable fact that something meaningful has happened in the domain.

- Use past-tense names that express business meaning.
- Capture the relevant information about the occurrence.
- Raise events as a consequence of meaningful domain behavior.
- Distinguish business occurrences from technical notifications.
- Consider whether the event is internal to the context or needs to be published externally.

Examples include `OrderConfirmed`, `ShipmentDispatched`, and `PaymentReceived`.

**Ask:** Would a domain expert recognize this as something meaningful that happened in the business?

### Domain Service

A Domain Service expresses domain behavior that does not naturally belong to one Entity or Value Object.

- Use it when an operation involves domain concepts but does not have a natural object owner.
- Keep its behavior focused on a meaningful domain responsibility.
- Use domain-oriented inputs and outputs.
- Avoid turning it into a general-purpose collection of business operations.
- Keep infrastructure coordination outside the domain service unless the domain behavior genuinely requires an abstraction for an external capability.

A Domain Service is not simply a class containing business logic that was difficult to place elsewhere.

**Ask:** Is this operation genuinely part of the domain but independent of the responsibility of any single Entity or Value Object?

### Repository

A Repository provides a collection-like abstraction for retrieving and persisting Aggregates.

- Define Repository interfaces around domain-relevant retrieval and persistence needs.
- Work with Aggregate Roots rather than exposing arbitrary database tables.
- Hide persistence details from the domain model.
- Keep query requirements separate when they do not need to return domain Aggregates.
- Avoid creating a Repository for every Entity or database table.

A Repository should help the application work with domain objects without knowing how they are stored.

**Ask:** Does this abstraction help retrieve and persist an Aggregate, or is it merely exposing database operations through another interface?

### Factory

A Factory encapsulates object creation when construction is complex, requires coordination, or must enforce important creation rules.

- Use meaningful creation operations.
- Validate required information and business constraints.
- Ensure newly created objects begin in a valid state.
- Hide construction details when they would otherwise complicate the caller.
- Prefer constructors or simple creation methods when they are sufficient.

Factories are useful when creation itself represents a meaningful domain operation, not merely because an object has several properties.

**Ask:** Does creating this object involve enough domain knowledge or complexity to deserve an explicit creation mechanism?

## Designing with Tactical Patterns

When modeling a business capability, work through these questions:

1. **Understand the language:** What concepts and actions do domain experts use? What do they mean in concrete scenarios?
2. **Identify behavior:** What can happen to these concepts? Which operations represent meaningful business intentions?
3. **Discover identity:** Which concepts have identity and lifecycle? Which are defined entirely by their values?
4. **Find invariants:** Which rules must remain true when business operations complete?
5. **Define Aggregates:** What is the smallest boundary capable of protecting those invariants?
6. **Assign responsibilities:** Which behavior belongs to Entities, Value Objects, Domain Services, or other domain concepts?
7. **Define persistence:** How should Aggregates be retrieved and persisted without allowing storage structures to dictate the model?
8. **Identify events:** Which meaningful business facts should be represented as Domain Events?
9. **Test behavior:** Can the model demonstrate valid operations, rejected operations, and important business scenarios?

## Common Modeling Mistakes

- Turning every noun into an Entity.
- Turning every database table into an Aggregate.
- Treating Aggregates as object graphs rather than consistency boundaries.
- Using generic setters that allow invalid state transitions.
- Moving business rules into Application Services until the domain model becomes an anemic data structure.
- Creating Domain Services for behavior that naturally belongs to an Entity or Value Object.
- Publishing every internal state change as an Integration Event.
- Creating repositories for individual tables rather than Aggregate boundaries.
- Introducing factories for trivial object construction.
- Modeling technical requirements instead of business concepts.

## Practical Principle

Tactical patterns are tools for expressing a domain model, not a checklist of classes that every application must contain.

Start with business behavior and rules. Discover the concepts responsible for that behavior, establish the consistency boundaries that protect important invariants, and use abstractions only when they clarify the model.

**Do not mechanically turn every noun into an Entity or every table into an Aggregate. Let the domain language and its rules shape the design.**