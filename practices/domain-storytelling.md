# Domain Storytelling

Domain Storytelling is a collaborative technique for exploring a business domain through stories told by the people who perform the work.

Instead of beginning with requirements, process diagrams, database structures, or software features, participants describe how work actually happens. They tell stories about specific situations, explaining who does what, with which objects, and in what order.

A Domain Story makes these interactions visible. It helps developers, domain experts, analysts, and other stakeholders develop a shared understanding of business activities without immediately translating everything into software concepts.

For example, consider a logistics company that transports goods from one location to another.

A requirement might say:

> The system must support shipment dispatch.

But what does dispatch actually involve?

Does assigning a vehicle mean that a shipment has been dispatched? Must the goods be loaded first? Who authorizes departure? What happens if the driver changes? Does the dispatcher record the departure before or after the vehicle leaves?

A Domain Storytelling session allows the people who perform this work to explain the actual process, including the decisions and exceptions hidden behind a seemingly simple requirement.

The objective is not to document a process merely for documentation's sake. It is to understand the domain through the language and experiences of the people who know it.

## What Is a Domain Story?

A Domain Story is a visual representation of a concrete business scenario, told from the perspective of someone involved in the work.

It consists of several fundamental elements:

- **Actors:** People, roles, departments, or external parties that perform activities.
- **Work Objects:** Business objects that actors create, use, change, or exchange.
- **Activities:** Actions performed by actors on or with work objects.
- **Sequence:** The order in which activities occur.
- **Annotations:** Additional information such as business rules, decisions, conditions, questions, and exceptions.

A simple story might look like this:

```text
Dispatcher
    |
    | 1. Assigns
    v
Vehicle

Dispatcher
    |
    | 2. Assigns
    v
Driver

Warehouse Operator
    |
    | 3. Loads
    v
Shipment

Dispatcher
    |
    | 4. Authorizes
    v
Dispatch

Driver
    |
    | 5. Departs with
    v
Shipment
```

This is a simplified representation, not a complete process specification.

Its value comes from making the activities and their sequence visible so participants can discuss whether the story accurately reflects their work.

For example, a dispatcher might interrupt the story:

> We cannot authorize dispatch immediately after loading. The vehicle must first pass the required checks.

That correction is valuable domain knowledge. It reveals a missing activity and potentially an important business rule.

## The Building Blocks of Domain Storytelling

### 1. Actors

Actors represent the people or roles participating in the story.

Examples include:

- Customer
- Dispatcher
- Warehouse Operator
- Driver
- Accountant
- Sales Representative
- Supplier
- Insurance Agent
- External Payment Provider

An actor does not have to be an individual person. It can represent a business role, team, organization, or external system when that perspective is relevant.

Consider an insurance claim process:

```text
Customer
Claims Officer
Insurance Assessor
Repair Workshop
```

Each actor has different responsibilities and participates in different activities.

A Customer submits a claim, a Claims Officer reviews it, an Insurance Assessor evaluates the damage, and a Repair Workshop performs the approved repair.

The story makes these responsibilities visible.

**Important:** An actor in a Domain Story is not automatically an Entity, Aggregate, user account, or software component. It represents a participant in the business activity. The software model may represent that participant differently.

### 2. Work Objects

Work Objects are the things actors work with, create, update, or exchange.

Examples include:

- Shipment
- Waybill
- Invoice
- Payment
- Reservation
- Insurance Claim
- Purchase Order
- Delivery Receipt
- Stock Reservation

Consider a purchasing process:

```text
Purchasing Officer
    |
    | Creates
    v
Purchase Request
    |
    | Reviewed by
    v
Department Manager
    |
    | Approves
    v
Approved Purchase Request
    |
    | Used to create
    v
Purchase Order
```

Notice that a Purchase Request and a Purchase Order are not necessarily the same object.

A request expresses an intention to purchase. An order represents a commitment to a supplier.

If participants initially use the same word for both, the story can help reveal the distinction.

Work Objects may also change during the story. A draft invoice may become an approved invoice, or a shipment may become ready for dispatch.

These changes can reveal important business states and transitions.

**Important:** A Work Object is not automatically a database table or domain class. It is a concept used in the participants' work. Its implementation should be discovered later through modeling.

### 3. Activities

Activities describe what actors do with Work Objects.

Examples include:

- Creates an order.
- Reviews an application.
- Approves a payment.
- Assigns a vehicle.
- Reserves inventory.
- Authorizes dispatch.
- Records delivery.
- Issues a refund.

The wording of an activity matters.

Compare:

```text
Updates Shipment
```

with:

```text
Assigns Vehicle to Shipment
```

The first describes a generic change. The second expresses a business intention.

A useful Domain Story uses verbs and expressions that participants naturally use to describe their work.

Activities can also expose important distinctions.

For example, a warehouse operator might:

1. Pick goods.
2. Pack goods.
3. Complete packing.
4. Hand goods over to the carrier.

These may appear to be variations of a generic `ProcessOrder` activity in software, but they represent distinct steps in the actual business.

### 4. Sequence

The sequence shows how activities unfold.

A story is not just a collection of actors and objects. The order of activities helps explain dependencies, decisions, and business rules.

Consider a hotel reservation:

```text
Guest
  |
  | Requests
  v
Reservation

Receptionist
  |
  | Checks availability for
  v
Room

Receptionist
  |
  | Confirms
  v
Reservation

Guest
  |
  | Pays deposit for
  v
Reservation
```

The sequence raises questions:

- Is the room reserved while the guest is paying?
- Can payment happen before confirmation?
- What happens if payment fails?
- Does the reservation expire if the guest does not pay?
- Can the receptionist confirm a reservation without a deposit?

These questions help participants move beyond a happy-path description.

### 5. Decisions, Conditions, and Exceptions

Real business processes are rarely linear.

Domain Storytelling should make important decisions and alternative paths visible.

Consider a loan application:

```text
Customer
    |
    | Submits
    v
Loan Application
    |
    | Reviewed by
    v
Loan Officer
    |
    | Evaluates
    v
Credit Assessment
```

The assessment may lead to different outcomes:

```text
                  Credit Assessment
                         |
                  +------+------+
                  |             |
               Approved       Rejected
                  |             |
                  v             v
          Loan Offer       Rejection Notice
```

The participants may reveal additional rules:

- Applications above a certain amount require another approval.
- Missing documents put the application on hold.
- A customer can submit additional documents.
- A rejected application may be reviewed again under specific conditions.

These details should be captured when they matter to understanding the business.

Do not turn every minor technical condition into a part of the story. Focus on decisions and exceptions that have business meaning.

## Example 1: Understanding a Shipment Dispatch Process

Consider a logistics company that manages the transportation of goods.

The initial requirement is:

> The system must allow dispatchers to dispatch shipments.

The requirement appears straightforward, but it leaves many questions unanswered.

A Domain Storytelling session begins with a dispatcher describing a real shipment.

### The Initial Story

The dispatcher explains:

1. A shipment is created from a waybill.
2. A dispatcher assigns a vehicle.
3. A driver is assigned to the vehicle.
4. The warehouse operator loads the goods.
5. The dispatcher authorizes dispatch.
6. The driver leaves the warehouse.
7. The dispatcher records the departure.

The story can be represented as:

```text
Dispatcher
    |
    | Creates
    v
Shipment

Dispatcher
    |
    | Assigns
    v
Vehicle

Dispatcher
    |
    | Assigns
    v
Driver

Warehouse Operator
    |
    | Loads
    v
Shipment

Dispatcher
    |
    | Authorizes
    v
Dispatch

Driver
    |
    | Departs with
    v
Shipment

Dispatcher
    |
    | Records
    v
Departure
```

Even this initial story is more informative than the original requirement.

It reveals several actors, work objects, and activities.

### Exploring the Story

The facilitator asks:

> Can the dispatcher authorize dispatch immediately after assigning a vehicle?

The dispatcher answers:

> No. The goods must be loaded first, and the vehicle must pass the required checks.

This reveals a dependency that was missing from the initial understanding.

Another question:

> Is assigning a vehicle the same as dispatching a shipment?

The dispatcher explains:

> No. A vehicle can be assigned several hours before departure. The shipment is only dispatched when the vehicle actually leaves under the required authorization.

This is an important language discovery.

The team initially used `Assigned` and `Dispatched` almost interchangeably. The domain experts distinguish them.

The Ubiquitous Language can now express:

- `ShipmentAssigned`
- `LoadingCompleted`
- `DispatchAuthorized`
- `ShipmentDispatched`

### Exploring Exceptions

The facilitator asks:

> What happens if the assigned vehicle becomes unavailable after loading?

The dispatcher explains:

> We may need to assign another vehicle. Depending on the situation, the goods might need to be unloaded and loaded again.

Another question:

> What happens if the driver leaves without the required authorization?

The answer may reveal a compliance violation or an exceptional process that must be recorded.

The story now exposes more than the normal sequence. It reveals business constraints and exceptional scenarios.

### What Did the Team Learn?

The session may lead to several discoveries:

1. Assigning a vehicle and dispatching a shipment are different business activities.
2. Loading must be completed before dispatch authorization.
3. Vehicle and driver assignments can change.
4. Dispatch authorization is a meaningful business decision.
5. Actual departure is a distinct business event.
6. Exceptional situations require explicit handling.

These discoveries can influence the domain model, state transitions, business rules, Domain Events, and tests.

For example, the model may need operations such as:

```csharp
shipment.AssignVehicle(vehicleId, driverId);
shipment.CompleteLoading();
shipment.AuthorizeDispatch();
shipment.Dispatch();
```

The purpose of Domain Storytelling is not to derive this code mechanically. The story helps the team understand the business well enough to make informed modeling decisions.

## Example 2: Discovering Hidden Business Rules in an Insurance Claim

Consider an insurance company that wants to automate its claim management process.

The initial requirement is:

> Customers should be able to submit insurance claims online.

A Domain Storytelling session with a Claims Officer and Insurance Assessor reveals the actual workflow.

### The Story

1. A Customer reports an incident.
2. The Customer submits an Insurance Claim.
3. A Claims Officer checks the submitted documents.
4. If documents are missing, the Claims Officer requests additional documents.
5. The Customer submits the missing documents.
6. An Insurance Assessor evaluates the damage.
7. The Claims Officer reviews the assessment.
8. The company approves or rejects the claim.
9. If approved, the company calculates compensation.
10. The Customer receives the decision.

A simplified story:

```text
Customer
   |
   | Reports
   v
Incident
   |
   | Submits
   v
Insurance Claim
   |
   | Reviewed by
   v
Claims Officer
   |
   | Requests missing documents
   v
Customer
   |
   | Submits
   v
Additional Documents
   |
   | Evaluated by
   v
Insurance Assessor
   |
   | Produces
   v
Damage Assessment
   |
   | Reviewed by
   v
Claims Officer
   |
   | Approves or Rejects
   v
Claim Decision
```

### What the Story Reveals

The initial requirement suggested a simple submission workflow.

The actual story reveals:

- A claim may be incomplete.
- Document review and damage assessment are different activities.
- Different roles have different responsibilities.
- The claim can be waiting for customer action.
- Approval and compensation calculation are distinct decisions.

The team may discover that `ClaimStatus = Pending` is not sufficiently expressive.

Pending what?

- Waiting for documents?
- Waiting for an assessment?
- Waiting for approval?
- Waiting for a customer response?

The model may need more precise states or explicit business concepts.

The team can also identify questions requiring further exploration:

- Can an assessor request additional evidence?
- Can a claim be partially approved?
- Can the customer appeal a rejection?
- Who can authorize compensation above a certain amount?

These questions can become inputs to Example Mapping, domain workshops, and further modeling.

## Example 3: Understanding a Payment and Refund Process

Consider an online marketplace.

The initial assumption is that a refund is simply a payment with a negative amount.

A Domain Storytelling session with the finance team reveals a different process.

### The Story

1. A Customer requests a refund for an order.
2. A Support Agent checks the reason for the request.
3. The Support Agent determines whether the request is eligible.
4. If the request requires inspection, the Customer returns the goods.
5. A Warehouse Operator receives and inspects the returned goods.
6. The Support Agent approves or rejects the refund.
7. The Payment Team initiates the refund.
8. The Payment Provider processes the refund.
9. The Customer receives the refunded amount.

The story reveals that a refund is not simply the opposite of a payment.

It has its own workflow, actors, rules, and lifecycle.

For example:

- A refund may be requested but not approved.
- An approved refund may still be processing.
- A payment provider may reject a refund.
- A partial refund may be issued.
- The refund may be completed after the original order has been closed.

The team may discover that `Refund` deserves an explicit domain concept rather than being represented as a negative `Payment`.

This discovery can influence the model, integration contracts, and business events.

## Example 4: Discovering Differences Between Roles

Domain Storytelling can also reveal that people describe the same process differently because they have different responsibilities.

Consider an order fulfillment process.

The Sales team says:

> We complete the order when the customer confirms it.

The Warehouse team says:

> We complete the order when all items are packed.

The Shipping team says:

> We complete the order when the goods are delivered.

All three statements may be correct within their respective contexts.

The problem is the overloaded term `Complete`.

A Domain Storytelling session with representatives from all three teams can reveal that they are describing different business outcomes:

| Context | Meaning of completion |
|---|---|
| Sales | Order confirmed |
| Warehouse | Packing completed |
| Shipping | Delivery completed |

This is a valuable discovery for Strategic Design.

The team should not necessarily force all three contexts to use one universal `OrderCompleted` concept.

Instead, each context can express its own meaningful language and publish relevant facts to other contexts.

For example:

```text
Sales:
  OrderConfirmed

Warehouse:
  PackingCompleted

Shipping:
  ShipmentDelivered
```

This illustrates how Domain Storytelling can contribute to discovering language differences and potential Bounded Context boundaries.

## How to Conduct a Domain Storytelling Session

Domain Storytelling is most effective when participants explore concrete situations together rather than attempting to produce a complete process specification in one meeting.

A practical session can follow these steps.

### 1. Define the Scope

Start with a clear question.

Examples:

- How does a shipment move from creation to dispatch?
- How does a customer request and receive a refund?
- How does a loan application get approved?
- How does a warehouse handle returned goods?
- How does an organization process a purchase request?

Avoid starting with an overly broad question such as:

> How does the entire company work?

A focused scope makes it easier to tell a meaningful story.

### 2. Invite the Right Participants

Include people who actually perform or understand the work.

Depending on the topic, participants may include:

- Domain experts.
- Operational employees.
- Business analysts.
- Developers.
- Testers.
- Product owners.
- Representatives of related departments.

Do not rely exclusively on managers or documentation when the people performing the work can explain the actual process.

For example, a warehouse manager may describe the official process, while a warehouse operator can explain how exceptions are handled during a busy shift.

Both perspectives can be valuable.

### 3. Ask for a Concrete Story

Ask a participant to describe a real or realistic example.

For instance:

> Think about the last shipment you dispatched. Can you walk us through what happened, from the moment the shipment was ready until the vehicle left?

This is more useful than asking:

> How does dispatch work in general?

A concrete story encourages participants to mention actual decisions, objects, terminology, and exceptions.

### 4. Capture Actors, Work Objects, and Activities

As the participant tells the story, capture:

- Who performs each activity?
- What object do they work with?
- What exactly do they do?
- What happens next?

Use the participants' own terminology.

If a dispatcher says `Release Shipment`, do not immediately replace it with `Dispatch` because that is the term used in the current application.

First, understand what `Release` means in their work.

### 5. Visualize the Sequence

Represent the activities in their actual order.

Keep the story visible so participants can correct it.

For example:

```text
Customer
   |
   | Places
   v
Order
   |
   | Checked by
   v
Sales Representative
   |
   | Confirms
   v
Confirmed Order
   |
   | Sent to
   v
Warehouse
   |
   | Picks and packs
   v
Packed Order
   |
   | Handed to
   v
Carrier
```

The diagram is a conversation aid. It should not become a substitute for the conversation.

### 6. Ask Questions and Challenge Assumptions

Explore the details that could change the understanding of the domain.

Useful questions include:

- Why does this activity happen?
- Who is responsible for it?
- What must be true before it can happen?
- What information is needed?
- What happens if the activity fails?
- Can the activity be repeated?
- Can it be reversed?
- Who makes the decision?
- Are there alternative paths?
- Does everyone use this term in the same way?

For example:

> Can a shipment be dispatched without a completed loading record?

The answer may reveal a business rule, an exception, or a difference between official procedures and actual practice.

### 7. Explore Alternative Stories

Do not stop after documenting the happy path.

Ask participants to describe variations:

- What happens when something goes wrong?
- What happens when a required document is missing?
- What happens when an actor is unavailable?
- What happens when an activity is repeated?
- What happens when two participants disagree?
- What happens when the process is interrupted?

For example, in a refund process, explore:

- Full refund.
- Partial refund.
- Refund after goods inspection.
- Refund without returning the goods.
- Refund rejected by the payment provider.

Alternative stories often reveal more important business rules than the normal path.

### 8. Validate the Story

Read the story back to the participants.

Ask:

- Does this represent what actually happens?
- Is anything missing?
- Are the activities in the correct order?
- Are the terms accurate?
- Are there situations where this story is different?
- Do other people involved in the process describe it differently?

Validation matters because a diagram can look convincing while still containing incorrect assumptions.

A Domain Story is only useful if the people who perform the work recognize it as a reasonable representation of their experience.

### 9. Record Questions and Uncertainties

Not every question needs to be resolved during the session.

Record:

- Disagreements.
- Ambiguous terms.
- Unverified assumptions.
- Missing business rules.
- Exceptional scenarios.
- Questions requiring another participant.
- Decisions that need further investigation.

For example:

```text
Open Questions

1. Can dispatch authorization be revoked?
2. Who is allowed to replace an assigned driver?
3. Does vehicle inspection happen before or after loading?
4. Can a shipment be partially dispatched?
5. What happens if the vehicle leaves but departure
   has not been recorded?
```

These questions become inputs to subsequent exploration and modeling.

## Domain Storytelling and Ubiquitous Language

Domain Storytelling is particularly valuable for discovering and refining Ubiquitous Language.

Instead of asking domain experts to define terms in isolation, it allows the team to discover how words are used in actual business situations.

Consider the word `Approved`.

In a loan application process, participants may use it to describe:

- Documents being accepted.
- A credit assessment being accepted.
- A loan being authorized.
- A loan offer being accepted by the customer.

These are different meanings.

By walking through stories, the team can discover the distinctions and identify the appropriate terminology.

For each important term, ask:

- Who uses this expression?
- In which situation?
- What does it mean?
- What happens before and after it?
- Is there another term for the same concept?
- Does the same word mean something different to another team?

The resulting language should be reflected in the model and implementation where appropriate.

For example:

```csharp
loanApplication.Submit();
creditAssessment.Approve();
loan.Authorize();
loanOffer.Accept();
```

These operations express different business intentions.

Domain Storytelling therefore contributes to Ubiquitous Language through concrete examples, not merely through the creation of a glossary.

## Domain Storytelling and Other DDD Techniques

Domain Storytelling is complementary to other domain exploration and modeling techniques.

| Technique | Main focus | How it complements Domain Storytelling |
|---|---|---|
| Event Storming | Business events, commands, policies, and process flow | Helps explore the timeline and consequences of business occurrences |
| Example Mapping | Rules, examples, and questions around a requirement | Helps clarify conditions and acceptance scenarios |
| Event Modeling | Workflows, events, commands, and read models | Helps explore how business processes and information flows can be represented |
| Domain-Driven Design Interviews | Understanding domain concepts and expertise | Provides deeper context behind the stories |
| Context Mapping | Relationships between Bounded Contexts | Helps represent the boundaries and dependencies discovered during exploration |
| Ubiquitous Language | Shared, context-specific terminology | Captures the language and meanings discovered in the stories |

For example, a Domain Story about shipment dispatch may reveal that loading must be completed before dispatch authorization.

The team can then use Example Mapping to clarify the exact rules and exceptions.

An Event Storming session may explore the resulting business events:

```text
ShipmentCreated
VehicleAssigned
LoadingCompleted
DispatchAuthorized
ShipmentDispatched
```

The team can then investigate which Bounded Context owns each concept and how the contexts communicate.

These techniques do not replace one another. Each provides a different perspective on the domain.

## Common Mistakes

### 1. Starting with Software Features

Asking participants to describe what the application should do can lead to discussions about screens, buttons, and existing system behavior.

Instead, ask them to describe the actual business work.

The goal is to understand the domain before designing its software representation.

### 2. Documenting Only the Happy Path

A story that describes only the ideal process can hide important business rules.

For example, a refund process that only describes successful refunds says little about eligibility, inspection, rejection, partial refunds, or provider failures.

Explore variations and exceptions deliberately.

### 3. Using Technical Language Too Early

Replacing business terminology with classes, APIs, databases, or services too early can distort the conversation.

If a participant says `Release Shipment`, first understand what that expression means.

Do not immediately translate it into `UpdateShipmentStatus()`.

### 4. Assuming One Story Represents the Entire Domain

Different people may experience the same process differently.

A Sales Representative, Warehouse Operator, and Dispatcher may describe different parts of order fulfillment.

These stories are not necessarily contradictory. They may reveal different responsibilities or context-specific meanings.

Compare stories rather than forcing them into one universal process.

### 5. Confusing Actors with Software Components

A dispatcher is a business actor, not automatically a `DispatcherService`.

A warehouse operator is a participant in the work, not automatically a microservice.

The story describes the domain. Software responsibilities should be designed after understanding it.

### 6. Turning Every Detail into a Business Rule

Not every activity or variation deserves a complex domain model.

Distinguish between meaningful business decisions and incidental operational details.

For example, whether a warehouse operator uses a tablet or a printed form may not affect the domain model. Whether a shipment can leave without authorization probably does.

### 7. Treating the Diagram as the Final Model

A Domain Story is an exploration artifact, not a complete domain model or implementation specification.

It can reveal candidate concepts, rules, events, and boundaries, but these need further investigation and validation.

Do not mechanically turn every noun into an Entity or every activity into a Domain Service.

## Practical Domain Storytelling Checklist

Before the session:

- [ ] Define a focused business question.
- [ ] Identify the process or scenario to explore.
- [ ] Invite people who perform or understand the work.
- [ ] Prepare a shared visual workspace.
- [ ] Agree on the scope and purpose of the session.

During the session:

- [ ] Ask participants to tell a concrete story.
- [ ] Identify actors and their responsibilities.
- [ ] Capture Work Objects and activities.
- [ ] Preserve the sequence of actions.
- [ ] Use the participants' own terminology.
- [ ] Explore decisions, conditions, and exceptions.
- [ ] Ask questions about ambiguous terms.
- [ ] Record disagreements and assumptions.
- [ ] Compare alternative scenarios where relevant.

After the session:

- [ ] Validate the story with participants.
- [ ] Clarify unresolved questions.
- [ ] Identify important domain terms.
- [ ] Record discovered business rules.
- [ ] Identify potential Domain Events and concepts for further exploration.
- [ ] Compare stories from different roles or contexts.
- [ ] Feed relevant discoveries into modeling and implementation discussions.

## Final Thought

A business process is not merely a sequence of steps. It is a sequence of decisions and actions performed by people who understand the meaning of their work.

Domain Storytelling gives those people a way to make that understanding visible and share it with the team building the software.

The most valuable discoveries often happen when someone interrupts the story:

> Wait, that is not what happens in practice.

That interruption may expose an overlooked rule, an ambiguous term, a missing activity, or a fundamental misunderstanding of the domain.

**Listen to the people who do the work. Let them tell the story. Then build a model that expresses what the story reveals.**