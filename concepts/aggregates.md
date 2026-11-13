# Aggregates

An Aggregate is a cluster of domain objects that are treated as a single unit for enforcing business invariants. It defines a consistency boundary within which domain rules must be preserved when a business operation is completed.

An Aggregate has one designated entry point called the **Aggregate Root**. External objects interact with the Aggregate through this root rather than modifying its internal objects directly. The root is responsible for protecting the invariants of the entire Aggregate, not merely its own state.

The purpose of an Aggregate is not to represent an object graph, mirror a database schema, or collect everything that belongs to the same business concept. Its purpose is to establish a boundary around rules that must remain consistent together.

Consider an order in an e-commerce system. An order may contain several order lines, each with its own quantity and price. The order also has a status and a total amount. Some rules apply across these objects:

- An order cannot be confirmed if it has no items.
- An order line must have a positive quantity.
- The order total must reflect the prices and quantities of its lines.
- A cancelled order cannot be confirmed.
- Items cannot be added after an order has been finalized.

These rules involve multiple objects, but they must be protected as part of a coherent business operation. The `Order` is therefore a natural candidate for an Aggregate Root, while `OrderLine` belongs inside the Aggregate.

The important question is not whether these objects are related. It is whether the business requires their changes to be consistent together.

## 1. The Purpose of an Aggregate: Protecting Invariants

An invariant is a business rule that must remain true whenever an operation completes successfully.

Aggregates provide a mechanism for enforcing invariants at the point where state changes. They help ensure that the system cannot enter a state that violates the business rules simply because an application service, API endpoint, background job, or another developer forgot to perform a validation.

For example, consider a bank account:

- Its balance cannot fall below the permitted overdraft limit.
- A withdrawal cannot exceed the available funds.
- A closed account cannot accept withdrawals.
- A transaction must have a positive amount.

These are not merely validation rules belonging to a user interface. They are domain invariants that must hold regardless of where the operation originates.

A poorly designed implementation might allow any application service to modify the balance:

```csharp
account.Balance -= amount;
```

This makes it possible for different parts of the application to implement withdrawals differently. One may check the available balance, another may forget, and a third may allow withdrawals from closed accounts.

Instead, the Aggregate Root exposes meaningful business operations:

```csharp
public sealed class BankAccount
{
    private readonly List<AccountTransaction> _transactions = [];

    public Guid Id { get; private set; }
    public decimal Balance { get; private set; }
    public AccountStatus Status { get; private set; }

    public IReadOnlyCollection<AccountTransaction> Transactions
        => _transactions.AsReadOnly();

    public void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new DomainException(
                "Withdrawal amount must be positive.");

        if (Status != AccountStatus.Active)
            throw new DomainException(
                "Only active accounts can be used for withdrawals.");

        if (Balance < amount)
            throw new DomainException(
                "Insufficient funds.");

        Balance -= amount;

        _transactions.Add(
            AccountTransaction.Withdrawal(amount));
    }
}
```

The caller does not decide how to update the balance or record the withdrawal. It asks the Aggregate to perform a business operation.

```csharp
account.Withdraw(100);
```

The Aggregate protects the relationship between the account balance and its transactions.

This is the central responsibility of an Aggregate: **make invalid state transitions difficult or impossible to express through its public interface.**

### Invariants versus ordinary validation

Not every validation rule needs to be enforced inside an Aggregate.

Consider the following cases:

| Rule | Likely responsibility |
|---|---|
| Withdrawal amount must be positive | Bank Account Aggregate |
| Closed accounts cannot be used | Bank Account Aggregate |
| Customer email must have a valid format | Value Object or input validation |
| A user must be authenticated | Application/security boundary |
| A customer must have sufficient credit to place an order | Depends on the consistency requirement and domain model |
| A product must be visible in the current search results | Read model or application-level concern |

The distinction is important. Aggregates should protect domain invariants, not become containers for every validation rule in the system.

Some rules can be enforced locally within an Aggregate. Others depend on information owned by another Aggregate, an external system, or a business process spanning multiple transactions. Those rules require different modeling strategies.

## 2. The Aggregate Root

The Aggregate Root is the only object through which external code can access and modify the Aggregate's internal state.

It has three primary responsibilities:

1. Provide a controlled interface for interacting with the Aggregate.
2. Enforce invariants that involve the Aggregate's objects.
3. Control state transitions so that the Aggregate cannot be left inconsistent.

Consider an order containing multiple order lines.

```csharp
public sealed class Order
{
    private readonly List<OrderLine> _lines = [];

    public Guid Id { get; private set; }
    public OrderStatus Status { get; private set; }

    public IReadOnlyCollection<OrderLine> Lines
        => _lines.AsReadOnly();

    public decimal TotalAmount =>
        _lines.Sum(line => line.Subtotal);

    public void AddItem(
        Guid productId,
        string productName,
        decimal unitPrice,
        int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException(
                "Items can only be added to draft orders.");

        if (quantity <= 0)
            throw new DomainException(
                "Quantity must be greater than zero.");

        if (unitPrice < 0)
            throw new DomainException(
                "Unit price cannot be negative.");

        var line = new OrderLine(
            productId,
            productName,
            unitPrice,
            quantity);

        _lines.Add(line);
    }

    public void RemoveItem(Guid productId)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException(
                "Items can only be removed from draft orders.");

        var line = _lines.SingleOrDefault(
            x => x.ProductId == productId);

        if (line is null)
            throw new DomainException(
                "The product does not exist in this order.");

        _lines.Remove(line);
    }

    public void Confirm()
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException(
                "Only draft orders can be confirmed.");

        if (_lines.Count == 0)
            throw new DomainException(
                "An order must contain at least one item.");

        Status = OrderStatus.Confirmed;
    }
}
```

The `OrderLine` objects are part of the Aggregate's internal model:

```csharp
public sealed class OrderLine
{
    public Guid ProductId { get; private set; }
    public string ProductName { get; private set; }
    public decimal UnitPrice { get; private set; }
    public int Quantity { get; private set; }

    public decimal Subtotal => UnitPrice * Quantity;

    internal OrderLine(
        Guid productId,
        string productName,
        decimal unitPrice,
        int quantity)
    {
        ProductId = productId;
        ProductName = productName;
        UnitPrice = unitPrice;
        Quantity = quantity;
    }
}
```

Notice that external code cannot directly construct an `OrderLine` through its public interface or modify the order's internal collection.

The caller interacts with the Aggregate through operations such as:

```csharp
order.AddItem(productId, "Keyboard", 80, 2);
order.RemoveItem(productId);
order.Confirm();
```

It does not manipulate the internal collection:

```csharp
// These operations should not be part of the public domain interface.

order.Lines.Add(new OrderLine(...));
order.Lines.First().Quantity = 10;
```

The exact access modifiers and collection design depend on the language and persistence technology. The principle does not: external code should not be able to bypass the Aggregate Root and violate its invariants.

### The root protects the entire Aggregate

An Aggregate Root is not simply the main entity in a group of related classes.

Suppose an order contains several order lines. If changing an order line can make the order inconsistent, that operation must be controlled by the root.

For example, directly changing a line's quantity might bypass:

- Restrictions on modifying confirmed orders.
- Order-level quantity limits.
- Rules about discounts.
- Rules about minimum order values.
- Recalculation of dependent amounts.

The root provides the boundary within which those rules can be enforced.

This does not mean every operation must contain all the logic in one large class. The root can delegate calculations and behavior to its internal entities and Value Objects. What matters is that the operation is initiated through the root and the Aggregate's invariants remain protected.

## 3. Designing Aggregates Around Business Invariants

One of the most common mistakes in DDD is designing Aggregates around relationships between objects instead of the business rules that govern them.

Developers naturally tend to group objects that reference one another, share a database table relationship, or appear together in a user interface.

For example, a typical e-commerce domain might contain:

- Customer
- Address
- Order
- OrderLine
- Product
- Category
- Payment
- Shipment
- Inventory
- Warehouse

A naive design might place all these objects inside one large Customer Aggregate because they are somehow related to a customer.

That design may appear convenient initially, but it creates a large consistency boundary. A change to a customer's address might require loading the entire object graph. A stock update might involve the same Aggregate as an order confirmation. Independent operations can become unnecessarily coupled.

Instead, start with business operations and the invariants they must preserve.

### Example: Order and Order Lines

Consider these business rules:

- An order must contain at least one item before confirmation.
- An order line must have a positive quantity.
- The order total must reflect its lines.
- A confirmed order cannot be edited.

These rules involve the order and its lines. They need to be protected together.

A reasonable model is:

```text
Order (Aggregate Root)
 ├── OrderLine
 ├── OrderLine
 └── OrderLine
```

The Order Aggregate is responsible for enforcing the rules governing its lines.

### Example: Product and Inventory

Now consider a product and its inventory.

A product may have:

- Name
- Description
- Brand
- Category
- Price
- Publication status

Inventory may have:

- Warehouse
- Available quantity
- Reserved quantity
- Reorder threshold

These concepts are related, but they have different business responsibilities.

Changing a product description should not require loading inventory. Reserving stock should not require loading product descriptions, images, and category information.

They may therefore belong to separate Aggregates:

```text
Product (Aggregate Root)
 ├── ProductName
 ├── ProductDescription
 └── ProductPrice

InventoryItem (Aggregate Root)
 ├── WarehouseId
 ├── AvailableQuantity
 └── ReservedQuantity
```

The two Aggregates can refer to one another using identity:

```csharp
public sealed class InventoryItem
{
    public Guid Id { get; private set; }
    public Guid ProductId { get; private set; }
    public Guid WarehouseId { get; private set; }

    public int AvailableQuantity { get; private set; }
    public int ReservedQuantity { get; private set; }
}
```

The `ProductId` is a reference to another Aggregate, not an invitation to embed the entire Product object.

This separation allows product information and inventory state to evolve independently.

It also raises an important modeling question: should the product price be copied into an order line when the order is placed?

Usually, yes. An order often needs to preserve the price agreed at the time of purchase. If the product's current price changes tomorrow, the historical order should not silently change with it.

The order line therefore stores a snapshot of the relevant commercial information rather than relying on the current Product Aggregate.

This is an example of modeling around business meaning rather than simply navigating object references.

## 4. Keep Aggregates Small

Aggregates should be as small as the consistency rules allow.

A small Aggregate is generally easier to understand, load, modify, persist, test, and protect against concurrent changes.

However, small does not mean splitting every entity into a separate Aggregate. If a business invariant genuinely requires multiple objects to change together, separating them can make the model more complicated and weaken consistency guarantees.

The goal is not to minimize the number of objects in an Aggregate. It is to establish the smallest boundary that can reliably protect its invariants.

### Example: A shopping cart

A shopping cart might contain:

```text
ShoppingCart (Aggregate Root)
 ├── CartItem
 ├── CartItem
 └── CartItem
```

The cart may enforce rules such as:

- A cart item must have a positive quantity.
- The same product should not appear as duplicate lines.
- A cart cannot be checked out if it is empty.
- A cart may have a maximum number of items.
- A cart cannot be modified after checkout.

These rules can reasonably be enforced within the ShoppingCart Aggregate.

But should the Product Aggregate also be inside the cart?

Usually not.

The cart needs enough product information to perform its own operations, but it does not need to own the product's complete lifecycle.

A cart item might reference the product by identity and retain a suitable price or product-information snapshot, depending on the business rules.

### Example: A warehouse

Imagine a warehouse containing hundreds of thousands of inventory items.

A naive design might be:

```text
Warehouse (Aggregate Root)
 ├── InventoryItem
 ├── InventoryItem
 ├── InventoryItem
 └── ...
```

If every stock reservation requires loading and locking the entire warehouse Aggregate, the design becomes expensive and creates unnecessary contention.

A better boundary may be an individual inventory position, perhaps identified by product and warehouse:

```text
InventoryItem (Aggregate Root)
 ├── ProductId
 ├── WarehouseId
 ├── AvailableQuantity
 └── ReservedQuantity
```

This allows independent inventory operations to proceed concurrently for different items.

However, consider a warehouse-wide invariant such as:

> The total number of occupied storage positions must never exceed the warehouse's capacity.

If this rule must be enforced strictly and immediately, an individual inventory item cannot protect it on its own. The design needs another consistency mechanism, such as a capacity-reservation Aggregate, a dedicated allocation process, or an appropriate transactional constraint.

The lesson is not that warehouse-level rules should always be ignored. It is that their consistency requirements must be made explicit before choosing the Aggregate boundary.

## 5. Reference Other Aggregates by Identity

Aggregates should generally reference other Aggregates by their identifiers rather than by direct object references.

For example:

```csharp
public sealed class Order
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
}
```

Instead of:

```csharp
public sealed class Order
{
    public Guid Id { get; private set; }
    public Customer Customer { get; private set; }
}
```

The first model makes the boundary explicit. The Order Aggregate knows which customer it belongs to, but it does not own the Customer Aggregate.

This has several advantages:

- Aggregates can be loaded independently.
- Changes to one Aggregate do not require loading another.
- Persistence does not need to reconstruct large interconnected object graphs.
- Aggregate boundaries remain explicit.
- Concurrent modifications are less likely to affect unrelated business operations.

There is an important distinction between referencing an Aggregate and needing information from it.

For example, an order may need to verify that a customer is eligible to place an order. The application service can retrieve the relevant customer information, or a domain policy can evaluate the rule using supplied information.

The Order Aggregate should not necessarily load the entire Customer Aggregate just to check a customer's status.

A simplified application service might look like this:

```csharp
public async Task ConfirmOrder(
    Guid orderId,
    CancellationToken cancellationToken)
{
    var order = await orderRepository.GetById(
        orderId,
        cancellationToken);

    var customer = await customerRepository.GetById(
        order.CustomerId,
        cancellationToken);

    if (!customer.CanPlaceOrders)
        throw new DomainException(
            "This customer cannot place orders.");

    order.Confirm();

    await orderRepository.Save(
        order,
        cancellationToken);
}
```

This example is appropriate only if customer eligibility is an application-level precondition or if the relevant information can be treated as a decision input.

If the business requires a guarantee that customer eligibility and order confirmation are coordinated atomically, this implementation alone is insufficient. That stronger requirement needs an explicit consistency design.

## 6. Cross-Aggregate Consistency

An Aggregate provides a consistency boundary, but business processes frequently involve several Aggregates.

For example, confirming an order may involve:

1. Confirming the order.
2. Reserving inventory.
3. Authorizing payment.
4. Scheduling shipment.

These operations affect different parts of the domain. It is often neither practical nor desirable to place all of them inside one enormous Aggregate.

Instead, each Aggregate protects its own invariants, while the broader business process coordinates their interactions.

Consider an inventory reservation:

```csharp
public sealed class InventoryItem
{
    public Guid Id { get; private set; }
    public int AvailableQuantity { get; private set; }
    public int ReservedQuantity { get; private set; }

    public void Reserve(int quantity)
    {
        if (quantity <= 0)
            throw new DomainException(
                "Reservation quantity must be positive.");

        if (AvailableQuantity < quantity)
            throw new DomainException(
                "Insufficient inventory.");

        AvailableQuantity -= quantity;
        ReservedQuantity += quantity;
    }

    public void Release(int quantity)
    {
        if (quantity <= 0)
            throw new DomainException(
                "Release quantity must be positive.");

        if (ReservedQuantity < quantity)
            throw new DomainException(
                "Cannot release more than the reserved quantity.");

        ReservedQuantity -= quantity;
        AvailableQuantity += quantity;
    }
}
```

The InventoryItem Aggregate protects its own stock invariants.

The Order Aggregate protects order state and order-line rules.

Neither Aggregate needs to own the other.

A business process coordinates the interaction:

```text
Order
  |
  | OrderConfirmed
  v
Inventory Reservation
  |
  | InventoryReserved
  v
Payment Authorization
  |
  | PaymentAuthorized
  v
Order Fulfilment
```

The exact sequence depends on the business. Some systems reserve inventory before payment authorization; others authorize payment first. The important point is that the workflow is a separate concern from the invariants inside each Aggregate.

If inventory reservation fails, the order may need to be cancelled or moved into a state requiring intervention. If payment authorization fails after stock has been reserved, the reservation may need to be released.

These are process-level rules. They can be modeled using domain events, process managers, or a Saga, depending on the requirements.

### Strong consistency versus eventual consistency

Not every business rule needs to be enforced synchronously across the entire system.

A useful distinction is:

- **Strong consistency within an Aggregate:** An operation must preserve the Aggregate's invariants before its transaction completes.
- **Eventual consistency between Aggregates:** Separate Aggregates may be updated in separate transactions, with the system converging toward the required business state through messages or events.

For example, after an order is confirmed, the customer's loyalty points might be updated asynchronously.

A short delay may be acceptable because the points update does not determine whether the order itself is valid.

By contrast, allowing two customers to reserve the final available unit of a product is not acceptable if the inventory rule requires that stock cannot be oversold.

The inventory reservation operation must therefore enforce its quantity invariant atomically.

Do not use eventual consistency as an excuse to weaken an important business rule. First identify the actual consistency requirement, then choose the appropriate design.

## 7. Aggregate Size, Performance, and Concurrency

Aggregate boundaries have direct consequences for performance and concurrency.

When an Aggregate is loaded and modified, the system must ensure that concurrent operations do not silently overwrite one another or violate its invariants.

Consider an inventory item with one unit available.

Two customers attempt to reserve it at nearly the same time.

Both requests might read:

```text
AvailableQuantity = 1
```

Both then attempt to reserve the item.

Without concurrency protection, the system may accept both reservations and oversell the inventory.

One common solution is optimistic concurrency control.

An Aggregate can carry a version number:

```csharp
public sealed class InventoryItem
{
    public Guid Id { get; private set; }
    public int AvailableQuantity { get; private set; }

    public long Version { get; private set; }

    public void Reserve(int quantity)
    {
        if (AvailableQuantity < quantity)
            throw new DomainException(
                "Insufficient inventory.");

        AvailableQuantity -= quantity;
    }
}
```

The persistence mechanism updates the record only if the version has not changed since it was loaded.

Conceptually:

```sql
UPDATE InventoryItems
SET AvailableQuantity = @NewQuantity,
    Version = Version + 1
WHERE Id = @Id
  AND Version = @ExpectedVersion;
```

If the update affects no rows, another transaction has changed the Aggregate. The operation must be retried or rejected according to the business and technical requirements.

Optimistic concurrency is particularly useful when conflicts are relatively uncommon.

Pessimistic locking may be more appropriate in some high-contention situations, but it introduces different trade-offs involving lock duration, throughput, and deadlocks.

The Aggregate design influences these decisions.

A large Aggregate can create unnecessary conflicts because unrelated operations share the same concurrency boundary. A smaller Aggregate can allow more operations to proceed independently.

For example, if all products in a warehouse belong to one Warehouse Aggregate, reserving stock for unrelated products may create conflicts. If inventory is modeled as independent product-and-warehouse positions, those reservations can often proceed independently.

However, splitting an Aggregate simply to avoid concurrency conflicts is not sufficient if it breaks an invariant that genuinely must be protected atomically.

The business rule determines the boundary. Performance considerations help evaluate the consequences of that decision.

## 8. Aggregates and Database Transactions

An Aggregate is a domain-model concept, not a database design pattern.

In a conventional relational implementation, changes to an Aggregate are often persisted within a single database transaction.

For example, confirming an order may update:

- The order status.
- One or more order-line records.
- An order-level value or timestamp.

These changes should be committed together when they represent one Aggregate operation.

However, the Aggregate is not automatically equivalent to a database table.

An Aggregate may be persisted across multiple tables. Conversely, a database table may contain data belonging to several different Aggregates.

Consider:

```text
Orders
OrderLines
Products
InventoryItems
Customers
```

A possible mapping is:

```text
Order Aggregate
 ├── Orders
 └── OrderLines

Product Aggregate
 └── Products

InventoryItem Aggregate
 └── InventoryItems

Customer Aggregate
 └── Customers
```

This is a persistence mapping, not a universal rule. The domain model determines the Aggregate boundaries; the persistence model maps those boundaries to storage.

Likewise, using an ORM does not mean that the entire navigation graph it can load belongs to one Aggregate.

A developer may be able to navigate from Order to Customer to Address to Orders. That does not mean those objects should be part of the same consistency boundary.

The important question remains: which objects must be changed together to preserve the business invariants?

### Repository responsibilities

Repositories are generally designed around Aggregate Roots.

For example:

```csharp
public interface IOrderRepository
{
    Task<Order?> GetById(
        Guid id,
        CancellationToken cancellationToken);

    Task Add(
        Order order,
        CancellationToken cancellationToken);

    Task Save(
        Order order,
        CancellationToken cancellationToken);
}
```

The application service retrieves the Aggregate Root, invokes its behavior, and asks the repository to persist the result.

```csharp
var order = await orderRepository.GetById(
    orderId,
    cancellationToken);

if (order is null)
    throw new OrderNotFoundException(orderId);

order.Confirm();

await orderRepository.Save(
    order,
    cancellationToken);
```

There is generally no need for a separate repository for every internal entity.

For example, an `OrderLineRepository` would be questionable if OrderLine is not an Aggregate Root and cannot be modified independently.

The repository should expose the persistence boundary of the Aggregate, not reproduce every database table as a repository abstraction.

## 9. Commands Should Express Business Intent

An Aggregate should expose meaningful operations rather than generic setters that allow arbitrary state manipulation.

Compare:

```csharp
order.SetStatus(OrderStatus.Confirmed);
```

with:

```csharp
order.Confirm();
```

The first describes a technical state assignment. The second expresses a business operation.

The difference matters because confirming an order may require several rules:

- The order must be in a confirmable state.
- It must contain at least one item.
- Required customer information must be available.
- The order may need to record when it was confirmed.
- A domain event may need to be raised.

A generic setter does not communicate or protect those rules.

Similarly:

```csharp
account.SetBalance(500);
```

is generally less expressive than:

```csharp
account.Deposit(500);
```

And:

```csharp
reservation.SetStatus(ReservationStatus.Cancelled);
```

is less meaningful than:

```csharp
reservation.Cancel(reason);
```

A well-designed Aggregate API communicates what the business is trying to accomplish and controls how the state changes.

This does not mean every property needs a dedicated method. The goal is to model meaningful business behavior, not create unnecessary ceremony.

## 10. When an Aggregate Boundary Is Wrong

Aggregate design is an iterative process. A boundary that initially appears reasonable may reveal problems when real business scenarios are modeled.

There are several common warning signs.

### 10.1 The Aggregate contains too much

Suppose a Customer Aggregate contains:

```text
Customer
 ├── Addresses
 ├── Orders
 │    ├── OrderLines
 │    └── Payments
 ├── LoyaltyPoints
 ├── SupportTickets
 └── Notifications
```

This is likely an overly broad boundary.

Updating a customer's phone number should not require loading all historical orders, payments, and support tickets.

The Aggregate may be grouping objects based on ownership or navigation relationships rather than consistency requirements.

### 10.2 The Aggregate exposes internal objects

If external code can freely modify child entities, the Aggregate Root cannot reliably protect its invariants.

For example:

```csharp
order.Lines[0].Quantity = -10;
```

If this is possible without going through the root, the Aggregate's boundary is not being respected.

### 10.3 The Aggregate requires too many unrelated objects

If every operation requires loading a large graph of objects, investigate whether all those objects truly participate in the same consistency rules.

An Aggregate should not become a convenient excuse for loading an entire domain model.

### 10.4 A single Aggregate creates excessive contention

If unrelated business operations constantly conflict because they modify the same Aggregate, its boundary may be too broad.

Investigate whether the operations really need to share an atomic consistency boundary.

### 10.5 The Aggregate is just a data container

An entity with dozens of public setters and no meaningful behavior may not be protecting anything.

For example:

```csharp
public class Order
{
    public Guid Id { get; set; }
    public OrderStatus Status { get; set; }
    public decimal TotalAmount { get; set; }
}
```

This may be a useful data-transfer structure, but it does not demonstrate the responsibilities of a rich Aggregate Root.

Not every domain model must be highly object-oriented, but the design must still provide a reliable place for enforcing the invariants that matter.

### 10.6 The Aggregate tries to coordinate an entire business process

A common mistake is putting every step of a workflow into one Aggregate.

For example, an Order Aggregate should not necessarily call a payment gateway, reserve inventory from several warehouses, send notifications, and arrange delivery from inside its `Confirm()` method.

Those responsibilities belong to application services, domain services, or process managers, depending on their nature.

The Aggregate should protect its own state transitions. A broader process should coordinate interactions between Aggregates.

## 11. A Practical Method for Discovering Aggregate Boundaries

Aggregate design becomes easier when approached through business scenarios rather than class diagrams.

Use the following process.

### Step 1: Identify meaningful business operations

Start with commands expressed in domain language:

- Place an order.
- Add an item to a cart.
- Confirm a reservation.
- Withdraw money.
- Reserve inventory.
- Cancel a shipment.
- Approve an application.

Avoid starting with CRUD operations such as `UpdateOrder` or `SaveCustomer`. Those names often hide the business decisions that matter.

### Step 2: Identify the invariants

For each operation, ask:

- What must be true before this operation?
- What must be true after it completes?
- Which rules must never be violated?
- Which rules are business-critical?
- What happens when a rule cannot be satisfied?

Write these rules in language that business stakeholders can understand.

### Step 3: Identify the state that must change together

Determine which pieces of state must be updated atomically to preserve the invariants.

For example, when withdrawing money:

- The account balance decreases.
- A withdrawal transaction is recorded.

If these two changes must always happen together, they belong within the same consistency boundary.

### Step 4: Identify the Aggregate Root

Choose the object that provides the meaningful business interface for the operation.

The root should be responsible for enforcing the invariants of the boundary.

### Step 5: Identify references to other Aggregates

Determine which related concepts have independent lifecycles and consistency requirements.

Use their identities rather than embedding their entire object graphs.

### Step 6: Examine concurrency and performance

Ask:

- How frequently will this Aggregate be loaded?
- How many objects will it contain?
- Which operations can happen concurrently?
- Will unrelated operations compete for the same version or lock?
- Can the Aggregate be loaded and persisted efficiently?

These questions do not replace business modeling. They reveal the technical consequences of the chosen boundary.

### Step 7: Test the boundary against real scenarios

Try normal, exceptional, and concurrent scenarios.

For example:

- Two users reserve the last available item.
- An order is cancelled while payment authorization is in progress.
- A customer changes their address after placing an order.
- An inventory reservation is released twice.
- An order is confirmed while another operation attempts to add an item.

If the design cannot explain how these situations preserve the business rules, revisit the boundaries.

## 12. Practical Exercises

### Exercise 1: Design an E-Commerce Order Aggregate

Model an order with the following requirements:

- An order contains one or more items.
- Each item has a product identifier, name, unit price, and quantity.
- Quantities must be positive.
- An order cannot be confirmed if it is empty.
- A confirmed order cannot be modified.
- An order can be cancelled only if it has not been shipped.
- The total amount must reflect the order lines.

Your tasks:

1. Identify the Aggregate Root.
2. Identify the entities and Value Objects inside the Aggregate.
3. List its invariants.
4. Design the public methods.
5. Implement the Aggregate in your preferred language.
6. Write unit tests for valid and invalid state transitions.

Consider whether the Product itself belongs inside the Order Aggregate. Explain why or why not.

### Exercise 2: Model Inventory Reservation

Design an Inventory Aggregate with these requirements:

- Inventory belongs to a warehouse and a product.
- Available quantity cannot become negative.
- A reservation must have a positive quantity.
- A reservation cannot exceed available stock.
- Reserved stock can be released.
- A reservation must not be released more than once.

Your tasks:

1. Identify the Aggregate boundary.
2. Decide whether a reservation is an internal entity, a Value Object, or an independent Aggregate.
3. Model reservation and release operations.
4. Implement optimistic concurrency control.
5. Write tests for competing reservation requests.

Consider what changes if inventory is tracked by product, warehouse, storage location, and batch.

### Exercise 3: Find the Boundaries in a Booking System

Consider a hotel booking system with:

- Guest
- Hotel
- Room
- RoomAvailability
- Reservation
- Payment
- Invoice

A reservation must not overlap with another confirmed reservation for the same room.

Your tasks:

1. Identify potential Aggregates.
2. Identify which invariants belong to each Aggregate.
3. Decide how overlapping reservations are prevented.
4. Explain how payment and reservation confirmation interact.
5. Consider how the design changes when a hotel has thousands of rooms and reservations.

Pay particular attention to the overlap rule. An individual Reservation Aggregate cannot necessarily guarantee that no other Reservation overlaps with it. You must identify the shared consistency boundary or concurrency mechanism that protects room availability.

### Exercise 4: Refactor an Anemic Domain Model

Start with a model that exposes public setters and allows application services to modify internal state directly.

Refactor it so that:

- Business operations are expressed through meaningful methods.
- Invariants are protected by the Aggregate Root.
- Internal entities cannot be modified independently.
- Other Aggregates are referenced by identity.
- Persistence is handled through an Aggregate-oriented repository.

Write tests that demonstrate that invalid state transitions are rejected.

### Exercise 5: Challenge an Existing Aggregate

Find an Aggregate in an existing codebase.

Ask:

- Does its boundary reflect business invariants?
- Does it contain objects that change independently?
- Can external code bypass its root?
- Does it load unnecessary data?
- Does it create excessive concurrency conflicts?
- Are its methods expressed in business language?
- Does it coordinate responsibilities that belong to a broader process?

Document the original design, identify the problems, and propose a revised boundary.

## 13. Completion Criteria

You should be comfortable with Aggregate design when you can:

- Explain why an Aggregate is a consistency boundary rather than an object graph.
- Distinguish an Aggregate Root from ordinary entities and Value Objects.
- Identify invariants from business scenarios.
- Design Aggregate boundaries around those invariants.
- Keep Aggregates small without splitting rules that require atomic consistency.
- Reference other Aggregates by identity.
- Express business operations through meaningful commands.
- Explain the difference between consistency within an Aggregate and coordination between Aggregates.
- Understand how database transactions and concurrency control support Aggregate consistency.
- Recognize oversized, anemic, and overly coupled Aggregates.
- Test Aggregate behavior, including invalid state transitions and concurrency-sensitive scenarios.

The real skill is not learning to draw boxes around entities. It is learning to recognize which decisions must be protected together, which can evolve independently, and where the boundary between them should exist.

**An Aggregate is where a business rule becomes a consistency guarantee.**