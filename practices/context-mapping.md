# Context Mapping

Context Mapping makes the relationships between Bounded Contexts explicit.

A Bounded Context defines the boundary within which a particular domain model and its Ubiquitous Language are consistent. But Bounded Contexts rarely exist in isolation. They depend on other contexts, exchange information, coordinate business processes, and sometimes share parts of their models.

These relationships introduce design decisions that are just as important as the design of each context itself.

Consider an e-commerce platform with separate Bounded Contexts for:

- **Sales:** Manages shopping carts, orders, pricing, and order confirmation.
- **Payment:** Manages payment authorization, capture, refunds, and payment methods.
- **Inventory:** Manages stock, reservations, and replenishment.
- **Shipping:** Manages shipment planning, carrier coordination, and delivery.
- **Customer Management:** Manages customer profiles, contact information, and account status.

These contexts are related, but they do not necessarily share the same model.

For example, Sales may use the concept of an `Order`, Inventory may work with `StockReservation`, Payment may deal with `PaymentAuthorization`, and Shipping may operate on a `Shipment`.

Trying to force all these concepts into one shared model would create unnecessary coupling. Each context should own the model that expresses its responsibilities.

Context Mapping helps answer questions such as:

- Which Bounded Context owns a particular business concept?
- Which context depends on another?
- Who controls the model or contract used for communication?
- How much influence does an upstream team have over a downstream team?
- Should models be shared, translated, or kept independent?
- How should changes in one context affect the others?
- Where should responsibility for integration and translation live?

A Context Map is the resulting picture of these relationships, together with the decisions and assumptions behind them.

## Why Context Mapping Matters

Identifying Bounded Contexts is only part of Strategic Design. The relationships between them can have a significant impact on the architecture, team organization, and ability to evolve the system.

Imagine that the Sales context needs to know whether an order can be fulfilled.

The Inventory context owns stock availability and reservation rules. Sales should not directly manipulate Inventory's internal data structures or reproduce its business rules.

Instead, the contexts need an explicit relationship.

For example:

```text
Sales Context
     |
     | Request stock reservation
     v
Inventory Context
     |
     | Reservation confirmed / rejected
     v
Sales Context
```

This diagram shows a dependency, but it does not explain enough to guide implementation.

Questions remain:

- Does Sales adapt to Inventory's model?
- Does Inventory provide an API designed for Sales?
- Do both contexts share a contract?
- Does Sales translate Inventory's response into its own language?
- Who is responsible when the contract changes?
- What happens if Inventory is unavailable?

Context Mapping makes these decisions explicit.

It is important to remember that a Context Map is not simply a diagram of applications, APIs, or databases. It describes relationships between domain models and the teams responsible for them.

## Core Elements of a Context Map

Before choosing a relationship pattern, identify the important elements of the relationship.

### 1. Bounded Contexts

Identify the contexts involved and the responsibility of each.

For example:

```text
Sales
  Owns:
    Shopping Cart
    Order
    Sales Price
    Order Confirmation

Inventory
  Owns:
    Stock Item
    Stock Reservation
    Available Quantity
    Replenishment

Shipping
  Owns:
    Shipment
    Carrier Assignment
    Dispatch
    Delivery
```

The same business word may have different meanings in different contexts.

For example, `Product` in Sales may represent a sellable catalog item with a description and price. In Inventory, a related concept may represent a stock-keeping unit with warehouse quantities and storage rules.

The contexts may refer to the same real-world product, but their models serve different purposes.

### 2. Ownership

Every model needs a clear owner.

Ownership means responsibility for defining, maintaining, and evolving the model. It includes business rules, terminology, data, and the contracts exposed to other contexts.

For example:

| Concept | Owning context | Reason |
|---|---|---|
| Order | Sales | Owns the customer's purchasing agreement and order lifecycle |
| Payment Authorization | Payment | Owns authorization and payment-provider interactions |
| Stock Reservation | Inventory | Owns the allocation of available stock |
| Shipment | Shipping | Owns transportation and delivery |
| Customer Profile | Customer Management | Owns customer contact and profile information |

Ownership does not mean that other contexts cannot maintain their own representations of the information.

Sales may store a customer's name and delivery address as part of an order snapshot. That does not make Sales the owner of the customer's profile.

The important distinction is between owning a concept and keeping a representation of information needed by another context.

### 3. Upstream and Downstream

An upstream context provides information, services, or capabilities that another context depends on.

A downstream context consumes or depends on what the upstream context provides.

For example:

```text
Inventory (Upstream)
       |
       | Stock Reservation Contract
       v
Sales (Downstream)
```

Inventory is upstream because Sales depends on its stock reservation capability.

This relationship does not automatically mean that Inventory is more important or has greater organizational authority. It describes the direction of dependency and influence.

The relationship becomes particularly important when the upstream context changes its model or contract.

For example:

- Inventory changes the meaning of `AvailableQuantity`.
- Sales depends on the old definition.
- The change may cause incorrect order confirmation unless the dependency is addressed.

Context Mapping helps make these dependencies visible before they become production incidents.

### 4. Integration Contracts

A relationship needs a way for contexts to communicate.

An integration contract describes what one context makes available to another and the expectations governing that interaction.

For example, Inventory might expose a reservation contract:

```json
{
  "reservationId": "R-1042",
  "orderId": "O-2048",
  "status": "Confirmed",
  "items": [
    {
      "sku": "LAPTOP-15",
      "quantity": 2
    }
  ]
}
```

The contract should make clear:

- What each field means.
- Which context owns the information.
- Which values are valid.
- What guarantees are provided.
- How errors are represented.
- How changes are versioned or kept compatible.
- Whether consumers can rely on immediate consistency or eventual updates.

A contract should express the information consumers need, not expose the upstream context's internal implementation.

For example, exposing Inventory's internal database tables directly would make downstream contexts depend on storage decisions that should remain private.

### 5. Translation and Model Compatibility

Two contexts may use different languages and structures for related business concepts.

For example:

```text
Sales Context:
  Order
  OrderLine
  RequestedQuantity

Inventory Context:
  StockReservation
  StockItem
  AllocatedQuantity
```

Sales and Inventory may both deal with quantities, but `RequestedQuantity` and `AllocatedQuantity` are not necessarily interchangeable.

The relationship needs an explicit translation or mapping strategy.

Without it, the models can gradually become coupled through assumptions that neither team has consciously agreed to share.

## Context Mapping Patterns

Context Mapping patterns describe common relationships between Bounded Contexts.

They are not technical integration technologies. They describe how teams and models relate to one another, including who controls the model, how much collaboration exists, and how dependencies are managed.

The patterns are not mutually exclusive. A relationship can combine several of them.

### 1. Shared Kernel

A Shared Kernel is a small part of the domain model that two or more Bounded Contexts deliberately share.

The participating teams agree to maintain and evolve that shared model together.

For example, consider a financial platform with separate contexts for:

- Account Management.
- Money Transfers.
- Billing.

All three may need to represent monetary amounts and currency.

The teams might agree to share a carefully defined `Money` Value Object:

```csharp
public sealed record Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (string.IsNullOrWhiteSpace(currency))
            throw new ArgumentException("Currency is required.");

        Amount = amount;
        Currency = currency;
    }
}
```

The shared type could establish consistent representation and validation rules.

However, the teams should not assume that every concept related to money belongs in the Shared Kernel.

For example:

- Billing may have `InvoiceAmount`.
- Transfers may have `TransferAmount`.
- Account Management may have `AvailableBalance`.

These concepts may share a monetary representation but have different business meanings and rules.

A Shared Kernel should remain deliberately small.

**When to use it**

- Two contexts have a genuine need to share a stable part of the model.
- Teams can collaborate on changes.
- The shared concepts have consistent meanings across those contexts.
- The cost of maintaining duplicated representations is greater than the cost of coordination.

**What to watch for**

A Shared Kernel introduces coordination costs.

Suppose the Billing team wants to change the shared `Money` representation to support additional precision, while the Transfers team is not ready to adopt the change.

Both teams must coordinate the migration.

A Shared Kernel that gradually expands into a large shared domain model can become a source of coupling and slow down independent evolution.

**Practical guidance**

- Keep the shared model small.
- Define exactly which concepts are shared.
- Agree on ownership and change procedures.
- Require tests for shared behavior.
- Avoid putting context-specific business rules into shared types.
- Review whether continued sharing is still worthwhile.

A shared code library alone does not automatically constitute a Shared Kernel. The shared model and the agreement to evolve it together are what matter.

### 2. Customer-Supplier

In a Customer-Supplier relationship, an upstream context provides capabilities or information to a downstream context and takes the downstream context's needs into account.

The downstream team is the customer. The upstream team is the supplier.

Consider an e-commerce platform:

```text
Inventory (Supplier)
       |
       | Reservation capability
       v
Sales (Customer)
```

Sales needs Inventory to support a business operation: reserving stock for an order.

The Sales team communicates its requirements to the Inventory team. The Inventory team considers those requirements when designing and evolving its API or integration contract.

For example, Sales may require an operation such as:

```text
ReserveStock
  OrderId
  Items
  ReservationExpiry
```

Inventory may then provide a response:

```text
StockReservationConfirmed
  ReservationId
  OrderId
  ExpiresAt
```

The two teams collaborate on the contract, but Inventory retains ownership of its internal model and rules.

**When to use it**

- One context provides capabilities needed by another.
- The downstream context has meaningful requirements that the upstream team can consider.
- Teams can communicate and coordinate changes.
- The upstream context is willing to support the needs of its consumers.

**What to watch for**

A Customer-Supplier relationship requires real collaboration.

If Inventory makes changes without considering Sales' needs, the relationship may become Conformist in practice.

If Sales has to repeatedly negotiate for changes that Inventory refuses to prioritize, the relationship may become a source of delivery friction.

**Practical guidance**

- Make downstream requirements visible.
- Agree on integration contracts.
- Establish communication and change coordination.
- Define expectations for compatibility and support.
- Discuss priorities when several consumers depend on the same upstream context.

Customer-Supplier is especially useful when the upstream team has enough autonomy to consider consumer requirements without losing ownership of its own model.

### 3. Conformist

In a Conformist relationship, the downstream context adopts the upstream context's model because it cannot influence the upstream design or because adapting independently would be too expensive.

Consider a company integrating with an external payment provider.

The provider exposes a payment model:

```text
PaymentIntent
  Id
  Amount
  Currency
  Status
  ClientSecret
```

The company's Payment context must work with this model.

The company cannot usually ask the provider to redesign its API to match the company's Ubiquitous Language.

The downstream context therefore conforms to the provider's contract.

This does not mean that the company's entire domain model should become a copy of the provider's model.

It means that the integration must understand and work with the upstream representation.

For example, the provider might use statuses such as:

```text
requires_payment_method
requires_confirmation
processing
succeeded
canceled
```

The company's own model may use:

```text
PaymentPending
PaymentAuthorized
PaymentCaptured
PaymentFailed
```

If the company directly adopts the provider's terminology and status transitions throughout its domain model, it becomes tightly coupled to that provider.

A better option may be to use Conformist behavior at the integration boundary and translate the provider's model into the company's own model.

**When to use it**

- The upstream provider controls the model and contract.
- There is little or no opportunity to influence upstream decisions.
- Adapting to the upstream representation is acceptable.
- The cost of maintaining a separate model is not justified.

**What to watch for**

Conformity can spread beyond the integration boundary.

For example, if every part of the application starts using the payment provider's status names, replacing the provider becomes much more difficult.

**Practical guidance**

- Understand the upstream model precisely.
- Keep external terminology from spreading unnecessarily.
- Document assumptions about provider behavior.
- Test mappings and exceptional responses.
- Consider an Anti-Corruption Layer if the external model threatens the integrity of the local model.

Conformist does not mean careless integration. It means accepting the upstream model as the basis of the relationship.

### 4. Anti-Corruption Layer

An Anti-Corruption Layer (ACL) protects a Bounded Context's model from the concepts, assumptions, and structures of another system.

It translates between the external model and the local model.

Consider a logistics company integrating with an old transportation management system.

The legacy system exposes a shipment representation:

```text
Waybill
  WaybillNo
  VehicleNo
  DriverCode
  StatusCode
  ExitDate
```

The new Shipping context has its own language:

```text
Shipment
  ShipmentId
  AssignedVehicle
  AssignedDriver
  DispatchStatus
  DispatchedAt
```

The legacy model may contain ambiguous status codes:

```text
StatusCode = 3
```

The new context should not allow this unexplained value to spread throughout its domain model.

Instead, an ACL translates the external representation:

```text
Legacy StatusCode 3
        |
        v
Legacy Waybill Translator
        |
        v
ShipmentDispatched
```

A simplified implementation might look like:

```csharp
public sealed class LegacyWaybillTranslator
{
    public ShipmentStatus TranslateStatus(int statusCode)
    {
        return statusCode switch
        {
            1 => ShipmentStatus.Created,
            2 => ShipmentStatus.Assigned,
            3 => ShipmentStatus.Dispatched,
            4 => ShipmentStatus.Delivered,
            _ => throw new UnsupportedLegacyStatusException(
                statusCode)
        };
    }
}
```

The translation should be based on verified business meaning, not assumptions inferred from the numeric values.

An ACL can translate more than field names. It may also:

- Convert external terminology into local terminology.
- Map external status codes to local states.
- Translate requests and responses.
- Interpret legacy error codes.
- Convert external identifiers.
- Handle differences in data structures.
- Protect local business rules from external assumptions.

**When to use it**

- Integrating with legacy systems.
- Consuming external APIs with incompatible models.
- Working with systems whose terminology conflicts with the local Ubiquitous Language.
- Protecting a new model from an upstream model that changes independently.
- Integrating with systems whose internal design cannot be controlled.

**What to watch for**

An ACL is not a place to hide every integration problem.

A translation layer that simply forwards every external object without meaningful interpretation does not provide much protection.

An overly complicated ACL can also become a second domain model, duplicating business rules that should belong elsewhere.

**Practical guidance**

- Define the local model first.
- Make translations explicit and testable.
- Keep external DTOs separate from local domain objects.
- Handle unknown values deliberately.
- Record assumptions about legacy behavior.
- Avoid leaking external terminology into local business logic.

A useful distinction: Conformist describes the relationship in which the downstream model adopts the upstream model. An ACL is a design technique for protecting a local model through translation. A system can use both: it may conform to an external API at the integration boundary while an ACL protects the internal domain model.

### 5. Separate Ways

Separate Ways means that two Bounded Contexts choose not to integrate because the cost of coordination or shared modeling is greater than the business value of doing so.

Not every related concept needs to be shared.

Consider an organization with separate Sales and Marketing contexts.

Both contexts maintain customer-related information.

Sales needs:

```text
Customer
  CustomerId
  CreditLimit
  PurchaseHistory
  AccountStatus
```

Marketing needs:

```text
AudienceMember
  ContactPreferences
  CampaignInteractions
  EngagementScore
  ConsentStatus
```

Although both contexts deal with people, they have different purposes and rules.

The teams may decide to maintain separate models rather than force a shared Customer model.

This can be a deliberate design decision, not a failure of integration.

Another example is an internal reporting application that needs a small set of historical customer attributes. It may be simpler to maintain its own read-optimized representation than to introduce a complex dependency on the Sales context.

**When to use it**

- The contexts have different responsibilities and language.
- Sharing a model would introduce unnecessary coupling.
- The business value of integration is low.
- Coordination between teams would cost more than maintaining independent models.
- Independent evolution is more valuable than consistency across the contexts.

**What to watch for**

Separate Ways does not mean ignoring important business requirements.

If Sales and Marketing must comply with the same customer consent decisions, maintaining completely independent consent information could introduce serious inconsistencies.

The team should identify which information genuinely needs coordination and which concepts can remain independent.

**Practical guidance**

- Document why integration is not worthwhile.
- Identify information that must remain consistent for business or regulatory reasons.
- Avoid accidental duplication of authoritative data.
- Revisit the decision when business requirements change.
- Do not confuse independent models with uncontrolled data copying.

Separate Ways is a legitimate strategic choice when the contexts do not need a stronger relationship.

### 6. Published Language

Published Language is a well-defined language or contract used by multiple Bounded Contexts to communicate.

It is particularly useful when several consumers need to exchange information through a stable and explicit representation.

Consider an e-commerce platform where the Order context publishes order-related events to several consumers:

```text
Order Context
      |
      | OrderConfirmed
      |
      +------> Inventory Context
      |
      +------> Payment Context
      |
      +------> Analytics Context
      |
      +------> Notification Context
```

Each consumer needs information about the confirmed order, but they should not depend on the internal Order Aggregate or its persistence model.

The Order context publishes a contract:

```json
{
  "eventId": "E-10021",
  "eventType": "OrderConfirmed",
  "version": 1,
  "occurredAt": "2026-11-13T10:30:00Z",
  "orderId": "O-2048",
  "customerId": "C-102",
  "currency": "EUR",
  "items": [
    {
      "productId": "P-501",
      "quantity": 2
    }
  ]
}
```

This representation is the Published Language.

It provides a stable way for consumers to understand the event without depending on the Order context's internal classes.

The contract should have clear semantics. For example, `OrderConfirmed` should mean that the order has passed the relevant confirmation rules, not merely that a database record has been created.

**When to use it**

- Several contexts consume information from the same upstream context.
- Consumers need a stable contract.
- Internal models differ across contexts.
- Integration must be independent of the upstream implementation.
- External partners or multiple teams need a common communication format.

**What to watch for**

A Published Language can become difficult to evolve if it exposes internal implementation details or changes without considering consumers.

A contract that contains every field from the upstream database is not necessarily a good Published Language.

**Practical guidance**

- Design contracts around consumer needs.
- Define terms and semantics, not only field types.
- Establish ownership and versioning policies.
- Consider backward compatibility.
- Test consumer expectations.
- Avoid exposing internal domain objects directly.

Published Language is a model and contract decision. JSON, XML, Protocol Buffers, Avro, and other formats are possible representations, not the pattern itself.

## Combining Context Mapping Patterns

Real systems rarely use only one pattern.

A relationship may combine patterns to address different aspects of the dependency.

### Example 1: Customer-Supplier + Published Language + Anti-Corruption Layer

Return to the e-commerce example.

Sales depends on Inventory to reserve stock.

The teams agree on the reservation requirements, so the relationship is Customer-Supplier.

Inventory publishes a stable reservation contract, so Published Language is also involved.

Sales translates the contract into its own model, so it uses an Anti-Corruption Layer.

```text
             Inventory Context
             (Upstream Supplier)
                     |
                     | Published Language
                     | StockReservationConfirmed
                     v
              Integration Contract
                     |
                     v
             Sales Anti-Corruption Layer
                     |
                     | Translation
                     v
                Sales Context
                (Downstream Customer)
```

These patterns are not competing choices.

- Customer-Supplier describes the collaboration and influence between teams.
- Published Language describes the shared communication model.
- Anti-Corruption Layer describes how Sales protects its own model from the external representation.

### Example 2: Conformist + Anti-Corruption Layer

A company integrates with an external payment provider.

The provider controls its API and the company cannot influence its design. The relationship is Conformist.

However, the company's Payment context wants to maintain its own language and business rules.

It therefore translates the provider's model through an ACL.

```text
External Payment Provider
       |
       | Provider API
       v
Provider Integration Adapter
       |
       v
Anti-Corruption Layer
       |
       v
Company Payment Context
```

The company conforms to the provider's contract at the boundary without allowing the provider's entire model to become its own.

### Example 3: Shared Kernel Across Internal Contexts

Two internal contexts share a small set of stable concepts.

```text
Account Management
       |
       | Shared Kernel
       |
Money Transfer
```

Both teams agree on the shared representation of `Money`, but each context retains its own business rules.

Account Management owns account status and account restrictions. Money Transfer owns transfer authorization and execution.

The Shared Kernel does not imply that both contexts share their entire domain model.

### Example 4: Separate Ways Between Related Contexts

Sales and Marketing both maintain information about customers.

```text
Sales Context          Marketing Context
     |                       |
  Customer              Audience Member
     |                       |
 Purchase Rules         Campaign Rules
     |                       |
     +---- No shared domain model ----+
```

They maintain independent models because their responsibilities differ.

If a common consent decision must be respected, the organization can establish a specific integration for that requirement without merging the entire customer model.

## Context Mapping Patterns Compared

| Pattern | Relationship | Who controls the model? | Main consideration |
|---|---|---|---|
| Shared Kernel | Teams share part of a model | Participating teams jointly | Coordination and shared evolution |
| Customer-Supplier | Upstream serves downstream needs | Upstream, with customer input | Collaboration and prioritization |
| Conformist | Downstream adopts upstream model | Upstream | Limited influence and dependency |
| Anti-Corruption Layer | Downstream translates external model | Local context owns its model | Translation and protection |
| Separate Ways | Contexts remain independent | Each context | Cost of integration versus value |
| Published Language | Contexts communicate through a defined contract | Contract owner, with consumer needs considered | Stability and compatibility |

The table is a quick reference, not a substitute for understanding the relationship. Some patterns describe team collaboration, others describe model sharing or translation. They can therefore be combined.

## Context Mapping Is Not the Same as Technical Integration

One of the most common mistakes is treating Context Mapping patterns as technical integration mechanisms.

They answer different questions.

| Technical mechanism | Context Mapping question |
|---|---|
| REST API | What model and contract does the provider expose, and who controls it? |
| Message broker | What business information is published, and what relationship exists between the contexts? |
| Shared database | Who owns the data and the rules governing it? |
| Shared library | Are the teams actually sharing a domain model and coordinating its evolution? |
| Microservices | What are the model boundaries and dependencies between the services? |

For example:

- REST does not automatically mean Customer-Supplier.
- Messaging does not automatically mean Published Language.
- Sharing a database does not automatically mean Shared Kernel.
- Using an ACL does not automatically mean Conformist.
- Deploying two microservices does not prove that they represent two independent Bounded Contexts.

A single relationship can be implemented using synchronous APIs, asynchronous messaging, files, or other mechanisms. The Context Mapping decision concerns the relationship between the models and their owners.

## How to Create a Context Map

A Context Map should emerge from understanding the contexts and their relationships, not from drawing boxes before understanding the domain.

A practical approach is:

### 1. Identify the Bounded Contexts

Start by identifying the contexts in scope.

For example:

```text
Sales
Inventory
Payment
Shipping
Customer Management
```

Clarify each context's responsibility and the model it owns.

### 2. Identify Dependencies

Ask which contexts need information or capabilities from others.

For example:

- Sales needs Inventory to reserve stock.
- Sales needs Payment to authorize payment.
- Shipping needs Sales to provide fulfillment information.
- Customer Management provides customer contact information to Sales.

Record the direction of each dependency.

### 3. Identify Model Ownership

For each important concept, identify the owning context.

For example:

| Concept | Owner |
|---|---|
| Order | Sales |
| Stock Reservation | Inventory |
| Payment Authorization | Payment |
| Shipment | Shipping |
| Customer Profile | Customer Management |

Also identify where other contexts maintain their own representations of that information.

### 4. Choose the Relationship Pattern

For each relationship, ask:

- Can the downstream team influence the upstream model?
- Is there a reason to share part of the model?
- Does the downstream context need to protect its own language?
- Would independent models be simpler?
- Do several consumers need a stable published contract?

Choose the pattern that reflects the actual relationship, not the pattern that sounds most sophisticated.

### 5. Define Integration Contracts

Specify what information crosses the boundary.

For example, a Sales-to-Shipping contract might communicate that an order is ready for fulfillment.

```text
OrderReadyForFulfillment
  OrderId
  DeliveryAddress
  Items
  RequestedDeliveryDate
```

Shipping should not need access to the internal Order Aggregate or Sales database.

### 6. Define Translation Responsibilities

Identify which context translates the information and where that translation belongs.

For example:

```text
External Carrier API
        |
        v
Carrier Integration Adapter
        |
        v
Shipping Model
```

The Shipping context should not allow carrier-specific terminology and status codes to spread throughout its domain model.

### 7. Record Risks and Assumptions

A useful Context Map records more than relationship names.

For each relationship, document:

- Upstream and downstream contexts.
- Model ownership.
- Chosen relationship pattern.
- Integration contract.
- Communication mechanism.
- Translation responsibilities.
- Change coordination expectations.
- Known risks and assumptions.
- Open questions.

For example:

```text
Relationship: Inventory -> Sales

Upstream: Inventory
Downstream: Sales

Pattern:
  Customer-Supplier
  Published Language
  Downstream ACL

Contract:
  StockReservationConfirmed
  StockReservationRejected

Ownership:
  Inventory owns reservation rules.
  Sales owns order confirmation rules.

Translation:
  Sales translates reservation responses
  into its own order fulfillment model.

Risk:
  Inventory reservation may expire before
  payment confirmation completes.

Open question:
  What happens when payment succeeds
  but the reservation has expired?
```

That final question may lead to further domain exploration and a more explicit business process.

## Common Context Mapping Mistakes

### Treating Context Maps as System Diagrams

A diagram showing applications and API connections does not necessarily represent a Context Map.

A Context Map should communicate model relationships, ownership, and dependencies.

### Assuming One Bounded Context Equals One Microservice

A Bounded Context is a model boundary, not automatically a deployment boundary.

A context may be implemented as a module within a modular monolith. Several contexts may share a deployment while maintaining distinct models.

### Sharing Too Much

A Shared Kernel should not become a common library containing every Entity, DTO, enum, and business rule.

The more concepts teams share, the more coordination they need.

### Ignoring Model Translation

Passing external DTOs directly into the domain model can introduce foreign concepts and assumptions.

Explicit translation helps preserve the local Ubiquitous Language.

### Choosing Patterns Without Understanding the Relationship

Selecting Customer-Supplier because one team calls another team's API does not make the relationship Customer-Supplier.

Understand the collaboration, influence, ownership, and model dependencies first.

### Treating Integration Technology as the Relationship Pattern

A message broker does not automatically imply Published Language. A REST API does not automatically imply Customer-Supplier.

Choose the relationship deliberately, then select an appropriate technical mechanism.

### Forgetting to Revisit the Map

Relationships change as teams, business capabilities, and systems evolve.

A relationship that began as Conformist may become Customer-Supplier when the organization gains influence over the upstream system. Two contexts that once shared a model may eventually need to separate.

A Context Map should evolve with the domain and organization.

## Practical Questions Before Choosing a Pattern

Use these questions when evaluating a relationship between two Bounded Contexts:

| Question | Why it matters |
|---|---|
| Which context owns the model? | Establishes responsibility and authority. |
| Which context depends on the other? | Reveals upstream and downstream direction. |
| Can the downstream team influence upstream decisions? | Helps distinguish Customer-Supplier from Conformist. |
| Do the contexts genuinely share the same concepts and rules? | Helps evaluate Shared Kernel. |
| Do they need to communicate through a stable contract? | Helps evaluate Published Language. |
| Does the external model conflict with the local language? | May justify an Anti-Corruption Layer. |
| Is integration worth its coordination cost? | Helps evaluate Separate Ways. |
| Who handles translation and contract changes? | Makes responsibilities explicit. |
| What happens when the upstream context is unavailable or changes? | Reveals operational and evolutionary risks. |

## Final Thought

Bounded Contexts define where models and their languages are consistent. Context Mapping explains how those models relate to one another.

The goal is not to eliminate every difference between contexts or force every team to share the same model. It is to make dependencies visible, establish ownership, and choose deliberate ways to manage relationships.

A good Context Map does more than show which systems communicate. It explains who owns the meaning of the information, who depends on it, how that meaning crosses boundaries, and what happens when either side changes.

**Model boundaries define where language is consistent. Context Maps make the relationships between those languages explicit.**