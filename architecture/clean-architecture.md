# Clean Architecture

Clean Architecture organizes software so that business rules do not depend on frameworks, databases, user interfaces, or external services. Its central concern is **dependency direction**: outer technical details depend on inner policies, not the other way around.

The architecture aims to keep business logic independent of the technologies used to deliver, persist, and operate the system. This makes it possible to change infrastructure without necessarily rewriting the domain logic.

Clean Architecture can support Domain-Driven Design, but it is not DDD itself. A project can have a perfectly clean dependency graph and still have a weak domain model. It can also have a rich domain model without implementing every layer or abstraction commonly associated with Clean Architecture.

The distinction matters: **DDD helps us understand and model the business; Clean Architecture helps us organize the software so that technical decisions do not dominate that model.**

## The central idea: Dependency Rule

The Dependency Rule states that source-code dependencies should point inward, toward policies and abstractions that are more central to the application.

An outer layer can depend on an inner layer, but an inner layer should not know about the outer layer.

For example:

- The domain model should not depend on Entity Framework Core.
- Application use cases should not depend on ASP.NET Core controllers.
- Business policies should not depend on SQL Server, Redis, RabbitMQ, or a particular cloud provider.
- Infrastructure implementations should depend on interfaces or contracts defined closer to the application or domain policies they support.

Consider a logistics system that handles shipment dispatching.

The business concept of dispatching a shipment should not depend on whether the system uses SQL Server, an HTTP API, or a message broker. Those technologies may be necessary to execute the operation, but they should not define what dispatching means.

The dependency direction should preserve that distinction.

## The typical structure

Clean Architecture is commonly represented as concentric circles. The exact number and names of layers vary between implementations, but a typical organization includes the following:

### 1. Domain Layer

The Domain Layer contains the business model and the rules that define valid business behavior.

It may include:

- Entities and Value Objects.
- Aggregates and Aggregate Roots.
- Domain Services.
- Domain Events.
- Domain policies and specifications.
- Domain-level abstractions, where they express genuine domain concepts or requirements.

For example, a `Shipment` Aggregate might expose operations such as:

```csharp
public class Shipment
{
    public ShipmentId Id { get; private set; }
    public ShipmentStatus Status { get; private set; }

    public void Dispatch()
    {
        if (Status != ShipmentStatus.ReadyForDispatch)
            throw new InvalidOperationException(
                "Only shipments ready for dispatch can be dispatched.");

        Status = ShipmentStatus.Dispatched;
    }
}
```

The example is intentionally small. A real model might also need to consider assigned vehicles, dispatch authorization, dispatch time, business exceptions, and the meaning of readiness.

The important point is that the rule governing whether a shipment can be dispatched belongs to the domain model. It should not exist only inside a controller or a database query.

The Domain Layer should not reference infrastructure frameworks merely to implement this behavior.

### 2. Application Layer

The Application Layer coordinates use cases. It determines what needs to happen to fulfill an application request, while leaving business decisions to the domain model.

Typical responsibilities include:

- Receiving and validating application commands.
- Loading the required Aggregates.
- Invoking domain behavior.
- Coordinating multiple domain objects when a use case requires it.
- Managing transaction boundaries.
- Calling external capabilities through abstractions.
- Returning results or application-level errors.

For example, a `DispatchShipmentHandler` might:

1. Receive a `DispatchShipment` command.
2. Load the corresponding Shipment Aggregate.
3. Ask the Aggregate to dispatch the shipment.
4. Persist the updated Aggregate.
5. Publish or schedule resulting events through an appropriate mechanism.

```csharp
public class DispatchShipmentHandler
{
    private readonly IShipmentRepository _shipments;

    public DispatchShipmentHandler(
        IShipmentRepository shipments)
    {
        _shipments = shipments;
    }

    public async Task Handle(
        DispatchShipment command,
        CancellationToken cancellationToken)
    {
        var shipment = await _shipments.GetByIdAsync(
            command.ShipmentId,
            cancellationToken);

        if (shipment is null)
            throw new ShipmentNotFoundException(command.ShipmentId);

        shipment.Dispatch();

        await _shipments.SaveAsync(
            shipment,
            cancellationToken);
    }
}
```

The handler coordinates the use case. The Aggregate decides whether the requested state transition is valid.

This distinction is fundamental. If the handler contains all the business rules while the domain objects merely hold properties, the application may follow Clean Architecture structurally while drifting away from DDD's emphasis on a behavior-rich domain model.

Not every application needs a separate handler class for every operation. The important thing is the separation of responsibilities, not the number of classes.

### 3. Interface Adapters

The Interface Adapters layer translates between external representations and the application's internal model.

It commonly contains:

- HTTP controllers and API endpoints.
- Request and response models.
- Presenters.
- View models.
- Persistence mapping code.
- Message consumers and producers.
- Adapters for external APIs.

For example, an HTTP endpoint might receive a JSON request and translate it into an application command:

```csharp
public record DispatchShipmentRequest(
    Guid ShipmentId);

[ApiController]
[Route("api/shipments")]
public class ShipmentsController : ControllerBase
{
    private readonly DispatchShipmentHandler _handler;

    public ShipmentsController(
        DispatchShipmentHandler handler)
    {
        _handler = handler;
    }

    [HttpPost("{id:guid}/dispatch")]
    public async Task<IActionResult> Dispatch(
        Guid id,
        CancellationToken cancellationToken)
    {
        await _handler.Handle(
            new DispatchShipment(new ShipmentId(id)),
            cancellationToken);

        return NoContent();
    }
}
```

The controller knows about HTTP. The application handler knows about the use case. The domain model knows about shipment dispatching.

These responsibilities should not be confused.

### 4. Infrastructure and Frameworks

The outermost layer contains technical implementations and external dependencies.

Examples include:

- Entity Framework Core and database providers.
- SQL Server, PostgreSQL, or other persistence technologies.
- Redis and distributed caches.
- RabbitMQ and Kafka.
- External HTTP clients.
- File systems and object storage.
- Logging, telemetry, and monitoring integrations.
- Framework-specific dependency injection and hosting configuration.

Infrastructure implements the capabilities required by inner layers.

For example, the application may define:

```csharp
public interface IShipmentRepository
{
    Task<Shipment?> GetByIdAsync(
        ShipmentId id,
        CancellationToken cancellationToken);

    Task SaveAsync(
        Shipment shipment,
        CancellationToken cancellationToken);
}
```

Infrastructure can implement that contract using Entity Framework Core:

```csharp
public class EfShipmentRepository : IShipmentRepository
{
    private readonly ShippingDbContext _context;

    public EfShipmentRepository(ShippingDbContext context)
    {
        _context = context;
    }

    public Task<Shipment?> GetByIdAsync(
        ShipmentId id,
        CancellationToken cancellationToken)
    {
        return _context.Shipments
            .FirstOrDefaultAsync(
                x => x.Id == id,
                cancellationToken);
    }

    public async Task SaveAsync(
        Shipment shipment,
        CancellationToken cancellationToken)
    {
        _context.Update(shipment);

        await _context.SaveChangesAsync(cancellationToken);
    }
}
```

The example illustrates dependency direction, not a universal repository implementation. In particular, mapping a rich domain model to persistence structures requires deliberate design. Directly updating a domain object through an ORM is not always appropriate, and the persistence strategy must respect Aggregate boundaries and consistency requirements.

The Application Layer depends on `IShipmentRepository`, not on `EfShipmentRepository` or `ShippingDbContext`.

Infrastructure supplies the implementation at runtime through dependency injection.

## How the dependency direction works

A simplified dependency graph might look like this:

```text
          API / Presentation
                  |
                  v
             Application
                  |
                  v
               Domain
                  ^
                  |
             Infrastructure
                  |
                  v
       Database / Broker / External APIs
```

This diagram is conceptual. Infrastructure can implement application-facing interfaces, while the application depends on those interfaces. At runtime, dependency injection connects the implementations to their consumers.

The important distinction is between **compile-time dependencies** and **runtime collaboration**.

An application service may call `IShipmentRepository`. At runtime, the dependency injection container supplies `EfShipmentRepository`. This does not mean the Application Layer depends on Entity Framework Core.

## Clean Architecture and DDD

Clean Architecture and DDD address different but complementary concerns.

| Concern | DDD | Clean Architecture |
|---|---|---|
| Primary focus | Understanding and modeling the business domain | Organizing dependencies and technical responsibilities |
| Main question | What does the business mean, and how should we represent it? | How do we prevent technical details from controlling business policies? |
| Core concepts | Ubiquitous Language, Bounded Contexts, Aggregates, Entities, Value Objects, Domain Events | Dependency Rule, layers, boundaries, ports and adapters |
| Main benefit | A model that reflects business knowledge and behavior | A system whose core policies are less coupled to technical choices |
| Typical failure | A model that does not represent the actual business | Excessive abstraction and layers without meaningful boundaries |

They should not be treated as competing architectural approaches.

A DDD-based system can use Clean Architecture to protect its domain model. However, applying Clean Architecture does not automatically produce a DDD model.

For example, a system can have:

- A `Domain` project containing only property classes.
- An `Application` project containing all business decisions.
- An `Infrastructure` project containing repositories.
- An `API` project containing controllers.

The dependency graph may look clean, but the domain model can still be anemic.

The folder structure does not prove that the business model is well designed.

## Clean Architecture within a Bounded Context

One particularly important consideration in DDD is that architectural boundaries should support domain boundaries.

A Bounded Context defines the boundary within which a particular model and its language are consistent. Clean Architecture does not define these boundaries for us.

For example, a logistics platform might contain separate Bounded Contexts for:

- Shipment Management.
- Fleet Management.
- Fuel Allocation.
- Billing.

Each context may have its own domain model, application use cases, adapters, and infrastructure implementations.

A possible structure is:

```text
src/
├── ShipmentManagement/
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── API/
│
├── FleetManagement/
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── API/
│
├── FuelAllocation/
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── API/
│
└── Billing/
    ├── Domain/
    ├── Application/
    ├── Infrastructure/
    └── API/
```

This is one possible organization, not a mandatory project structure.

The important thing is that each context owns its model and its rules. A shared `Domain` project containing entities from every part of the business can undermine those boundaries, even if the solution follows Clean Architecture's dependency direction.

Likewise, placing every context in a separate deployable service is not a requirement. Bounded Contexts can exist within a modular monolith.

## The double-edged sword: When Clean Architecture becomes harmful

Clean Architecture is useful when it protects meaningful business policies from technical volatility. It becomes counterproductive when its implementation introduces more complexity than it removes.

Several problems deserve particular attention.

### 1. Confusing layers with responsibilities

Creating `Domain`, `Application`, `Infrastructure`, and `Presentation` projects does not automatically establish good separation of concerns.

A use case that simply forwards a request through five classes without adding coordination, policy, or translation may be unnecessary indirection.

The question should not be whether every operation passes through every layer. It should be whether each boundary has a meaningful responsibility.

### 2. Abstracting everything

An interface for every class, a repository for every table, and a service for every operation can create an architecture full of abstractions without meaningful variation or isolation.

For example, creating `IShipmentService`, `ShipmentService`, `IShipmentManager`, `ShipmentManager`, and `ShipmentProcessor` for a single operation does not necessarily improve the design.

Introduce abstractions when they establish a useful boundary, isolate a volatile dependency, express a meaningful contract, or enable a legitimate variation.

Do not introduce them merely because a diagram contains an interface at every boundary.

### 3. Turning the Domain Layer into a collection of data classes

A common implementation mistake is to keep domain entities free of framework references but also free of meaningful behavior.

The result is often a set of classes with getters and setters while all business logic lives in application handlers.

For example:

```csharp
shipment.Status = ShipmentStatus.Dispatched;
```

This may be appropriate in a simple data-oriented model, but if dispatching has business rules, the operation should normally be expressed through domain behavior:

```csharp
shipment.Dispatch();
```

The second expression gives the domain model an opportunity to enforce its own invariants and communicate intent.

Framework independence is not the same as domain richness.

### 4. Treating the repository pattern as a mandatory abstraction

Clean Architecture is often implemented with repository interfaces, but it does not require a repository for every persistence operation.

Repositories are particularly useful when they represent domain-oriented access to Aggregates and hide persistence mechanics.

They are less useful when they merely wrap an ORM's generic CRUD API without adding a meaningful boundary.

The goal is not to hide every database operation behind an interface. The goal is to prevent persistence concerns from taking control of business policy.

### 5. Overengineering simple applications

Not every application needs four projects, multiple abstractions, command handlers, mapping libraries, and elaborate dependency injection configuration.

A small application with limited domain complexity may be better served by a simpler architecture.

Architecture should reflect the complexity and expected evolution of the problem, not the popularity of a particular template.

### 6. Mistaking framework independence for framework avoidance

Clean Architecture does not mean frameworks are inherently bad.

ASP.NET Core, Entity Framework Core, and messaging frameworks can be productive tools. The objective is to keep their influence within appropriate boundaries.

Avoiding a framework at all costs can be just as harmful as coupling the entire domain model to it.

The real question is whether the framework is controlling business decisions or merely helping implement them.

## Practical guidelines

When applying Clean Architecture in a DDD-based system:

1. Discover the Bounded Contexts before deciding how to organize the solution.
2. Keep domain behavior and invariants in the domain model where they belong.
3. Put use-case coordination in the Application Layer.
4. Keep HTTP, messaging, persistence, and external-system concerns at the boundaries.
5. Define abstractions where they establish meaningful dependency boundaries.
6. Avoid turning every class into an interface and every operation into a service.
7. Do not confuse project structure with domain design.
8. Choose the simplest architecture that protects the boundaries that actually matter.
9. Revisit architectural decisions when the domain or its technical constraints change.

## A practical exercise

Take a business operation from an existing system, such as dispatching a shipment.

Identify:

- The business rules that determine whether dispatching is allowed.
- The domain objects responsible for enforcing those rules.
- The application use case that coordinates the operation.
- The external interfaces involved, such as HTTP or messaging.
- The persistence and infrastructure mechanisms required.
- The dependencies between these components.

Then imagine replacing SQL Server with another database, or replacing an HTTP endpoint with a message consumer.

Ask yourself:

- Which parts of the domain model should change?
- Which application policies should remain unchanged?
- Which adapters and infrastructure implementations need to be replaced?
- Are there abstractions that genuinely protect the model, or are some merely adding indirection?

The objective is not to make every technology replaceable at zero cost. It is to understand which decisions should be isolated and why.

## Completion criteria

You should be able to:

- Explain the Dependency Rule and distinguish compile-time dependencies from runtime collaboration.
- Describe the responsibilities of the Domain, Application, Interface Adapters, and Infrastructure layers.
- Explain how Clean Architecture can support DDD without being a DDD methodology.
- Organize architectural components around meaningful Bounded Contexts.
- Identify anemic domain models and unnecessary abstractions even when the project structure looks clean.
- Justify architectural boundaries based on business and technical needs rather than convention.

## Further reading

- Robert C. Martin, *Clean Architecture: A Craftsman's Guide to Software Structure and Design*.
- Alistair Cockburn, [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/).
- Martin Fowler, [Presentation Domain Data Layering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html).
- Vaughn Vernon, *Implementing Domain-Driven Design*.
- Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software*.