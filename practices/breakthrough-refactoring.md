# Breakthrough Refactoring

One of the most important ideas in Eric Evans' *Domain-Driven Design: Tackling Complexity in the Heart of Software* is **refactoring toward deeper insight**.

In DDD, refactoring is not limited to improving code structure or removing technical debt. It can also be a way of expressing newly discovered domain knowledge. As the team develops a deeper understanding of the business, the domain model should evolve to reflect that understanding.

Sometimes the result is a small improvement. Sometimes it is a fundamental change in how the software represents the business.

Consider a logistics system where developers initially believe that assigning a truck to a shipment means the shipment has been dispatched. After conversations with dispatchers, they discover that assignment, loading, authorization, and actual departure are separate business activities, each with its own rules.

The original model may work for simple scenarios, but it cannot accurately represent the business process. Adding more conditions to the existing `DispatchShipment()` method will not solve the underlying problem. The team needs to reconsider what dispatching actually means.

This is the essence of breakthrough refactoring.

## What Is Breakthrough Refactoring?

Breakthrough refactoring is a significant model improvement driven by a deeper understanding of the domain.

It happens when the team discovers that its existing model does not adequately express the business, its rules, or the relationships between its concepts.

The discovery may reveal that:

- Two concepts previously treated as one actually have different responsibilities.
- A seemingly simple operation represents a more meaningful business process.
- An important business rule has been hidden inside application logic.
- A relationship between domain concepts has been misunderstood.
- A concept that was previously missing deserves an explicit representation.
- The current model reflects technical or organizational assumptions rather than the business itself.

For example, consider an e-commerce application with a `Customer` class containing an `IsActive` property.

Initially, the team assumes that an inactive customer cannot place orders. Later, domain experts explain that a customer account can be active or suspended, while individual purchasing permissions depend on additional conditions, such as credit restrictions or fraud investigations.

The original Boolean property has combined several different business concepts.

A deeper understanding may lead to explicit concepts such as `CustomerAccountStatus`, `CreditRestriction`, and `PurchasingEligibility`, each with its own meaning and rules.

The response is not simply to add another condition or workaround. It is to reconsider the model and reshape it around the new understanding.

**Ordinary refactoring improves the structure of an existing model. Breakthrough refactoring can change the model itself because the team has learned something important about the domain.**

## Characteristics of Breakthrough Refactoring

### Driven by New Understanding

Breakthrough refactoring begins with a discovery.

That discovery might come from conversations with domain experts, Event Storming, Example Mapping, investigating unusual business scenarios, analyzing production behavior, or questioning an assumption that everyone previously accepted.

For example, a hotel reservation system initially treats a reservation as confirmed as soon as a customer submits a booking request.

During an exploration session, the reservation team explains that a booking request is not a confirmed reservation. Availability must be checked, payment authorization may be required, and the reservation can remain pending until confirmation.

This changes the team's understanding of the process.

The original model:

```csharp
public class Reservation
{
    public Guid Id { get; private set; }
    public DateTime CheckIn { get; private set; }
    public DateTime CheckOut { get; private set; }
    public bool IsConfirmed { get; private set; }
}
```

The model does not clearly distinguish between requesting a reservation, confirming it, and authorizing payment.

The team may discover that these are distinct business decisions and transitions. The new model should express those distinctions rather than treating confirmation as a simple Boolean update.

The important part is that the team learns something that changes how it understands the domain.

### Significant Conceptual Changes

The resulting changes may go beyond renaming classes or extracting methods.

They can include:

- Introducing previously undiscovered domain concepts.
- Separating concepts that were incorrectly combined.
- Changing relationships between Entities and Value Objects.
- Redesigning Aggregate boundaries.
- Moving behavior to the concepts responsible for it.
- Changing how a business process is represented.
- Revisiting Bounded Context boundaries when the discovery has strategic implications.

Consider a payment system that initially represents all payment activity using a single `Payment` Entity.

As the team investigates refunds, it discovers that a refund is not simply a negative payment. It has its own lifecycle, approval rules, limits, and relationship to the original payment.

The model may evolve from:

```text
Payment
  Amount
  Status
  IsRefund
```

Into something more expressive:

```text
Payment
  Authorize()
  Capture()
  Fail()

Refund
  Request()
  Approve()
  Reject()
  Complete()
```

This is not merely a class decomposition. It represents a newly discovered distinction in the business.

Not every breakthrough requires a large architectural change. The significance comes from the improvement in understanding, not the number of files changed.

### Improved Model Expressiveness

The purpose is to make the model communicate the business more accurately.

A better model should help developers and domain experts reason about business behavior, recognize important distinctions, and discuss changes using a shared language.

The code should not merely contain the right information. Its structure and behavior should express what that information means.

Consider this operation:

```csharp
shipment.Status = ShipmentStatus.Completed;
```

What does `Completed` mean?

Has the shipment been loaded? Has the vehicle left the warehouse? Has it arrived at its destination? Has the customer received the goods?

If the domain distinguishes these situations, a generic status update hides important business meaning.

A more expressive model might expose operations such as:

```csharp
shipment.AssignVehicle(vehicleId);
shipment.CompleteLoading();
shipment.AuthorizeDispatch();
shipment.Dispatch();
shipment.RecordDelivery();
```

Each operation represents a business intention and provides a place to enforce its corresponding rules.

The goal is not to create a method for every sentence in a business conversation. It is to make meaningful distinctions explicit when they affect business behavior.

### Iterative and Exploratory

Breakthrough refactoring is often exploratory.

The team may try different representations, compare their ability to express business scenarios, and revise the model as further questions emerge.

For example, while modeling inventory, the team may initially consider `AvailableQuantity` a simple property:

```csharp
public int AvailableQuantity { get; private set; }
```

Further exploration reveals that stock can be physically present but reserved for other orders. Some stock may also be damaged, quarantined, or awaiting inspection.

The team now needs to distinguish:

- Physical stock.
- Reserved stock.
- Available stock.
- Unavailable or quarantined stock.

A possible calculation is:

```text
Available Stock = Physical Stock - Reserved Stock - Unavailable Stock
```

But even this formula may be insufficient if reservations expire or stock is allocated across warehouses.

The team can explore alternative models and validate them against real scenarios before settling on a design.

The process is not about finding a perfect abstraction in one attempt. It is about progressively discovering a more useful model.

## How Breakthrough Refactoring Relates to DDD

### Ubiquitous Language

Ubiquitous Language is the foundation for communicating domain understanding through the model.

As conversations reveal more precise meanings, distinctions, and relationships, the language evolves. The implementation should evolve with it.

For example, if domain experts distinguish between `Shipment Assigned` and `Shipment Dispatched`, the model should not continue representing both as a generic `ShipmentUpdated` operation.

The distinction might be:

- **Assigned:** A vehicle and driver have been allocated to the shipment.
- **Loaded:** The goods have been loaded onto the assigned vehicle.
- **Dispatched:** The vehicle has actually departed under the required authorization.

These terms represent different business facts, not alternative names for the same operation.

The model may therefore evolve from:

```csharp
shipment.UpdateStatus("Dispatched");
```

Into:

```csharp
shipment.AssignVehicle(vehicleId, driverId);
shipment.CompleteLoading();
shipment.AuthorizeDispatch();
shipment.Dispatch();
```

The corresponding Domain Events can also express the distinctions:

```text
ShipmentAssigned
ShipmentLoadingCompleted
ShipmentDispatchAuthorized
ShipmentDispatched
```

Breakthrough refactoring helps bring the code into alignment with the language and understanding shared by the team.

### Domain Exploration

Domain exploration provides the knowledge that can trigger breakthrough refactoring.

Techniques such as Event Storming, Example Mapping, Domain Storytelling, and collaborative modeling help uncover:

- Missing concepts.
- Conflicting assumptions.
- Hidden business rules.
- Exceptional scenarios.
- Differences in terminology.
- Previously misunderstood relationships.

Consider a subscription platform.

The initial implementation has a simple operation:

```text
CancelSubscription
```

During an Event Storming session, the team discovers that customers can request cancellation, but the subscription may remain active until the end of the paid billing period.

The business distinguishes between:

1. Cancellation requested.
2. Cancellation scheduled.
3. Subscription access ended.
4. Subscription terminated.

The team also discovers that some subscriptions can be reactivated before their scheduled end date.

This exploration reveals that cancellation is not an immediate state change. It is a business process with a lifecycle.

The original `CancelSubscription()` operation may need to be replaced with explicit operations and transitions that reflect the actual business rules.

These discoveries can reveal that the existing model needs more than incremental adjustments.

### Strategic Design

Some discoveries affect only the internal structure of a Bounded Context. Others reveal that the boundaries between models or responsibilities need reconsideration.

For example, imagine an e-commerce platform where the `Order` context initially owns both order fulfillment and shipment tracking.

As the business grows, the team discovers that shipment tracking has distinct responsibilities:

- Managing shipment assignments.
- Coordinating carriers.
- Tracking transportation milestones.
- Handling delivery attempts.
- Managing delivery exceptions.

The Order context cares about whether an order has been fulfilled. The Shipping context cares about the lifecycle of a shipment.

These are related but different models.

The discovery may justify introducing a separate Shipping Bounded Context, with an explicit integration contract between Order and Shipping.

The relationship might look like:

```text
Order Context
     |
     | OrderReadyForFulfillment
     v
Shipping Context
     |
     | ShipmentDispatched
     | ShipmentDelivered
     v
Order Context
```

This is not simply a decision to create another microservice. It is a recognition that the business has distinct responsibilities and models.

Such discoveries can lead to changes in Bounded Context boundaries, Context Maps, ownership, and integration contracts.

These changes should be based on domain understanding, not simply on a desire to split an application into more services.

### Tactical Design

Tactical patterns provide the building blocks for expressing the improved model.

As understanding develops, the team may discover that:

- A primitive value should become a meaningful Value Object.
- A concept previously modeled as a Value Object actually has identity.
- An Aggregate boundary does not protect the required invariants.
- A business operation belongs to an Entity rather than an Application Service.
- A missing Domain Event represents an important business occurrence.

Consider a money transfer system.

The initial implementation might accept two account identifiers and an amount:

```csharp
public void Transfer(
    Guid sourceAccountId,
    Guid destinationAccountId,
    decimal amount)
{
    // Load accounts.
    // Check balance.
    // Update balances.
    // Save changes.
}
```

The team initially places all business decisions inside an Application Service.

Further exploration reveals important rules:

- The source and destination accounts must be different.
- The source account must be permitted to transfer funds.
- The transfer amount must be positive.
- The source account must have sufficient available funds.
- The transfer must be recorded as a meaningful business operation.

The model may evolve to include:

- A `Money` Value Object that prevents invalid monetary values.
- An `Account` Entity responsible for its balance-related rules.
- A `Transfer` concept representing the business operation.
- A Domain Event such as `FundsTransferred`.

The Application Service still coordinates the use case, but it no longer owns every business decision.

The patterns are not the objective. They are tools for expressing the model that the team has discovered.

### Testing and Refactoring

Tests make model evolution safer.

They help preserve existing business behavior while the implementation changes, expose assumptions encoded in the old model, and verify that the new model supports the scenarios discovered during exploration.

However, tests should not blindly preserve the old design. When the team discovers that an old assumption was wrong, the tests may need to change as well.

Consider the earlier shipment example.

An existing test might assert:

```text
Assigning a vehicle changes shipment status to Dispatched.
```

After talking to dispatchers, the team discovers that assignment does not mean dispatch.

The test itself encodes an incorrect business assumption.

It should be replaced with scenarios such as:

```text
Given a shipment awaiting assignment
When a vehicle is assigned
Then the shipment becomes Assigned

Given a shipment with an assigned vehicle
When loading is completed
Then the shipment becomes ReadyForDispatch

Given a shipment awaiting dispatch authorization
When dispatch is authorized and the vehicle departs
Then the shipment becomes Dispatched
```

The objective is to preserve valid business behavior, not every detail of the previous implementation.

## Breakthrough Refactoring in Practice

A practical approach involves several connected activities.

### 1. Recognize the Need for Change

Look for signs that the current model no longer represents the domain effectively:

- Developers repeatedly struggle to explain what a concept means.
- Business rules are scattered across unrelated components.
- New requirements require increasingly complicated conditions.
- The same term represents different concepts in different scenarios.
- Workarounds accumulate around an existing abstraction.
- Domain experts frequently correct the team's interpretation.

For example, suppose a logistics system has a `CompleteWaybill()` method that performs all of the following:

- Validates the shipment.
- Checks the driver's eligibility.
- Updates vehicle information.
- Changes shipment status.
- Calculates charges.
- Sends notifications.
- Updates delivery records.

Every new business rule makes the method more complicated.

This may look like a code organization problem, but it may also indicate that the team has combined several distinct business operations into one concept.

These symptoms do not automatically prove that a breakthrough is needed, but they are good reasons to investigate.

### 2. Return to the Domain

Do not begin by redesigning classes.

Talk to domain experts, explore concrete scenarios, challenge assumptions, and investigate the business rules behind the problem.

Ask questions that expose meaning:

- What actually happens in this situation?
- Why does this operation exist?
- Are these two concepts really the same?
- What conditions must hold before this action is allowed?
- What happens when the expected process does not occur?

For example, instead of asking a dispatcher what `CompleteWaybill()` should do, walk through an actual shipment:

- When is a waybill created?
- When is a vehicle assigned?
- When does loading begin and end?
- Who authorizes departure?
- Can a shipment be assigned but never dispatched?
- What happens if the driver changes after assignment?
- Can a dispatched shipment be returned or redirected?

These questions can expose distinctions that were invisible in the original implementation.

The goal is to understand the discovery before deciding how to represent it in code.

### 3. Refine the Ubiquitous Language

Clarify the terminology that describes the new understanding.

Identify ambiguous expressions, define important distinctions, and agree on language that domain experts and developers can use consistently.

For example, the team may discover that people use the word `Delivery` to describe three different things:

- A delivery order.
- The transportation of goods.
- The successful handover of goods to a recipient.

The team should clarify these meanings within the relevant Bounded Contexts instead of forcing one overloaded term into the model.

Update the model's vocabulary, examples, and tests accordingly.

### 4. Explore Alternative Models

Consider different ways of expressing the newly discovered concepts.

Evaluate each alternative against actual business scenarios.

For the shipment example, compare two possible designs.

**Model A: One generic status**

```text
Shipment
  Status
  VehicleId
  DriverId
  IsLoaded
  IsAuthorized
```

**Model B: Explicit business operations**

```text
Shipment
  AssignVehicle()
  CompleteLoading()
  AuthorizeDispatch()
  Dispatch()
  RecordDelivery()
```

Model A may be sufficient for a very simple application. But if each operation has its own rules, responsible people, and valid transitions, Model B may express the domain more accurately.

Evaluate each alternative:

- Does it represent the rules accurately?
- Does it make responsibilities clearer?
- Does it eliminate misleading abstractions?
- Does it make important business operations easier to understand?
- Does it introduce unnecessary complexity?

Do not assume that a more elaborate model is automatically a better model.

### 5. Refactor in Manageable Steps

A breakthrough can require substantial change, but implementation should remain controlled.

- Establish a reliable baseline of existing behavior.
- Identify the parts of the model affected by the discovery.
- Introduce the new concepts incrementally where possible.
- Keep the system working through intermediate changes.
- Update tests and integration contracts as the model evolves.
- Review the result with domain experts.

For example, if a legacy application stores shipment state in a single status column, the team does not necessarily need to replace the entire persistence model immediately.

It can:

1. Introduce explicit domain operations.
2. Map the existing status values to the new model.
3. Add tests for the newly discovered business transitions.
4. Change the application workflow to use the new operations.
5. Update persistence and integration contracts where necessary.
6. Migrate historical data if the new distinctions must be represented historically.

When working with legacy systems, techniques such as characterization tests, seams, and incremental replacement can help make otherwise risky changes manageable.

### 6. Validate the New Model

The refactoring is not complete simply because the code compiles or the tests pass.

Validate that the new model expresses the discovered business understanding.

For the shipment example, ask dispatchers whether the new distinctions accurately represent their daily work.

Can they explain why a shipment is assigned but not dispatched? Can they identify when dispatch authorization is required? Does the model represent exceptional situations such as vehicle replacement or failed loading?

Ask developers whether the new structure makes behavior and responsibilities easier to reason about.

The strongest evidence of improvement is that the model makes previously confusing business scenarios clearer.

## Example: A Library Domain

The logistics example demonstrates how a business process can be misunderstood. A different kind of breakthrough happens when the team discovers that a single concept actually represents several entities with different identities and lifecycles.

Consider a library system in which the initial model represents a book as a single Entity.

```csharp
public class Book
{
    public Guid Id { get; private set; }
    public string Title { get; private set; }
    public string Author { get; private set; }
    public bool IsBorrowed { get; private set; }
}
```

This model appears reasonable for a small library application.

However, as the team explores real scenarios, it discovers that:

- A library can own several physical copies of the same book.
- The same intellectual work can have multiple editions.
- Different editions may have different ISBNs.
- Each physical copy has its own barcode and condition.
- One copy can be borrowed while another remains available.
- A damaged copy may be unavailable even though it has not been borrowed.

The original model cannot express these distinctions correctly.

For example, if one copy is borrowed, setting `Book.IsBorrowed` to `true` incorrectly suggests that every copy of that book is unavailable.

The team discovers three distinct concepts:

- `Work` represents the intellectual creation.
- `Edition` represents a particular published version.
- `LibraryCopy` represents an individual physical item.

The revised model might look like:

```text
Work
  Id
  Title
  Authors

Edition
  Id
  WorkId
  ISBN
  Publisher
  PublicationDate

LibraryCopy
  Id
  EditionId
  Barcode
  Condition
  AvailabilityStatus
```

Lending operations now apply to a specific `LibraryCopy`, not to the abstract work itself.

The model can express meaningful rules:

- A borrowed copy cannot be borrowed again.
- A damaged copy cannot be checked out.
- Two physical copies of the same edition can have different availability states.
- Withdrawing one copy does not remove the intellectual work from the catalog.

This is more than a code cleanup. It is a conceptual improvement that makes the model reflect distinctions that matter to the business.

The important breakthrough was not introducing three classes. It was discovering that the business had been treating three different concepts as one.

## Benefits of Breakthrough Refactoring

- **Deeper Domain Understanding:** The team develops a more accurate understanding of business concepts, rules, and relationships.
- **Improved Model Expressiveness:** The software communicates business intent more clearly. For example, `AuthorizeDispatch()` communicates more than a generic status update.
- **Better Collaboration:** Domain experts and developers can discuss the business using more precise language and distinguish operations that were previously confused.
- **Reduced Accidental Complexity:** Misleading abstractions and workarounds can be replaced with a model that better represents the domain.
- **Greater Adaptability:** New requirements can fit naturally into a model with clearer responsibilities and boundaries.
- **Better Design Decisions:** The team makes architectural and implementation decisions based on improved domain knowledge.

These benefits are not automatic. A refactoring can introduce new complexity or simply replace one inadequate model with another. The new design must be validated against the domain.

## Common Mistakes

- Treating every refactoring as a breakthrough.
- Changing the model based on technical preferences without discovering new domain knowledge.
- Introducing more abstractions and assuming that complexity means sophistication.
- Refactoring without involving domain experts.
- Preserving outdated assumptions because existing tests encode them.
- Attempting a large rewrite without a safe migration strategy.
- Confusing a change in deployment architecture with a deeper understanding of the domain.
- Treating the first improved model as final rather than continuing to learn.
- Assuming that a more detailed model is always better, even when the business does not need those distinctions.

For example, splitting a simple `Customer` class into `CustomerProfile`, `CustomerIdentity`, `CustomerPreferences`, and `CustomerMetadata` purely to make the code look more sophisticated is not breakthrough refactoring. Unless the business has meaningful distinctions or rules that justify those concepts, the result is probably unnecessary complexity.

## Connection to Other DDD Concepts

| Concept | Its relationship to breakthrough refactoring |
|---|---|
| Ubiquitous Language | Expresses the evolving understanding and provides the vocabulary for the model. |
| Domain Exploration | Reveals new knowledge that may challenge the existing model. |
| Strategic Design | Helps reconsider responsibilities and boundaries when discoveries affect multiple models. |
| Tactical Design | Provides building blocks for expressing the improved model. |
| Refactoring | Provides techniques for changing the implementation safely. |
| Testing | Helps preserve valid behavior and verify the new model. |

## Further Reading

- **Domain-Driven Design: Tackling Complexity in the Heart of Software**, Eric Evans. The primary source for refactoring toward deeper insight and the evolution of domain models.
- **Refactoring: Improving the Design of Existing Code**, Martin Fowler. A practical catalog of techniques for restructuring code safely.
- **Working Effectively with Legacy Code**, Michael Feathers. Strategies for making difficult-to-change systems safer to modify.
- **Implementing Domain-Driven Design**, Vaughn Vernon. Practical guidance for implementing DDD concepts and evolving domain models.
- **Growing Object-Oriented Software, Guided by Tests**, Steve Freeman and Nat Pryce. A detailed treatment of using tests to guide design and development.
- **97 Things Every Programmer Should Know**, edited by Kevlin Henney. A collection of essays on software development, including ideas relevant to design and refactoring.

## Final Thought

A domain model is not a description of what the team understood when the project started. It is an evolving expression of what the team understands now.

Sometimes the most valuable improvement is not making the existing model cleaner. It is discovering that the model was describing the wrong thing.

The breakthrough is not the number of classes introduced, the number of files changed, or the architectural pattern adopted. It is the moment when a more accurate understanding of the business leads to a model that makes its complexity easier to reason about.

**Refactor the code to reflect what you have learned. And keep learning from the model you build.**