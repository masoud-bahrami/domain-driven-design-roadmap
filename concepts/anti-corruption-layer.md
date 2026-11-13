# Anti-Corruption Layer

An Anti-Corruption Layer (ACL) is an integration pattern used to protect the domain model of a Bounded Context from the concepts, structures, terminology, and assumptions of another system.

When two systems communicate, they do not necessarily share the same understanding of the business. They may use different names for the same concept, represent the same information differently, follow different business rules, or have entirely different models of how the world works.

The challenge is not simply making their data compatible. It is preventing the model of one system from gradually becoming part of the model of another.

Consider a modern order-management system integrating with a legacy inventory application. The legacy system may expose a concept called `StockRecord`, containing fields such as:

- `ItemNo`
- `WhCode`
- `QtyOnHand`
- `QtyAllocated`
- `QtyBlocked`
- `RecordStatus`

The modern system, however, may use concepts such as:

- `ProductId`
- `WarehouseId`
- `AvailableQuantity`
- `ReservedQuantity`
- `InventoryStatus`

These concepts may look similar, but their meanings are not necessarily identical.

For example, the legacy system might calculate available quantity as:

```text
Available Quantity = QtyOnHand - QtyAllocated - QtyBlocked
```

The modern system might define availability differently, taking pending reservations and damaged stock into account.

Directly using the legacy `StockRecord` throughout the modern domain would make the local model dependent on the legacy system's terminology and assumptions.

An Anti-Corruption Layer provides an explicit boundary between these models. It translates the external representation into concepts that make sense inside the local Bounded Context.

```text
┌──────────────────────────────┐
│       Legacy Inventory       │
│                              │
│  StockRecord                 │
│  ItemNo                      │
│  QtyOnHand                   │
│  QtyAllocated                │
│  QtyBlocked                  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│   Anti-Corruption Layer      │
│                              │
│  - Request translation       │
│  - Response translation      │
│  - Terminology mapping       │
│  - Error translation         │
│  - Protocol adaptation       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│   Modern Inventory Context   │
│                              │
│  InventoryItem               │
│  ProductId                   │
│  AvailableQuantity           │
│  ReservedQuantity            │
│  InventoryStatus             │
└──────────────────────────────┘
```

The local model does not need to know how the legacy system stores or calculates its information. It works with concepts that belong to its own domain.

The Anti-Corruption Layer is therefore not just a technical adapter. It is a boundary between different models of the business.

## 1. Why Anti-Corruption Layers Matter

Models influence how developers think about a domain. Once external terminology and assumptions enter a codebase, they tend to spread.

A legacy database schema may initially be used only by one integration component. A few months later, its table names appear in application services, its status codes become domain enums, and its unusual calculation rules are repeated across several modules.

Eventually, the new system is no longer expressing its own model. It is expressing a variation of the legacy model.

This is model corruption.

An ACL prevents this gradual leakage by ensuring that external concepts are translated before they enter the local domain.

There are several reasons to introduce one.

### 1.1 Protecting the Ubiquitous Language

Different systems may use different terminology for concepts that appear similar.

For example, a transportation platform may use:

| Legacy terminology | Local terminology |
|---|---|
| Waybill | Shipment |
| Vehicle Owner | Carrier |
| Driver Code | DriverId |
| Delivery Status | ShipmentStatus |
| Cargo Weight | DeclaredWeight |

These mappings are not always one-to-one. A legacy `Waybill` may represent a broader concept than a modern `Shipment`, including administrative and financial information.

The ACL translates only the information and meaning required by the local context.

The local domain should not need to call a shipment a `Waybill` simply because that is what an external system calls it.

This is especially important when multiple Bounded Contexts use different terminology for related concepts.

The same word can mean different things in different contexts. Conversely, different words can refer to similar concepts without making them interchangeable.

An ACL helps preserve those distinctions.

### 1.2 Isolating Legacy Technical Constraints

Legacy systems often expose technical decisions that should not become permanent design constraints in a new system.

Examples include:

- Fixed-width identifiers.
- Numeric status codes.
- Dates represented as strings.
- Boolean values represented as `0` and `1`.
- Monetary values stored as integers in an undocumented unit.
- SOAP operations with unusually broad request objects.
- Database procedures that return multiple result sets.
- Error codes that combine technical and business failures.
- Null values used to represent several different states.

The local domain should not have to understand these details.

For example, a legacy system may represent shipment status as:

```text
0 = Unknown
1 = Registered
2 = InProgress
3 = Delivered
4 = Cancelled
```

The local system might use a richer model:

```csharp
public enum ShipmentStatus
{
    Pending,
    Accepted,
    InTransit,
    Delivered,
    Cancelled,
    RequiresReview
}
```

The ACL translates the external status into the local representation.

```csharp
public static class LegacyShipmentStatusMapper
{
    public static ShipmentStatus Map(int status)
    {
        return status switch
        {
            1 => ShipmentStatus.Accepted,
            2 => ShipmentStatus.InTransit,
            3 => ShipmentStatus.Delivered,
            4 => ShipmentStatus.Cancelled,
            _ => ShipmentStatus.RequiresReview
        };
    }
}
```

The mapping should be based on actual business semantics, not merely matching numeric values to enum positions.

An unknown status should not silently become a valid local state. Depending on the business requirements, it may need to trigger an integration error, quarantine the message, or move the shipment into a state requiring review.

### 1.3 Preventing External Assumptions from Spreading

An external system may make assumptions that are valid within its own context but incorrect in yours.

For example:

- The legacy system considers a shipment complete when a driver records delivery.
- The modern system considers a shipment complete only after proof of delivery has been verified.
- A payment provider considers an authorization successful when funds are reserved.
- The local payment context distinguishes authorization from capture.
- A CRM considers a customer inactive after six months without interaction.
- The subscription context considers a customer active while a subscription remains valid.

These are differences in business meaning, not simply differences in field names.

The ACL should translate the external concepts into local meaning without pretending that the two models are identical.

### 1.4 Supporting Independent Evolution

An ACL reduces the impact of changes in external systems.

Suppose a third-party provider changes its API:

```json
{
  "customer_no": "10042",
  "full_name": "Jane Doe",
  "is_active": 1
}
```

Later, it changes the response:

```json
{
  "customer": {
    "id": "10042",
    "display_name": "Jane Doe",
    "status": "ACTIVE"
  }
}
```

If the provider's response structure is used throughout the application, this change may affect many components.

If the external structure is isolated behind an ACL, only the integration boundary needs to be updated, provided the business meaning remains compatible.

This does not eliminate integration maintenance. It limits the area in which external changes can propagate.

## 2. The Main Responsibilities of an Anti-Corruption Layer

An ACL can include several forms of translation and adaptation.

Its exact responsibilities depend on the systems being integrated, but the following are common.

### 2.1 Request Translation

The local application expresses a request in its own language. The ACL converts it into the format expected by the external system.

For example, the local system may request:

```csharp
public sealed record ReserveInventoryCommand(
    Guid ProductId,
    Guid WarehouseId,
    int Quantity);
```

The legacy system may expect:

```csharp
public sealed class LegacyReservationRequest
{
    public string ItemNo { get; set; } = "";
    public string WhCode { get; set; } = "";
    public int Qty { get; set; }
}
```

The ACL performs the translation:

```csharp
public sealed class LegacyInventoryAdapter
{
    private readonly ILegacyInventoryClient _client;
    private readonly IProductCodeResolver _productCodes;
    private readonly IWarehouseCodeResolver _warehouseCodes;

    public LegacyInventoryAdapter(
        ILegacyInventoryClient client,
        IProductCodeResolver productCodes,
        IWarehouseCodeResolver warehouseCodes)
    {
        _client = client;
        _productCodes = productCodes;
        _warehouseCodes = warehouseCodes;
    }

    public async Task Reserve(
        ReserveInventoryCommand command,
        CancellationToken cancellationToken)
    {
        var request = new LegacyReservationRequest
        {
            ItemNo = await _productCodes.Resolve(
                command.ProductId,
                cancellationToken),

            WhCode = await _warehouseCodes.Resolve(
                command.WarehouseId,
                cancellationToken),

            Qty = command.Quantity
        };

        await _client.Reserve(
            request,
            cancellationToken);
    }
}
```

The local application knows about product IDs, warehouse IDs, and inventory reservations. It does not need to know about legacy item numbers or warehouse codes.

### 2.2 Response Translation

The ACL translates external responses into local representations.

For example, a legacy inventory response might be:

```json
{
  "item_no": "PRD-001",
  "wh_code": "WH-01",
  "qty_on_hand": 100,
  "qty_allocated": 20,
  "qty_blocked": 5
}
```

The local context may expect:

```csharp
public sealed record InventoryAvailability(
    Guid ProductId,
    Guid WarehouseId,
    int AvailableQuantity);
```

The adapter can perform the mapping and calculation:

```csharp
public sealed class InventoryResponseMapper
{
    public InventoryAvailability Map(
        LegacyInventoryResponse response,
        Guid productId,
        Guid warehouseId)
    {
        var availableQuantity =
            response.QuantityOnHand
            - response.QuantityAllocated
            - response.QuantityBlocked;

        return new InventoryAvailability(
            productId,
            warehouseId,
            availableQuantity);
    }
}
```

The calculation belongs in the ACL only if it correctly translates the legacy system's definition into the local concept.

If the local context has a different definition of availability, the ACL must not blindly reuse the legacy calculation. It should translate the raw information into local inputs, leaving the local domain to apply its own rules.

This distinction is important: translation should preserve meaning, not just produce a convenient value.

### 2.3 Terminology and Semantic Translation

Sometimes the most important translation is not structural.

Consider a legacy insurance system that exposes:

```text
PolicyState = 5
```

Its documentation says that this means the policy is "closed".

But in the local insurance context, a policy can be:

- Active
- Expired
- Cancelled
- Suspended
- Settled
- Closed

The external concept of `Closed` may represent several different local states.

The ACL needs enough information to determine the correct local meaning. If the legacy system does not expose that information, the mapping may require additional queries or an explicit `Unknown`/`RequiresReview` state.

Do not hide semantic uncertainty behind a mapper that always returns a plausible-looking value.

A translation that produces the wrong business meaning is worse than an integration failure that is visible.

### 2.4 Error Translation

External systems have their own error models.

A payment provider may return:

```text
101 = Invalid Card
102 = Insufficient Funds
103 = Expired Card
500 = Internal Error
```

The local payment context may use:

```csharp
public enum PaymentFailureReason
{
    InvalidPaymentMethod,
    InsufficientFunds,
    PaymentMethodExpired,
    ProviderUnavailable,
    Unknown
}
```

The ACL translates external errors into local concepts:

```csharp
public static class PaymentErrorMapper
{
    public static PaymentFailureReason Map(int code)
    {
        return code switch
        {
            101 => PaymentFailureReason.InvalidPaymentMethod,
            102 => PaymentFailureReason.InsufficientFunds,
            103 => PaymentFailureReason.PaymentMethodExpired,
            500 => PaymentFailureReason.ProviderUnavailable,
            _ => PaymentFailureReason.Unknown
        };
    }
}
```

This allows the local application to handle payment failures without depending on provider-specific codes.

However, not every external error should become a business failure.

Consider the following distinctions:

| External result | Local interpretation |
|---|---|
| Invalid card | Payment method rejected |
| Insufficient funds | Payment declined |
| Request timeout | Outcome may be unknown |
| HTTP 503 | Provider temporarily unavailable |
| Invalid API credentials | Integration configuration failure |
| Unknown provider code | Unrecognized integration response |

A timeout is particularly important.

If a payment request times out, the provider may have processed the payment even though the local system never received the response.

The ACL should not automatically translate a timeout into `PaymentFailed`. The correct local state may be `PaymentOutcomeUnknown`, followed by a status inquiry or reconciliation process.

An ACL must preserve important uncertainty instead of converting every technical problem into a definitive business outcome.

### 2.5 Protocol and Communication Adaptation

An ACL may also adapt communication protocols.

For example:

- A local REST API communicates with a legacy SOAP service.
- A modern event-driven application consumes messages from a legacy message queue.
- A local application uses HTTP while an external system requires a proprietary TCP protocol.
- A third-party service exposes polling APIs while the local application uses events.

The ACL can hide these differences behind a local interface.

```csharp
public interface IInventoryGateway
{
    Task<InventoryAvailability> GetAvailability(
        Guid productId,
        Guid warehouseId,
        CancellationToken cancellationToken);

    Task Reserve(
        ReserveInventoryCommand command,
        CancellationToken cancellationToken);
}
```

The implementation may use SOAP, REST, a database procedure, or a message broker.

The local application depends on the business-facing interface rather than the external protocol.

## 3. An Anti-Corruption Layer in Practice

Consider a modern logistics platform integrating with an old transportation-management system.

The legacy system represents a shipment using a structure like this:

```csharp
public sealed class LegacyWaybillDto
{
    public string WaybillNo { get; set; } = "";
    public string VehicleNo { get; set; } = "";
    public string DriverNo { get; set; } = "";
    public int StatusCode { get; set; }
    public decimal CargoWeight { get; set; }
    public string RegistrationDate { get; set; } = "";
}
```

The modern logistics context uses:

```csharp
public sealed class Shipment
{
    public ShipmentId Id { get; private set; }
    public VehicleId VehicleId { get; private set; }
    public DriverId DriverId { get; private set; }
    public ShipmentStatus Status { get; private set; }
    public Weight CargoWeight { get; private set; }
    public DateTimeOffset RegisteredAt { get; private set; }
}
```

The local model has stronger types and explicit domain concepts.

An ACL can translate the legacy response:

```csharp
public sealed class LegacyShipmentMapper
{
    private readonly IVehicleResolver _vehicles;
    private readonly IDriverResolver _drivers;

    public LegacyShipmentMapper(
        IVehicleResolver vehicles,
        IDriverResolver drivers)
    {
        _vehicles = vehicles;
        _drivers = drivers;
    }

    public async Task<ShipmentImportData> Map(
        LegacyWaybillDto source,
        CancellationToken cancellationToken)
    {
        var vehicleId = await _vehicles.Resolve(
            source.VehicleNo,
            cancellationToken);

        var driverId = await _drivers.Resolve(
            source.DriverNo,
            cancellationToken);

        if (!DateTimeOffset.TryParse(
                source.RegistrationDate,
                out var registeredAt))
        {
            throw new LegacyIntegrationException(
                $"Invalid registration date for waybill {source.WaybillNo}.");
        }

        return new ShipmentImportData(
            new ShipmentId(source.WaybillNo),
            vehicleId,
            driverId,
            MapStatus(source.StatusCode),
            Weight.FromKilograms(source.CargoWeight),
            registeredAt);
    }

    private static ShipmentStatus MapStatus(int code)
    {
        return code switch
        {
            1 => ShipmentStatus.Registered,
            2 => ShipmentStatus.InTransit,
            3 => ShipmentStatus.Delivered,
            4 => ShipmentStatus.Cancelled,
            _ => throw new LegacyIntegrationException(
                $"Unsupported shipment status: {code}")
        };
    }
}
```

The example illustrates several responsibilities:

- Translating legacy identifiers into local identities.
- Converting external status codes into local domain concepts.
- Parsing external date representations.
- Converting numeric values into domain-specific types.
- Handling unsupported or invalid external data explicitly.

In a production implementation, date parsing should use the documented external format and culture rather than relying on the machine's default settings. The mapper should also distinguish invalid data from temporarily unavailable reference data.

The local domain can then operate on `ShipmentImportData` without depending on `LegacyWaybillDto`.

## 4. Anti-Corruption Layer versus Other Integration Patterns

An ACL is often confused with several other patterns. They can be used together, but they solve different problems.

### 4.1 Anti-Corruption Layer versus Adapter

An Adapter makes one interface compatible with another.

For example, it may allow a local application to call a service that exposes a different technical interface.

An ACL goes further: it protects the local domain model from the external model's semantics and assumptions.

An Adapter can be part of an ACL, but not every Adapter is an ACL.

Consider a wrapper that exposes a third-party payment API without changing any of its concepts:

```csharp
public Task<ProviderPaymentResponse> Charge(
    ProviderPaymentRequest request);
```

This is primarily an adapter.

A more complete ACL might expose:

```csharp
public Task<PaymentAuthorizationResult> Authorize(
    AuthorizePayment command);
```

It translates local payment intent into the provider's API and translates provider outcomes into local payment concepts.

The difference is not the number of classes involved. It is the degree of protection provided to the local model.

### 4.2 Anti-Corruption Layer versus Open Host Service

An Open Host Service is an integration pattern in which a Bounded Context provides a well-defined service interface for other contexts to consume.

It is about how a context exposes its capabilities to other systems.

An ACL is about how a context protects itself when consuming another system's model.

They can complement each other:

```text
Bounded Context A
      |
      | Open Host Service
      v
Integration Contract
      |
      v
Anti-Corruption Layer
      |
      v
Bounded Context B
```

The first context exposes a stable contract. The second translates that contract into its own model.

### 4.3 Anti-Corruption Layer versus Conformist

A Conformist relationship exists when a downstream context adopts the upstream model because it has little influence over the upstream design or because the cost of translation is not justified.

In this relationship, the downstream context accepts the upstream model rather than protecting itself with an independent model.

For example, a reporting application may deliberately use the same terminology and data structures as an existing enterprise system.

This can be a reasonable choice when:

- The upstream model is sufficiently suitable.
- The downstream application has limited domain behavior.
- The cost of maintaining a separate model exceeds the expected benefit.
- The integration is intentionally read-only or tightly coupled.

Not every integration requires an ACL.

An ACL adds translation code, testing requirements, and operational complexity. It should be introduced when protecting the local model provides meaningful value.

### 4.4 Anti-Corruption Layer versus Shared Kernel

A Shared Kernel is a small part of the domain model that two or more Bounded Contexts intentionally share and maintain together.

An ACL takes the opposite approach: it keeps models separate and translates between them.

A Shared Kernel can reduce duplication when the shared concepts are genuinely stable and jointly owned. But sharing too much model creates coupling between contexts.

Use a Shared Kernel when shared concepts and governance are intentional. Use an ACL when the models need to remain independent.

## 5. Where Should the Anti-Corruption Layer Live?

The ACL belongs at the boundary of the Bounded Context that needs protection.

It should translate external representations into local concepts before those representations reach the local domain.

A common implementation structure might look like this:

```text
src/
├── Logistics.Domain/
│   ├── Shipments/
│   ├── Vehicles/
│   ├── Drivers/
│   └── ValueObjects/
│
├── Logistics.Application/
│   ├── Shipments/
│   └── Abstractions/
│
├── Logistics.Infrastructure/
│   ├── Persistence/
│   └── ExternalIntegrations/
│       └── LegacyTransportation/
│           ├── LegacyTransportationClient.cs
│           ├── LegacyShipmentMapper.cs
│           ├── LegacyRequestMapper.cs
│           ├── LegacyErrorMapper.cs
│           └── LegacyTransportModels.cs
│
└── Logistics.Api/
```

The domain should not reference the legacy transport models.

The application layer may define an interface representing the capability it needs:

```csharp
public interface IShipmentInformationProvider
{
    Task<ShipmentInformation?> Find(
        ShipmentId shipmentId,
        CancellationToken cancellationToken);
}
```

The infrastructure layer implements that interface using the ACL:

```csharp
public sealed class LegacyShipmentInformationProvider
    : IShipmentInformationProvider
{
    private readonly ILegacyTransportationClient _client;
    private readonly LegacyShipmentMapper _mapper;

    public LegacyShipmentInformationProvider(
        ILegacyTransportationClient client,
        LegacyShipmentMapper mapper)
    {
        _client = client;
        _mapper = mapper;
    }

    public async Task<ShipmentInformation?> Find(
        ShipmentId shipmentId,
        CancellationToken cancellationToken)
    {
        var response = await _client.GetWaybill(
            shipmentId.Value,
            cancellationToken);

        if (response is null)
            return null;

        return _mapper.Map(response);
    }
}
```

The application layer depends on the local interface. The infrastructure implementation handles the external protocol and model.

The precise project structure depends on the architecture. An ACL does not require a separate project, microservice, or deployment unit. It requires an explicit and well-defined translation boundary.

## 6. Anti-Corruption Layers in Event-Driven Systems

The same principles apply when systems communicate through messages rather than direct API calls.

Suppose a legacy system publishes:

```json
{
  "event_type": "WB_STATUS_CHANGED",
  "waybill_no": "WB-10042",
  "status": 3,
  "event_date": "2026/11/13 14:30:00"
}
```

The local context uses an event such as:

```csharp
public sealed record ShipmentDelivered(
    ShipmentId ShipmentId,
    DateTimeOffset DeliveredAt);
```

A message consumer should not pass the legacy message directly into the domain.

Instead, it translates the message:

```csharp
public sealed class LegacyWaybillConsumer
{
    private readonly IShipmentApplicationService _shipments;

    public LegacyWaybillConsumer(
        IShipmentApplicationService shipments)
    {
        _shipments = shipments;
    }

    public async Task Handle(
        LegacyWaybillMessage message,
        CancellationToken cancellationToken)
    {
        var shipmentEvent = Map(message);

        await _shipments.Handle(
            shipmentEvent,
            cancellationToken);
    }

    private static ShipmentDelivered Map(
        LegacyWaybillMessage message)
    {
        if (message.Status != 3)
            throw new UnsupportedLegacyMessageException(
                message.EventType);

        if (!DateTimeOffset.TryParseExact(
                message.EventDate,
                "yyyy/MM/dd HH:mm:ss",
                CultureInfo.InvariantCulture,
                DateTimeStyles.AssumeUniversal,
                out var deliveredAt))
        {
            throw new InvalidLegacyMessageException(
                "Invalid delivery date.");
        }

        return new ShipmentDelivered(
            new ShipmentId(message.WaybillNo),
            deliveredAt);
    }
}
```

In production, the consumer should also consider:

- Message versioning.
- Duplicate message delivery.
- Idempotency.
- Out-of-order events.
- Unknown event types.
- Invalid payloads.
- Retry and dead-letter policies.
- Time-zone semantics.
- Schema evolution.

An ACL protects the domain model, but it does not automatically solve messaging reliability.

For example, if the legacy system publishes the same delivery message three times, the local context must still avoid applying the business operation three times.

That is an idempotency concern, not a mapping concern.

Likewise, if a delivery event arrives before a shipment-registration event, the application needs an explicit strategy for handling out-of-order messages.

Keep these responsibilities distinct, even when they are implemented in the same integration component.

## 7. Anti-Corruption Layers and Legacy Modernization

ACLs are particularly useful when replacing legacy systems incrementally.

A complete rewrite is often too risky or expensive. Instead, organizations introduce new Bounded Contexts around selected business capabilities while existing applications continue to operate.

During this transition, new components may need to communicate with the legacy system.

An ACL creates a boundary that allows the new model to evolve independently.

For example:

```text
                  ┌──────────────────────┐
                  │    Legacy System     │
                  │                      │
                  │  Existing Database  │
                  │  Existing Rules      │
                  │  Existing API        │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ Anti-Corruption Layer│
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │    New Context       │
                  │                      │
                  │  New Domain Model    │
                  │  New Business Rules  │
                  │  New API             │
                  └──────────────────────┘
```

This approach can be combined with the Strangler Fig pattern.

Over time, selected capabilities are implemented in the new system. Traffic is redirected to the new implementation while the legacy system continues to support capabilities that have not yet been replaced.

The ACL can initially translate between the old and new models. As migration progresses, some translations may become unnecessary because the legacy capability is retired.

### Example: Replacing a legacy customer system

Suppose a legacy CRM owns customer information. A new subscription context needs customer identity and eligibility information.

The new context should not necessarily adopt the CRM's entire customer model.

It might need only:

```csharp
public sealed record SubscriberProfile(
    SubscriberId Id,
    CustomerName Name,
    bool IsEligible);
```

The CRM may expose:

```csharp
public sealed class LegacyCustomerResponse
{
    public string CustomerNo { get; set; } = "";
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public int CustomerType { get; set; }
    public int CustomerStatus { get; set; }
    public string CreditFlag { get; set; } = "";
    public string RegistrationChannel { get; set; } = "";
}
```

The ACL translates the subset of information that the subscription context needs.

It should not import every CRM field simply because it is available.

This keeps the local model focused on its own responsibilities.

## 8. Common Anti-Corruption Layer Mistakes

An ACL is useful, but it can become another source of complexity if it is poorly designed.

### 8.1 Renaming Fields Without Translating Meaning

Changing `WaybillNo` to `ShipmentId` does not automatically create an ACL.

If the local domain still follows all the legacy system's assumptions, the translation is only cosmetic.

A real ACL translates business meaning, not just property names.

### 8.2 Leaking External DTOs into the Domain

For example:

```csharp
public class Shipment
{
    public LegacyWaybillDto Source { get; set; }
}
```

This directly couples the local domain to the external representation.

External DTOs should remain at the integration boundary.

### 8.3 Creating a Pass-Through Layer

An adapter that simply forwards every external method and response without translating the model may add little value.

For example:

```csharp
public Task<LegacyCustomerDto> GetCustomer(
    string customerNo)
{
    return _legacyClient.GetCustomer(customerNo);
}
```

If every caller still understands `LegacyCustomerDto`, the boundary has not protected the local model.

A pass-through adapter can still be useful when the external contract is intentionally adopted, but it should not be mistaken for a meaningful ACL.

### 8.4 Translating Everything into Generic Objects

Avoid replacing external DTOs with generic structures such as:

```csharp
Dictionary<string, object>
```

or:

```csharp
object response;
```

The goal is not to hide the external model behind an untyped interface. The goal is to create an explicit local model.

Use well-defined types that express the information and meaning the local context needs.

### 8.5 Silently Ignoring Unknown Values

Suppose a legacy status mapper does this:

```csharp
return status switch
{
    1 => ShipmentStatus.Registered,
    2 => ShipmentStatus.InTransit,
    _ => ShipmentStatus.Unknown
};
```

This may be valid if `Unknown` is an intentional local state with defined behavior.

But if the system treats every unrecognized status as a normal state, a breaking change in the legacy system can go unnoticed.

Unknown values should have an explicit handling strategy:

- Reject and report the message.
- Quarantine the record for investigation.
- Map to a meaningful local fallback state.
- Accept the value only if the business explicitly permits it.

Do not silently turn unexpected external behavior into apparently valid domain data.

### 8.6 Putting Business Rules in the Wrong Place

An ACL translates between models. It should not become a second domain model containing all the business behavior of the local context.

For example, an adapter may translate a provider's payment status into `PaymentAuthorizationStatus`.

But the rule that an order cannot be fulfilled before payment authorization belongs to the order or fulfilment domain, not to the provider mapper.

Keep translation responsibilities separate from local business decisions.

### 8.7 Making the ACL Too Large

An ACL can become a large integration module with hundreds of mappings, protocol handlers, retry policies, business workflows, and unrelated responsibilities.

Organize it around external systems and the capabilities they provide.

Separate request mapping, response mapping, error mapping, protocol communication, and workflow coordination when doing so improves clarity.

Do not create a class for every trivial mapping merely to increase the number of abstractions.

The objective is a clear boundary, not an elaborate framework.

## 9. Testing an Anti-Corruption Layer

An ACL deserves explicit tests because it is responsible for preserving meaning across system boundaries.

A mapping bug can be more dangerous than an ordinary integration failure. The system may continue operating while making incorrect business decisions.

Consider a legacy inventory response:

```json
{
  "qty_on_hand": 100,
  "qty_allocated": 20,
  "qty_blocked": 5
}
```

A test should verify the expected local interpretation:

```csharp
[Fact]
public void Should_Map_Legacy_Inventory_To_Availability()
{
    var response = new LegacyInventoryResponse
    {
        QuantityOnHand = 100,
        QuantityAllocated = 20,
        QuantityBlocked = 5
    };

    var result = _mapper.Map(
        response,
        ProductId,
        WarehouseId);

    Assert.Equal(75, result.AvailableQuantity);
}
```

Other important test categories include:

### 9.1 Mapping Tests

Verify that:

- Identifiers are translated correctly.
- Status codes map to the correct local states.
- Dates and time zones are interpreted correctly.
- Monetary values preserve their units and precision.
- Optional and missing fields are handled correctly.
- External values do not leak into local domain types.

### 9.2 Error Translation Tests

Verify that:

- Known external errors map to the correct local failures.
- Timeouts are not incorrectly interpreted as definitive business failures.
- Authentication and configuration failures remain distinguishable.
- Unknown error codes are handled explicitly.

### 9.3 Contract Tests

Verify that the integration behaves as expected against representative external responses.

This can involve:

- Captured API responses.
- Provider sandbox environments.
- Consumer-driven contract tests.
- Versioned message fixtures.
- Recorded legacy database outputs.

A contract test helps detect when an external system changes its contract in a way that breaks the ACL.

### 9.4 Integration Tests

Test the complete translation path:

```text
Local Command
     |
     v
Request Mapper
     |
     v
External Client
     |
     v
Legacy Response
     |
     v
Response Mapper
     |
     v
Local Result
```

Use representative scenarios, including malformed responses and unexpected external values.

### 9.5 Characterization Tests for Legacy Systems

Legacy behavior is not always documented correctly.

Before replacing or wrapping a legacy operation, capture what it actually does for representative inputs.

Characterization tests help document existing behavior, including behavior that may be surprising or inconsistent.

This is particularly valuable when the legacy implementation cannot be modified or when its business rules are buried in stored procedures and undocumented services.

Do not assume that an existing behavior is correct simply because it has existed for years. First establish what it does, then decide whether the ACL should preserve, normalize, or explicitly reject that behavior.

## 10. Practical Exercises

### Exercise 1: Integrate a Legacy Inventory System

Imagine a legacy inventory system exposes this response:

```json
{
  "item_no": "P-1001",
  "warehouse_code": "W-01",
  "quantity": 100,
  "allocated": 20,
  "blocked": 5,
  "status": 1
}
```

Your local Inventory Context uses:

- ProductId
- WarehouseId
- AvailableQuantity
- InventoryStatus

Tasks:

1. Define the local model.
2. Define the external DTO.
3. Create request and response mappers.
4. Translate the legacy status into a local concept.
5. Decide how to handle unknown status codes.
6. Write mapping and integration tests.

Consider whether `quantity - allocated - blocked` is actually the correct definition of availability in your local context.

### Exercise 2: Integrate a Payment Provider

A payment provider exposes:

```json
{
  "transaction_id": "TX-12345",
  "result_code": 102,
  "amount": 500000,
  "currency": "IRR"
}
```

Your Payment Context uses:

- PaymentId
- AuthorizationStatus
- Money
- PaymentFailureReason

Tasks:

1. Design the provider-facing DTO.
2. Define local payment concepts.
3. Translate success and failure responses.
4. Handle unknown result codes.
5. Model timeout and uncertain payment outcomes.
6. Write tests for successful, rejected, and uncertain payments.

Pay particular attention to the difference between a rejected payment and a payment whose outcome is unknown.

### Exercise 3: Consume Legacy Events

A legacy transportation system publishes messages containing:

- Waybill number
- Vehicle number
- Driver number
- Numeric status
- Registration date

Your Logistics Context uses:

- ShipmentId
- VehicleId
- DriverId
- ShipmentStatus
- RegisteredAt

Tasks:

1. Define the external message contract.
2. Define the local event or command.
3. Implement the translation.
4. Handle unknown event types and status codes.
5. Consider duplicate and out-of-order messages.
6. Write tests using captured message examples.

### Exercise 4: Protect a New Bounded Context from an Existing CRM

An existing CRM exposes a large Customer model containing personal details, marketing preferences, credit information, support history, and registration metadata.

Your new Subscription Context needs only customer identity, eligibility, and subscription-related information.

Tasks:

1. Identify the concepts needed by the Subscription Context.
2. Define its local model.
3. Design an ACL that translates the CRM response.
4. Avoid importing irrelevant CRM fields.
5. Decide how changes in customer eligibility are communicated.
6. Explain what happens if the CRM becomes unavailable.

### Exercise 5: Evaluate an Existing Integration

Find an integration in an existing application.

Ask:

- Are external DTOs used inside the domain?
- Does the local code use external terminology?
- Are external status codes repeated across multiple components?
- Are provider-specific errors exposed to application services?
- Are assumptions about external data duplicated?
- Are unknown values silently ignored?
- Can the external system change its API without affecting the local domain?

Document the existing boundary and propose an improved design.

## 11. Completion Criteria

You should understand Anti-Corruption Layers when you can:

- Explain why different Bounded Contexts may have different models of the same business concepts.
- Distinguish semantic translation from simple field mapping.
- Identify when an ACL provides meaningful value.
- Translate external requests, responses, events, and errors.
- Protect the local Ubiquitous Language from foreign terminology.
- Keep external DTOs and protocol details outside the local domain.
- Handle unknown values and uncertain outcomes explicitly.
- Distinguish ACL responsibilities from business rules and workflow coordination.
- Explain the difference between an ACL, Adapter, Open Host Service, Conformist, and Shared Kernel.
- Test translations against representative external behavior.
- Use ACLs as part of incremental legacy modernization.

An Anti-Corruption Layer does not make two systems share the same model. It allows them to communicate without requiring them to become the same model.

**Integration is not the absence of boundaries. Good integration is the ability to cross boundaries without losing meaning.**