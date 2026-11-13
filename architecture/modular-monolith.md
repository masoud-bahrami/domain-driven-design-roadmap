# Modular Monolith

A **Modular Monolith** is an application deployed as a single unit but internally organized into well-defined modules. Each module owns a cohesive set of responsibilities, exposes explicit interfaces, and controls access to its internal implementation.

Unlike a traditional monolith in which different parts of the system can freely depend on one another, a modular monolith establishes boundaries that make dependencies and ownership explicit.

It combines two important properties:

- **Unified deployment:** The application is built, released, and operated as one deployment unit.
- **Modular design:** Internally, the application is divided into modules with clear responsibilities, contracts, and dependency rules.

The objective is not merely to organize code. It is to make parts of the system independently understandable and changeable while avoiding the operational complexity of distributing them across processes.

## Modular Monolith and Domain-Driven Design

A modular monolith is a particularly useful architectural choice for implementing a system discovered through Domain-Driven Design.

DDD helps identify Subdomains, Bounded Contexts, and the relationships between their models. A modular monolith provides a way to represent those boundaries in code without requiring each Bounded Context to become an independently deployed service.

For example, consider a logistics platform with the following Bounded Contexts:

- **Cargo:** Manages cargo descriptions, classifications, and cargo-related rules.
- **Fleet:** Manages vehicles, drivers, and their operational availability.
- **Quota:** Manages fuel allocation, eligibility, and consumption rules.
- **Shipping:** Manages shipment planning, dispatch, and delivery progress.

These contexts can be implemented as separate modules inside one application.

Each module maintains its own model and Ubiquitous Language. The Shipping module should not manipulate Fleet's internal entities or directly execute Quota's business rules. Instead, it collaborates through explicitly defined contracts.

This distinction matters because a Bounded Context is a boundary of meaning, not necessarily a process or deployment boundary.

A modular monolith allows the software to preserve semantic boundaries without immediately introducing distributed communication.

## What Makes a Monolith Modular?

Creating a directory for each business capability does not make an application modular.

A meaningful module should have several characteristics.

### 1. Clear Business Responsibility

Each module should own a cohesive business capability or a clearly defined part of the domain.

For example, a Fleet module might own:

- Vehicle registration and lifecycle.
- Driver assignment and eligibility.
- Vehicle availability.
- Fleet-related business rules.

It should not become a general-purpose container for every feature that happens to involve a vehicle.

Likewise, the Shipping module may need to know whether a vehicle is available, but that does not mean it should own Fleet's availability rules.

The responsibility should follow the domain model and the decisions each part of the business owns.

### 2. Explicit Module Interfaces

Modules should communicate through deliberate interfaces rather than accessing one another's internal implementation.

For example, Shipping might need to check whether a vehicle can be assigned to a shipment.

A module contract could expose an operation such as:

```csharp
public interface IFleetOperations
{
    Task<VehicleAssignmentResult> AssignVehicleAsync(
        AssignVehicleCommand command,
        CancellationToken cancellationToken);
}
```

The Fleet module implements this contract and retains responsibility for its own rules.

Shipping does not need to know how Fleet validates vehicle availability, checks driver eligibility, or persists assignments.

The interface should express a meaningful capability, not expose a collection of generic CRUD operations simply because the underlying module has database tables.

Depending on the system, module contracts may consist of:

- Application-level commands and queries.
- Explicit request and response types.
- Published domain or integration events.
- Read-only contracts for information owned by another module.

Not every internal class needs to become part of a module's public API. The public surface should remain small enough to preserve freedom to change the implementation.

### 3. Controlled Dependencies

A module should not depend on another module's internal classes, repositories, or persistence implementation.

For example, this is a problematic dependency:

```csharp
// Shipping module
using Fleet.Infrastructure.Persistence;

var vehicle = await fleetDbContext.Vehicles
    .FirstOrDefaultAsync(x => x.Id == vehicleId);
```

Shipping now depends on Fleet's database structure and persistence implementation.

A change to Fleet's schema can break Shipping even when Fleet's public business contract has not changed.

A better approach is to depend on a contract:

```csharp
// Shipping module
public class ShipmentPlanningService
{
    private readonly IFleetQueries _fleetQueries;

    public ShipmentPlanningService(IFleetQueries fleetQueries)
    {
        _fleetQueries = fleetQueries;
    }

    public async Task PlanAsync(
        Guid shipmentId,
        Guid vehicleId,
        CancellationToken cancellationToken)
    {
        var vehicle = await _fleetQueries.GetAvailabilityAsync(
            vehicleId,
            cancellationToken);

        if (!vehicle.IsAvailable)
            throw new InvalidOperationException(
                "The selected vehicle is not available.");

        // Continue shipment planning.
    }
}
```

The example is illustrative: the actual business operation and consistency requirements may call for a stronger contract than a simple availability query.

The important point is that Shipping depends on a deliberate capability exposed by Fleet, not on Fleet's internal database.

### 4. Data Ownership

A module should own the data required to implement its responsibilities.

This does not necessarily mean every module must have a separate physical database. A modular monolith can use one database while maintaining logical ownership.

For example:

- Fleet owns vehicle and driver data.
- Shipping owns shipment and dispatch data.
- Quota owns allocation and consumption data.

The modules may share the same SQL Server instance and even the same database, but their tables and persistence operations should have clear ownership.

Other modules should not directly modify tables they do not own.

A practical arrangement might look like:

```text
Database
│
├── Fleet
│   ├── Vehicles
│   ├── Drivers
│   └── VehicleAssignments
│
├── Shipping
│   ├── Shipments
│   ├── Dispatches
│   └── Deliveries
│
└── Quota
    ├── Allocations
    └── ConsumptionRecords
```

Separate schemas can help make ownership visible:

```text
Fleet.Vehicles
Fleet.Drivers
Shipping.Shipments
Shipping.Dispatches
Quota.Allocations
Quota.ConsumptionRecords
```

Schemas are useful organizational and permission boundaries, but they do not enforce modularity by themselves. Application-level access restrictions and database permissions may also be necessary.

The goal is to prevent accidental coupling through shared persistence.

### 5. Internal Autonomy

A module should be able to change its internal implementation without forcing unrelated modules to change.

For example, Fleet might replace its vehicle availability calculation, introduce a new persistence strategy, or reorganize its internal domain model.

If Shipping depends only on Fleet's public contract, those changes should not affect Shipping.

This is a useful test of whether a module boundary is real:

**Can the module's internal design change without requiring consumers to understand or modify it?**

If not, the boundary may be only organizational.

## A Possible Project Structure

A modular monolith can be organized in different ways. The structure should reflect the system's actual boundaries rather than impose a universal folder convention.

One possible .NET solution structure is:

```text
Logistics.sln
│
├── src
│   │
│   ├── Logistics.Api
│   │   ├── Controllers
│   │   ├── Middleware
│   │   └── Program.cs
│   │
│   ├── Modules
│   │   │
│   │   ├── Fleet
│   │   │   ├── Fleet.Contracts
│   │   │   ├── Fleet.Domain
│   │   │   ├── Fleet.Application
│   │   │   └── Fleet.Infrastructure
│   │   │
│   │   ├── Shipping
│   │   │   ├── Shipping.Contracts
│   │   │   ├── Shipping.Domain
│   │   │   ├── Shipping.Application
│   │   │   └── Shipping.Infrastructure
│   │   │
│   │   └── Quota
│   │       ├── Quota.Contracts
│   │       ├── Quota.Domain
│   │       ├── Quota.Application
│   │       └── Quota.Infrastructure
│   │
│   └── BuildingBlocks
│       ├── Observability
│       └── Messaging
│
└── tests
    ├── Architecture.Tests
    ├── Fleet.Tests
    ├── Shipping.Tests
    └── Quota.Tests
```

This is an example, not a mandatory structure.

A small module may not need four separate projects. Splitting every module into Domain, Application, Infrastructure, and Contracts projects can introduce unnecessary ceremony.

The essential requirement is that module boundaries are explicit and enforced. Projects, namespaces, internal visibility, dependency rules, and architecture tests can all contribute to that enforcement.

## Communication Between Modules

Modules can collaborate through several mechanisms.

### Direct In-Process Calls

A module can expose an application contract that another module invokes directly.

For example, Shipping calls Fleet to request a vehicle assignment.

This is often the simplest approach when the operation needs an immediate result and both modules run within the same process.

However, avoid exposing internal domain objects across module boundaries. Use contracts that express the required business interaction.

### In-Process Events

A module can publish an event when something meaningful happens, allowing other modules to react without directly depending on the publisher's implementation.

For example:

1. Fleet completes a vehicle assignment.
2. Fleet publishes `VehicleAssigned`.
3. Shipping receives the event and updates its shipment workflow.

The event should represent a business fact, not merely a notification that a database row changed.

In-process events can reduce direct coupling, but they also introduce questions about ordering, error handling, transaction boundaries, and whether handlers execute synchronously or asynchronously.

An in-process event is not automatically equivalent to a durable message. If the application crashes before an in-memory notification is processed, the event may be lost.

### Shared Database Transactions

A modular monolith can sometimes coordinate changes across modules within one database transaction.

This can be useful when a business operation genuinely requires immediate consistency across multiple modules.

However, shared transaction capability should not become an excuse to ignore module ownership. Cross-module transactions create coupling and should be deliberate.

Ask whether the operation represents one genuine business consistency requirement or whether the modules are simply too tightly coupled.

### Asynchronous Background Processing

A module may use background workers to process work without blocking the initiating operation.

For example, after a shipment is dispatched, a module may schedule notifications or update downstream reporting projections.

This can introduce asynchronous behavior without requiring the module to become a separately deployed microservice.

## Enforcing Module Boundaries

A modular monolith should make accidental coupling difficult.

Several techniques can help.

**Project references**

Use project references to define which modules can depend on which contracts. Avoid referencing another module's Infrastructure or Domain project unless the relationship is explicitly justified.

**Internal visibility**

Use C# `internal` types to prevent implementation details from becoming accessible throughout the solution.

**Architecture tests**

Use tools such as ArchUnitNET or NetArchTest to verify rules such as:

- Shipping cannot reference Fleet.Infrastructure.
- Fleet cannot reference Shipping.Domain.
- Modules can access one another only through approved contracts.
- Domain projects cannot depend on Infrastructure projects.

**Database permissions and ownership**

Use schemas and, where appropriate, database permissions to prevent modules from modifying another module's tables.

**Code review and dependency documentation**

Record the intended module dependencies and review changes that introduce new cross-module relationships.

These mechanisms complement each other. A folder structure or naming convention alone cannot guarantee modularity.

## Modular Monolith and Microservices

A modular monolith and a microservices architecture can implement similar domain boundaries with different deployment and operational characteristics.

| Dimension | Modular Monolith | Microservices |
|---|---|---|
| Deployment | One application deployment | Independent service deployments |
| Process boundaries | Usually one process | Multiple processes |
| Communication | In-process contracts and messaging | Network APIs and messaging |
| Data ownership | Logical ownership can be enforced within shared infrastructure | Usually independently owned persistence |
| Transactions | Local and cross-module database transactions are possible | Cross-service transactions require distributed coordination or business workflows |
| Failure handling | Fewer network-related failure modes | Partial failures and network uncertainty |
| Operational complexity | Generally lower | Generally higher |
| Independent scaling | Limited by deployment boundaries, though workers can be separated | Services can scale independently |
| Team autonomy | Depends on module ownership and coordination | Can support stronger operational autonomy |

Neither architecture is inherently more aligned with DDD.

A modular monolith can have excellent Bounded Context boundaries. A microservices system can have poorly designed boundaries and tightly coupled domain models.

The quality of the model and the clarity of ownership matter more than the deployment style.

## Evolving a Modular Monolith into Microservices

A modular monolith can provide a practical path toward microservices, but extraction should be treated as an architectural change rather than a deployment configuration change.

Suppose Shipping initially operates inside the logistics application.

Over time:

- Shipping develops a dedicated team.
- Its workload grows substantially.
- It needs independent deployment.
- Its operational requirements differ from the rest of the platform.
- Its interactions with Fleet and Quota are already defined through explicit contracts.

Shipping may now be a reasonable candidate for extraction.

A possible migration sequence is:

1. Confirm that Shipping has a coherent business responsibility and a stable model.
2. Identify every dependency on other modules, including direct database access.
3. Define explicit APIs or integration contracts for cross-boundary interactions.
4. Establish ownership of Shipping's data.
5. Decide how transactions and consistency will work after extraction.
6. Introduce network communication and failure-handling mechanisms.
7. Deploy and operate Shipping independently.
8. Remove obsolete in-process dependencies and verify that the remaining modules respect the new boundary.

The extraction may require introducing an outbox, idempotent consumers, retries, or a process manager to preserve business workflows.

It is also possible that the extraction is never worthwhile. A modular monolith does not need to be viewed as an unfinished microservices architecture.

## Common Mistakes

**Folders instead of boundaries**

Organizing code into `Shipping`, `Fleet`, and `Quota` folders while allowing unrestricted dependencies does not create a modular monolith.

**Shared domain models**

A shared class library containing common business entities can gradually erase the distinctions between Bounded Contexts. Reuse should not take priority over semantic consistency.

**Direct cross-module database access**

A module that reads and writes another module's tables bypasses its business rules and makes its persistence model part of the consumer's implementation.

**A giant shared BuildingBlocks project**

A shared technical library can be useful for infrastructure concerns, but it should not become a home for unrelated domain concepts. Business concepts belong to the contexts that own their meaning.

**Too many abstractions**

Every module does not need its own messaging framework, repository framework, dependency injection abstraction, and elaborate project hierarchy. Introduce structure where it protects a meaningful boundary.

**Circular module dependencies**

If Shipping depends on Fleet, Fleet depends on Quota, and Quota depends on Shipping, the modules are not independently understandable. Revisit their responsibilities, contracts, and business relationships.

**Treating every interaction as an event**

Events are useful when something meaningful has happened and other parts of the system need to react. An immediate request for a decision may be better represented as a direct call. Choosing asynchronous messaging for every interaction can make workflows harder to understand.

**Assuming extraction will always be easy**

A modular monolith can make extraction easier by establishing boundaries early, but moving a module into another process still requires solving distributed data, communication, security, and operational concerns.

## Practical Guidelines

- Discover Bounded Contexts before deciding how to organize modules.
- Give every module a clear business responsibility and Ubiquitous Language.
- Keep module interfaces small, explicit, and intention-revealing.
- Let each module own its domain model and persistence operations.
- Prevent direct access to another module's internal implementation.
- Make dependencies visible and enforce them with automated tests.
- Use direct calls when an immediate answer is required and events when a business fact should trigger reactions.
- Avoid introducing distributed-system complexity before independent deployment provides meaningful value.
- Revisit boundaries when domain knowledge, team ownership, or operational requirements change.

## Exercise

Design a modular monolith for a logistics platform.

Start with four Bounded Contexts: Cargo, Fleet, Quota, and Shipping.

For each module:

1. Define its business responsibility and Ubiquitous Language.
2. Identify the domain concepts and rules it owns.
3. Define its public contracts.
4. Identify which modules it depends on.
5. Decide which data it owns.
6. Identify any cross-module workflows.
7. Write architecture tests that prevent unauthorized dependencies.

Then implement one interaction between Shipping and Fleet.

Verify that Shipping can request a vehicle assignment without accessing Fleet's domain entities, repositories, or database tables directly.

Finally, choose one module and explain what additional work would be required to extract it as an independently deployed microservice.

## Completion Criteria

You should be able to:

- Explain what makes a monolith modular rather than merely organized into folders.
- Distinguish Bounded Contexts from modules and deployment units.
- Define explicit module contracts and dependency rules.
- Establish data ownership within a shared database.
- Choose appropriate communication mechanisms between modules.
- Identify and prevent circular dependencies and cross-module persistence access.
- Use architecture tests to enforce modularity.
- Explain how a modular monolith differs from microservices.
- Evaluate whether extracting a module would provide enough value to justify distributed-system complexity.

**A modular monolith is not a compromise between good design and microservices. It is an architectural choice that preserves meaningful boundaries while keeping deployment and operations relatively simple.**