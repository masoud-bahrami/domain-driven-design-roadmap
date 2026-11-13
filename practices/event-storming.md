# Event Storming

Event Storming is a collaborative workshop format for exploring a business domain through the events that occur within it and the decisions, commands, actors, policies, and systems surrounding those events.

Originally introduced by Alberto Brandolini, Event Storming brings domain experts, developers, testers, business analysts, and other stakeholders together around a shared visual workspace.

Instead of beginning with requirements, database structures, classes, or architecture diagrams, participants explore what happens in the business.

They identify meaningful business events, arrange them in a plausible sequence, and investigate what causes those events, who is responsible, which rules influence them, and what happens when things go wrong.

For example, consider a logistics company that transports goods from one location to another.

A requirement might say:

> The system must support shipment dispatch and delivery tracking.

But this statement does not explain what actually happens.

- Who creates a shipment?
- When is a vehicle assigned?
- What happens before loading?
- Who authorizes dispatch?
- What happens if the vehicle breaks down?
- How is departure recorded?
- What happens when delivery fails?
- Can a shipment be partially delivered?

Event Storming provides a collaborative way to explore these questions.

The resulting model helps participants develop a shared understanding of the business, discover hidden rules, identify important concepts, and investigate possible Bounded Context boundaries.

## What Is Event Storming?

Event Storming explores a domain by starting with **Domain Events**, which represent things that have happened and matter to the business.

Events are written in the past tense because they describe facts:

- Order Placed
- Payment Authorized
- Stock Reserved
- Shipment Dispatched
- Goods Delivered
- Refund Approved
- Invoice Issued
- Account Suspended

Participants place these events on a shared timeline and investigate the activities and decisions that lead to them.

For example:

```text
Order Placed
     |
     v
Payment Authorized
     |
     v
Stock Reserved
     |
     v
Order Confirmed
     |
     v
Shipment Created
     |
     v
Shipment Dispatched
     |
     v
Goods Delivered
```

This is a simplified happy-path sequence. A real workshop would explore alternative paths, dependencies, policies, and exceptions.

For example:

- What happens if payment is rejected?
- What happens if stock cannot be reserved?
- Can an order be confirmed before payment is captured?
- What happens if a shipment is dispatched but delivery fails?

These questions often reveal more than the initial sequence itself.

The central idea is to explore the business through what happens, why it happens, who makes it happen, and what consequences follow.

## The Three Event Storming Formats

The original Event Storming approach identifies three main workshop formats. They address different levels of exploration and should not be confused with one another.

### 1. Big Picture Event Storming

Big Picture Event Storming explores a business domain at a broad level.

Its purpose is to understand the overall business process, identify major events, discover organizational boundaries, and reveal areas of uncertainty.

It is particularly useful when participants do not yet have a shared understanding of the domain or when the organization wants to explore a large business capability.

Consider an e-commerce business.

A Big Picture session might discover the following sequence:

```text
Customer Registered
       |
       v
Product Added to Cart
       |
       v
Order Placed
       |
       v
Payment Authorized
       |
       v
Stock Reserved
       |
       v
Order Confirmed
       |
       v
Shipment Created
       |
       v
Shipment Dispatched
       |
       v
Goods Delivered
       |
       v
Order Completed
```

The purpose is not to specify every validation rule or implementation detail.

Instead, participants explore the overall business journey.

They may discover that:

- Payment and inventory are managed by different teams.
- Shipping begins only after order confirmation.
- Customer registration is not always required before purchasing.
- Returns and refunds follow a separate process.
- Some steps involve external organizations.

These discoveries help the team understand the business landscape and identify areas that deserve deeper exploration.

**When to use it**

- Starting work in an unfamiliar domain.
- Exploring a large business capability.
- Aligning stakeholders around the overall process.
- Discovering major business events and organizational boundaries.
- Identifying unclear or disputed areas before detailed design.

**Typical outcome**

A broad picture of the domain, its major events, participants, business processes, potential boundaries, and open questions.

### 2. Process-Level Event Storming

Process-Level Event Storming explores a particular business process in greater detail.

It starts with a focused scope and investigates the commands, actors, policies, decisions, and exceptions surrounding the events.

Consider the shipment dispatch process discovered during a Big Picture session.

A Process-Level workshop might explore:

```text
Shipment Created
      |
      v
[Assign Vehicle]
      |
      v
Vehicle Assigned
      |
      v
[Assign Driver]
      |
      v
Driver Assigned
      |
      v
[Load Shipment]
      |
      v
Loading Completed
      |
      v
[Authorize Dispatch]
      |
      v
Dispatch Authorized
      |
      v
[Record Departure]
      |
      v
Shipment Dispatched
```

Participants can now investigate the business rules behind each transition.

For example:

- Can a vehicle be assigned before the shipment is ready?
- Must the driver have the required qualifications?
- Can loading be completed in multiple stages?
- Who is authorized to approve dispatch?
- Can authorization be revoked?
- What happens if the assigned vehicle becomes unavailable?

The workshop may reveal that vehicle assignment and dispatch authorization belong to different roles and follow different rules.

It may also reveal that a shipment can be loaded but cannot be dispatched until a required inspection is completed.

**When to use it**

- Exploring a specific business workflow.
- Clarifying complex business rules.
- Understanding alternative paths and exceptions.
- Investigating operational bottlenecks.
- Discovering dependencies between activities and decisions.

**Typical outcome**

A detailed representation of a business process, including its events, commands, actors, policies, exceptions, and unresolved questions.

### 3. Software Design-Level Event Storming

Software Design-Level Event Storming explores how the discovered domain behavior can be represented in software.

It moves closer to implementation while keeping business understanding at the center of the discussion.

The team investigates:

- Which components handle commands?
- Which Aggregates protect business invariants?
- Which Domain Events are produced?
- Which policies react to events?
- Which read models support user queries?
- Which external systems participate?
- Where should responsibilities and boundaries exist?

For example, consider the event:

```text
Stock Reserved
```

The team may explore its implementation:

```text
Reserve Stock Command
        |
        v
Inventory Application Service
        |
        v
Stock Reservation Aggregate
        |
        | Validates availability
        v
Stock Reserved Domain Event
        |
        v
Order Fulfillment Policy
        |
        v
Confirm Order Command
```

The discussion may reveal that:

- Inventory owns stock reservation rules.
- Sales owns order confirmation.
- The contexts communicate through an explicit contract.
- A failed reservation must produce a different business outcome.
- A read model is needed to display reservation status.

This format helps connect domain behavior to software design without assuming that every sticky note should become a class, service, or message.

**When to use it**

- Designing or refining a domain model.
- Investigating Aggregate responsibilities.
- Exploring Domain Events and policies.
- Understanding application workflows.
- Discussing integration between Bounded Contexts.
- Translating domain discoveries into implementation decisions.

**Typical outcome**

A candidate software design that connects commands, domain behavior, Aggregates, events, policies, read models, and integration boundaries.

It is a design hypothesis that should be validated, not a complete implementation specification.

## The Main Building Blocks of Event Storming

Event Storming commonly uses colored sticky notes to distinguish different kinds of information.

The exact colors and conventions can vary between workshops and tools. The meaning of the elements matters more than the color.

### 1. Domain Events

<text color="#D97706" weight="semibold" inline>Orange</text>

Domain Events represent meaningful business facts that have already happened.

They are typically expressed in the past tense.

Examples:

- Order Placed
- Payment Authorized
- Stock Reserved
- Shipment Assigned
- Loading Completed
- Shipment Dispatched
- Goods Delivered
- Refund Approved

Consider the difference between these statements:

```text
Dispatch Shipment
Shipment Dispatched
```

The first expresses an intention to perform an action. It is a Command.

The second expresses a fact that the action has happened. It is a Domain Event.

This distinction is fundamental.

A command may be rejected. An event represents something that has already occurred.

For example:

```text
Command:
  Reserve Stock

Possible outcomes:
  Stock Reserved
  Stock Reservation Rejected
```

Not every command produces the event that its name suggests.

**Practical guidance**

- Use past-tense names.
- Express business facts rather than technical operations.
- Make the meaning understandable to domain experts.
- Avoid generic names such as `Data Updated` or `Status Changed`.
- Include events from exceptional paths when they matter.

An event should communicate something meaningful about the business, not merely that a database row changed.

### 2. Commands

<text color="#3B82F6" weight="semibold" inline>Blue</text>

Commands express intentions or requests to perform an action.

Examples:

- Place Order
- Authorize Payment
- Reserve Stock
- Assign Vehicle
- Approve Refund
- Dispatch Shipment

A command usually originates from an actor, another system, or a policy reacting to an event.

For example:

```text
Customer
   |
   | Place Order
   v
Order Placed
```

Or:

```text
Payment Authorized
       |
       v
[Confirm Order]
       |
       v
Order Confirmed
```

Commands can be accepted, rejected, or deferred depending on business rules.

For example, `Reserve Stock` may be rejected if the requested quantity is unavailable.

**Practical guidance**

- Use imperative names.
- Identify who or what issues the command.
- Ask what conditions must hold before it can succeed.
- Explore possible rejection outcomes.
- Avoid treating commands and events as interchangeable.

A command describes what someone wants to happen. An event describes what actually happened.

### 3. Actors

<text color="#A855F7" weight="semibold" inline>Purple</text>

Actors represent people, roles, organizations, or external participants involved in the process.

Examples:

- Customer
- Dispatcher
- Warehouse Operator
- Driver
- Finance Officer
- Payment Provider
- External Carrier

Consider a refund process:

```text
Customer
   |
   | Requests Refund
   v
Refund Requested
```

The Customer initiates the process, but a Support Agent may review the request and a Finance Officer may authorize the refund.

Identifying actors helps reveal responsibilities and organizational dependencies.

**Practical guidance**

- Identify the role performing the action, not necessarily a specific individual.
- Distinguish internal participants from external systems or organizations.
- Ask whether different roles can perform the same activity.
- Investigate approval and authorization responsibilities.

An actor is a participant in the business process, not automatically a software component or user account.

### 4. Policies

<text color="#16A34A" weight="semibold" inline>Green</text>

Policies represent business reactions that occur when certain events happen.

They are often expressed as:

> Whenever X happens, do Y.

For example:

```text
Payment Authorized
       |
       v
[When payment is authorized,
 reserve stock]
       |
       v
Reserve Stock Command
```

Another example:

```text
Stock Reserved
       |
       v
[When stock is reserved,
 confirm the order]
       |
       v
Confirm Order Command
```

Policies are useful for discovering business rules and reactions that connect different parts of a process.

Consider a subscription system:

```text
Payment Failed
      |
      v
[Notify Customer]
      |
      v
Send Payment Failure Notification
```

Or:

```text
Subscription Expired
      |
      v
[Revoke Access]
      |
      v
Revoke Subscription Access
```

A policy may be implemented using an Application Service, Process Manager, event handler, or other mechanism. The workshop should first clarify the business reaction rather than prematurely choosing an implementation.

**Practical guidance**

- Express the triggering event.
- Describe the business reaction.
- Identify the resulting command.
- Explore conditions and exceptions.
- Determine whether the reaction is immediate or delayed.

A policy is not necessarily a Domain Event handler. It is a way to express a business rule or reaction during exploration.

### 5. Read Models

<text color="#0891B2" weight="semibold" inline>Light blue</text>

Read Models represent information needed by people or systems to make decisions or understand the current state of the business.

Examples:

- Available Stock
- Order Details
- Shipment Tracking Information
- Customer Account Summary
- Payment History
- Outstanding Invoices

Consider a dispatcher who needs to know which shipments are ready for dispatch.

The dispatcher may use a read model containing:

```text
ReadyForDispatch
  ShipmentId
  Vehicle
  Driver
  LoadingStatus
  InspectionStatus
  DispatchAuthorization
```

The dispatcher uses this information to decide which shipment to process.

A Read Model is not necessarily a separate database or a CQRS implementation. During Event Storming, it simply identifies information required by a participant or process.

**Practical guidance**

- Identify who needs the information.
- Clarify what decision or action it supports.
- Distinguish business information from technical diagnostics.
- Identify which context owns the underlying facts.

### 6. External Systems

External Systems represent systems or services outside the process being explored.

Examples:

- Payment Gateway
- SMS Provider
- Carrier Platform
- Tax Authority
- Banking Network
- Identity Provider

Consider a payment workflow:

```text
Payment Service
      |
      | Sends authorization request
      v
Payment Provider
      |
      | Returns authorization result
      v
Payment Authorized
```

The external system may have its own behavior, terminology, failure modes, and availability constraints.

Making it visible helps participants explore dependencies that might otherwise remain hidden.

For example:

- What happens when the payment provider is unavailable?
- Can a request time out even though the provider processed it?
- Can the provider send duplicate notifications?
- How are rejected payments handled?

External systems should be represented when they contribute meaningful behavior to the story.

### 7. Hotspots

<text color="#DC2626" weight="semibold" inline>Pink</text>

Hotspots identify uncertainty, disagreement, complexity, or areas requiring further investigation.

Examples:

```text
Why can a shipment be dispatched
without a completed inspection?
```

```text
Does payment authorization mean
that money has actually been captured?
```

```text
Who owns the customer's consent status?
```

Hotspots are particularly valuable because they prevent the workshop from hiding uncertainty behind a polished diagram.

A disagreement is not necessarily a problem to solve immediately. It is useful information about the current understanding of the domain.

**Practical guidance**

- Record unresolved questions.
- Capture contradictory explanations.
- Identify assumptions that need validation.
- Avoid inventing answers just to complete the diagram.
- Assign follow-up investigation where necessary.

## A Practical Event Storming Session

A successful workshop needs a clear scope, the right participants, and enough room for exploration.

The following approach is suitable for a Process-Level workshop, but can be adapted for other formats.

### 1. Agree on the Scope

Define the business process or capability to explore.

Good examples:

- How does an order move from placement to fulfillment?
- How does a shipment move from creation to delivery?
- How does a customer request a refund?
- How does a loan application get approved?
- How does an organization process a purchase request?

Avoid scopes that are too broad.

For example, `Explore the entire logistics platform` may produce an overwhelming diagram without enough depth to support meaningful decisions.

A more useful scope might be:

`Explore the process from shipment creation to dispatch.`

Also clarify what is outside the scope.

### 2. Invite People with Relevant Domain Knowledge

Include participants who understand the actual work.

Depending on the topic, invite:

- Domain experts.
- Operational employees.
- Business analysts.
- Developers.
- Testers.
- Product owners.
- Representatives of related departments.

For example, exploring shipment dispatch may require:

- A dispatcher.
- A warehouse operator.
- A driver or transport representative.
- A domain analyst.
- Developers responsible for the relevant software.

Do not rely exclusively on managers or existing documentation. People who perform the work often know about exceptions and practical constraints that official process descriptions omit.

### 3. Discover Domain Events

Start by asking participants:

> What happens during this process?

Ask them to write business events on sticky notes.

Encourage past-tense language.

For example:

```text
Shipment Created
Vehicle Assigned
Driver Assigned
Loading Started
Loading Completed
Inspection Completed
Dispatch Authorized
Shipment Dispatched
Delivery Attempted
Goods Delivered
```

At this stage, avoid spending too much time debating implementation details.

The purpose is to build an initial picture of what happens.

### 4. Arrange Events in a Plausible Timeline

Place the events in the order in which they occur.

For example:

```text
Shipment Created
       |
       v
Vehicle Assigned
       |
       v
Driver Assigned
       |
       v
Loading Started
       |
       v
Loading Completed
       |
       v
Inspection Completed
       |
       v
Dispatch Authorized
       |
       v
Shipment Dispatched
       |
       v
Delivery Attempted
       |
       v
Goods Delivered
```

Do not assume the first timeline is correct.

Ask participants whether the sequence reflects reality.

For example, a dispatcher may explain that inspection occurs before loading in some warehouses, while another warehouse performs it afterward.

That difference may reveal variations in the process or separate business rules.

### 5. Explore Commands and Actors

For each important event, ask:

- What caused this event?
- Who initiated the action?
- What command or intention was involved?
- What conditions must hold?

For example:

```text
Dispatcher
    |
    | Assign Vehicle
    v
Vehicle Assigned
```

Then explore:

- Must the vehicle be available?
- Must it have sufficient capacity?
- Can it be assigned to multiple shipments?
- Who can change the assignment?
- Can assignment be reversed?

These questions help discover business rules and responsibilities.

### 6. Add Policies and External Systems

Look for automatic reactions and dependencies.

For example:

```text
Payment Authorized
       |
       v
[Reserve Stock]
       |
       v
Stock Reserved
```

Or:

```text
Shipment Dispatched
       |
       v
[Notify Carrier Platform]
       |
       v
Carrier Platform Updated
```

Explore whether these reactions are mandatory, conditional, immediate, or asynchronous.

Identify external systems that influence the process.

### 7. Identify Read Models

Ask participants what information they need to perform their work.

For example, a dispatcher may need:

- Shipments awaiting vehicle assignment.
- Vehicles currently available.
- Shipments waiting for inspection.
- Shipments authorized for dispatch.
- Shipments with failed delivery attempts.

These information needs may reveal candidate Read Models and queries.

They also help distinguish operational views from the business events that produce them.

### 8. Explore Exceptions and Hotspots

The happy path is only the beginning.

Ask participants to explore situations such as:

- Vehicle becomes unavailable after assignment.
- Driver is replaced before departure.
- Loading fails.
- Inspection is rejected.
- Dispatch authorization is revoked.
- Vehicle departs without the expected record.
- Carrier rejects the shipment.
- Customer is unavailable during delivery.

For example:

```text
Vehicle Assigned
       |
       v
Vehicle Becomes Unavailable
       |
       v
[Assign Replacement Vehicle]
       |
       v
Vehicle Reassigned
```

This raises further questions:

- Must the shipment be unloaded?
- Does the previous driver assignment remain valid?
- Is another inspection required?
- Who approves the replacement?

Mark unresolved questions and disagreements as hotspots.

Do not force the group to resolve every issue during the session.

### 9. Identify Possible Boundaries

Once the business process is more understandable, look for areas with different responsibilities, language, rules, and ownership.

For example, the shipment workshop may reveal:

```text
Shipment Planning
  - Shipment creation
  - Vehicle assignment
  - Driver assignment

Warehouse Operations
  - Loading
  - Inspection
  - Loading completion

Transportation
  - Dispatch authorization
  - Departure
  - Delivery tracking
```

These may suggest separate responsibilities or candidate Bounded Contexts.

However, do not assume that every group of activities deserves a separate context or service.

Investigate whether the concepts and rules genuinely form distinct models.

### 10. Capture Decisions and Validate the Result

At the end of the workshop:

- Review the event timeline.
- Confirm important terminology.
- Record agreed business rules.
- Preserve unresolved questions.
- Identify assumptions requiring verification.
- Document potential boundaries.
- Agree on follow-up investigations.

Ask domain experts to review the result.

A diagram is not validated merely because participants have stopped adding sticky notes.

The participants should recognize it as a reasonable representation of the process and its important variations.

## Worked Example: Order Fulfillment

Consider an e-commerce business that wants to improve its order fulfillment process.

The initial requirement is:

> When a customer places an order, the system should confirm it and arrange delivery.

A Big Picture discussion identifies the main events:

```text
Customer Registered
Order Placed
Payment Authorized
Stock Reserved
Order Confirmed
Shipment Created
Shipment Dispatched
Goods Delivered
```

The team then runs a Process-Level workshop focused on order confirmation.

### The Initial Process

```text
Customer
   |
   | Place Order
   v
Order Placed
   |
   v
[Authorize Payment]
   |
   v
Payment Authorized
   |
   v
[Reserve Stock]
   |
   v
Stock Reserved
   |
   v
[Confirm Order]
   |
   v
Order Confirmed
```

### Exploring Business Rules

The team asks what happens when payment authorization fails.

The Payment specialist explains:

> We do not confirm an order when payment authorization fails.

The team adds an alternative path:

```text
Order Placed
     |
     v
[Authorize Payment]
     |
     +----------------------+
     |                      |
     v                      v
Payment Authorized     Payment Rejected
     |                      |
     v                      v
[Reserve Stock]       [Notify Customer]
```

Next, the Inventory specialist explains:

> Payment authorization does not guarantee that stock can be reserved. Stock can be taken by another order before the reservation is completed.

This reveals a second failure path:

```text
Payment Authorized
       |
       v
[Reserve Stock]
       |
       +----------------------+
       |                      |
       v                      v
Stock Reserved       Stock Reservation Rejected
       |                      |
       v                      v
[Confirm Order]      [Handle Payment Authorization]
```

The team now needs to investigate what happens when payment is authorized but stock cannot be reserved.

Possible business decisions include:

- Void the payment authorization.
- Try another warehouse.
- Offer an alternative product.
- Keep the order pending.
- Notify the customer and cancel the order.

The correct decision depends on the business rules. Event Storming makes the decision visible instead of allowing developers to invent it.

### Discovering a Policy

The team also discovers that once stock is reserved, the order should be confirmed automatically.

This can be represented as:

```text
Stock Reserved
       |
       v
[When stock is reserved,
 confirm the order]
       |
       v
Confirm Order Command
       |
       v
Order Confirmed
```

This policy connects the Inventory and Sales responsibilities.

The team can now investigate whether it belongs in a simple event reaction or requires a longer-running process with explicit state and recovery behavior.

### Discovering Potential Context Boundaries

The workshop reveals that:

- Sales owns order confirmation.
- Payment owns payment authorization.
- Inventory owns stock reservation.
- Shipping owns transportation and delivery.

These are candidate Bounded Contexts because they have distinct responsibilities and business rules.

The team can investigate their relationships using Context Mapping.

For example, Sales may act as a downstream consumer of Inventory's reservation capability, while Inventory publishes a reservation result through a stable integration contract.

### What Did the Team Learn?

The workshop revealed:

1. Order placement and order confirmation are different business events.
2. Payment authorization and payment capture may represent different payment stages.
3. Stock reservation can fail after payment authorization.
4. The business needs an explicit decision for failed reservation.
5. Order confirmation depends on successful reservation.
6. Different contexts own different business decisions.
7. A policy connects the reservation result to order confirmation.

These discoveries can influence the Ubiquitous Language, domain model, Aggregate design, Domain Events, integration contracts, and tests.

The important outcome is not the diagram itself. It is the shared understanding developed while creating and challenging it.

## Event Storming and DDD

Event Storming is especially useful because its discoveries feed into other DDD activities.

### Ubiquitous Language

Event Storming exposes the language people use to describe business behavior.

For example, participants may initially use `Order Completed` to describe several different situations:

- The order was confirmed.
- The goods were packed.
- The shipment was dispatched.
- The customer received the goods.

Exploring the timeline helps distinguish these meanings.

The team may introduce more precise events:

```text
Order Confirmed
Packing Completed
Shipment Dispatched
Goods Delivered
```

These distinctions can then be reflected in the Ubiquitous Language and the domain model.

### Domain Events

Event Storming makes business events visible and helps identify which occurrences deserve explicit representation.

For example:

```text
Shipment Assigned
Loading Completed
Dispatch Authorized
Shipment Dispatched
```

These events may reveal meaningful changes in the business lifecycle.

However, not every sticky note automatically becomes a Domain Event in the implementation.

The team should evaluate whether the occurrence represents a meaningful business fact within the relevant Bounded Context.

### Aggregates

Commands and business rules can reveal where consistency boundaries may be required.

For example, the workshop may discover that an order must never be confirmed unless its required items have been reserved.

This raises questions:

- Which model owns the order confirmation invariant?
- Which model owns stock reservation?
- Can these decisions happen in one transaction?
- Is eventual coordination acceptable?
- What happens when one operation succeeds and another fails?

These discussions may lead to Aggregate design or a Process Manager.

But the sticky-note diagram should not be used to mechanically derive Aggregates.

### Bounded Contexts and Context Mapping

Event Storming can reveal where terminology, rules, responsibilities, and ownership differ.

For example, the Sales team may use `Order`, while Shipping uses `Shipment`.

The team can investigate whether these concepts belong to different models and how the contexts should communicate.

Context Mapping then makes the relationships, ownership, dependencies, and integration patterns explicit.

### CQRS and Read Models

Read Models can emerge when participants explain the information they need to perform activities.

For example:

- A dispatcher needs shipments awaiting dispatch.
- A warehouse operator needs items waiting to be packed.
- A finance officer needs payments awaiting reconciliation.

These information needs can help shape query models.

However, identifying a Read Model does not mean that the system automatically needs a separate database or a full CQRS architecture.

## Event Storming vs. Domain Storytelling

These techniques are complementary, but they start from different perspectives.

| Aspect | Domain Storytelling | Event Storming |
|---|---|---|
| Starting point | Stories told by people performing the work | Events that happen in the domain |
| Main focus | Actors, Work Objects, Activities, and sequence | Events, Commands, Policies, Actors, and decisions |
| Perspective | How people perform a particular activity | How the business changes over time |
| Useful for | Understanding real workflows and responsibilities | Discovering business behavior, rules, and dependencies |
| Typical output | Visual Domain Stories | Event timeline and related business elements |

For example, Domain Storytelling may reveal how a dispatcher assigns a vehicle and authorizes departure.

Event Storming can then explore the business events, commands, and policies surrounding those activities.

Neither technique replaces the other. Choose the format that best supports the questions being explored.

## Common Mistakes

### 1. Treating Event Storming as a Diagramming Exercise

The purpose is not to create an impressive wall of sticky notes.

The value comes from the conversations, questions, corrections, and discoveries.

A diagram without meaningful domain exploration is just a diagram.

### 2. Starting with Technical Architecture

Beginning with microservices, APIs, databases, and infrastructure can constrain the discussion before the business is understood.

Explore business behavior first. Discuss implementation when the workshop's scope calls for it.

### 3. Treating Every Sticky Note as a Domain Event

Not every activity or technical action is a Domain Event.

`Update Database`, `Call API`, and `Send HTTP Request` are usually implementation details, not meaningful business facts.

Distinguish between business events and technical operations.

### 4. Confusing Commands and Events

`Approve Refund` is a command.

`Refund Approved` is an event.

The command expresses an intention. The event represents a fact.

A command may be rejected, so do not assume that issuing a command means the corresponding event occurred.

### 5. Ignoring Exceptions

A happy-path-only model can hide the most important business rules.

Explore failures, cancellations, timeouts, rejected decisions, retries, and alternative paths.

### 6. Deriving Aggregates Mechanically

A cluster of events does not automatically define an Aggregate.

Aggregate boundaries should be designed around business invariants and consistency requirements.

Use Event Storming to discover candidate responsibilities and rules, then investigate them further.

### 7. Treating the First Timeline as the Truth

Different participants may describe the same process differently.

These disagreements should be explored, not erased.

The workshop should make uncertainty visible rather than manufacture agreement.

### 8. Trying to Solve Every Question in One Workshop

Some questions require further investigation, additional participants, or a separate modeling session.

Record open questions and assign follow-up work instead of allowing one unresolved detail to consume the entire workshop.

### 9. Confusing a Workshop Output with the Final Domain Model

Event Storming produces discoveries and hypotheses.

The actual domain model requires further refinement, validation, and implementation.

Do not mechanically translate every sticky note into a class, method, event, or service.

## Practical Event Storming Checklist

**Before the workshop**

- [ ] Define the purpose and scope.
- [ ] Choose the appropriate Event Storming format.
- [ ] Invite relevant domain experts and participants.
- [ ] Prepare a sufficiently large physical or digital workspace.
- [ ] Explain the purpose and basic notation.
- [ ] Establish how decisions and open questions will be recorded.

**During the workshop**

- [ ] Identify meaningful Domain Events.
- [ ] Arrange events in a plausible timeline.
- [ ] Explore Commands and Actors.
- [ ] Identify Policies and business reactions.
- [ ] Identify external systems and dependencies.
- [ ] Discover Read Models and information needs.
- [ ] Explore exceptions and alternative paths.
- [ ] Mark disagreements, assumptions, and hotspots.
- [ ] Identify candidate responsibilities and boundaries.
- [ ] Keep the discussion grounded in business language.

**After the workshop**

- [ ] Review the timeline with participants.
- [ ] Validate important events and terminology.
- [ ] Record business rules and decisions.
- [ ] Preserve unresolved questions.
- [ ] Identify follow-up investigations.
- [ ] Refine the Ubiquitous Language.
- [ ] Investigate candidate Aggregates and Bounded Contexts.
- [ ] Update relevant domain models and integration discussions.
- [ ] Share the results with participants and stakeholders.

## Further Reading

- **Introducing EventStorming**, Alberto Brandolini. The primary resource for understanding the method, its philosophy, and workshop formats.
- **EventStorming.com**, Alberto Brandolini. Practical resources and explanations of Event Storming.
- **Domain-Driven Design: Tackling Complexity in the Heart of Software**, Eric Evans. Foundational concepts for modeling complex domains.
- **Domain-Driven Design Distilled**, Vaughn Vernon. A concise introduction to strategic and tactical DDD.
- **Learning Domain-Driven Design**, Vlad Khononov. Practical guidance for domain exploration, strategic design, and implementation.

## Final Thought

Event Storming helps teams move from assumptions about how a business works to a shared understanding of what actually happens.

The most valuable moments often occur when someone challenges an event, questions the sequence, or explains an exception that nobody else knew about.

A missing event can reveal a missing business concept. A disputed event can expose conflicting assumptions. A policy can reveal an implicit business rule. A hotspot can identify a decision that the organization has never properly resolved.

**Do not focus on arranging sticky notes. Focus on discovering the business behavior they represent.**