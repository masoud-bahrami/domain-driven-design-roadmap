# Hexagonal Architecture

Hexagonal Architecture, also known as **Ports and Adapters**, is an architectural style introduced by Alistair Cockburn. It aims to keep an application's business capabilities independent of the technologies and mechanisms through which the application communicates with the outside world.

Its central idea is that an application should not be designed around a particular database, user interface, message broker, or external service. Instead, the application defines the interactions it needs, and external components connect to it through explicit boundaries.

These boundaries are called **Ports**, and the components that connect external technologies to those ports are called **Adapters**.

Hexagonal Architecture complements Domain-Driven Design by protecting the domain model and application behavior from infrastructure details. It is particularly useful when a system has multiple interaction mechanisms, when external dependencies need to change independently, or when application behavior needs to be tested without requiring real infrastructure.

However, Hexagonal Architecture is not a requirement for DDD. Like other architectural styles, it should be adopted when its boundaries solve a real problem rather than because every dependency is expected to have an interface.

## The central idea: The application is at the center

Traditional application structures often organize software around technical delivery mechanisms.

For example, an application might be designed primarily around an HTTP API, with controllers calling database-oriented services and those services containing business logic.

This can make the application dependent on the technologies through which it is accessed.

Hexagonal Architecture reverses that perspective.

The application is the center. It exposes capabilities through ports and uses other ports to communicate with external systems.

External technologies connect through adapters.

The familiar hexagon is a conceptual representation, not a requirement to have exactly six sides or six components. Each side represents a possible interaction boundary.

<escape>
```text
       HTTP Adapter              Message Consumer
             |                          |
             v                          v
        +-----------------------------------+
        |                                   |
        |          APPLICATION              |
        |                                   |
        |     Use Cases and Domain Model    |
        |                                   |
        +-----------------------------------+
             ^             |
             |             v
       CLI Adapter     Database Adapter
                           |
                           v
                      SQL Server

                  External API Adapter
                           |
                           v
                     External System
```
</escape>

The application does not need to know whether a request came from an HTTP endpoint, a scheduled job, a message consumer, or a command-line interface.

Likewise, it does not need to know whether its data is persisted in SQL Server, PostgreSQL, or another storage technology.

It interacts through ports that express the capabilities and contracts it requires.

## Ports: Defining meaningful boundaries

A Port represents an interaction contract between the application and something outside it.

Ports are not merely interfaces introduced to satisfy dependency injection. They describe how the application communicates with its environment.

There are two major categories.

### 1. Driving Ports (Primary Ports)

Driving Ports expose application capabilities to the outside world.

They represent operations that external actors can request the application to perform.

Examples include:

- Dispatch a shipment.
- Reserve inventory.
- Cancel an order.
- Register a customer.
- Calculate a transportation charge.

These ports are often represented by application use cases.

For example:

```csharp
public interface IDispatchShipment
{
    Task ExecuteAsync(
        ShipmentId shipmentId,
        CancellationToken cancellationToken);
}
```

An HTTP controller, a message consumer, or a scheduled process can invoke this capability.

The application does not need to know which adapter initiated the operation.

A useful way to think about a driving port is:

*What can the application do, and how can an external actor ask it to do it?*

### 2. Driven Ports (Secondary Ports)

Driven Ports represent capabilities the application needs from its environment.

For example:

- Load and persist a Shipment Aggregate.
- Publish an integration event.
- Obtain a current exchange rate.
- Send an SMS notification.
- Read information from an external fleet system.
- Obtain the current time.

The application defines the required contract, while an external adapter provides its implementation.

For example:

```csharp
public interface IShipmentRepository
{
    Task<Shipment?> GetByIdAsync(
        ShipmentId shipmentId,
        CancellationToken cancellationToken);

    Task SaveAsync(
        Shipment shipment,
        CancellationToken cancellationToken);
}
```

An infrastructure adapter can implement this port using Entity Framework Core and SQL Server.

Another implementation might use an in-memory repository for testing.

The application depends on the repository contract, not on the database technology.

A useful way to think about a driven port is:

*What does the application need from the outside world to fulfill its responsibilities?*

## Adapters: Connecting the application to the outside world

An Adapter translates between an external technology or interaction mechanism and an application's ports.

Adapters handle technical details while keeping the application-facing contracts stable.

They are commonly divided into two categories.

### 1. Driving Adapters (Primary Adapters)

Driving Adapters initiate application behavior by calling its driving ports.

Examples include:

- REST API controllers.
- GraphQL endpoints.
- Message consumers.
- Command-line interfaces.
- Scheduled jobs.
- User interface controllers.
- WebSocket handlers.

Consider an HTTP endpoint that receives a shipment dispatch request.

The controller translates the HTTP request into an application operation:

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

The controller is an adapter. It knows about HTTP, routing, and request handling. It does not decide whether a shipment is eligible for dispatch.

Another adapter could initiate the same use case from a message:

```csharp
public class DispatchShipmentConsumer
{
    private readonly IDispatchShipment _dispatchShipment;

    public DispatchShipmentConsumer(
        IDispatchShipment dispatchShipment)
    {
        _dispatchShipment = dispatchShipment;
    }

    public Task Consume(
        DispatchShipmentMessage message,
        CancellationToken cancellationToken)
    {
        return _dispatchShipment.ExecuteAsync(
            new ShipmentId(message.ShipmentId),
            cancellationToken);
    }
}
```

The HTTP controller and message consumer are different adapters, but they invoke the same application capability.

This is one of the main benefits of the architecture: application behavior is not tied to a single delivery mechanism.

### 2. Driven Adapters (Secondary Adapters)

Driven Adapters implement the capabilities required by the application.

Examples include:

- SQL Server repositories.
- Entity Framework Core implementations.
- Redis cache adapters.
- RabbitMQ publishers.
- Kafka producers.
- External HTTP clients.
- File storage adapters.
- SMS gateways.
- System clock implementations.

For example:

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

The repository adapter translates the application's persistence contract into operations supported by the database and ORM.

The application does not need to know how the database connection is established, how SQL queries are executed, or which ORM is used.

As with any repository implementation, persistence mapping must respect the domain model and Aggregate boundaries. An ORM's ability to load and update objects does not automatically make its persistence model a good domain model.

## How the pieces work together

Consider the shipment dispatch operation.

An HTTP request enters through a driving adapter. The adapter invokes the driving port, implemented by an application use case.

The use case loads the Shipment Aggregate through a driven port, asks the Aggregate to perform the domain operation, and persists the result through the same port.

The runtime infrastructure provides the database adapter.

```text
              HTTP Request
                   |
                   v
          ShipmentsController
             (Driving Adapter)
                   |
                   v
           IDispatchShipment
              (Driving Port)
                   |
                   v
        DispatchShipmentHandler
             (Application)
                   |
                   v
          IShipmentRepository
              (Driven Port)
                   |
                   v
         EfShipmentRepository
             (Driven Adapter)
                   |
                   v
              SQL Server
```

The important distinction is that the application defines the interaction contracts. The adapters implement or invoke them.

At compile time, the application depends on abstractions that it owns. At runtime, dependency injection connects those abstractions to their concrete implementations.

## Hexagonal Architecture and DDD

Hexagonal Architecture and DDD complement each other, but they solve different problems.

| Concern | DDD | Hexagonal Architecture |
|---|---|---|
| Primary focus | Understanding and modeling the business domain | Isolating application behavior from external technologies |
| Main question | What does the business mean, and how should we represent it? | How does the application interact with its environment without depending on it? |
| Important concepts | Ubiquitous Language, Bounded Contexts, Aggregates, Entities, Value Objects, Domain Events | Ports, Adapters, application boundaries |
| Main benefit | A model that reflects business knowledge and behavior | An application that is less coupled to delivery and infrastructure mechanisms |
| Typical failure | A model that does not represent the actual business | Excessive interfaces and indirection without meaningful boundaries |

DDD tells us what the model should represent. Hexagonal Architecture helps us protect that model from the technologies surrounding it.

For example, a Shipment Aggregate should enforce the business rules governing dispatching regardless of whether the operation was initiated through HTTP, messaging, or a scheduled process.

The Aggregate should not need to know about:

- ASP.NET Core.
- RabbitMQ.
- SQL Server.
- Redis.
- External HTTP APIs.

Those concerns belong outside the domain model.

However, a system can implement Hexagonal Architecture perfectly and still have a poor domain model. Its ports may be cleanly defined, its adapters well isolated, and its dependencies correctly directed, while its domain objects remain anemic and its Ubiquitous Language poorly understood.

The architecture protects the model. It does not create the model for us.

## Hexagonal Architecture within a Bounded Context

In DDD, the Bounded Context is a semantic boundary: it defines where a particular model and its language are consistent.

Hexagonal Architecture can be applied inside that boundary.

For example, a logistics platform might contain Shipment Management, Fleet Management, and Fuel Allocation Bounded Contexts.

Each context can define its own application capabilities and required external interactions.

```text
Shipment Management
├── Domain
├── Application
│   ├── Driving Ports
│   └── Driven Ports
├── Adapters
│   ├── HTTP
│   ├── Messaging
│   ├── SQL Server
│   └── External APIs
│
Fleet Management
├── Domain
├── Application
│   ├── Driving Ports
│   └── Driven Ports
├── Adapters
│   ├── HTTP
│   ├── Messaging
│   └── SQL Server
│
Fuel Allocation
├── Domain
├── Application
│   ├── Driving Ports
│   └── Driven Ports
├── Adapters
│   ├── HTTP
│   ├── Messaging
│   └── SQL Server
```

This is a conceptual organization. It does not require separate projects, databases, or deployable services.

A Bounded Context can be implemented as a module in a modular monolith, with its own ports and adapters.

The important thing is to avoid confusing technical boundaries with semantic boundaries. A shared repository interface does not establish a Bounded Context, and a separate adapter project does not create model ownership.

## Testing with Ports and Adapters

One of the practical benefits of Hexagonal Architecture is that application behavior can be tested without requiring every external dependency.

Because the application interacts with its environment through ports, tests can provide controlled adapters.

For example, a use case can be tested using an in-memory repository:

```csharp
public class InMemoryShipmentRepository : IShipmentRepository
{
    private readonly Dictionary<ShipmentId, Shipment> _shipments = new();

    public Task<Shipment?> GetByIdAsync(
        ShipmentId shipmentId,
        CancellationToken cancellationToken)
    {
        _shipments.TryGetValue(shipmentId, out var shipment);

        return Task.FromResult(shipment);
    }

    public Task SaveAsync(
        Shipment shipment,
        CancellationToken cancellationToken)
    {
        _shipments[shipment.Id] = shipment;

        return Task.CompletedTask;
    }
}
```

This allows application tests to focus on use-case behavior without depending on a real SQL Server instance.

However, an in-memory adapter does not prove that the production database adapter works correctly. The production adapter still needs integration tests against the actual persistence technology.

A useful testing strategy includes:

- **Domain tests:** Verify business rules and invariants without infrastructure.
- **Application tests:** Verify use cases through their ports, using controlled test adapters.
- **Adapter tests:** Verify translation between external protocols and application contracts.
- **Integration tests:** Verify actual interaction with databases, brokers, and external services.
- **Contract tests:** Verify compatibility between adapters and external systems.

The ability to replace an adapter for a test is useful, but replacing every dependency with a mock can also hide integration problems. Test the boundaries that carry real technical risk.

## When to Use Hexagonal Architecture

Hexagonal Architecture is particularly useful when:

- An application has multiple entry points, such as HTTP, messaging, and scheduled jobs.
- Business behavior should be independent of a particular delivery mechanism.
- External services or infrastructure technologies are expected to change.
- The application integrates with legacy systems that require translation.
- Domain and application behavior need to be tested independently of infrastructure.
- Different implementations of an external capability are genuinely required.
- The system needs clear boundaries between business policies and technical mechanisms.

For example, a shipment management application may initially use SQL Server and HTTP APIs. Later, it may need to consume dispatch requests from RabbitMQ and publish integration events to other contexts.

With meaningful ports, these additional adapters can be introduced without forcing the domain model to understand messaging infrastructure.

## When Hexagonal Architecture Becomes Overengineering

The architecture is useful only when its boundaries provide value.

A common mistake is to treat every dependency as a reason to create an interface and every class as something that must have an adapter.

Consider a simple application with one database, one delivery mechanism, and no meaningful expectation of changing either.

Creating separate interfaces, implementations, mapping layers, and projects for every small operation may add more maintenance cost than flexibility.

Several warning signs deserve attention.

### 1. Creating ports for every class

Not every dependency represents an architectural boundary.

An interface should express a meaningful contract or isolate a dependency that the application should not know about.

A `ShipmentNameFormatter` does not necessarily need an interface simply because it is a class.

### 2. Confusing ports with dependency injection

Dependency injection is a mechanism for supplying implementations. Hexagonal Architecture is a way of organizing application boundaries.

Using dependency injection does not automatically make an application hexagonal.

Likewise, an application can follow the principles of Ports and Adapters without requiring an elaborate dependency injection framework.

### 3. Creating generic ports that hide domain meaning

A generic `IDataService` or `IRepository<T>` may be convenient, but it can obscure the actual capabilities required by a context.

Compare:

```csharp
public interface IDataService
{
    Task<T?> GetAsync<T>(Guid id);
    Task SaveAsync<T>(T entity);
}
```

with:

```csharp
public interface IShipmentRepository
{
    Task<Shipment?> GetByIdAsync(ShipmentId id);
    Task SaveAsync(Shipment shipment);
}
```

The second contract communicates its purpose in the language of the domain.

Not every repository needs to be specialized, but abstractions should not erase important domain distinctions.

### 4. Putting business rules inside adapters

An adapter translates between the external world and the application. It should not become the place where business decisions are made.

For example, an HTTP controller should not decide whether a shipment can be dispatched based on its current status. It should invoke the appropriate application capability and allow the domain model to enforce the rule.

### 5. Assuming every adapter must be replaceable

Replaceability is useful when it protects against real volatility or enables meaningful testing.

But making every technical component replaceable has a cost. Some dependencies are stable, well-understood, and unlikely to change.

Architecture should protect important decisions, not pursue theoretical replaceability.

### 6. Confusing technical isolation with domain isolation

A project may have perfect separation between HTTP, application, and database code while combining several unrelated business models into one domain module.

Hexagonal Architecture does not replace Strategic DDD. Discovering Subdomains, defining Bounded Contexts, and establishing ownership remain separate design activities.

## Practical Guidelines

When applying Hexagonal Architecture:

1. Identify the application capabilities that external actors can invoke.
2. Define driving ports around meaningful use cases.
3. Identify the external capabilities the application needs.
4. Define driven ports around meaningful contracts.
5. Implement adapters for the actual technologies and protocols.
6. Keep domain behavior independent of delivery and persistence mechanisms.
7. Keep application use cases responsible for coordination, not infrastructure details.
8. Use adapters to translate external representations into application concepts.
9. Test domain behavior, application use cases, and technical adapters at appropriate levels.
10. Avoid creating abstractions that do not protect a meaningful boundary.
11. Apply the architecture within Bounded Contexts rather than treating it as a substitute for defining them.
12. Prefer a simple implementation when the additional architectural boundaries do not provide enough value.

## A Practical Exercise

Take an existing shipment dispatch operation and redesign its interactions using Ports and Adapters.

Assume the application must support:

- Dispatching through an HTTP API.
- Receiving dispatch requests through a message broker.
- Persisting shipment state in SQL Server.
- Publishing `ShipmentDispatched` integration events.
- Sending customer notifications through an external SMS provider.

Identify the following:

**1. Driving ports**

What application capabilities are exposed? Is there one dispatch use case or several distinct business operations?

**2. Driving adapters**

Which adapters invoke those capabilities? How should HTTP and messaging requests be translated into application commands?

**3. Driven ports**

What capabilities does the application require from external systems? Which contracts should the application own?

**4. Driven adapters**

Which implementations connect the application to SQL Server, the broker, and the SMS provider?

**5. Domain responsibilities**

Which business rules belong inside the Shipment Aggregate, and which operations belong to application coordination?

**6. Failure handling**

What happens if the shipment is successfully dispatched but event publication fails? Would an Outbox be appropriate?

**7. Testing**

Which behaviors can be tested without infrastructure, and which adapters need integration tests?

Finally, consider whether every proposed interface represents a useful boundary. Remove abstractions that add indirection without protecting a meaningful dependency or expressing a useful contract.

## Completion Criteria

You should be able to:

- Explain the purpose of Hexagonal Architecture and its relationship to DDD.
- Distinguish Ports from Adapters.
- Distinguish driving ports and adapters from driven ports and adapters.
- Explain how the application defines the contracts it needs.
- Design application boundaries around meaningful capabilities.
- Keep domain behavior independent of infrastructure and delivery mechanisms.
- Apply Ports and Adapters within a Bounded Context.
- Test application behavior using controlled adapters without confusing those tests with infrastructure integration tests.
- Identify when interfaces and abstractions add unnecessary complexity.
- Explain why dependency injection alone does not constitute Hexagonal Architecture.

## Further Reading

- Alistair Cockburn, [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/).
- Alistair Cockburn, [Hexagonal Architecture: The Original 2005 Article](https://alistair.cockburn.us/hexagonal-architecture/).
- Robert C. Martin, *Clean Architecture: A Craftsman's Guide to Software Structure and Design*.
- Vaughn Vernon, *Implementing Domain-Driven Design*.
- Martin Fowler, [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html).