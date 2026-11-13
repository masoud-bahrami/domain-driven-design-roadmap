# Layered Architecture

Layered Architecture organizes software into horizontal areas, each responsible for a particular category of work. Instead of structuring the application entirely around technical components or individual features, it separates responsibilities such as presentation, application coordination, domain behavior, and infrastructure.

It is one of the most familiar architectural styles in enterprise software and can provide a straightforward foundation for implementing Domain-Driven Design.

However, the existence of layers does not guarantee good separation of concerns. A system can have separate Presentation, Application, Domain, and Infrastructure projects while its business rules remain scattered across controllers, services, repositories, and database queries.

The most important question is not how many layers an application has, but **whether each layer has a clear responsibility and whether dependencies preserve the integrity of the domain model.**

## The central idea: Separation of responsibilities

A layered architecture divides an application into horizontal areas. Each layer has a particular responsibility and collaborates with other layers through defined boundaries.

A common organization in DDD-oriented applications is:

1. **Presentation Layer:** Handles interaction with users and external clients.
2. **Application Layer:** Coordinates use cases and application workflows.
3. **Domain Layer:** Contains the business model, rules, and behavior.
4. **Infrastructure Layer:** Implements technical capabilities such as persistence, messaging, and external integrations.

A typical conceptual structure looks like this:

```text
+--------------------------------------+
|            Presentation              |
|       HTTP APIs, UI, Controllers     |
+--------------------------------------+
                   |
                   v
+--------------------------------------+
|             Application              |
|      Use Cases, Handlers, DTOs       |
+--------------------------------------+
                   |
                   v
+--------------------------------------+
|               Domain                 |
|  Aggregates, Entities, Value Objects |
|  Domain Services, Domain Events      |
+--------------------------------------+
                   ^
                   |
+--------------------------------------+
|            Infrastructure            |
|  Database, ORM, Messaging, External  |
|  Services, File Storage              |
+--------------------------------------+
```

This diagram represents responsibilities and conceptual relationships. It does not prescribe a universal project structure.

The critical point is that business rules belong to the Domain Layer, application workflows belong to the Application Layer, and technical implementation details belong to Infrastructure.

## The layers

### 1. Presentation Layer

The Presentation Layer provides an entry point through which external actors interact with the application.

It may include:

- REST API controllers and endpoints.
- GraphQL endpoints.
- Web interfaces.
- Mobile application APIs.
- Command-line interfaces.
- Request and response models.
- Input parsing and presentation-specific validation.

Its responsibility is to translate external requests into application operations and translate their results into appropriate responses.

For example, a logistics application may expose an endpoint for dispatching a shipment:

```csharp
[ApiController]
[Route("api/shipments")]
public class ShipmentsController : ControllerBase
{
    private readonly IDispatchShipment _dispatchShipment;

    public ShipmentsController(
        IDispatchShipment dispatchShipment)
    {
        _dispatchShipment = dispatchShipment;
    }

    [HttpPost("{id:guid}/dispatch")]
    public async Task<IActionResult> Dispatch(
        Guid id,
        CancellationToken cancellationToken)
    {
        await _dispatchShipment.ExecuteAsync(
            new ShipmentId(id),
            cancellationToken);

        return NoContent();
    }
}
```

The controller handles HTTP concerns and invokes an application capability.

It should not contain the business rules that determine whether a shipment can be dispatched.

There is an important distinction between presentation validation and business validation.

Checking whether an HTTP request contains a valid identifier is a presentation or input-validation concern. Checking whether a shipment is eligible for dispatch is a domain concern.

### 2. Application Layer

The Application Layer coordinates the execution of use cases.

It defines what the application can do and orchestrates the domain objects and external capabilities required to accomplish those operations.

Typical responsibilities include:

- Handling application commands and queries.
- Loading and saving Aggregates.
- Invoking domain behavior.
- Coordinating multiple domain objects or services.
- Defining application transaction boundaries.
- Managing application-level authorization and workflow coordination.
- Returning application results.
- Initiating integration with external systems through appropriate abstractions.

For example, a dispatch use case might:

1. Receive a dispatch request.
2. Load the Shipment Aggregate.
3. Invoke its `Dispatch()` operation.
4. Persist the resulting state.
5. Ensure resulting events are recorded or published through the appropriate mechanism.

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
            throw new ShipmentNotFoundException(
                command.ShipmentId);

        shipment.Dispatch();

        await _shipments.SaveAsync(
            shipment,
            cancellationToken);
    }
}
```

The handler coordinates the use case. It does not decide independently whether dispatching is valid.

That decision belongs to the domain model.

The Application Layer should generally remain relatively thin, but thin does not mean meaningless. Complex workflows, authorization coordination, transaction management, and interactions across multiple Aggregates may legitimately require substantial application-level logic.

### 3. Domain Layer

The Domain Layer represents the business concepts, rules, and behavior of the Bounded Context.

It is the most important layer from a DDD perspective because it contains the model through which the team expresses and implements its understanding of the business.

It may include:

- Entities.
- Value Objects.
- Aggregates and Aggregate Roots.
- Domain Services.
- Domain Events.
- Domain policies and specifications.
- Domain-level abstractions where appropriate.

Consider the Shipment Aggregate:

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

This is a simplified example, but it illustrates an important principle: the domain object owns the business operation and protects the invariant.

The Domain Layer should express the Ubiquitous Language of its Bounded Context. Its types, operations, and relationships should communicate business meaning rather than merely reproduce database structures.

A Domain Layer containing only classes with public getters and setters is not necessarily a meaningful domain model.

### 4. Infrastructure Layer

The Infrastructure Layer contains the technical implementations required by the application.

It may include:

- Database contexts and ORM configurations.
- Repository implementations.
- SQL queries and persistence mappings.
- Message broker integrations.
- External API clients.
- Caching implementations.
- File and object storage.
- Logging and monitoring integrations.
- Framework-specific configuration.

For example, the application may define an `IShipmentRepository` contract, while Infrastructure implements it using Entity Framework Core and SQL Server.

```csharp
public class EfShipmentRepository : IShipmentRepository
{
    private readonly ShippingDbContext _context;

    public EfShipmentRepository(ShippingDbContext context)
    {
        _context = context;
    }

    public Task<Shipment?> GetByIdAsync(
        ShipmentId shipmentId,
        CancellationToken cancellationToken)
    {
        return _context.Shipments
            .FirstOrDefaultAsync(
                x => x.Id == shipmentId,
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

Infrastructure provides the implementation. The Application Layer depends on the contract rather than the concrete persistence technology.

This distinction becomes particularly important when using an ORM that encourages persistence-oriented object models. The database schema should not dictate the meaning and behavior of the domain model.

## Dependency direction matters

A traditional layered architecture is often represented as a stack:

```text
Presentation
     |
     v
Application
     |
     v
Domain
     ^
     |
Infrastructure
```

However, the exact dependency rules vary between layered implementations.

In a DDD-oriented architecture, the Domain Layer should not depend on Infrastructure or Presentation. Application code should also avoid depending directly on concrete infrastructure implementations.

Infrastructure can implement abstractions defined by the inner layers.

This is commonly achieved through Dependency Inversion and dependency injection.

For example:

```text
Application
     |
     v
IShipmentRepository
     ^
     |
EfShipmentRepository
     |
     v
SQL Server
```

The Application Layer defines the contract it needs. Infrastructure supplies the implementation.

This means the runtime call flow and compile-time dependency direction are not necessarily the same.

An application use case can call a repository implementation at runtime without having a compile-time dependency on its concrete class.

## Layered Architecture and DDD

Layered Architecture is a structural approach. DDD is an approach to understanding and designing software around the business domain.

They complement each other, but they are not interchangeable.

| Concern | DDD | Layered Architecture |
|---|---|---|
| Primary focus | Business understanding and domain modeling | Separation of technical and logical responsibilities |
| Main question | What does the business mean, and how should we represent it? | Where should each responsibility live? |
| Important concepts | Ubiquitous Language, Bounded Contexts, Aggregates, Entities, Value Objects | Presentation, Application, Domain, Infrastructure |
| Main benefit | A model that reflects business knowledge and behavior | A recognizable structure with explicit responsibility boundaries |
| Typical failure | A model disconnected from business reality | Layers that become pass-through wrappers or contain misplaced responsibilities |

DDD gives meaning to the Domain Layer. Layered Architecture provides a way to organize the implementation around that model.

For example, a `Shipment` class should not exist merely because the database contains a `Shipments` table. It should represent a meaningful business concept within the Shipment Management context.

Likewise, a `DispatchShipmentHandler` should not become the owner of every dispatch rule simply because it belongs to the Application Layer.

**The layers should organize the model, not dictate what the model means.**

## Layered Architecture within a Bounded Context

In DDD, a Bounded Context defines the boundary within which a particular model and language are consistent.

Layered Architecture can be applied independently within each Bounded Context.

For example:

```text
src/
├── ShipmentManagement/
│   ├── Presentation/
│   ├── Application/
│   ├── Domain/
│   └── Infrastructure/
│
├── FleetManagement/
│   ├── Presentation/
│   ├── Application/
│   ├── Domain/
│   └── Infrastructure/
│
└── FuelAllocation/
    ├── Presentation/
    ├── Application/
    ├── Domain/
    └── Infrastructure/
```

This organization can help teams keep each context's model and responsibilities separate.

However, it is not mandatory to create four projects for every context. The same separation can exist through folders, namespaces, modules, or other structures.

Nor does each Bounded Context need its own deployment unit.

A modular monolith can contain several Bounded Contexts, each organized using layers, while remaining a single deployable application.

The important point is to preserve semantic ownership and avoid creating a shared Domain Layer that combines unrelated business models.

## Common problems and architectural traps

Layered Architecture is familiar and straightforward, but it has several failure modes.

### 1. Pass-through layers

A pass-through layer forwards requests to the next layer without adding meaningful responsibility.

For example:

```text
Controller
    |
    v
ShipmentService
    |
    v
ShipmentManager
    |
    v
ShipmentProcessor
    |
    v
ShipmentRepository
```

If every component simply forwards the call, the architecture has introduced indirection without improving separation.

A layer should exist because it has a responsibility, not because a diagram looks incomplete without it.

### 2. Anemic Domain Model

One of the most damaging problems in DDD-oriented layered systems is a Domain Layer containing only data structures.

For example:

```csharp
public class Shipment
{
    public Guid Id { get; set; }
    public ShipmentStatus Status { get; set; }
    public DateTime? DispatchedAt { get; set; }
}
```

The business rules then accumulate in application services:

```csharp
public async Task Dispatch(Guid shipmentId)
{
    var shipment = await _repository.GetByIdAsync(shipmentId);

    if (shipment.Status != ShipmentStatus.ReadyForDispatch)
        throw new InvalidOperationException();

    shipment.Status = ShipmentStatus.Dispatched;
    shipment.DispatchedAt = DateTime.UtcNow;

    await _repository.SaveAsync(shipment);
}
```

The problem is not that the class contains properties. It is that business behavior and invariants have been placed outside the object responsible for representing them.

A richer domain model could express the operation directly:

```csharp
shipment.Dispatch(clock.UtcNow);
```

The appropriate implementation depends on the domain, but the business decision should remain in the domain model.

### 3. Database-driven domain design

Another common problem occurs when the domain model is designed as a reflection of database tables.

For example:

```text
Shipment
ShipmentDetail
ShipmentStatus
ShipmentHistory
ShipmentDriver
ShipmentVehicle
```

These may be reasonable persistence structures, but they do not automatically describe the domain model.

The business might distinguish between assigning a vehicle, preparing a shipment, dispatching it, and confirming its departure. Those distinctions should emerge from domain exploration and Ubiquitous Language, not from the existing schema.

Persistence models and domain models may overlap, but they solve different problems.

### 4. Application services that become domain services

The Application Layer often becomes overloaded because developers place every business rule in application services.

A useful distinction:

- Application services coordinate a use case.
- Domain services express domain behavior that does not naturally belong to a particular Entity or Value Object.

For example, an application handler may load two Aggregates and coordinate a workflow. A domain policy may determine whether a particular combination of business conditions permits an operation.

The location should follow the responsibility, not the class name.

### 5. Infrastructure leaking into the domain

If domain classes inherit from ORM-specific base classes, use framework attributes extensively, or depend on infrastructure services, technical concerns begin influencing the model.

Some persistence technologies require compromises, and not every framework interaction can be eliminated. But the domain should not become a passive representation of what the ORM finds convenient.

Keep technical dependencies at the appropriate boundaries.

### 6. Treating every layer as a separate project

Separate projects can enforce compile-time boundaries, but they also introduce configuration, dependency management, build complexity, and additional friction.

For a small application, folders and namespaces may be sufficient.

For a larger system, separate projects can provide useful enforcement.

The decision should reflect the size of the system, team organization, and importance of the boundaries.

### 7. Shared layers that undermine Bounded Contexts

A common organizational mistake is to create one global Domain project containing entities, enums, and services used by every module.

This may look like reuse, but it can gradually create a shared model whose terminology and assumptions are inconsistent.

For example, `Customer` may mean a buyer in one context and a credit-account holder in another.

Sharing the same class can force both contexts into a model that accurately represents neither.

Share concepts only when they genuinely have the same meaning and ownership. Otherwise, let each context model the concept according to its own language.

## When to Use Layered Architecture

Layered Architecture is a reasonable choice when:

- The team benefits from a familiar and understandable structure.
- The application has recognizable presentation, application, domain, and infrastructure responsibilities.
- The domain model needs protection from persistence and framework concerns.
- The system is not so complex that more specialized architectural boundaries are necessary.
- The team wants a straightforward foundation that can evolve as the domain becomes better understood.
- The architecture can enforce dependency direction without excessive abstractions.

It is often a good starting point for a DDD-oriented modular monolith.

However, it should not be selected simply because it is familiar. If the application has complex external integrations, multiple delivery mechanisms, or strong requirements for independent adapters, Hexagonal Architecture may offer a more explicit way to organize those interactions.

These styles are not mutually exclusive. A layered application can use Ports and Adapters at its boundaries.

## Practical Guidelines

When applying Layered Architecture in a DDD system:

1. Define each layer by its responsibility, not just its folder name.
2. Keep domain behavior and invariants in the Domain Layer.
3. Use the Application Layer to coordinate use cases.
4. Keep presentation concerns out of business logic.
5. Keep persistence and external integration implementations in Infrastructure.
6. Make dependency direction explicit and protect inner layers from concrete infrastructure dependencies.
7. Do not design domain objects merely by copying database tables.
8. Avoid pass-through layers that introduce no meaningful responsibility.
9. Apply the structure within Bounded Contexts rather than creating one universal domain model.
10. Use separate projects when compile-time enforcement justifies their cost.
11. Do not confuse a thin Application Layer with an anemic Domain Model.
12. Allow the architecture to evolve as the complexity and needs of the domain become clearer.

## A Practical Exercise

Take an existing shipment management module and organize it using Layered Architecture.

Identify the responsibilities of each layer.

**1. Presentation**

Where do HTTP requests, message consumers, and external client inputs enter the application?

**2. Application**

Which use cases does the application expose? Which operations require coordination, transaction management, or external interactions?

**3. Domain**

Which business concepts, rules, and invariants belong to the Shipment Management model? Does the Shipment Aggregate enforce its own valid state transitions?

**4. Infrastructure**

Which technical implementations provide persistence, messaging, caching, and external integrations?

**5. Dependencies**

Can the Domain Layer be compiled and tested without referencing the database or web framework? Does the Application Layer depend on contracts rather than concrete infrastructure implementations?

**6. Responsibility**

Are any layers merely forwarding calls? Are business decisions hiding in controllers, application handlers, or repositories?

Finally, take one business operation, such as dispatching a shipment, and trace it from the incoming request to the resulting domain state and persistence operation.

The goal is not to maximize the number of layers. It is to make responsibilities and dependencies understandable.

## Completion Criteria

You should be able to:

- Explain the purpose of Layered Architecture.
- Describe the responsibilities of Presentation, Application, Domain, and Infrastructure.
- Explain why dependency direction matters in a DDD implementation.
- Distinguish application coordination from domain behavior.
- Recognize anemic domain models and database-driven designs.
- Identify pass-through layers and unnecessary indirection.
- Organize layers within Bounded Contexts without confusing them with deployment boundaries.
- Decide when separate projects provide useful architectural enforcement.
- Explain how Layered Architecture can coexist with Hexagonal and Clean Architecture.

## Further Reading

- Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software*.
- Vaughn Vernon, *Implementing Domain-Driven Design*.
- Martin Fowler, [Presentation Domain Data Layering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html).
- Martin Fowler, [Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html).
- Robert C. Martin, *Clean Architecture: A Craftsman's Guide to Software Structure and Design*.