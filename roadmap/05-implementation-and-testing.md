# 05 Implementation and Testing

Domain-Driven Design is not a programming language, framework, or architectural style. It is an approach to understanding and modeling complex business domains.

Its principles can be applied using different programming paradigms, including Object-Oriented Programming (OOP), Functional Programming (FP), and combinations of both.

The underlying ideas remain the same:

- Model the business rather than merely its data.
- Make important business rules explicit.
- Preserve the integrity of domain concepts.
- Keep models and their language consistent within their boundaries.
- Separate domain decisions from technical concerns.
- Continuously validate the model against real business behavior.

What changes is how these ideas are expressed in code.

An object-oriented implementation may use classes, encapsulated state, and methods that control state transitions. A functional implementation may use immutable data structures, algebraic data types, and functions that transform domain state. A hybrid implementation may combine both approaches.

None of these approaches is inherently more DDD than the others.

The quality of an implementation depends on how accurately it represents the domain, how effectively it protects business rules, and how easily the system can evolve as the business changes.

Testing is an essential part of this process. It provides feedback about whether the implementation behaves as the domain requires, whether architectural boundaries are respected, and whether integrations preserve the expected contracts.

## Goal

Learn how to implement and test DDD applications across different programming paradigms.

By the end of this section, you should be able to:

- Translate DDD concepts into different programming styles.
- Understand how OOP and FP represent Entities, Value Objects, Aggregates, and domain behavior.
- Choose implementation techniques based on the domain and the team's context.
- Keep domain logic independent of infrastructure and external dependencies.
- Design a testing strategy around business behavior and architectural boundaries.
- Test domain rules, application workflows, persistence, and integrations at appropriate levels.
- Use TDD and BDD as feedback and collaboration practices.
- Build confidence in the system without relying exclusively on manual testing or excessive end-to-end tests.

## 1. DDD Across Programming Paradigms

DDD is concerned with the design of the domain model, not with a particular programming paradigm.

The same business concept can be represented in different ways while preserving its meaning.

Consider an Order:

- It has an identity.
- It contains items.
- It has a lifecycle.
- It must obey business rules.
- It may transition from Draft to Confirmed or Cancelled.
- Certain transitions produce meaningful business events.

An OOP implementation may represent Order as a class with encapsulated state and methods.

An FP implementation may represent Order as an immutable record and use functions to produce a new state after validating an operation.

Both can express the same domain behavior.

The important question is not whether the implementation uses classes or functions. It is whether the model makes the business rules explicit and prevents invalid operations.

### 1.1 DDD with Object-Oriented Programming

Object-Oriented Programming has a long history with DDD. Many of the foundational tactical patterns were originally presented through object-oriented examples.

OOP provides a natural way to represent domain concepts as objects with identity, state, behavior, and relationships.

#### Mapping DDD Concepts to OOP

**Entities**

Entities are commonly represented as classes with identity and controlled state transitions.

An Order, for example, has an `OrderId` that remains stable throughout its lifecycle. Its state may change, but its identity does not.

**Value Objects**

Value Objects can be represented using immutable classes, records, or structs.

A `Money` object, for example, can encapsulate an amount and currency, enforce valid values, and implement value-based equality.

**Aggregates**

Aggregates are represented by clusters of objects with an Aggregate Root controlling access to their internal state.

The root exposes operations that enforce the Aggregate's invariants.

**Domain Services**

Domain Services are typically implemented as classes or functions that encapsulate domain operations that do not naturally belong to an individual Entity or Value Object.

**Repositories**

Repositories are usually represented through interfaces that describe how Aggregates are retrieved and persisted, with concrete implementations in the infrastructure layer.

#### Common OOP Practices

- Encapsulate state and expose meaningful domain operations.
- Use identity-based equality for Entities and value-based equality for Value Objects.
- Prefer composition when assembling domain behavior.
- Use polymorphism when different domain concepts genuinely have different behavior behind a common abstraction.
- Use inheritance selectively rather than building deep hierarchies around superficial similarities.
- Use Factories, Strategies, and other patterns when they make domain responsibilities clearer.
- Keep Aggregate internals inaccessible to external callers.

#### Example: An Order Aggregate in C#

```csharp
public sealed class Order
{
    private readonly List<OrderItem> _items = [];

    public OrderId Id { get; }
    public OrderStatus Status { get; private set; }

    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    public Money Total =>
        _items.Aggregate(
            Money.Zero,
            (total, item) => total + item.Subtotal);

    private Order(OrderId id)
    {
        Id = id;
        Status = OrderStatus.Draft;
    }

    public static Order Create(OrderId id)
    {
        return new Order(id);
    }

    public void AddItem(
        ProductId productId,
        int quantity,
        Money unitPrice)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException(
                "Only draft orders can be modified.");

        if (quantity <= 0)
            throw new ArgumentOutOfRangeException(
                nameof(quantity));

        _items.Add(
            new OrderItem(productId, quantity, unitPrice));
    }

    public void Confirm()
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException(
                "Only draft orders can be confirmed.");

        if (_items.Count == 0)
            throw new InvalidOperationException(
                "An order must contain at least one item.");

        Status = OrderStatus.Confirmed;
    }
}

public sealed record OrderItem(
    ProductId ProductId,
    int Quantity,
    Money UnitPrice)
{
    public Money Subtotal => UnitPrice * Quantity;
}
```

This is a simplified example, but it demonstrates several important ideas:

- Order is the Aggregate Root.
- Its internal collection is not directly mutable by external callers.
- Business rules are enforced through domain operations.
- OrderItem is modeled as an immutable value in this example.
- The Aggregate controls its own lifecycle.

Notice that the example does not make every property publicly mutable or expose a generic `Update` method.

Also, whether OrderItem should be a Value Object or an Entity depends on the domain. If individual order lines have an independent identity, lifecycle, or behavior, modeling them as Entities may be more appropriate.

**Important:** OOP does not require mutable domain models. Immutable objects and functional techniques can be used within an object-oriented design.

### 1.2 DDD with Functional Programming

Functional Programming offers another way to represent domain models.

It emphasizes:

- Immutability.
- Pure functions.
- Explicit data transformations.
- Composition.
- Algebraic Data Types.
- Explicit handling of errors and effects.

Instead of allowing an object to change its internal state, a functional implementation often represents domain operations as functions that receive a valid state and return a new state or an error.

This approach can make business rules easier to reason about because the transformation is explicit.

#### Mapping DDD Concepts to FP

**Entities**

An Entity can be represented as an immutable record or data structure containing a stable identity.

Its identity distinguishes it from other Entities even when its attributes change.

**Value Objects**

Value Objects map naturally to immutable records and algebraic data types.

The type system can enforce valid values and make invalid states difficult or impossible to represent.

**Aggregates**

An Aggregate can be represented as a data structure containing its state, with functions controlling valid transitions.

A function receives the current Aggregate and returns either a new valid Aggregate or a domain error.

**Domain Services**

Domain operations can be represented as pure functions that operate on domain values and return results without performing external side effects.

**Repositories**

Repositories can be represented through function types or interfaces that isolate persistence operations from domain logic.

The actual database interaction remains at the application or infrastructure boundary.

#### Common FP Practices

- Use immutable data structures.
- Model alternatives and states with discriminated unions or other algebraic data types.
- Use pattern matching to handle domain states explicitly.
- Represent missing values and failures explicitly.
- Prefer small, composable functions.
- Separate pure domain transformations from side effects.
- Use types to encode business constraints.
- Make invalid states difficult to construct.

#### Example: An Order Model in F#

```fsharp
type OrderId = OrderId of Guid

type ProductId = ProductId of Guid

type OrderStatus =
    | Draft
    | Confirmed
    | Cancelled

type Money = {
    Amount: decimal
    Currency: string
}

type OrderItem = {
    ProductId: ProductId
    Quantity: int
    UnitPrice: Money
}

type Order = {
    Id: OrderId
    Status: OrderStatus
    Items: OrderItem list
}

type OrderError =
    | OrderIsNotEditable
    | OrderHasNoItems
    | InvalidQuantity

let addItem item order =
    match order.Status with
    | Draft when item.Quantity > 0 ->
        Ok {
            order with
                Items = item :: order.Items
        }
    | Draft ->
        Error InvalidQuantity
    | _ ->
        Error OrderIsNotEditable

let confirm order =
    match order.Status, order.Items with
    | Draft, [] ->
        Error OrderHasNoItems

    | Draft, _ ->
        Ok {
            order with
                Status = Confirmed
        }

    | _ ->
        Error OrderIsNotEditable
```

The example uses immutable records and functions to model the Aggregate's behavior.

The `confirm` function does not modify the existing Order. It returns a new version when the transition is valid, or an explicit error when it is not.

This style makes state transitions visible and allows the compiler to help enforce the model's structure.

In a production implementation, additional types and constructors should enforce constraints such as valid Money values, supported currencies, and valid identifiers. The example intentionally focuses on the modeling approach.

#### Functional Workflows

A common functional approach is to compose small functions into a larger business workflow.

For example:

```text
Receive Input
     |
     v
Validate Request
     |
     v
Load Domain Data
     |
     v
Apply Business Rules
     |
     v
Persist Changes
     |
     v
Publish Relevant Events
```

Each step can have an explicit input and output type.

Pure domain operations remain separate from effects such as database access, HTTP requests, and message publication.

This makes the workflow easier to test because the business transformations can be tested independently of the infrastructure.

### 1.3 Comparing OOP and FP

| Concern | Object-Oriented Approach | Functional Approach |
|---|---|---|
| Entities | Objects with stable identity | Immutable data with stable identity |
| Value Objects | Immutable classes, records, or structs | Immutable records and algebraic types |
| Aggregates | Root controls internal state | Functions transform Aggregate state |
| Business behavior | Methods and domain services | Composable functions |
| State changes | Often controlled mutation | New state returned from transformations |
| Errors | Exceptions or explicit result types | Often explicit result types or sum types |
| Side effects | Encapsulated behind objects and interfaces | Isolated at application boundaries |
| Composition | Object composition and polymorphism | Function composition and type composition |

These are tendencies, not strict rules.

An OOP system can use immutable data and pure functions. An FP system can model Entities, Aggregate boundaries, and identity explicitly.

The goal is to choose the representation that communicates the domain most clearly.

### 1.4 Hybrid Approaches

Many production systems combine object-oriented and functional techniques.

For example:

- Use classes for Aggregate Roots and encapsulation.
- Use immutable records for Value Objects.
- Use pure functions for calculations and complex transformations.
- Use application services to coordinate effects.
- Use interfaces to isolate external dependencies.
- Use algebraic types or result types to represent domain errors.

A C# application can use immutable records, pattern matching, and pure functions without becoming a purely functional application.

Likewise, an F# application can use interfaces and encapsulated components where they make sense.

A hybrid approach is often a practical choice, especially when working with existing systems, established teams, or languages that support multiple styles.

---

## 2. Choosing an Implementation Approach

The programming paradigm should be selected based on the problem, the team, and the surrounding system.

Consider the following dimensions.

### Team Expertise

A paradigm that the team understands and applies consistently is often more valuable than a theoretically elegant approach that nobody can maintain.

This does not mean avoiding learning new techniques. It means accounting for the team's ability to design, review, test, and operate the resulting software.

### Domain Characteristics

Different domains may benefit from different modeling techniques.

For example:

- A complex stateful workflow may benefit from explicit state transitions and strong encapsulation.
- A calculation-heavy domain may benefit from pure functions and immutable transformations.
- A financial domain may benefit from precise Value Objects, explicit transaction states, and carefully controlled side effects.
- A rule-heavy domain may benefit from types and functions that make valid operations explicit.

These are tendencies, not rules. Both OOP and FP can model these domains effectively.

### Existing Codebase

A new application has more freedom in its implementation choices.

A legacy application has existing constraints:

- Established frameworks and libraries.
- Persistence models.
- Integration contracts.
- Team conventions.
- Deployment architecture.
- Existing domain behavior that must be preserved.

Introducing DDD does not necessarily require rewriting the application or replacing its programming paradigm.

It is often more effective to introduce explicit domain models around the most important business operations and evolve the architecture incrementally.

### Operational and Technical Requirements

Consider:

- Performance and memory usage.
- Concurrency requirements.
- Persistence technology.
- Integration protocols.
- Deployment constraints.
- Observability and troubleshooting.
- Long-term maintainability.

The implementation should support the domain without introducing unnecessary architectural complexity.

---

## 3. Separating Domain Logic from Technical Concerns

Regardless of programming paradigm, a DDD implementation should distinguish business decisions from technical mechanisms.

A business operation may require:

- Validating input.
- Retrieving domain data.
- Applying business rules.
- Updating an Aggregate.
- Persisting changes.
- Publishing an event.
- Calling an external service.

These responsibilities are related, but they do not all belong in the domain model.

A useful separation is:

**Domain Model**

Contains business concepts, rules, invariants, and domain behavior.

**Application Layer**

Coordinates use cases and manages the execution flow.

**Infrastructure**

Implements persistence, messaging, external services, and other technical dependencies.

**Presentation**

Receives external requests and translates them into application-level operations.

For example, consider a `PlaceOrder` operation:

```text
HTTP Request
     |
     v
Presentation Layer
     |
     v
Application Use Case
     |
     +----> Load Customer
     |
     +----> Load Product Information
     |
     +----> Create and Validate Order
     |
     +----> Persist Order
     |
     +----> Publish Integration Event
     |
     v
HTTP Response
```

The domain model should not need to know whether the request came from an HTTP API, a background worker, a message consumer, or a command-line application.

Likewise, the domain should not depend on a particular database or message broker.

This separation makes the domain model easier to test and allows infrastructure to evolve without rewriting business rules.

### Persistence Should Support the Model

A common implementation mistake is allowing the database schema to dictate the domain model.

For example, a database may contain:

```text
Orders
OrderItems
Customers
Products
```

That does not mean the domain model must reproduce these tables as Entities with the same relationships.

The domain model should reflect business concepts and consistency requirements. Persistence mapping should translate between the model and its storage representation.

This may involve:

- ORM mapping.
- Data mappers.
- Separate persistence models.
- Serialization and deserialization.
- Explicit conversion between domain types and DTOs.

A relational database may store a Value Object across several columns. An ORM may map several domain objects into one table or one Aggregate into multiple tables.

The storage representation is an implementation detail, not the definition of the domain.

### External Integrations Should Respect Context Boundaries

When communicating with another Bounded Context or an external provider:

- Use explicit integration contracts.
- Translate external representations into local domain concepts.
- Avoid sharing internal domain objects across context boundaries.
- Handle timeouts, retries, and failures explicitly.
- Make message processing idempotent where duplicate delivery is possible.
- Test serialization and contract compatibility.

An Anticorruption Layer may be appropriate when the external model does not fit the local domain.

The objective is to keep the local model consistent even when external systems have different terminology, assumptions, or data structures.

---

## 4. Testing as Part of Domain Design

Testing is not merely a way to detect defects after implementation.

It is a way to express expected behavior, challenge assumptions, and receive continuous feedback about the quality of the model.

A well-designed domain model should make important business rules observable and testable.

For example:

```text
Given an order with no items
When the customer attempts to confirm it
Then the operation must be rejected
```

This scenario communicates a business rule independently of the programming language, database, or API.

It can become an automated unit test, an acceptance test, or part of a broader test suite.

### 4.1 Testing the Domain Model

The domain model should be testable without requiring external infrastructure whenever possible.

Focus on:

- Entity behavior.
- Value Object validation and equality.
- Aggregate invariants.
- State transitions.
- Domain Services.
- Domain Event generation.
- Domain error conditions.

For example, test that:

- An order cannot be confirmed without items.
- A cancelled order cannot be modified.
- A Money calculation respects currency rules.
- A shipment cannot be dispatched before preparation.
- A successful operation produces the expected domain occurrence.

These tests should verify observable business behavior rather than internal implementation details.

### 4.2 Testing Application Use Cases

Application tests verify that use cases coordinate the required operations correctly.

For example, a `PlaceOrder` use case may need to:

1. Retrieve customer information.
2. Load product details.
3. Create an Order.
4. Apply the appropriate domain behavior.
5. Persist the Aggregate.
6. Publish an event after the required transaction succeeds.

Application tests should verify that the workflow handles:

- Successful execution.
- Missing or unavailable data.
- Domain rule violations.
- Persistence failures.
- External service failures.
- Transaction boundaries.
- Expected application results.

Use test doubles when they help isolate the application workflow, but avoid mocking every internal method or reproducing the implementation in the test.

### 4.3 Testing Persistence

Persistence integration tests verify that the domain model is correctly mapped to and from its storage representation.

They should cover:

- Aggregate persistence and retrieval.
- Value Object mapping.
- Entity identity.
- Relationships and owned objects.
- Transaction behavior.
- Optimistic concurrency.
- Database constraints.
- Serialization and deserialization where relevant.

For example, an Order Aggregate may behave correctly in memory but fail to restore its internal state after being loaded from the database.

A unit test cannot detect that problem. A persistence integration test can.

Use the actual persistence technology in these tests whenever the behavior depends on its implementation.

A mocked repository cannot prove that an ORM mapping, database constraint, or transaction behaves correctly.

### 4.4 Testing External Integrations

Integration boundaries deserve explicit testing because they are where independently developed components meet.

Examples include:

- REST API request and response contracts.
- Message serialization and deserialization.
- Message publishing and consumption.
- Authentication and authorization integration.
- Third-party service adapters.
- File storage.
- External data providers.

Contract tests can verify that a provider and its consumers agree on the shape and meaning of an interface.

Consumer-driven contract testing is particularly useful when multiple teams evolve services independently.

The objective is not to test every possible behavior of every external system. It is to verify the assumptions your application makes about the systems it depends on.

### 4.5 End-to-End Testing

End-to-end tests verify important business journeys across the integrated application.

For example:

```text
Customer Places Order
        |
        v
Order Is Accepted
        |
        v
Payment Is Authorized
        |
        v
Shipment Is Created
        |
        v
Customer Can Track Order
```

These tests provide confidence that the important parts of the system work together.

However, they are usually slower, more expensive, and more fragile than focused domain and integration tests.

Do not attempt to test every business rule through the user interface.

Test detailed rules at the domain level, test infrastructure boundaries through integration tests, and reserve end-to-end tests for a small number of high-value journeys.

### 4.6 Testing Pyramid and Test Strategy

A useful starting point is the Testing Pyramid.

```text
              /\
             /  \
            / E2E\
           /------\
          /Contract \
         /Integration\
        /------------\
       / Domain & Unit \
      /________________\
```

The general idea is to have many focused, fast tests, a meaningful set of integration and contract tests, and a smaller number of end-to-end tests.

The exact distribution depends on the application. The pyramid is a heuristic, not a mandatory numerical ratio.

| Test Level | Main Purpose | Example |
|---|---|---|
| Domain unit tests | Verify business rules | An order cannot be confirmed without items |
| Application tests | Verify use-case coordination | Placing an order invokes the expected operations |
| Persistence integration tests | Verify database behavior | An Aggregate can be saved and reloaded correctly |
| Contract tests | Verify integration agreements | A message consumer can process the published contract |
| End-to-end tests | Verify critical user journeys | A customer can place and track an order |

A useful rule is to test a behavior at the lowest level that can provide meaningful confidence.

If a higher-level test finds a defect that could have been detected by a focused lower-level test, consider adding that lower-level regression test as well.

---

## 5. Testing Techniques and Practices

### 5.1 Test-Driven Development (TDD)

Test-Driven Development is a development practice in which tests guide the implementation.

The familiar cycle is:

```text
Red
 |
 v
Write a failing test
 |
 v
Green
 |
 v
Implement the simplest solution
 |
 v
Refactor
 |
 +------> Repeat
```

In a DDD context, TDD can help discover the behavior an object or function needs to express.

For example:

1. Write a test that confirms an order cannot be confirmed without items.
2. Implement the minimum behavior needed to satisfy it.
3. Refactor the Aggregate while preserving the test.
4. Add more examples for valid and invalid transitions.

TDD is not about writing tests for every private method before writing the implementation.

It is about using observable behavior to guide design and protect important rules.

### 5.2 Behavior-Driven Development (BDD)

BDD focuses on discovering, communicating, and validating expected behavior through concrete examples.

It encourages collaboration between developers, domain experts, testers, and other stakeholders.

A scenario can be expressed using Given, When, and Then:

```gherkin
Feature: Order confirmation

  Scenario: Confirming an order with valid items

    Given an order containing at least one item
    When the customer confirms the order
    Then the order should be confirmed
```

BDD is more than writing Gherkin files.

Its value comes from the conversations that establish what the examples mean and the feedback that automated scenarios provide.

A useful BDD workflow connects:

- Domain exploration.
- Example discovery.
- Acceptance criteria.
- Implementation.
- Automated verification.

Not every unit test needs to be written in Gherkin. Use the format where it improves shared understanding.

### 5.3 Property-Based Testing

Example-based tests verify specific scenarios. Property-based testing verifies general properties across many generated inputs.

For example, instead of testing only a few Money calculations, define properties such as:

- Adding zero should not change the amount.
- Adding compatible monetary values should preserve the currency.
- A valid quantity multiplied by a unit price should produce the expected subtotal.

Property-based testing can be especially useful for:

- Financial calculations.
- Parsers and validators.
- Complex domain transformations.
- State machines.
- Serialization and deserialization.
- Mathematical and allocation rules.

The properties must still be grounded in the domain. Generating random values without meaningful invariants does not automatically create valuable tests.

### 5.4 State Transition Testing

Many domain concepts have explicit lifecycles.

An Order may move through states such as:

```text
Draft
  |
  v
Confirmed
  |
  v
Fulfillment
  |
  v
Shipped
  |
  v
Delivered
```

Some transitions may be prohibited.

For example:

- A Delivered order cannot return to Draft.
- A Cancelled order cannot be shipped.
- A Draft order cannot be delivered.

State transition tests should cover both permitted and prohibited transitions.

For domains with many states and transitions, consider modeling the lifecycle explicitly with a state machine or algebraic data types.

### 5.5 Concurrency Testing

Business invariants must remain valid under concurrent requests, not just sequential unit tests.

Consider inventory allocation:

- Two customers attempt to reserve the last available item.
- Both requests read the same available quantity.
- Both attempt to reserve it.

A sequential domain unit test may pass, while the actual system allows an oversell because persistence does not enforce the necessary concurrency behavior.

Concurrency testing should cover:

- Optimistic concurrency.
- Competing updates.
- Duplicate commands.
- Idempotent processing.
- Transaction isolation.
- Message ordering where required.

These tests often require the real database or infrastructure component involved in the operation.

### 5.6 Architecture Tests

Architecture tests verify structural rules that should remain true as the codebase evolves.

Examples:

- The Domain project must not reference Infrastructure.
- Domain objects must not depend on HTTP or database frameworks.
- Bounded Contexts must not access one another's internal persistence models.
- Public integration contracts must not expose internal implementation types.
- Dependency direction must remain consistent with the intended architecture.

Architecture tests help prevent gradual erosion of boundaries.

They are particularly valuable in larger systems where multiple developers and teams contribute to the same codebase.

---

## 6. Test Doubles: Mocks, Stubs, and Fakes

Test doubles are substitutes used to isolate the system under test from dependencies.

They can be useful, but should be chosen according to the behavior being tested.

**Stub**

Provides predefined responses to a dependency's calls.

For example, a stubbed exchange-rate provider returns a known rate for a particular currency pair.

**Mock**

Verifies expected interactions with a dependency.

For example, a mock can verify that an application service requested a payment authorization after the order was validated.

**Fake**

Provides a working but simplified implementation.

For example, an in-memory Repository can support domain or application tests without a real database.

#### Use Test Doubles Carefully

A common mistake is mocking every dependency and testing that methods call other methods in a particular sequence.

Such tests may pass even when the business outcome is incorrect.

Prefer verifying meaningful behavior and results.

Use mocks when the interaction itself is important, such as ensuring that an external payment operation is not invoked after a business rule rejects an order.

Use a real database integration test when you need confidence in actual persistence behavior.

Use a fake when its behavior is sufficiently representative of the dependency for the test's purpose.

Avoid building elaborate fake infrastructure that duplicates the real system's behavior inaccurately.

---

## 7. Testing Across Programming Paradigms

The testing strategy should reflect the implementation style without changing the business expectations.

### OOP Testing

In an object-oriented implementation, tests often:

- Construct Entities and Value Objects.
- Invoke public domain methods.
- Verify state transitions.
- Verify emitted Domain Events.
- Test Aggregate invariants.
- Use test doubles for external dependencies.

For example:

```csharp
[Fact]
public void Confirm_ShouldRejectAnEmptyOrder()
{
    var order = Order.Create(OrderId.New());

    Assert.Throws<InvalidOperationException>(
        () => order.Confirm());
}
```

This test verifies a business rule through the Aggregate's public behavior.

It does not need a database or application server.

### FP Testing

In a functional implementation, tests often:

- Construct immutable domain values.
- Call pure functions.
- Verify returned values.
- Verify explicit error cases.
- Test properties across generated inputs.
- Test workflow composition separately from side effects.

For example:

```fsharp
[<Fact>]
let ``Confirm rejects an empty order`` () =
    let order = {
        Id = OrderId(Guid.NewGuid())
        Status = Draft
        Items = []
    }

    let result = confirm order

    Assert.Equal(
        Error OrderHasNoItems,
        result
    )
```

Pure functions are particularly straightforward to test because their outputs depend on their inputs rather than hidden external state.

However, functional systems still need integration tests for persistence, message delivery, serialization, and other effects.

### Testing Hybrid Implementations

In a hybrid system:

- Test pure calculations as functions.
- Test Aggregate behavior through public methods.
- Test application workflows independently.
- Test infrastructure adapters against their actual dependencies.
- Test integration contracts between contexts.

The objective is always the same: validate the domain behavior and the system boundaries at the appropriate level.

---

## 8. Continuous Integration and Quality Feedback

Tests provide the most value when they are part of the normal development workflow.

A Continuous Integration pipeline should run relevant checks automatically whenever code changes.

A practical pipeline may include:

1. Restore dependencies.
2. Compile the application.
3. Run static analysis and code quality checks.
4. Run domain unit tests.
5. Run application and integration tests.
6. Run contract tests where applicable.
7. Produce build artifacts.
8. Run deployment or acceptance checks in an appropriate environment.

Not every test needs to run at every stage. Slow or environment-dependent tests may have a separate execution strategy.

The important thing is to provide fast feedback for common changes while maintaining confidence in the full system.

### Test Quality Matters

A high test count does not necessarily mean a reliable system.

Pay attention to:

- Test execution time.
- Flaky tests.
- Difficult-to-understand test setup.
- Tests coupled to implementation details.
- Missing coverage of important business rules.
- Integration boundaries that are never exercised.
- Production defects that repeatedly escape the test suite.

Tests should be maintained as part of the product, not treated as disposable development artifacts.

---

## Practical Exercise: Implement and Test a Business Use Case

Choose one meaningful business operation from a domain you understand.

Examples:

- Place and confirm an order.
- Enroll a student in a course.
- Allocate a vehicle to a delivery.
- Reserve inventory.
- Approve an insurance claim.
- Issue a shipment.

Use the same business operation to explore implementation and testing.

### Part A: Define Expected Behavior

Describe the use case in business language.

Identify:

1. The actor initiating the operation.
2. The business intention.
3. Preconditions.
4. Business rules and invariants.
5. Valid state transitions.
6. Expected outcomes.
7. Rejected operations and exceptional cases.

Write at least three concrete scenarios: one successful scenario, one invalid operation, and one important edge case.

### Part B: Choose an Implementation Style

Implement the same use case using either:

- Object-Oriented Programming.
- Functional Programming.
- A hybrid approach.

Explain why you chose that approach.

Identify how your implementation represents:

- Entities.
- Value Objects.
- Aggregate boundaries.
- Domain behavior.
- Domain Events.
- Application orchestration.
- Persistence and external dependencies.

If possible, implement the same small domain operation in a second style and compare the trade-offs.

### Part C: Implement the Use Case

Keep domain decisions inside the domain model and orchestration in the application layer.

The implementation should:

- Enforce business invariants.
- Handle invalid transitions.
- Keep external dependencies at appropriate boundaries.
- Persist the required state.
- Produce meaningful domain or integration events where needed.
- Avoid exposing internal domain structures as external contracts.

### Part D: Build the Test Suite

Write tests for the following:

**Domain Tests**

- Entity behavior.
- Value Object validation and equality.
- Aggregate invariants.
- Valid and invalid state transitions.
- Domain Events.

**Application Tests**

- Successful execution.
- Domain rule rejection.
- Missing data.
- Persistence failures.
- External dependency failures.

**Integration Tests**

- Repository persistence and retrieval.
- Database mapping.
- Transaction behavior.
- Message serialization and consumption, if relevant.

**Contract or End-to-End Tests**

- At least one important integration contract.
- At least one critical business journey.

### Part E: Evaluate the Quality of the Tests

Ask:

- Do the tests describe business behavior clearly?
- Can domain tests run without infrastructure?
- Are tests checking outcomes rather than private implementation details?
- Are important persistence and integration boundaries covered?
- Are there duplicate tests that provide no additional confidence?
- Can a developer understand a failure quickly?
- Do the tests protect the business rules that matter most?

**Deliverable:** A working use case, its domain model, a test suite, and a short explanation of the implementation and testing decisions.

---

## Completion Criteria

You have completed this section when you can:

### Implementation

- Explain why DDD is independent of a particular programming paradigm.
- Describe how Entities, Value Objects, Aggregates, and Domain Services can be implemented using OOP and FP.
- Explain the differences between mutable and immutable domain models.
- Use algebraic data types and explicit error types where appropriate.
- Explain how OOP and FP techniques can be combined.
- Choose an implementation style based on domain requirements, team expertise, and existing constraints.
- Separate domain behavior from application orchestration.
- Keep persistence and external integrations from defining the domain model.
- Explain why the database schema should not dictate Aggregate boundaries.

### Testing

- Write behavior-focused tests for domain rules.
- Test Aggregate invariants and state transitions.
- Test application workflows independently from infrastructure.
- Test persistence mappings using real persistence infrastructure where needed.
- Test external integration contracts.
- Explain the purpose and trade-offs of unit, integration, contract, and end-to-end tests.
- Apply TDD and BDD appropriately.
- Understand when property-based, concurrency, and architecture tests provide additional confidence.
- Use mocks, stubs, and fakes without coupling tests unnecessarily to implementation details.
- Integrate automated tests into a continuous integration workflow.

---

## Recommended Resources

### Domain Modeling and Programming Paradigms

**[Domain Modeling Made Functional](https://pragprog.com/titles/swdddf/domain-modeling-made-functional/)** — Scott Wlaschin.

An excellent resource for understanding how DDD concepts can be expressed through functional programming and F#.

It covers domain discovery, types, Entities, Value Objects, Aggregates, workflows, error handling, persistence, and testing. Although the examples use F#, many of the modeling principles apply to other languages.

**[Functional and Reactive Domain Modeling](https://www.manning.com/books/functional-and-reactive-domain-modeling)** — Debasish Ghosh.

Explores functional domain models, composition, reactive systems, and the relationship between domain modeling and distributed application design. Examples use Scala.

**[Patterns, Principles, and Practices of Domain-Driven Design](https://www.wiley.com/en-us/Patterns%2C+Principles%2C+and+Practices+of+Domain-Driven+Design-p-9781118714706)** — Scott Millett and Nick Tune.

A useful resource for implementing DDD in object-oriented systems, particularly with C#.

### Testing and Design

**[Growing Object-Oriented Software, Guided by Tests](https://growing-object-oriented-software.com/)** — Steve Freeman and Nat Pryce.

A practical guide to TDD, object-oriented design, test doubles, and building maintainable software through tests.

**[The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)** — Martin Fowler.

Explains different test levels, their trade-offs, and how to build an effective testing strategy.

**[Unit Testing Best Practices for .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)** — Microsoft.

Practical guidance for writing readable, maintainable, and reliable unit tests in .NET applications.

**[Testing in .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/)** — Microsoft.

An overview of unit, integration, and load testing, along with the available .NET testing tools.

### Further Learning

- **[F# for Fun and Profit](https://fsharpforfunandprofit.com/)** — Scott Wlaschin. A substantial collection of functional programming articles, including domain modeling and practical F# techniques.
- **[Martin Fowler: Test-Driven Development](https://martinfowler.com/bliki/TestDrivenDevelopment.html)** — A useful introduction to the principles and practice of TDD.
- **[Cucumber: BDD and Example Mapping](https://cucumber.io/docs/bdd/example-mapping/)** — Practical material on discovering and clarifying behavior through concrete examples.

---

Implementation is where the model meets the constraints of the real world: programming languages, frameworks, databases, integrations, teams, and deployment environments.

Testing is how we continuously check that those constraints have not distorted the meaning of the domain.

The best implementation is not necessarily the one with the most patterns, the most sophisticated type system, or the highest number of tests.

It is the one that expresses the business clearly, protects the rules that matter, and gives the team enough confidence to change the system without losing that understanding.