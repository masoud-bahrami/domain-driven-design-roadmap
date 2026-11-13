# Supple Design

A domain model is not something we discover once, implement, and then preserve forever.

As a team learns more about a business, its understanding changes. Previously hidden rules become visible, ambiguous concepts acquire precise meanings, and business processes reveal distinctions that were not apparent at the beginning.

The model must evolve with that understanding.

But not every design makes change equally easy. Some designs make business rules explicit, communicate intent, and allow developers to explore alternative models. Others hide meaning behind generic methods, technical abstractions, tangled dependencies, and implementation details.

Eric Evans addresses this challenge in Part III, Chapter 10 of *Domain-Driven Design: Tackling Complexity in the Heart of Software*, titled **Supple Design**.

Supple Design is a set of design principles and patterns that make a domain model easier to understand, reason about, and change without losing its connection to the underlying domain.

Its central concern is not merely writing maintainable code. It is making the model itself more expressive and easier to work with.

A supple design helps developers and domain experts understand what the software means, explore alternative designs, and incorporate new domain knowledge without fighting the existing implementation.

## What Is Supple Design?

Supple Design describes qualities of a domain model and its implementation that allow it to remain understandable and adaptable as the domain evolves.

Consider a logistics system that initially models shipment dispatch like this:

```csharp
public class ShipmentService
{
    public void Process(Shipment shipment)
    {
        // Validate shipment
        // Assign vehicle
        // Assign driver
        // Complete loading
        // Authorize dispatch
        // Update shipment status
    }
}
```

The method `Process` tells us almost nothing about the business behavior.

What does processing mean? Does it assign a vehicle? Does it dispatch the shipment? Does it complete loading? Which business rules are being enforced?

Now consider a more expressive model:

```csharp
shipment.AssignVehicle(vehicle);
shipment.AssignDriver(driver);
shipment.CompleteLoading();
shipment.AuthorizeDispatch();
shipment.Dispatch();
```

The second version communicates more about the domain.

It exposes business intentions, makes the sequence easier to understand, and gives the model meaningful places to enforce rules.

Suppose the business later introduces a requirement that a shipment cannot be dispatched until its inspection has been completed.

In the expressive model, the team can introduce an explicit inspection concept and enforce the rule within the relevant domain behavior.

The design does not guarantee that the change will be trivial. It does, however, provide a structure in which the new understanding can be represented without deciphering a generic processing operation.

### Qualities of Supple Design

A supple design has several important qualities.

**Expressiveness**

The model and its implementation communicate domain concepts, business intentions, and rules clearly.

A developer should be able to understand important behavior without mentally translating generic technical operations into business meaning.

**Malleability**

The design can be modified without requiring disproportionate changes across unrelated parts of the system.

A change in one business rule should not unexpectedly break several unrelated workflows simply because their implementations are tightly coupled.

**Understandability**

The model should help developers reason about the business rather than force them to reconstruct its meaning from scattered conditions, data structures, and technical conventions.

**Explorability**

Developers should be able to experiment with alternative models and discover better representations of the domain.

A design that is difficult to change discourages experimentation. A supple design makes it easier to test a new understanding against the existing model.

These qualities reinforce one another. A model that communicates its intent clearly is generally easier to reason about, and a model that is easier to reason about is generally easier to evolve.

## The Supple Design Patterns

Evans describes several patterns that contribute to Supple Design.

The following sections explain each pattern, its purpose, practical examples, and considerations when applying it.

### 1. Intention-Revealing Interfaces

An Intention-Revealing Interface makes the purpose of an operation clear through its name and usage.

A method should communicate what it does in terms of the domain, rather than expose generic technical operations that require callers to understand its implementation.

Consider:

```csharp
order.Process();
order.Update();
order.Handle();
```

These names do not communicate the business intention.

Compare them with:

```csharp
order.Confirm();
order.Cancel();
order.AddItem(product, quantity);
order.ApplyDiscount(discount);
```

The second set of operations expresses recognizable business actions.

This becomes particularly important when the domain has rules that should be enforced consistently.

For example:

```csharp
public class Order
{
    private readonly List<OrderItem> _items = new();

    public void Confirm()
    {
        if (_items.Count == 0)
            throw new InvalidOperationException(
                "An order cannot be confirmed without items.");

        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException(
                "Only draft orders can be confirmed.");

        Status = OrderStatus.Confirmed;
    }
}
```

The method communicates a business operation and protects the conditions under which it can happen.

A caller does not need to know how the order's internal state is represented.

Compare that with:

```csharp
order.Status = OrderStatus.Confirmed;
```

The second operation allows callers to modify the state without necessarily enforcing the rules associated with confirmation.

Intention-revealing interfaces are closely connected to Ubiquitous Language.

If domain experts say `Authorize Dispatch`, the model should not arbitrarily call the operation `ProcessShipmentStepThree`.

The names used in code should reflect the concepts and distinctions discovered through domain exploration.

**When to apply**

- Designing Aggregate operations.
- Naming Domain Services and Application Services.
- Defining commands and business operations.
- Exposing APIs for domain behavior.
- Replacing generic methods and setters with meaningful operations.

**Watch out**

Intention-revealing names are not enough by themselves.

A method called `ConfirmOrder` is not expressive if its implementation simply changes a status field while ignoring the rules that make confirmation meaningful.

The interface and its behavior must communicate the same intention.

### 2. Side-Effect-Free Functions

A Side-Effect-Free Function produces a result without changing observable state or causing externally visible effects.

Its output depends only on its inputs.

This makes it easier to understand, test, reuse, and reason about.

Consider calculating the total price of an order.

A function that calculates a total should not unexpectedly update the order, write to a database, send a notification, or modify a global variable.

A simple example:

```csharp
public static decimal CalculateTotal(
    IReadOnlyCollection<OrderItem> items)
{
    return items.Sum(item =>
        item.UnitPrice * item.Quantity);
}
```

For the same input, the function produces the same result.

It does not change the order or any other external state.

Now consider a function that combines calculation and mutation:

```csharp
public void CalculateTotal()
{
    Total = Items.Sum(item =>
        item.UnitPrice * item.Quantity);

    _repository.Save(this);
    _notificationService.NotifyCustomer(Id);
}
```

The method has several responsibilities:

- Calculate a value.
- Change internal state.
- Persist the object.
- Trigger an external notification.

Understanding its behavior requires understanding all these effects.

Separating calculation from side effects makes the design easier to reason about:

```csharp
var total = OrderPricing.CalculateTotal(order.Items);

order.SetCalculatedTotal(total);

await repository.Save(order);
```

The calculation is isolated, while state changes and persistence remain explicit.

#### Pure Functions and Domain Services

Side-effect-free functions are especially useful for calculations, policies, validation rules, and transformations.

For example, a pricing policy might calculate a discount:

```csharp
public static decimal CalculateDiscount(
    decimal orderTotal,
    decimal discountRate)
{
    return orderTotal * discountRate;
}
```

However, this does not mean every Domain Service must be a pure function.

Some domain operations legitimately involve changing domain state or coordinating domain behavior.

For example, a domain operation may evaluate a business decision and return a result that the caller uses to perform a state transition.

The important distinction is between a function that computes a result and an operation that changes state or interacts with the outside world.

A useful design is to isolate calculations and decisions where possible, while keeping necessary state changes explicit and controlled.

**When to apply**

- Pricing and financial calculations.
- Eligibility and qualification decisions.
- Validation and policy evaluation.
- Transformations between domain representations.
- Complex business calculations that need independent testing.

**Watch out**

Do not force every operation into a pure-function model.

An Aggregate operation such as `order.Confirm()` is expected to change the Aggregate's state. That is legitimate domain behavior.

The goal is to isolate unnecessary side effects, not eliminate all state changes from the domain model.

### 3. Assertions

Assertions express assumptions that a design depends on.

They make important conditions explicit and help detect situations in which the program has entered an unexpected state.

For example, suppose a pricing calculation assumes that an order contains at least one item.

An assertion can make this assumption visible:

```csharp
Debug.Assert(order.Items.Count > 0);
```

The assertion communicates an expectation to developers and can help detect programming errors during development.

Assertions are particularly useful when a condition should already have been guaranteed by another part of the model.

Consider an operation that expects a confirmed order:

```csharp
public static Invoice CreateInvoice(Order order)
{
    Debug.Assert(
        order.Status == OrderStatus.Confirmed);

    return new Invoice(order.Id, order.Items);
}
```

The assertion documents an assumption about the state of the order.

However, there is an important distinction between an assertion and business validation.

A customer may legitimately attempt to confirm an empty order. That is not necessarily a programming error. It is an invalid business operation that should be handled explicitly.

For example:

```csharp
public void Confirm()
{
    if (_items.Count == 0)
        throw new BusinessRuleViolationException(
            "An order must contain at least one item.");

    Status = OrderStatus.Confirmed;
}
```

This is business rule enforcement, not merely an assertion.

Assertions should not replace validation that must work in production or protect the system against invalid external input.

**When to apply**

- Documenting internal assumptions.
- Detecting impossible or unexpected states.
- Checking invariants during development.
- Verifying preconditions that should already be guaranteed.
- Making hidden dependencies between operations visible.

**Watch out**

Do not use assertions as the only protection for business invariants.

Assertions may be disabled or behave differently depending on the execution environment. Business rules must be enforced by the actual domain behavior.

### 4. Conceptual Contours

Conceptual Contours concern the shape and boundaries of the domain model.

A good design should reflect meaningful conceptual distinctions in the domain rather than impose arbitrary technical divisions.

Consider a logistics system.

Initially, the team may represent everything related to transportation using a single `Shipment` class:

```text
Shipment
  - Vehicle
  - Driver
  - Loading
  - Inspection
  - Dispatch
  - Delivery
  - Payment
  - Customer
```

Over time, the model becomes difficult to understand because it combines concepts that have different responsibilities, rules, and lifecycles.

Further domain exploration reveals that:

- Shipment Planning manages vehicle and driver assignment.
- Warehouse Operations manages loading and inspection.
- Transportation manages dispatch and delivery.
- Billing manages transportation charges.

These concepts have different responsibilities and potentially different models.

The design should reflect those conceptual distinctions.

Conceptual Contours are not simply a matter of creating more classes, folders, or services. They are about finding a model structure that corresponds to the way the domain itself is understood.

This is closely related to Bounded Contexts, but the two ideas should not be treated as identical.

A Bounded Context establishes a boundary within which a particular model and its language are consistent.

Conceptual Contours concern the meaningful shape of the model and how its concepts fit together. They can influence the design inside a Bounded Context as well as the boundaries between contexts.

**When to apply**

- A model contains concepts with different responsibilities.
- One class or Aggregate has accumulated unrelated behavior.
- The same term represents different concepts in different parts of the domain.
- The model becomes difficult to navigate as complexity grows.
- Domain exploration reveals previously hidden distinctions.

**Watch out**

Do not create conceptual boundaries merely because the code has become large.

A meaningful boundary should reflect a real distinction in the domain, responsibilities, rules, or model.

### 5. Standalone Classes

Standalone Classes minimize unnecessary dependencies so that a class can be understood and used with limited knowledge of other classes.

The goal is to reduce the mental effort required to understand a piece of the model.

Consider a simple Value Object:

```csharp
public sealed class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}
```

This class is relatively self-contained.

Understanding it does not require understanding a database, an application service, a message broker, or a large object graph.

Now consider a `Money` class that depends on:

- An order repository.
- A customer service.
- A payment gateway.
- An application configuration service.
- A database context.

These dependencies would make the class harder to understand and reuse.

A domain concept should not depend on unrelated infrastructure merely because that infrastructure happens to be available.

This principle is especially relevant when separating domain logic from persistence, messaging, and external integrations.

**When to apply**

- Designing Value Objects.
- Separating domain behavior from infrastructure.
- Reducing unnecessary dependencies.
- Making domain concepts independently understandable.
- Improving testability and reuse.

**Watch out**

Standalone does not mean every class must have zero dependencies.

An Aggregate may contain references to other domain concepts. A Domain Service may require a collaborator. A Repository may depend on persistence infrastructure.

The goal is not isolation at any cost. It is to avoid dependencies that make the model unnecessarily difficult to understand or change.

### 6. Closure of Operations

Closure of Operations is one of the most easily misunderstood Supple Design patterns.

It is not simply about grouping related methods inside a class.

The idea is that an operation on a type should produce a result of that same type when the operation's meaning allows it.

In other words, the type is closed under the operation.

Consider the concept of Money.

Adding two monetary values of the same currency should produce another Money value.

```csharp
public sealed class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException(
                "Currencies must match.");

        return new Money(
            Amount + other.Amount,
            Currency);
    }
}
```

The operation takes Money and returns Money.

```csharp
var subtotal = itemPrice.Add(shippingCost);
var total = subtotal.Add(tax);
```

The result of each operation can participate in another operation of the same kind.

The caller does not need to extract decimal values, perform arithmetic independently, and reconstruct a Money object after every calculation.

The domain concept remains intact throughout the operation.

Another example is a Date Range:

```csharp
public sealed class DateRange
{
    public DateOnly Start { get; }
    public DateOnly End { get; }

    public DateRange(DateOnly start, DateOnly end)
    {
        if (end < start)
            throw new ArgumentException(
                "End must not precede start.");

        Start = start;
        End = end;
    }

    public DateRange ShiftDays(int days)
    {
        return new DateRange(
            Start.AddDays(days),
            End.AddDays(days));
    }
}
```

Shifting a Date Range produces another Date Range.

This preserves the concept and allows operations to be composed naturally.

Closure is not appropriate for every operation.

For example, `Order.CalculateTotal()` should return Money rather than Order because the result represents a different domain concept.

The important point is that when an operation conceptually transforms one instance of a type into another instance of the same type, preserving that type makes the model more expressive.

**When to apply**

- Value Objects such as Money, Quantity, Distance, and DateRange.
- Mathematical and financial operations.
- Transformations that preserve a domain concept.
- Composable domain calculations.
- Operations where callers should continue working with the same conceptual type.

**Watch out**

Do not force operations to return the same type when the domain meaning requires a different result.

Closure should preserve conceptual integrity, not dictate method signatures mechanically.

### 7. Declarative Design

Declarative Design expresses what a model or operation means through a representation that communicates its intent, rather than forcing readers to reconstruct that meaning from a sequence of low-level instructions.

A declarative design makes important business concepts visible in the structure of the code.

Consider a customer eligibility rule:

```csharp
if (customer.Status == CustomerStatus.Active &&
    customer.CreditLimit >= order.Total &&
    customer.IsWithinDeliveryArea)
{
    // Accept order
}
```

This implementation contains the conditions, but the business meaning is distributed across a series of technical expressions.

Now consider a model that names the business decision:

```csharp
if (customer.CanPlace(order))
{
    // Accept order
}
```

The second version communicates the intention more directly.

The underlying conditions still need to be implemented, but they are encapsulated behind a domain-oriented operation.

Declarative Design can also make use of specifications, policies, and other domain abstractions.

For example:

```csharp
var eligibility =
    IsActiveCustomer
        .And(HasSufficientCredit)
        .And(IsWithinDeliveryArea);
```

The expression describes the business criteria without requiring the caller to understand every implementation detail.

This can be especially valuable when business rules are composable or need to be reused in different contexts.

However, declarative design does not mean hiding complexity behind layers of abstractions.

A declaration is useful only when it makes the business meaning clearer.

**When to apply**

- Complex business eligibility rules.
- Policies that combine multiple criteria.
- Reusable business constraints.
- Models where procedural code obscures domain intent.
- Situations where a domain expert should be able to recognize the expressed rule.

**Watch out**

Avoid creating elaborate rule engines or abstraction frameworks for simple conditions.

A straightforward `if` statement can be clearer than an unnecessary specification hierarchy.

### 8. Domain-Specific Languages

A Domain-Specific Language (DSL) provides a language tailored to expressing concepts and operations in a particular domain.

Unlike a general-purpose programming language, a DSL focuses on a specific problem area and uses concepts meaningful within that area.

A DSL can be internal to the programming language or external to it.

For example, a logistics system may need to express shipment eligibility rules.

A general implementation might look like:

```csharp
if (shipment.Weight <= vehicle.Capacity &&
    vehicle.IsAvailable &&
    driver.HasRequiredLicense &&
    shipment.Origin.Region == vehicle.Region)
{
    // Vehicle and driver are eligible
}
```

A domain-oriented expression could make the intent clearer:

```csharp
var eligibility =
    VehicleCanCarry(shipment)
        .And(VehicleIsAvailable)
        .And(DriverHasRequiredLicense)
        .And(ShipmentIsWithinOperatingRegion);
```

The resulting expression is closer to the language used to discuss the business rule.

An internal DSL may use method chaining, fluent interfaces, or domain-oriented expressions.

An external DSL may use a dedicated textual format or configuration language.

For example, a business might define delivery rules in a dedicated language:

```text
DeliveryRule:
  Region = North
  MaximumWeight = 1200 kg
  RequiredVehicleType = Refrigerated
  DriverQualification = HeavyVehicle
```

The software interprets the rule and applies it to relevant shipments.

A DSL can be valuable when domain rules are sufficiently complex, repeated, and important to express independently of their implementation.

**When to apply**

- Complex pricing and eligibility rules.
- Business policies that change frequently.
- Repeated rule expressions.
- Domains where experts need to review or maintain rule definitions.
- Situations where a specialized vocabulary significantly improves clarity.

**Watch out**

A DSL has a cost.

It introduces syntax, interpretation, validation, tooling, documentation, and maintenance concerns.

Do not create a new language simply to make a few conditions look different.

The language should solve a genuine communication or modeling problem.

## Specifications and Supple Design

The Specification pattern deserves special attention because it is closely related to the ideas behind Supple Design.

A Specification represents a business criterion that can be evaluated against an object.

For example:

- Is the customer active?
- Does the customer have sufficient credit?
- Is the shipment within the delivery area?
- Is the vehicle suitable for the shipment?

These criteria can be represented independently and combined.

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T candidate);
}
```

An example specification:

```csharp
public sealed class IsActiveCustomer
    : ISpecification<Customer>
{
    public bool IsSatisfiedBy(Customer customer)
    {
        return customer.Status == CustomerStatus.Active;
    }
}
```

Another:

```csharp
public sealed class HasSufficientCredit
    : ISpecification<Customer>
{
    private readonly decimal _requiredAmount;

    public HasSufficientCredit(decimal requiredAmount)
    {
        _requiredAmount = requiredAmount;
    }

    public bool IsSatisfiedBy(Customer customer)
    {
        return customer.AvailableCredit >= _requiredAmount;
    }
}
```

These specifications can be combined into a larger business criterion.

```csharp
var canPurchase =
    new IsActiveCustomer()
        .And(new HasSufficientCredit(order.Total));
```

This makes the composition of business rules explicit.

Specifications are particularly useful when criteria need to be combined, reused, tested independently, or represented in a form that makes business intent clear.

They can also support declarative design and help create a more supple model.

However, a Specification is not automatically the best place for every business rule.

For example, an Aggregate invariant that must always hold when an order is confirmed should still be protected by the Aggregate's behavior.

A Specification can express the criterion, but the model must ensure that the rule cannot be bypassed through another operation.

Similarly, a specification that requires external information, such as a remote credit assessment, may need orchestration outside a simple in-memory predicate.

The pattern should be used where it improves the expression and composition of domain knowledge, not merely because a rule contains a conditional statement.

## Supple Design in Practice

Supple Design is not achieved by applying all the patterns mechanically.

The patterns reinforce one another, but their value depends on the quality of the underlying domain understanding.

Consider the evolution of a shipment dispatch model.

### Initial Model

A team begins with:

```csharp
public class Shipment
{
    public int Status { get; set; }

    public void Process()
    {
        // Several unrelated operations
    }
}
```

The model has little domain meaning.

### Discovering the Language

Through discussions with dispatchers and warehouse operators, the team discovers that these are distinct business activities:

- Assigning a vehicle.
- Assigning a driver.
- Completing loading.
- Completing inspection.
- Authorizing dispatch.
- Recording departure.

The team updates its Ubiquitous Language.

### Making Intent Explicit

The model now exposes operations that correspond to those activities:

```csharp
shipment.AssignVehicle(vehicle);
shipment.AssignDriver(driver);
shipment.CompleteLoading();
shipment.CompleteInspection();
shipment.AuthorizeDispatch();
shipment.Dispatch();
```

The operations express business intentions.

### Protecting Invariants

The team discovers that dispatch requires:

- An assigned vehicle.
- An assigned qualified driver.
- Completed loading.
- A completed inspection.
- Dispatch authorization.

These conditions become part of the Aggregate's behavior.

```csharp
public void Dispatch()
{
    if (Vehicle is null)
        throw new BusinessRuleViolationException(
            "A vehicle must be assigned.");

    if (Driver is null)
        throw new BusinessRuleViolationException(
            "A driver must be assigned.");

    if (!LoadingCompleted)
        throw new BusinessRuleViolationException(
            "Loading must be completed.");

    if (!InspectionCompleted)
        throw new BusinessRuleViolationException(
            "Inspection must be completed.");

    if (!DispatchAuthorized)
        throw new BusinessRuleViolationException(
            "Dispatch must be authorized.");

    Status = ShipmentStatus.Dispatched;
}
```

The business operation protects the conditions that make dispatch valid.

### Isolating Calculations and Decisions

Suppose the system must calculate whether a vehicle has enough capacity for a shipment.

The team isolates the calculation into a side-effect-free operation:

```csharp
public static bool CanCarry(
    Vehicle vehicle,
    Shipment shipment)
{
    return vehicle.Capacity >= shipment.Weight;
}
```

The calculation becomes easier to test and reason about.

### Preserving Conceptual Integrity

The team introduces Value Objects for concepts such as Weight and Capacity rather than passing unrelated decimal values throughout the model.

```csharp
public sealed record Weight(decimal Kilograms);

public sealed record Capacity(decimal Kilograms);
```

The model can now express operations in terms of domain concepts instead of primitive values.

Each improvement makes the model more expressive and easier to evolve.

The result is not simply cleaner code. It is a more accurate representation of the business.

## How Supple Design Connects to Other DDD Concepts

Supple Design is closely connected to almost every other part of DDD.

| DDD concept | Connection to Supple Design |
|---|---|
| Ubiquitous Language | Provides the vocabulary that intention-revealing interfaces and declarative models express |
| Domain Exploration | Supplies the domain knowledge needed to create meaningful abstractions |
| Bounded Contexts | Defines the scope within which a model and its language remain consistent |
| Aggregates | Provide explicit boundaries for protecting invariants and controlling state changes |
| Value Objects | Often benefit from side-effect-free functions and closure of operations |
| Domain Services | Can isolate domain operations that do not naturally belong to an Entity or Value Object |
| Specifications | Express and compose business criteria |
| Refactoring | Improves the model as new understanding emerges |
| Domain Events | Make meaningful business facts explicit |
| Testing | Provides confidence when the model is changed and refined |

### Supple Design and Ubiquitous Language

Ubiquitous Language is the vocabulary of the model.

Supple Design helps that vocabulary become visible in the implementation.

A model that uses domain terminology in its names but hides its behavior behind generic methods is not genuinely expressive.

The language must be reflected in the operations, abstractions, and rules of the model.

### Supple Design and Refactoring

Refactoring is one of the primary ways to improve a design without changing its externally observable behavior.

As the team's understanding evolves, refactoring helps the model better express the discovered concepts and distinctions.

Sometimes the changes are small:

- Rename `Process` to `AuthorizeDispatch`.
- Extract a pricing calculation.
- Introduce a Value Object.
- Remove an unnecessary dependency.

Other times, the new understanding requires a deeper model change.

For example, the team may discover that `Shipment Assigned` and `Shipment Dispatched` represent different business states and should be modeled as separate concepts.

That may lead to a more substantial refactoring.

### Supple Design and Testing

A supple design is easier to test because its behavior and assumptions are more explicit.

Intention-revealing operations provide clear places to test business behavior.

Side-effect-free functions can be tested independently.

Assertions make internal assumptions visible.

Standalone classes reduce the need to construct large dependency graphs.

Automated tests provide confidence when refactoring changes the implementation.

Testing is not simply a final verification activity. It supports the continuous evolution of the model.

## Practical Guidelines

The following practices help make Supple Design part of everyday development.

**1. Start with domain meaning.**

Do not introduce abstractions before understanding the concepts and rules they are intended to represent.

Use conversations, Domain Storytelling, Event Storming, and Example Mapping to discover the business meaning.

**2. Make business intentions explicit.**

Prefer operations that communicate domain behavior over generic methods and unrestricted state mutation.

**3. Keep calculations and decisions understandable.**

Isolate pure calculations and policy evaluations where appropriate. Keep necessary state changes explicit.

**4. Protect invariants at the right boundaries.**

Use Aggregate behavior to protect rules that must hold within an Aggregate's consistency boundary.

Do not rely on callers to remember important conditions.

**5. Reduce unnecessary dependencies.**

A domain concept should not know about infrastructure unless that dependency is genuinely part of its responsibility.

**6. Preserve conceptual distinctions.**

Do not collapse different business concepts into one class or one status merely because they look similar in the database.

**7. Refactor continuously.**

Treat the model as something that evolves with domain understanding. Avoid allowing temporary implementation decisions to become permanent constraints.

**8. Test behavior, not just implementation details.**

Tests should demonstrate business rules, important state transitions, and rejected operations.

**9. Prefer clarity over cleverness.**

A design should reduce the effort needed to understand the domain. Abstractions that require more explanation than the business rule they represent are often unnecessary.

**10. Revisit the model with domain experts.**

The quality of the design depends on the accuracy of the understanding it represents.

## Common Mistakes

### 1. Treating Supple Design as a Checklist

Applying every pattern does not automatically produce a supple design.

The patterns are tools for solving design problems, not mandatory ingredients in every model.

A simple business concept may need only a clear class and a few operations.

### 2. Confusing Expressive Names with Expressive Models

Renaming `Process` to `Dispatch` does not make a model expressive if the business rules remain scattered across unrelated services and controllers.

The behavior must match the intention communicated by the interface.

### 3. Overusing Abstractions

Specifications, DSLs, interfaces, and domain services can improve a model, but they can also make it harder to understand when introduced without a genuine need.

Do not create abstractions simply because a design pattern exists.

### 4. Putting Every Rule in a Domain Service

Domain Services are useful when an operation does not naturally belong to a particular Entity or Value Object.

They should not become containers for all business logic.

Behavior that belongs to an Aggregate should generally remain within that Aggregate.

### 5. Treating Assertions as Business Validation

Assertions express assumptions about program state. They do not replace business rule enforcement.

Invalid customer actions must be handled as business outcomes, not merely detected through development assertions.

### 6. Confusing Closure of Operations with Class Cohesion

Closure is about operations whose results remain within the same conceptual type.

It is not simply grouping related methods inside one service or module.

### 7. Creating a DSL Too Early

A specialized language may be valuable for a complex, evolving rule system.

For a few straightforward conditions, it introduces unnecessary complexity.

### 8. Ignoring the Domain While Refactoring

Refactoring should improve the design, not merely reorganize files or reduce lines of code.

The model should become more understandable and more accurate in expressing the business.

## Practical Review Checklist

When reviewing a domain model, ask:

- [ ] Do class and method names communicate business intentions?
- [ ] Does the implementation use the Ubiquitous Language consistently?
- [ ] Are important business rules visible in the model?
- [ ] Are calculations and decisions isolated from unnecessary side effects?
- [ ] Are internal assumptions made explicit?
- [ ] Are business invariants enforced by the appropriate domain boundaries?
- [ ] Does the structure reflect meaningful conceptual distinctions?
- [ ] Are unnecessary dependencies minimized?
- [ ] Do operations preserve domain types when conceptually appropriate?
- [ ] Are business rules expressed declaratively where doing so improves clarity?
- [ ] Are Specifications used where criteria genuinely need to be represented and composed?
- [ ] Would a DSL provide meaningful value, or would it introduce unnecessary complexity?
- [ ] Can developers explain the model without translating every operation into business language?
- [ ] Can the model evolve without disproportionate changes to unrelated behavior?
- [ ] Do automated tests provide confidence for future refactoring?

## Further Reading

- **Domain-Driven Design: Tackling Complexity in the Heart of Software**, Eric Evans. Part III, especially Chapter 10, *Supple Design*. This is the primary source for the patterns discussed here.
- **Refactoring: Improving the Design of Existing Code**, Martin Fowler. A practical foundation for improving existing designs while preserving behavior.
- **Working Effectively with Legacy Code**, Michael Feathers. Useful for making existing systems safer to change, particularly when automated tests and clear boundaries are missing.
- **Implementing Domain-Driven Design**, Vaughn Vernon. Practical examples of implementing DDD concepts and patterns.
- **A Philosophy of Software Design**, John Ousterhout. Useful complementary material on complexity, abstraction, and making software easier to understand.

## Final Thought

The purpose of Supple Design is not to make every class elegant or every method clever.

It is to make the domain model a better tool for thinking.

When a new business rule emerges, the model should give the team a natural place to express it. When a concept becomes clearer, the code should be able to reflect that understanding. When the team discovers that an old abstraction no longer represents the domain, the design should allow it to be replaced.

This is why Supple Design is so closely connected to Ubiquitous Language and continuous domain exploration.

The language gives the model its meaning. Domain exploration deepens that meaning. Supple Design helps preserve and express it as the software evolves.

**A supple design does not merely tolerate change. It makes deeper understanding possible.**