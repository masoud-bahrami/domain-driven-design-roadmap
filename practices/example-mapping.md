# Example Mapping

Example Mapping is a collaborative technique for clarifying a user story, business rule, or requirement through **Rules, Examples, Questions, and Acceptance Criteria**.

It helps a team move from broad statements to concrete, testable understanding.

A requirement such as:

> Customers can cancel an order.

Sounds simple, but it leaves many questions unanswered.

- Can an order be cancelled after payment?
- What if the warehouse has already started preparing it?
- Can a customer cancel one item instead of the entire order?
- Is a cancellation always accepted?
- What happens to an existing payment authorization?
- Does cancellation trigger a refund?
- Can an order be cancelled after shipment?

Different people may have different answers, even though they believe they understand the same requirement.

Example Mapping makes these differences visible by asking participants to describe concrete situations and agree on the expected outcomes.

It is particularly useful before implementation, when a team needs to clarify business rules, identify missing decisions, and establish a shared understanding of acceptance criteria.

## What Is Example Mapping?

Example Mapping is commonly associated with the work of Matt Wynne and the Behaviour-Driven Development community.

It uses four types of cards, traditionally distinguished by color:

| Card | Typical color | Purpose |
|---|---|---|
| User Story | Yellow | Defines the behavior or requirement being explored |
| Rules | Blue | States the business rules governing that behavior |
| Examples | Green | Describes concrete scenarios that illustrate the rules |
| Questions | Red | Captures uncertainty, ambiguity, and decisions that remain unresolved |

The technique is deliberately simple.

Start with one user story. Identify the rules that govern it. Explore each rule through examples. Record questions that cannot yet be answered.

The result is a clearer understanding of the requirement and a set of scenarios that can guide implementation and testing.

## The Four Building Blocks

### 1. User Story

<text color="#D97706" weight="semibold" inline>Yellow</text>

The User Story identifies the behavior the team wants to explore.

For example:

> As a customer, I want to cancel an order so that I can stop a purchase I no longer need.

A user story provides a starting point, not a complete specification.

It describes a desired capability but does not necessarily explain the business rules governing it.

**Practical guidance**

- Choose one story or a sufficiently focused requirement.
- Make sure participants understand the business goal.
- Avoid combining several unrelated behaviors.
- Keep the scope small enough to explore through concrete examples.

If the story contains several independent workflows, split the discussion into smaller sessions.

### 2. Rules

<text color="#3B82F6" weight="semibold" inline>Blue</text>

Rules describe the business conditions and constraints that govern the behavior.

For the order cancellation story, participants might discover:

1. An order can be cancelled before fulfillment begins.
2. An order cannot be cancelled after it has been dispatched.
3. A cancellation must release reserved inventory.
4. Payment authorization must be voided when the order is cancelled.
5. If payment has already been captured, the customer may be entitled to a refund rather than an authorization reversal.

Notice that these statements describe different aspects of the business.

Some define when cancellation is permitted. Others describe consequences that must follow.

A useful rule should be precise enough to explore through examples.

For instance:

> An order can be cancelled before fulfillment begins.

Raises another question:

**What exactly does fulfillment begin mean?**

Does it mean when the warehouse receives the order, when picking starts, or when the first item is physically picked?

The rule may look clear until participants try to apply it to an actual situation.

**Practical guidance**

- Express one business rule per card.
- Use language understood by domain experts.
- Avoid vague expressions such as `usually`, `normally`, or `when appropriate`.
- Distinguish conditions from consequences.
- Identify rules that depend on other business concepts.
- Do not confuse a technical validation with a business rule.

Rules should explain the business behavior, not prescribe implementation details.

### 3. Examples

<text color="#16A34A" weight="semibold" inline>Green</text>

Examples describe concrete situations that demonstrate how a rule should behave.

They are the most important part of Example Mapping because they challenge assumptions and expose details hidden inside general statements.

Consider the rule:

> An order can be cancelled before fulfillment begins.

Participants may explore the following examples:

**Example 1: Order has not been processed**

- The customer places an order.
- Payment is authorized.
- The warehouse has not started fulfillment.
- The customer requests cancellation.

Expected outcome: cancellation is accepted, inventory reservation is released, and the payment authorization is voided.

**Example 2: Warehouse has started picking**

- The customer places an order.
- The warehouse starts picking the items.
- The customer requests cancellation.

Expected outcome: this requires clarification. Can the warehouse stop the picking process, or must the request be rejected?

**Example 3: Order has been dispatched**

- The warehouse completes fulfillment.
- The shipment is dispatched.
- The customer requests cancellation.

Expected outcome: the order cannot be cancelled through the normal cancellation process. A return process may be appropriate instead.

These examples reveal that `before fulfillment begins` may not be sufficiently precise.

They also expose a potential distinction between cancellation and return.

**Practical guidance**

- Describe a concrete situation, not another general rule.
- Include relevant starting conditions.
- State the action or trigger.
- Agree on the expected outcome.
- Explore both successful and unsuccessful cases.
- Include boundary conditions and unusual situations.
- Challenge examples that participants interpret differently.

A useful example should make it possible to answer:

> Given this situation, what should the business do?

### 4. Questions

<text color="#DC2626" weight="semibold" inline>Red</text>

Questions capture unresolved issues, missing knowledge, and disagreements.

For example:

- What exactly marks the beginning of fulfillment?
- Can a customer cancel an order after picking starts?
- Who can approve an exception?
- What happens when payment has already been captured?
- Can an order containing multiple shipments be partially cancelled?
- How long can a cancellation request remain pending?

These questions should remain visible.

Do not silently choose an interpretation just because the implementation needs to move forward.

A question may require:

- A domain expert's decision.
- Input from another department.
- Investigation of an existing business process.
- Clarification of legal or contractual constraints.
- A separate discussion about a related workflow.

**Practical guidance**

- Record questions as soon as they arise.
- Distinguish missing information from genuine business disagreement.
- Assign ownership for follow-up work.
- Avoid converting assumptions into rules without validation.
- Revisit unresolved questions before implementation depends on their answers.

An unanswered question is not a failed workshop. It is a discovery that would otherwise have remained hidden.

## A Practical Example Mapping Session

Consider the following user story:

> As a customer, I want to cancel an order so that I can stop a purchase I no longer need.

The team consists of:

- A product owner.
- A customer support representative.
- A warehouse representative.
- A developer.
- A tester.

The team places the User Story card at the center of the workspace.

### Step 1: Discover the Rules

The participants discuss what governs order cancellation.

They identify these initial rules:

**Rule 1:** Customers can cancel an order before fulfillment begins.

**Rule 2:** An order cannot be cancelled after it has been dispatched.

**Rule 3:** Cancelling an order must release its reserved inventory.

**Rule 4:** The payment operation depends on the payment's current state.

The fourth rule is deliberately broad. It needs more exploration.

### Step 2: Explore Each Rule Through Examples

The team starts with Rule 1.

**Rule: Customers can cancel an order before fulfillment begins.**

| Example | Situation | Expected outcome |
|---|---|---|
| 1 | Order placed, fulfillment not started | Cancellation accepted |
| 2 | Warehouse has started picking | Unclear |
| 3 | Warehouse has finished packing but shipment not dispatched | Unclear |

The examples expose an ambiguity: the participants do not agree on what `fulfillment begins` means.

The team records a question rather than inventing a definition.

Next, they explore Rule 2.

**Rule: An order cannot be cancelled after it has been dispatched.**

| Example | Situation | Expected outcome |
|---|---|---|
| 1 | Shipment dispatched, customer requests cancellation | Cancellation rejected |
| 2 | Shipment dispatched, customer refuses delivery | Investigate return or delivery refusal |
| 3 | Only part of the order has been dispatched | Clarify whether remaining items can be cancelled |

This reveals that a single order may involve multiple shipments and that cancellation may need to be considered at different levels.

Finally, they explore Rule 4.

**Rule: The payment operation depends on the payment's current state.**

| Example | Payment state | Expected outcome |
|---|---|---|
| 1 | Authorization approved, not captured | Void authorization |
| 2 | Payment captured | Refund may be required |
| 3 | Payment still pending | Investigate cancellation or reconciliation |
| 4 | Payment provider unavailable | Define retry and recovery behavior |

The team discovers that `cancel payment` is not one universal operation.

The correct action depends on the payment lifecycle.

### Step 3: Record Questions

The session has uncovered several questions:

- Does fulfillment begin when the warehouse receives an order or when picking starts?
- Can picking be stopped after it begins?
- Can an order with multiple shipments be partially cancelled?
- Who decides whether a cancellation exception is permitted?
- How should pending payments be handled?
- What happens if the payment provider is unavailable?

These questions are assigned to the relevant participants for investigation.

### Step 4: Refine the Rules

After consulting the warehouse representative, the team learns that fulfillment begins when the first item is picked.

The rule can now be refined:

> A customer can cancel an order while no item has been picked.

The team also discovers that warehouse supervisors can approve a cancellation after picking has started, provided the shipment has not been dispatched.

The original rule was too simplistic. The examples helped the participants discover the actual business policy.

### Step 5: Define Acceptance Criteria

Once the rules and examples are sufficiently clear, the team can express the agreed behavior as acceptance criteria.

For example:

**Scenario: Cancel an order before picking begins**

```gherkin
Given an order has been placed
And no items have been picked
When the customer requests cancellation
Then the order is cancelled
And reserved inventory is released
And the payment authorization is voided
```

**Scenario: Reject cancellation after dispatch**

```gherkin
Given an order has been dispatched
When the customer requests cancellation
Then the cancellation is rejected
And the customer is informed that the order must follow the return process
```

**Scenario: Cancel an order after picking begins with supervisor approval**

```gherkin
Given an order has items that have been picked
And the order has not been dispatched
When a warehouse supervisor approves cancellation
Then the order is cancelled
And the warehouse is instructed to stop fulfillment
And reserved inventory is reconciled
```

These scenarios can guide implementation and automated acceptance tests.

However, acceptance criteria should reflect decisions that have actually been validated. The team should not turn unresolved questions into invented behavior.

## Example Mapping and the DDD Practices Around It

Example Mapping complements other domain exploration techniques. It is particularly useful when the team needs to move from broad understanding to precise business behavior.

### Ubiquitous Language

Example Mapping helps refine the language used by domain experts and developers.

Consider the word `cancelled`.

Does it mean:

- The customer requested cancellation?
- The business accepted the request?
- Fulfillment was stopped?
- The payment was reversed?
- The order lifecycle was terminated?

These may be separate concepts and events.

Through concrete examples, the team can discover more precise terms such as:

- Cancellation Requested
- Cancellation Approved
- Order Cancelled
- Payment Authorization Voided
- Refund Issued

The goal is not to create more terminology unnecessarily. It is to distinguish meanings that have different business consequences.

The agreed terminology should become part of the Ubiquitous Language within the relevant Bounded Context.

### Domain Storytelling

Domain Storytelling helps participants explain how people perform work.

Example Mapping takes a particular behavior or rule from that understanding and explores its concrete outcomes.

For example, a Domain Story may describe how a support agent handles a cancellation request.

Example Mapping can then investigate what happens when the request arrives before picking, during picking, or after dispatch.

### Event Storming

Event Storming explores business behavior and the relationships between events, commands, actors, and policies.

Example Mapping explores a narrower question in greater detail.

For example, Event Storming may reveal the event `Order Cancelled`.

Example Mapping can investigate the exact conditions under which that event may occur.

It may reveal that cancellation is not a single decision but a process involving customer requests, warehouse approval, inventory release, and payment handling.

### Acceptance Test-Driven Development and BDD

Example Mapping can provide the foundation for acceptance criteria and executable specifications.

Once the team agrees on rules and examples, selected scenarios can be expressed using Given-When-Then.

For example:

```gherkin
Scenario: Cancel an order before fulfillment begins
  Given an order has been placed
  And no items have been picked
  When the customer requests cancellation
  Then the order is cancelled
  And the reserved inventory is released
```

These scenarios can be automated where appropriate.

However, not every Example Mapping card needs to become an automated test. The examples are primarily a tool for understanding. The team should select scenarios that provide meaningful coverage of business behavior.

## When to Use Example Mapping

Example Mapping is particularly useful when:

- A user story contains ambiguous business language.
- Different stakeholders interpret a requirement differently.
- A business rule has several conditions or exceptions.
- Acceptance criteria are vague or incomplete.
- A feature has important edge cases.
- The team is unsure what should happen in a particular scenario.
- A developer or tester discovers an unanswered business question.
- A change may affect several parts of a business process.

It is less useful when the behavior is already well understood, the rule is trivial, and the team can agree on the expected outcome without a separate workshop.

Do not turn every small requirement into a formal Example Mapping session. The technique should reduce uncertainty, not introduce unnecessary ceremony.

## Common Mistakes

### 1. Writing More Rules Instead of Exploring Examples

A team may produce a long list of abstract statements without discovering how they apply.

For example:

> Cancellation must respect the order lifecycle.

This sounds reasonable but does not explain what should happen in any particular situation.

Ask for concrete examples.

### 2. Treating Examples as Test Cases Only

Examples are not merely inputs and expected outputs for automated tests.

Their purpose is to explore meaning, challenge assumptions, and discover business rules.

Tests can be derived from the resulting understanding, but testing is not the entire purpose of the technique.

### 3. Ignoring Questions to Keep the Session Moving

A team may feel pressured to answer every question immediately.

This encourages assumptions and can lead to incorrect implementations.

Record unanswered questions and assign follow-up work.

### 4. Exploring Too Many User Stories at Once

A workshop that covers an entire feature set can quickly become unfocused.

Choose one behavior or a closely related group of rules. Split unrelated concerns into separate sessions.

### 5. Confusing Business Rules with Implementation Decisions

For example:

> The system must call the Payment API within two seconds.

This may be an important technical or operational requirement, but it is not necessarily a business rule.

First clarify the business outcome when payment authorization is delayed. Then discuss technical constraints and implementation.

### 6. Treating Acceptance Criteria as a Complete Domain Model

Acceptance criteria describe selected expected behaviors.

They do not necessarily capture every concept, relationship, invariant, or responsibility in the domain.

Use them as a bridge between domain understanding and implementation, not as a replacement for domain modeling.

### 7. Allowing One Person to Define the Rules

A product owner or developer may propose an interpretation, but the people who understand and own the business process should validate it.

Example Mapping works best when different perspectives are represented and disagreements are explored.

## Practical Example Mapping Checklist

**Before the session**

- [ ] Select one user story or focused requirement.
- [ ] Identify participants with relevant domain knowledge.
- [ ] Prepare the four types of cards.
- [ ] Explain the purpose of the session.
- [ ] Agree on the scope.

**During the session**

- [ ] Write the User Story.
- [ ] Identify the business rules.
- [ ] Explore each rule with concrete examples.
- [ ] Include successful and unsuccessful scenarios.
- [ ] Explore boundary conditions and exceptions.
- [ ] Record unresolved questions.
- [ ] Challenge assumptions and ambiguous language.
- [ ] Validate decisions with domain experts.

**After the session**

- [ ] Resolve or assign outstanding questions.
- [ ] Refine the rules using the new understanding.
- [ ] Agree on acceptance criteria.
- [ ] Identify scenarios suitable for automated tests.
- [ ] Update the Ubiquitous Language where necessary.
- [ ] Share the results with the implementation team.
- [ ] Revisit assumptions when new domain knowledge emerges.

## Further Reading

- **Example Mapping**, Matt Wynne. The original explanation and practical guidance for the technique.
- **Cucumber Blog: Example Mapping**. Articles and practical examples from the BDD community.
- **Specification by Example**, Gojko Adzic. A broader treatment of collaborative specification through concrete examples.
- **Discovery: Explore Behaviour Using Examples**, Gaspar Nagy and Seb Rose. Practical guidance on exploring requirements and behavior through examples.

## Final Thought

A requirement can sound perfectly clear until someone asks for a concrete example.

That is the strength of Example Mapping.

Rules describe what should happen. Examples reveal what those rules actually mean. Questions expose what the team does not yet know. Acceptance criteria preserve the understanding reached by the participants.

**Do not ask only whether everyone understands the requirement. Ask whether everyone agrees on what should happen in a concrete situation.**