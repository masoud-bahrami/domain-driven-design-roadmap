# 02. Domain Exploration

A domain model should not begin with a developer's assumptions about how a business works. It should emerge from a serious attempt to understand the business itself.

That understanding rarely comes from reading requirements documents, inspecting database schemas, interviewing one stakeholder, or studying an existing application in isolation.

Business knowledge is distributed across people, conversations, decisions, policies, documents, operational practices, and software systems. Some of it is explicit. Much of it is implicit. Some of it is contradictory, outdated, or known only by the people who deal with particular situations every day.

Domain exploration is the work of discovering, challenging, organizing, and validating that knowledge.

In Domain-Driven Design, this is not a preliminary analysis phase that ends when development begins. It is a continuous collaborative activity. As the team learns more about the domain, its understanding changes, its language becomes more precise, and its model evolves.

This chapter introduces practical techniques for making that knowledge visible and turning it into a shared understanding that can guide software design.

## Learning objectives

By the end of this chapter, you should be able to:

- Explain why domain exploration must involve both domain experts and software practitioners.
- Discover business processes through concrete scenarios rather than assumptions.
- Understand the purpose and formats of Event Storming.
- Use Domain Storytelling to capture actors, activities, and work objects.
- Use Example Mapping to clarify business rules and acceptance criteria.
- Choose an appropriate collaborative modeling technique for a particular problem.
- Facilitate workshops that encourage participation and expose disagreements.
- Distinguish facts, assumptions, policies, decisions, and unanswered questions.
- Turn workshop findings into useful models, decisions, and follow-up activities.

**The objective is not to produce the largest diagram or the most colorful workshop board. It is to understand the domain well enough to make better decisions about the software.**

---

## 1. Why domain exploration matters

Imagine a team building software for a logistics company.

The initial requirement sounds straightforward:

"Allow customers to create a shipment and track its delivery."

A developer might immediately imagine:

- A Shipment entity.
- A Customer entity.
- A Status property.
- A database table.
- A tracking API.
- A few endpoints for creating and updating shipments.

But the business process may be considerably more complicated.

For example:

- A customer requests a shipment.
- The company checks whether the shipment is eligible for the selected service.
- A shipment may require pricing approval.
- A carrier may accept or reject the assignment.
- A driver may collect the package.
- The package may pass through several facilities.
- A delivery attempt may fail.
- The recipient may refuse the package.
- The shipment may be returned to its origin.
- A shipment can contain multiple packages.
- Some shipment types require additional documentation.

Even this description leaves important questions unanswered.

What does "shipment created" mean? Is it created when the customer submits a request, when the company accepts it, or when a carrier agrees to transport it?

Does a failed delivery attempt mean the shipment has failed? Can it be attempted again? Who decides?

What happens when a shipment is partially delivered?

The team cannot answer these questions reliably by looking at a database schema or guessing what the business probably does.

It needs to explore the actual process with the people who understand it.

### 1.1 Knowledge is distributed

Different people understand different parts of the same business.

A customer support representative may know the exceptions customers encounter. A warehouse operator may understand the physical handling process. A finance specialist may know the rules for pricing and settlement. A product manager may understand commercial priorities. A developer may know how existing systems constrain the process.

None of these perspectives is necessarily complete.

A useful model emerges when these perspectives are brought together and examined.

### 1.2 Requirements are interpretations

Requirements documents are useful, but they are representations of what someone currently believes the software should do.

They may omit exceptions, contain ambiguous terms, or encode assumptions that have never been validated.

Consider the requirement:

"The system must prevent orders from being canceled after processing."

What does processing mean?

- Payment authorization?
- Payment capture?
- Warehouse picking?
- Shipment preparation?
- Carrier collection?

The requirement appears precise but contains an unresolved business concept.

Domain exploration helps reveal these ambiguities before they become implementation decisions.

### 1.3 Existing software is not necessarily the source of truth

An existing application can provide valuable evidence about how a business currently operates.

However, its behavior may reflect historical workarounds, technical limitations, outdated policies, or misunderstood requirements.

Do not assume that every existing database field or status represents a valid business concept.

Compare three things:

1. What the business says should happen.
2. What people actually do.
3. What the software currently implements.

The differences between these three views are often among the most valuable discoveries in domain exploration.

### 1.4 Exploration is not requirements collection

Traditional requirements collection often asks:

"What does the system need to do?"

Domain exploration asks deeper questions:

- What is happening in the business?
- Why does it happen?
- Who makes the decision?
- What information is needed?
- Which rules govern the decision?
- What happens when the expected situation does not occur?
- What does a particular term mean to the people using it?
- Where do different participants disagree?

The purpose is to understand the problem, not merely collect a list of requested features.

---

## 2. Collaborative software design

Collaborative software design is the practice of involving the people who hold relevant knowledge in the process of understanding and making software design decisions.

It is closely aligned with DDD because a useful domain model cannot be developed in isolation from the people who understand the domain.

Developers bring technical knowledge. Domain experts bring business knowledge. Product and design roles contribute user needs, priorities, and operational perspectives.

The model benefits from the interaction of these different kinds of knowledge.

### 2.1 Collaboration is more than interviewing

An interview is useful for collecting information, but collaborative modeling goes further.

In an interview:

- One person asks questions.
- Another person provides answers.
- The interviewer interprets and documents the information.

In a collaborative modeling session:

- Participants contribute their knowledge directly.
- The group creates a shared representation.
- Different interpretations become visible.
- Participants challenge and refine the model together.
- Decisions and unanswered questions are identified collectively.

The distinction matters.

People often recognize problems in a model more easily when they can see it, discuss it, and modify it themselves.

### 2.2 Who should participate?

The participants depend on the problem being explored.

A useful group may include:

- Domain experts who understand the business process.
- People who perform the actual work.
- Product managers or product owners.
- Business analysts.
- Developers and architects.
- Testers or quality engineers.
- UX researchers and designers.
- Operational or support representatives.
- External partners when their responsibilities affect the process.

Do not invite people simply because of their job titles. Invite them because they possess relevant knowledge or authority.

A senior manager may understand the business strategy but not the operational exceptions. An experienced operator may know exactly how those exceptions are handled but have little influence over formal decisions.

Both perspectives can be important.

### 2.3 Collaboration requires psychological safety

Participants must feel comfortable saying:

- "I don't know."
- "We do not always follow that process."
- "The documentation is wrong."
- "Different teams handle this differently."
- "That rule is only an assumption."
- "I disagree with that interpretation."

If people feel that disagreement will be punished, the workshop may produce apparent agreement while concealing important problems.

A facilitator should make it clear that the purpose is to understand the domain, not to defend existing processes or prove that a particular solution is correct.

### 2.4 The facilitator's responsibility

A facilitator guides the collaboration without becoming the owner of the domain knowledge or dictating the model.

Responsibilities include:

- Defining the purpose and scope.
- Ensuring the right people participate.
- Explaining the modeling technique.
- Encouraging balanced participation.
- Asking clarifying questions.
- Making assumptions and disagreements visible.
- Preventing premature technical discussions.
- Helping the group identify decisions and follow-up work.

A facilitator does not need to know every answer. In fact, pretending to know the answers can prevent the group from discovering them.

### 2.5 Collaborative design is a skill

Having a whiteboard and a collection of sticky notes does not automatically create collaboration.

Workshops can be affected by organizational hierarchy, confirmation bias, groupthink, dominant personalities, and unequal access to information.

A good facilitator pays attention to these dynamics.

For example, if a manager answers every question before the people doing the work have a chance to contribute, the group may miss important operational knowledge.

Sometimes the most useful question is not another technical question but:

"Who has a different experience of this process?"

**Recommended reading**

[Collaborative Software Design: How to Facilitate Domain Modeling Decisions](https://www.manning.com/books/collaborative-software-design) — Evelyn van Kelle, Gien Verschatse, and Kenny Baas-Schwegler.

This book explores collaborative modeling, facilitation, decision-making, cognitive biases, organizational dynamics, resistance, conflict resolution, and follow-up practices. It is particularly valuable for people who regularly facilitate modeling and design sessions.

---

## 3. Event Storming

Event Storming is a collaborative modeling technique introduced by Alberto Brandolini.

It helps groups explore complex business domains by making business events visible and arranging them into a shared representation.

The central idea is simple:

**Start by discovering what happens in the business, then explore the decisions, interactions, rules, and responsibilities behind those events.**

Instead of beginning with classes, database tables, services, or requirements, participants identify meaningful things that have happened.

For example, in an online retail business:

- Product Added to Cart.
- Order Placed.
- Payment Authorized.
- Order Confirmed.
- Inventory Reserved.
- Shipment Prepared.
- Shipment Dispatched.
- Delivery Completed.

These events help the group reconstruct the business process and discover where important decisions occur.

### 3.1 What is a Domain Event in Event Storming?

An event represents something that has happened and is meaningful to the business.

Events are usually written in the past tense.

Examples:

- `OrderPlaced`
- `PaymentAuthorized`
- `InventoryReserved`
- `ShipmentDispatched`

An event should describe a business occurrence rather than a technical implementation detail.

For example:

- `DatabaseRecordInserted` is a technical event.
- `OrderPlaced` is a business event.

The first describes an implementation operation. The second describes something meaningful in the domain.

### 3.2 Why start with events?

Events provide a useful way to reconstruct the business from observable outcomes.

People may disagree about how a process should work, but they can often describe specific things that have happened.

Starting with events encourages participants to share concrete experiences before debating the design.

It also helps reveal:

- Missing steps.
- Different paths through a process.
- Delays and bottlenecks.
- Business exceptions.
- Dependencies between departments.
- Unclear responsibilities.
- Important decisions.
- Differences between the intended and actual process.

### 3.3 Event Storming formats

Event Storming is not a single workshop format. Different formats support different levels of exploration.

Three widely used formats are Big Picture, Process Modeling, and Software Design.

#### 3.3.1 Big Picture Event Storming

Big Picture Event Storming explores a broad business domain or an important end-to-end business journey.

It aims to establish a shared overview of what happens across the organization.

Participants identify significant events and arrange them into a timeline.

The purpose is not to design every detail. It is to understand the overall landscape and discover where further exploration is needed.

Typical discoveries include:

- Major business activities.
- Different perspectives on the same process.
- Organizational hand-offs.
- Dependencies.
- Delays.
- Exceptions.
- Areas of uncertainty.
- Possible domain boundaries.

For example, a retail business might explore the journey from product discovery to order fulfillment and after-sales support.

The group may discover that "order completion" means different things to Sales, Finance, and Fulfillment.

That discovery is more valuable than producing a perfect timeline.

**When to use it:**

- You do not yet understand the overall domain.
- Multiple departments participate in a business process.
- The organization has conflicting views of how the business works.
- You need to identify areas that deserve deeper investigation.
- You want to explore possible Subdomains and Bounded Contexts.

Big Picture is an exploration tool, not a finished architecture or a definitive Context Map.

#### 3.3.2 Process Modeling Event Storming

Process Modeling focuses on a specific business process and explores its behavior in greater detail.

Once the group has identified an interesting area, it can examine the sequence of events and the decisions that lead to them.

A typical flow may include:

- Commands: requests or intentions to perform an action.
- Actors: people, roles, or systems initiating actions.
- Events: facts that have occurred.
- Policies: reactions or business rules triggered by events.
- Read Models: information needed to make decisions.
- External Systems: dependencies outside the process.
- Hot Spots: disagreements, uncertainties, or problematic areas.

Consider an order placement process:

1. A customer submits an order.
2. The system validates the order.
3. The order is accepted.
4. Payment authorization is requested.
5. Payment is authorized or rejected.
6. The order is confirmed or rejected according to the relevant rules.
7. Inventory reservation is initiated.

The group then explores alternative paths:

- What if the payment is rejected?
- What if inventory is unavailable?
- What if the customer submits the same request twice?
- What if the payment provider responds late?
- What if the order is canceled while payment is being processed?

The objective is to understand the process, including the situations that do not follow the expected path.

**When to use it:**

- A specific business process is poorly understood.
- Business rules and exceptions need clarification.
- Participants need to understand how decisions lead to events.
- You need to discover interactions between different parts of the process.
- You want to identify candidate models and responsibilities.

#### 3.3.3 Software Design Event Storming

Software Design Event Storming explores how software should support a discovered business process.

It moves closer to the design of the software model.

Participants may explore:

- Commands and their handlers.
- Aggregates and their responsibilities.
- Domain Events.
- Policies and process coordination.
- Read Models.
- External integrations.
- Potential Bounded Context boundaries.
- Consistency requirements.

For example, after exploring an order placement process, the group may ask:

- Which Aggregate is responsible for confirming an order?
- Which invariants must be protected?
- Which context owns payment authorization?
- Should inventory reservation happen synchronously?
- What should happen if a downstream process fails?
- Which events need to be communicated to other contexts?

These are design questions, not merely process discovery questions.

They should be informed by the domain knowledge discovered earlier.

**When to use it:**

- The business process is sufficiently understood.
- The team needs to explore a software design.
- Domain responsibilities and boundaries need clarification.
- The group wants to connect business behavior to implementation decisions.

Software Design Event Storming should not be used as an excuse to jump into architecture before the business behavior is understood.

### 3.4 Common Event Storming elements

Event Storming uses colored sticky notes as a lightweight visual language.

The exact notation can vary between formats and facilitators. The following are commonly used conventions.

| Element | Common color | Meaning |
|---|---|---|
| Domain Event | Orange | Something meaningful that happened |
| Command | Blue | An intention or request to perform an action |
| Actor | Yellow | A person, role, or system initiating an action |
| Policy | Lilac / Purple | A reaction or rule triggered by an event |
| Read Model | Green | Information used to make a decision |
| Aggregate | Often yellow or a distinct marker | A model responsible for handling a command and protecting consistency |
| External System | Often pink or a distinct marker | A system outside the explored boundary |
| Hot Spot | Pink | A question, conflict, risk, or uncertainty |

These colors are conventions, not the essence of the technique. Agree on the notation before the workshop and avoid introducing unnecessary symbols.

### 3.5 A simple Event Storming example

Consider a customer placing an order.

A simplified Process Modeling representation might look like this:

```text
Customer
   |
   v
[Place Order]
   |
   v
(Order Placed)
   |
   v
[Authorize Payment]
   |
   +--------------------------+
   |                          |
   v                          v
(Payment Authorized)    (Payment Rejected)
   |                          |
   v                          v
[Confirm Order]          [Reject Order]
   |                          |
   v                          v
(Order Confirmed)        (Order Rejected)
   |
   v
[Reserve Inventory]
   |
   v
(Inventory Reserved)
```

This is only a simplified illustration.

The workshop should explore the actual business rules, possible outcomes, timing, responsibilities, and exceptions.

For example:

- Is payment authorized before or after inventory is reserved?
- Can inventory be reserved temporarily?
- What happens if payment succeeds but inventory reservation fails?
- Can an order be confirmed before payment is captured?
- What happens if the payment provider does not respond?

The diagram does not answer these questions. It makes them visible.

### 3.6 How to facilitate a basic Event Storming session

**Before the workshop**

1. Define the business area or process to explore.
2. Identify the people with relevant knowledge.
3. Invite participants from different perspectives.
4. Prepare a sufficiently large modeling surface.
5. Provide sticky notes and markers, or prepare an equivalent digital board.
6. Explain the purpose and expected outcome.
7. Agree on a facilitator and a way to record unresolved questions.

Do not begin by asking participants to design a microservice architecture. The purpose is to discover the business.

**During the workshop**

**Step 1: Establish the scope**

Explain which business process is being explored.

For example:

"How does an order move from customer submission to confirmed fulfillment?"

Make sure everyone understands the scope.

**Step 2: Discover events**

Ask participants to write down meaningful events that have happened in the business.

Encourage individual contributions before group discussion.

Use past-tense business language.

**Step 3: Build the timeline**

Arrange events in their approximate chronological order.

Allow participants to move events and challenge the sequence.

Do not force agreement prematurely.

**Step 4: Explore alternative paths**

Ask what happens when the expected process does not occur.

Explore cancellations, rejections, failures, exceptions, retries, and manual interventions.

**Step 5: Identify decisions and responsibilities**

Introduce commands, actors, policies, and read models where they help explain the process.

Do not introduce every notation element just because it exists.

**Step 6: Identify hot spots**

Mark unresolved questions, contradictions, risks, and areas requiring further investigation.

Do not hide uncertainty behind an apparently complete diagram.

**Step 7: Summarize discoveries**

Ask the participants to explain the process using the model.

Record important terms, decisions, assumptions, and open questions.

**After the workshop**

- Review the model with participants.
- Validate important details with the people responsible for the process.
- Assign owners to unresolved questions.
- Identify areas that need deeper exploration.
- Preserve the artifacts and their context.
- Decide whether another workshop or a different technique is needed.

A workshop that ends with a photograph of sticky notes but no follow-up is rarely enough.

### 3.7 Common Event Storming mistakes

**Starting with solutions**

If participants begin discussing databases, microservices, frameworks, or APIs before understanding the process, the workshop can become an architecture meeting rather than a domain exploration.

**Treating the first timeline as the truth**

Different participants may have different experiences. The timeline is a representation of the group's current understanding, not automatically an authoritative description.

**Ignoring exceptions**

Happy paths are easy to model. Business complexity often lives in rejected, delayed, reversed, or exceptional cases.

**Overcomplicating the notation**

Too many colors and symbols can make participation difficult. Introduce notation gradually.

**Letting one person dominate**

The goal is to make distributed knowledge visible. A facilitator should encourage quieter participants and people with direct operational experience to contribute.

**Confusing events with commands**

"Approve Refund" is a command or intention. "Refund Approved" is an event. The distinction matters because a command may fail, while an event describes something that happened.

**Treating the output as architecture**

An Event Storming board can reveal possible boundaries and responsibilities, but those observations still need to be analyzed and validated before becoming architectural decisions.

**Further resources**

- [EventStorming - Official Website](https://www.eventstorming.com/)
- [Introducing EventStorming - Alberto Brandolini](https://www.eventstorming.com/book/)
- [EventStorming Resources](https://www.eventstorming.com/resources/)
- [EventStorming Patterns](https://www.eventstorming.com/patterns/)

The official resources include books, articles, videos, starter kits, and templates for different Event Storming formats.

---

## 4. Domain Storytelling

Domain Storytelling is a collaborative modeling technique developed by Stefan Hofer and Henning Schwentner.

It captures domain knowledge through concrete stories told by people who participate in the business.

A domain story describes:

- Who is involved.
- What they do.
- Which objects or information they work with.
- In what order activities happen.
- How responsibilities and interactions are connected.

It uses a simple visual language to represent these stories.

Unlike a conventional process diagram, Domain Storytelling emphasizes the language and perspective of the people who perform the work.

### 4.1 Why storytelling?

People often explain their work through examples.

A warehouse employee might say:

"First, I receive the delivery note from the supplier. Then I check the quantities against the purchase order. If everything matches, I register the received goods and move them to the storage area."

This short story contains important domain information:

- An actor: warehouse employee.
- Work objects: delivery note, purchase order, received goods.
- Activities: receive, check, register, move.
- A decision: whether the quantities match.
- A possible exception: a mismatch.

A Domain Story makes these relationships visible.

### 4.2 The basic notation

Domain Storytelling uses a small set of visual building blocks.

#### Actors

Actors are people, groups, or systems that actively participate in the story.

Examples:

- Customer.
- Warehouse Employee.
- Supplier.
- Payment Provider.
- Inventory System.

An actor performs an activity or initiates an interaction.

#### Work Objects

Work Objects are the things actors create, use, exchange, or modify.

Examples:

- Purchase Order.
- Delivery Note.
- Shipment.
- Invoice.
- Payment Confirmation.
- Customer Request.

Work Objects are labeled using terms from the domain language.

#### Activities

Activities describe what an actor does with or to a Work Object.

They are usually represented as arrows labeled with verbs.

Examples:

- Creates Purchase Order.
- Sends Delivery Note.
- Checks Received Goods.
- Approves Refund.
- Assigns Shipment.

The labels should reflect the language used by the people who understand the process.

#### Sequence Numbers

Activities are numbered to show the order in which the story unfolds.

This makes it possible to reconstruct the scenario as a sequence of meaningful actions.

#### Annotations

Annotations provide additional information about:

- Business rules.
- Exceptions.
- Alternative paths.
- Conditions.
- Assumptions.
- Domain terminology.
- Important observations.

#### Groups

Groups can be used to organize parts of a story that belong together.

For example, they may represent an organizational area, a location, an optional sequence, or a repeated activity.

### 4.3 A simple Domain Storytelling example

Consider the process of receiving goods from a supplier.

A domain expert might describe it as follows:

1. The supplier sends a delivery notification to the warehouse.
2. The warehouse employee retrieves the purchase order.
3. The employee compares the delivered quantities with the ordered quantities.
4. If they match, the employee registers the received goods.
5. If they do not match, the employee records the discrepancy.
6. The purchasing department reviews the discrepancy.
7. The supplier may be contacted for clarification.

The story can be represented with actors, work objects, and activities:

```text
Supplier
   |
   | 1. Sends
   v
Delivery Notification
   |
   v
Warehouse Employee
   |
   | 2. Retrieves
   v
Purchase Order
   |
   | 3. Compares
   v
Delivered Goods
   |
   +-----------------------+
   |                       |
   v                       v
Quantities Match      Discrepancy Found
   |                       |
   v                       v
Register Goods        Record Discrepancy
                           |
                           v
                    Purchasing Department
                           |
                           | Reviews
                           v
                      Discrepancy
```

This is a conceptual text representation, not a complete Domain Storytelling diagram. A proper story would use the pictographic notation and sequence numbers to make the actors, objects, and activities explicit.

### 4.4 How to facilitate a Domain Storytelling session

**Step 1: Choose a concrete scenario**

Avoid starting with a broad question such as:

"How does the warehouse work?"

Instead ask:

"Can you walk us through what happened the last time a supplier delivered goods?"

A concrete scenario helps participants explain actual work rather than idealized processes.

**Step 2: Let the domain expert tell the story**

Ask the person who performs or understands the process to describe it in their own words.

Do not immediately translate their terminology into technical concepts.

**Step 3: Identify actors and work objects**

Capture who participates and which objects or information they work with.

Use the terms the domain expert uses.

**Step 4: Identify activities**

Represent the actions connecting actors and work objects.

Prefer meaningful verbs such as "approves", "assigns", "reconciles", and "dispatches" over vague words such as "processes".

**Step 5: Establish the sequence**

Number the activities and reconstruct the scenario.

Ask whether anything has been omitted or placed in the wrong order.

**Step 6: Explore variations**

Ask what happens in different situations:

- What if the quantities do not match?
- What if the delivery is late?
- What if the purchase order has been canceled?
- What if the responsible employee is unavailable?
- What if the supplier sends incomplete documentation?

Capture these variations rather than forcing them into the main story.

**Step 7: Validate the story**

Ask the domain expert to walk through the diagram and explain it in their own words.

If the diagram cannot be explained naturally by the person whose work it represents, revisit the model.

### 4.5 When to use Domain Storytelling

Domain Storytelling is particularly useful when:

- You need to understand how people actually perform their work.
- Business processes involve many actors and hand-offs.
- Documentation is incomplete or outdated.
- Participants struggle to explain their work in abstract terms.
- You want to explore user journeys and operational workflows.
- You need to discover domain terminology and work objects.
- You want to understand differences between similar processes.

It is also useful when a business process is easier to explain through a specific scenario than through a list of formal requirements.

### 4.6 Domain Storytelling and Event Storming

These techniques can complement one another.

Event Storming emphasizes meaningful business events and the relationships between them.

Domain Storytelling emphasizes actors, activities, and work objects within a concrete scenario.

Consider the same warehouse process.

Event Storming might reveal:

- Goods Delivered.
- Delivery Inspected.
- Discrepancy Detected.
- Goods Received.
- Inventory Updated.

Domain Storytelling might reveal:

- The supplier sends a delivery notification.
- The warehouse employee compares quantities.
- The employee checks the purchase order.
- The employee records a discrepancy.
- The purchasing department contacts the supplier.

The two representations expose different aspects of the same domain.

Use the technique that best supports the question you are trying to answer.

**Further resources**

- [Domain Storytelling - Official Website](https://domainstorytelling.org/)
- [The Domain Storytelling Book](https://domainstorytelling.org/book/) — Stefan Hofer and Henning Schwentner.
- [Quick-Start Guide](https://domainstorytelling.org/quick-start-guide)
- [Domain Storytelling and DDD](https://domainstorytelling.org/domain-driven-design)

---

## 5. Example Mapping

Example Mapping is a collaborative technique introduced by Matt Wynne to help teams clarify requirements and business rules through concrete examples.

It is particularly useful when a team needs to understand what a specific user story or business capability should do before implementing it.

The central idea is:

**Instead of debating vague requirements, discuss concrete examples that demonstrate how the business rule should behave.**

A user story may sound clear at first but become ambiguous when the team examines different situations.

Consider:

"As a customer, I want to cancel my order so that I can stop a purchase I no longer need."

This raises several questions:

- Can every order be canceled?
- Can an order be canceled after payment?
- Can an order be canceled after shipment preparation?
- What happens to the payment?
- What happens if only part of the order has been shipped?
- Does cancellation require approval?

Example Mapping helps the team explore these questions systematically.

### 5.1 The four elements

Example Mapping commonly uses four types of cards.

| Element | Common color | Purpose |
|---|---|---|
| Story | Yellow | The feature or behavior being discussed |
| Rule | Blue | A business rule or acceptance criterion |
| Example | Green | A concrete example illustrating a rule |
| Question | Red | An unanswered question or uncertainty |

The structure is straightforward:

- A Story is placed at the top.
- Rules are placed underneath it.
- Examples are grouped under the relevant rules.
- Questions are recorded wherever clarification is needed.

### 5.2 A practical example

Consider the story:

**Cancel an order**

The group identifies the following rules.

**Rule 1: An order can only be canceled before shipment dispatch.**

Examples:

- A newly placed order can be canceled.
- A confirmed order that has not been dispatched can be canceled.
- A dispatched order cannot be canceled through this operation.

**Rule 2: Payment must be handled according to its current state.**

Examples:

- If payment has only been authorized, the authorization may need to be voided.
- If payment has been captured, a refund may be required.
- If payment has not been completed, no refund may be necessary.

**Rule 3: Cancellation must be recorded.**

Examples:

- The system records the cancellation time.
- The system records the reason for cancellation.
- The customer receives an appropriate confirmation.

The group may also discover questions:

- Can a customer cancel an individual item?
- Can customer support cancel an order after dispatch?
- What happens when the payment provider is unavailable?
- Can an order be canceled while inventory reservation is being processed?

These questions should remain visible until someone provides a reliable answer.

### 5.3 Why examples are powerful

A business rule expressed in abstract language may hide multiple interpretations.

Concrete examples make those interpretations easier to compare.

For example:

"Orders can be canceled before processing."

The team might assume everyone agrees on the meaning of processing.

But consider these examples:

- An order has been submitted but payment has not been authorized.
- Payment has been authorized but inventory has not been reserved.
- Inventory has been reserved but warehouse picking has not started.
- Warehouse picking has started but shipment has not been dispatched.

The examples force the group to clarify the exact business meaning of the rule.

This can expose important domain distinctions and improve the Ubiquitous Language.

### 5.4 How to facilitate Example Mapping

**Step 1: Choose one story**

Select a specific user story, business capability, or behavior that needs clarification.

Avoid attempting to clarify an entire product backlog in one session.

**Step 2: Write down the story**

Make sure the participants understand the intended outcome.

**Step 3: Identify rules**

Ask:

"What must be true for this behavior to be valid?"

Capture each distinct business rule.

**Step 4: Explore examples**

For each rule, ask participants to describe concrete situations.

Include:

- Normal cases.
- Boundary cases.
- Exceptions.
- Rejected cases.
- Alternative outcomes.

**Step 5: Record questions**

When the group cannot answer a question reliably, record it.

Do not turn an assumption into a rule simply to finish the workshop.

**Step 6: Review the map**

Check whether the examples adequately explain the rules.

If a rule has too many unrelated examples or generates many questions, it may need to be clarified or split.

**Step 7: Convert discoveries into useful outcomes**

The results may lead to:

- Clarified acceptance criteria.
- Additional domain conversations.
- New or refined user stories.
- Updated business rules.
- Acceptance tests.
- Changes to the domain model.

### 5.5 When to use Example Mapping

Example Mapping is especially useful when:

- A user story contains ambiguous behavior.
- Business rules have many exceptions.
- Developers and domain experts interpret requirements differently.
- Acceptance criteria need clarification.
- A team wants to discover scenarios before writing tests.
- A feature is small enough to explore through focused examples.

It is often useful during backlog refinement, discovery workshops, and the preparation of development work.

### 5.6 Example Mapping and acceptance tests

Example Mapping can provide the foundation for executable specifications and acceptance tests.

However, the map itself is not necessarily an automated test specification.

The team still needs to translate relevant examples into a suitable testing format, such as Given-When-Then scenarios or another form of executable specification.

For example:

```gherkin
Scenario: Cancel an order before shipment dispatch
  Given an order has been confirmed
  And the shipment has not been dispatched
  When the customer requests cancellation
  Then the order should be canceled
  And the applicable payment handling should be initiated
```

The exact expected payment behavior must be defined by the business rules. The scenario above is illustrative, not a universal cancellation rule.

**Further resources**

- [Example Mapping - Cucumber Documentation](https://cucumber.io/docs/bdd/example-mapping/)
- [Introducing Example Mapping - Matt Wynne](https://cucumber.io/blog/bdd/example-mapping-introduction/)
- [Gherkin Rules and Example Mapping](https://cucumber.io/blog/bdd/gherkin-rules/)

---

## 6. Choosing the right technique

The techniques introduced in this chapter solve related but different problems.

Choosing between them should depend on what the team needs to discover.

| Technique | Main question | Best suited for |
|---|---|---|
| Event Storming | What happens in the business, and what triggers it? | Discovering events, processes, decisions, and dependencies |
| Domain Storytelling | Who does what with which objects, and in what sequence? | Understanding real work, actors, interactions, and operational scenarios |
| Example Mapping | What exactly should this behavior do in specific situations? | Clarifying business rules, examples, and acceptance criteria |

### A practical selection guide

Use **Big Picture Event Storming** when the overall business landscape is unclear.

Use **Process Modeling Event Storming** when you need to explore a specific process, its events, decisions, and exceptions.

Use **Software Design Event Storming** when you have sufficient domain knowledge and want to explore how software should support the process.

Use **Domain Storytelling** when you need to understand how people actually perform their work and interact with business objects.

Use **Example Mapping** when a specific behavior or business rule needs to be clarified through examples.

You do not have to choose only one.

A team might:

1. Run a Big Picture Event Storming workshop to explore a broad business area.
2. Use Domain Storytelling to investigate an important operational process.
3. Run Process Modeling Event Storming to explore the events and decisions in that process.
4. Use Example Mapping to clarify the rules of a particular operation.
5. Use Software Design Event Storming to explore possible domain and software responsibilities.

This is one possible sequence, not a mandatory methodology.

The important thing is to choose a technique because it helps answer a question, not because it is fashionable.

---

## 7. Facilitating effective workshops

The success of a collaborative modeling session depends on more than the modeling technique.

It also depends on preparation, facilitation, participant dynamics, and what happens afterward.

### 7.1 Define the purpose

Every workshop should have a clear purpose.

Examples:

- Understand how an order moves from submission to fulfillment.
- Discover why payment reconciliation fails in certain situations.
- Clarify how shipment cancellation works.
- Identify the responsibilities involved in approving refunds.
- Explore differences between sales and inventory terminology.

Avoid vague objectives such as "Understand the whole business."

A focused objective helps participants contribute relevant knowledge and makes it easier to evaluate the outcome.

### 7.2 Define the scope

A workshop should have a meaningful boundary.

Specify:

- The business process.
- The starting point.
- The expected endpoint.
- The organizational area.
- The decisions or uncertainties being explored.

A broad scope may be appropriate for Big Picture Event Storming, but deeper modeling usually benefits from a more focused process or scenario.

### 7.3 Invite the right people

A workshop should include people who understand the relevant work.

Try to include different perspectives without making the group unnecessarily large.

If important knowledge is missing, record that limitation and arrange follow-up conversations.

Do not treat the absence of a domain expert as permission to invent the answer.

### 7.4 Establish workshop rules

Useful working agreements include:

- Describe actual behavior before proposing improvements.
- Use the language of the domain.
- Ask for examples when a statement is unclear.
- Make disagreements visible.
- Record assumptions separately from confirmed facts.
- Avoid jumping into implementation details too early.
- Give everyone an opportunity to contribute.
- Keep unresolved questions visible.

### 7.5 Encourage concrete examples

When someone says:

"Usually, the order is processed automatically."

Ask:

"Can you walk us through the last order that was processed?"

When someone says:

"The system rejects invalid requests."

Ask:

"What makes a request invalid? Can you give us an example?"

Concrete scenarios reveal the details that abstract statements hide.

### 7.6 Handle disagreements constructively

Disagreement is not necessarily a problem.

It may indicate:

- Different business practices.
- Different meanings of the same term.
- A policy that is applied inconsistently.
- A missing exception.
- A gap between documented and actual behavior.
- A genuine conflict between business objectives.

Instead of forcing consensus, ask:

- Who experiences this situation?
- Under what conditions does each interpretation apply?
- Is one interpretation an official policy and the other an operational workaround?
- Does this behavior differ between departments?
- What evidence would help resolve the disagreement?

Sometimes the correct outcome is to recognize that different contexts have different rules.

### 7.7 Watch for premature solutions

A common problem is that technical participants start proposing implementation choices before the group understands the business problem.

For example:

- "We should create a microservice."
- "This should be an event-driven workflow."
- "We need a separate database."
- "Let's add a status column."

These may eventually be reasonable decisions, but they should not replace domain exploration.

The facilitator can acknowledge the suggestion and record it for a later design discussion.

First understand what the business needs and why.

### 7.8 End with explicit outcomes

Every workshop should finish with a review of:

- What the group learned.
- Which terms were clarified.
- Which rules were identified.
- Which disagreements remain.
- Which assumptions need validation.
- Which decisions were made.
- Which questions require follow-up.
- Who is responsible for each next action.

A workshop without clear outcomes may create enthusiasm but little lasting value.

---

## 8. Facts, assumptions, policies, decisions, and questions

One of the most important skills in domain exploration is distinguishing different kinds of knowledge.

They should not all be treated as equally reliable statements.

### 8.1 Facts

Facts describe things that have happened or can be observed.

Examples:

- The customer submitted the order at 10:30.
- The payment provider rejected the authorization.
- The warehouse received the shipment yesterday.

Facts can be supported by observations, records, or reliable accounts of actual events.

### 8.2 Policies and business rules

Policies describe how the business expects a situation to be handled.

Examples:

- Orders must be paid before dispatch.
- Refunds above a certain amount require approval.
- Shipments containing restricted goods require additional documentation.

A policy may be formal or informal. Its source, scope, and authority should be understood.

A statement about what people usually do is not necessarily an official policy.

### 8.3 Assumptions

Assumptions are beliefs that have not yet been validated.

Examples:

- All customers have a verified phone number.
- Every shipment contains one package.
- Payment authorization always completes within a few seconds.

Assumptions are not inherently bad. They become dangerous when they are silently treated as facts.

Record them and determine which ones need validation.

### 8.4 Decisions

Decisions are conclusions reached by people with the appropriate authority after considering relevant information.

Examples:

- A refund request must be reviewed by the finance team.
- A shipment can be split into multiple packages.
- A particular exception will be handled manually for the first release.

Record the rationale and scope of important decisions.

A decision is not the same as a fact. It describes what the organization has decided to do.

### 8.5 Questions

Questions represent missing or unresolved knowledge.

Examples:

- Can an order be partially canceled?
- Who owns the decision to reject a shipment?
- Does the business allow multiple payment attempts?
- What happens when the carrier does not report a delivery result?

Questions should remain visible until they are answered or explicitly accepted as unresolved.

### 8.6 A practical classification

| Statement | Classification | Why |
|---|---|---|
| The payment provider rejected the authorization | Fact | Describes an observed occurrence |
| Orders must be paid before dispatch | Policy | Defines expected business behavior |
| Every customer has a verified phone number | Assumption | Requires validation |
| Refunds above a threshold require finance approval | Decision or policy | Depends on whether it is newly decided or an existing rule |
| Can an order be canceled after picking starts? | Question | Requires clarification |

The same statement may change classification as the team learns more.

For example, an assumption may become a confirmed fact, a formal policy, or a rejected belief.

The important thing is to preserve its current status and the evidence behind it.

---

## 9. From workshop output to a useful domain model

A workshop is not the final destination.

Its purpose is to produce knowledge that can guide further exploration, modeling, and implementation.

The output should be organized so that people who did not attend can understand what was discovered and why it matters.

### 9.1 Capture the important information

Depending on the technique, useful artifacts may include:

- Event timelines.
- Domain stories.
- Business process descriptions.
- Actor and responsibility maps.
- Business rules.
- Concrete examples.
- Glossary entries.
- Open questions.
- Assumptions.
- Decisions and their rationale.
- Identified exceptions.
- Candidate domain boundaries.

Do not preserve every sticky note without context. A large collection of artifacts is not necessarily useful documentation.

### 9.2 Build and refine the Ubiquitous Language

Review the terms used during the workshop.

Identify:

- Ambiguous words.
- Synonyms.
- Terms with multiple meanings.
- Important business distinctions.
- Terms used differently across departments.

Add validated definitions to the glossary of the relevant Bounded Context.

Do not force every department to use the same word if their models genuinely differ.

### 9.3 Identify candidate Subdomains and Bounded Contexts

Workshop findings may reveal areas of distinct business responsibility.

For example, an order fulfillment exploration may expose separate concerns around:

- Sales.
- Inventory.
- Warehouse Operations.
- Transportation.
- Delivery.

These are candidate boundaries, not automatic architectural decisions.

Further investigation is needed to understand ownership, model consistency, dependencies, and the relationships between the areas.

### 9.4 Connect discoveries to tactical modeling

Once the business behavior is sufficiently understood, the team can begin exploring tactical concepts.

For example:

- Which concepts have identity?
- Which concepts are values?
- Which business operations require consistency?
- Which invariants must be protected?
- Which events represent meaningful occurrences?
- Which behavior belongs to a particular Entity or Value Object?
- Which operations may require Domain Services?

The answers should emerge from the domain knowledge, not from a desire to use every DDD pattern.

### 9.5 Validate the model against real scenarios

A useful model should be able to explain both normal and unusual business situations.

Take the discovered process and test the proposed model against concrete examples.

Ask:

- Can the model represent the expected behavior?
- Can it represent important exceptions?
- Are its terms meaningful to domain experts?
- Does it enforce the rules that matter?
- Does it hide important business distinctions?
- Does it introduce concepts that nobody in the business recognizes?

If the model cannot explain real business scenarios, it needs further refinement.

### 9.6 Keep the model alive

Business processes change. New policies emerge. People discover exceptions. Organizational responsibilities evolve.

The model should evolve with that knowledge.

Workshop artifacts can be stored in a shared repository, linked to relevant documentation, and revisited when important changes occur.

A domain exploration session should not be treated as a one-time ceremony.

---

## 10. A complete practical exercise

This exercise combines the techniques in this chapter.

The goal is to explore a business process, develop a shared understanding, and produce useful artifacts that can be carried into strategic design.

### Choose a domain

Choose a business area you know or can access through domain experts.

Possible examples:

- Online retail.
- Logistics and transportation.
- Banking and payment processing.
- Food distribution.
- Subscription management.
- Healthcare administration.
- Warehouse operations.
- Insurance claims.

Choose a process with real decisions, constraints, and exceptions.

For this exercise, we will use order fulfillment.

### Part 1. Understand the process

Talk to at least two people involved in the process.

Ask each person independently to describe how an order moves from confirmation to delivery.

Record:

- Actors.
- Activities.
- Business objects.
- Decisions.
- Events.
- Exceptions.
- External systems.
- Unanswered questions.

Compare their explanations.

Do they use the same terms? Do they describe the same sequence? Do they agree on who owns each decision?

**Deliverable:** A short domain overview and a list of initial uncertainties.

### Part 2. Run a Big Picture Event Storming session

Invite relevant participants and explore the overall order fulfillment journey.

Identify the major events and arrange them chronologically.

Explore different paths and mark important areas of uncertainty.

**Deliverable:** An event timeline showing the major business events and hot spots.

### Part 3. Run a Domain Storytelling session

Choose one concrete scenario, such as a shipment that cannot be delivered.

Ask a domain expert to explain what happens.

Capture:

- Who participates?
- What objects are involved?
- Which actions take place?
- In what order?
- Which decisions are made?
- What happens next?

Explore alternative scenarios, such as a customer refusing delivery or a package being damaged.

**Deliverable:** A Domain Story describing the selected scenario.

### Part 4. Run a focused Process Modeling Event Storming session

Explore the selected process in greater detail.

Identify:

- Commands.
- Events.
- Actors.
- Policies.
- Read Models.
- External systems.
- Hot spots.

Clarify the relationships between the events and decisions.

**Deliverable:** A more detailed process model.

### Part 5. Use Example Mapping

Choose one important business operation, such as retrying a failed delivery.

Write the story, identify the rules, and collect examples.

For each rule, explore at least one normal case and one exceptional case.

Record unanswered questions.

**Deliverable:** An Example Map with clarified rules, examples, and questions.

### Part 6. Validate your discoveries

Review the artifacts with the people who understand the business.

Ask them to walk through the scenarios.

Correct misunderstandings and record unresolved disagreements.

Do not change a business rule simply because the technical implementation would be easier.

**Deliverable:** A validated set of models and a list of remaining uncertainties.

### Part 7. Reflect

Write a short report answering:

1. What did you discover that you did not know before?
2. Which assumptions turned out to be wrong?
3. Which terms had different meanings for different participants?
4. Which business rules were previously undocumented?
5. Which exceptions were missing from the original process?
6. Which decisions require further investigation?
7. Which areas might deserve separate domain models?
8. What would you investigate next?

**Deliverable:** A domain exploration report that can inform the next stage of DDD.

---

## 11. Common mistakes in domain exploration

A few recurring mistakes deserve special attention.

### Exploring the system instead of the domain

Starting with database tables, existing APIs, or source code can bias the team toward reproducing the current implementation.

Use these sources as evidence, but do not assume they accurately represent the business.

### Inviting only managers

Management perspectives are important, but operational knowledge often resides with people who perform the work.

Include the people who encounter the real exceptions and constraints.

### Asking abstract questions

Questions such as "How does your business process work?" often produce high-level descriptions.

Ask participants to describe a recent, concrete example instead.

### Treating every statement as a requirement

People may describe existing workarounds, personal preferences, policies, or assumptions.

Explore why a behavior exists before translating it into a software requirement.

### Forcing agreement

Disagreement may reveal different business contexts or conflicting policies.

Do not erase it simply to make the diagram look consistent.

### Modeling only the happy path

Exceptional cases frequently contain the most important business rules.

Explore failures, reversals, cancellations, retries, delays, and manual interventions.

### Treating workshop artifacts as permanent truth

The workshop captures the group's understanding at a particular moment.

Validate it, record its limitations, and revise it as new information emerges.

### Confusing discovery with design

Do not force every discovery into a class, Aggregate, service, or microservice.

First understand what the business means. Then decide how that knowledge should influence the model and software.

### Stopping after the workshop

A workshop should lead to further investigation, modeling, decisions, and validation.

Without follow-up, the results quickly become disconnected from the development process.

---

## 12. Completion criteria

You have developed a practical understanding of domain exploration when you can do more than describe the techniques.

**Collaborative exploration**

- [ ] Explain why domain knowledge is distributed across people and systems.
- [ ] Identify the participants needed for a particular exploration.
- [ ] Explain the responsibilities of a facilitator.
- [ ] Recognize the effects of hierarchy, assumptions, and group dynamics.
- [ ] Distinguish actual business behavior from intended behavior.

**Event Storming**

- [ ] Explain the purpose of Big Picture Event Storming.
- [ ] Explain Process Modeling and Software Design formats.
- [ ] Distinguish commands from events.
- [ ] Identify actors, policies, read models, and hot spots.
- [ ] Facilitate a basic Event Storming session.
- [ ] Explore exceptions and alternative paths.
- [ ] Explain why an Event Storming board is not automatically an architecture.

**Domain Storytelling**

- [ ] Explain actors, work objects, activities, and sequence numbers.
- [ ] Capture a concrete business scenario.
- [ ] Identify different perspectives and alternative paths.
- [ ] Validate a story with the people who perform the work.

**Example Mapping**

- [ ] Explain the relationship between stories, rules, examples, and questions.
- [ ] Clarify a business rule through concrete examples.
- [ ] Identify missing acceptance criteria.
- [ ] Translate relevant examples into testable scenarios.

**Knowledge validation**

- [ ] Distinguish facts, assumptions, policies, decisions, and questions.
- [ ] Record disagreements without hiding them.
- [ ] Identify areas that need further investigation.
- [ ] Connect exploration results to the Ubiquitous Language.
- [ ] Explain how discoveries can inform strategic and tactical design.

**Practical application**

- [ ] Facilitate or participate in at least one collaborative modeling session.
- [ ] Produce a useful representation of a real business process.
- [ ] Validate the representation with domain experts.
- [ ] Record important findings and unresolved questions.
- [ ] Explain what should be explored next and why.

---

## 13. Further reading and resources

The following resources are selected to provide a deeper understanding of collaborative modeling and domain exploration.

### Essential books

**[Collaborative Software Design: How to Facilitate Domain Modeling Decisions](https://www.manning.com/books/collaborative-software-design)**

Evelyn van Kelle, Gien Verschatse, and Kenny Baas-Schwegler.

A practical guide to collaborative modeling and decision-making. It covers Event Storming, Domain Storytelling, Example Mapping, facilitation, social dynamics, cognitive biases, conflict resolution, and follow-up.

Particularly recommended for facilitators, technical leaders, architects, and people responsible for collaborative design decisions.

**[Domain Storytelling: A Collaborative, Visual, and Agile Way to Build Domain-Driven Software](https://domainstorytelling.org/book)**

Stefan Hofer and Henning Schwentner.

The primary reference for the Domain Storytelling method. It introduces its visual notation, workshop format, scenario-based modeling, and relationship with DDD.

**[Introducing EventStorming](https://www.eventstorming.com/book/)**

Alberto Brandolini.

The author's evolving reference for Event Storming. It explains the philosophy and mechanics behind the technique and provides material for practitioners. Check the publication page for its current completion status.

**[The EventStorming Handbook](https://www.eventstorming.com/resources/)**

Paul Rayner.

A practical resource for understanding and facilitating Event Storming workshops.

### Official and practical resources

**Event Storming**

- [EventStorming Official Website](https://www.eventstorming.com/)
- [EventStorming Resources](https://www.eventstorming.com/resources/)
- [EventStorming Patterns](https://www.eventstorming.com/patterns/)
- [Introducing EventStorming](https://www.eventstorming.com/book/)

**Domain Storytelling**

- [Domain Storytelling Official Website](https://domainstorytelling.org/)
- [Quick-Start Guide](https://domainstorytelling.org/quick-start-guide)
- [Domain Storytelling and DDD](https://domainstorytelling.org/domain-driven-design)
- [The Domain Storytelling Book](https://domainstorytelling.org/book/)

**Example Mapping**

- [Example Mapping - Cucumber](https://cucumber.io/docs/bdd/example-mapping/)
- [Introducing Example Mapping - Matt Wynne](https://cucumber.io/blog/bdd/example-mapping-introduction/)
- [Gherkin Rules](https://cucumber.io/blog/bdd/gherkin-rules/)

### Recommended learning order

If you are new to collaborative modeling, a useful sequence is:

1. Read the introductory material on collaborative software design.
2. Learn the fundamentals of Event Storming.
3. Practice Domain Storytelling with a concrete business scenario.
4. Use Example Mapping to clarify the rules discovered.
5. Explore facilitation, decision-making, and organizational dynamics in greater depth.
6. Apply the techniques to a real domain and compare the results.

Do not try to master every technique before working with a domain expert. Start with a real problem and learn the technique that helps you explore it.

---

## What comes next?

Domain exploration gives the team a better understanding of how the business works, what its important events and decisions are, and where different perspectives and responsibilities exist.

But understanding a process is not the same as designing the structure of the domain.

The next challenge is to discover meaningful boundaries:

- Which areas of the business have distinct responsibilities?
- Which capabilities provide competitive advantage?
- Where should different models exist?
- Which teams should own those models?
- How should the models communicate?
- Which relationships create dependencies or translation costs?

These questions lead to Strategic Design.

**[03. Strategic Design](03-strategic-design.md)**

The objective is to move from discovering how the business works to understanding how its domain should be divided, modeled, and organized.