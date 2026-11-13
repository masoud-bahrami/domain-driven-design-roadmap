# Role Preparation

Domain-Driven Design is a collaborative approach to understanding and designing software around a business domain. It brings together people who understand the business, people who build the software, and people who understand how users interact with it.

This section explains how different roles contribute to that work and which concepts are useful for each of them.

It is not a certification checklist, a set of mandatory prerequisites, or an attempt to turn everyone into a software architect. Each role brings existing expertise that becomes more valuable when connected to the knowledge and perspectives of others.

The objective is to develop a shared understanding of the domain, improve the quality of conversations, and make better decisions together.

## How to read this guide

The learning recommendations use three categories:

- **Foundation:** Knowledge that is useful to have before or while participating in DDD activities.
- **Continuous learning:** Skills that improve through regular practice and should develop throughout a person's career.
- **As needed:** Topics worth exploring when a particular modeling, implementation, or collaboration problem makes them relevant.

You do not need to complete every book or master every subject before participating in a DDD initiative.

---

## 1. Software Developers

Developers are responsible for translating domain understanding into software. Their contribution is not limited to implementing requirements or translating diagrams into classes. They participate in discovering the model, challenging assumptions, exploring alternative designs, and ensuring that the resulting software expresses meaningful domain behavior.

DDD does not prescribe a single programming paradigm. Object-oriented and functional approaches can both support domain modeling. The important capability is being able to represent domain concepts, behavior, constraints, and relationships clearly.

### Essential foundations

#### 1.1 Programming fundamentals

**Learning focus:** Foundation and continuous learning.

Developers should be comfortable with the programming language and runtime they use. DDD introduces modeling and design challenges; it does not replace the need to understand programming fundamentals.

Useful subjects include:

- Variables, types, expressions, control flow, and functions.
- Data structures and algorithms.
- Collections, generics, and type systems.
- Error handling and exception management.
- Memory, concurrency, and asynchronous execution where relevant.
- Modules, packages, namespaces, and dependency management.

**Recommended resources:**

- [MIT OpenCourseWare: Introduction to Computer Science and Programming](https://ocw.mit.edu/courses/6-100l-introduction-to-computer-science-and-programming-using-python-fall-2022/)
- [Microsoft Learn: C# documentation](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [The Java Tutorials](https://docs.oracle.com/javase/tutorial/)
- [Python documentation](https://docs.python.org/3/tutorial/)

Choose language-specific resources according to your technology stack. The goal is not to master every feature of a language, but to understand its capabilities and limitations well enough to make deliberate design decisions.

#### 1.2 Object-oriented and functional design

**Learning focus:** Foundation and continuous learning.

Object-oriented programming is particularly useful when a domain model benefits from explicit identity, encapsulated behavior, and objects that protect their own state.

Understand:

- Encapsulation and information hiding.
- Abstraction and composition.
- Polymorphism and interfaces.
- Object collaboration and responsibility assignment.
- Immutability and value-oriented design.
- Functional composition and transformation.
- The relationship between data, behavior, and state.

**Recommended resources:**

- [Head First Object-Oriented Analysis and Design](https://www.oreilly.com/library/view/head-first-object-oriented/0596008678/) by Brett McLaughlin, Gary Pollice, and David West.
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.pearson.com/en-us/subject-catalog/p/design-patterns-elements-of-reusable-object-oriented-software/P200000003191) by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides.
- [Functional Programming in C#](https://www.manning.com/books/functional-programming-in-c-sharp) by Enrico Buonanno.
- [Functional Programming in Java](https://www.manning.com/books/functional-programming-in-java) by Pierre-Yves Saumont.

You do not need to choose between OOP and functional programming to practice DDD. Learn to select the approach that makes the model and its rules easier to understand.

#### 1.3 Software design principles

**Learning focus:** Foundation and continuous learning.

Good domain models require more than correctly named classes. Developers need to reason about responsibilities, dependencies, coupling, cohesion, and the consequences of design decisions.

Become familiar with:

- SOLID principles.
- DRY (Don't Repeat Yourself).
- KISS (Keep It Simple).
- Separation of Concerns.
- Dependency Inversion.
- High Cohesion and Low Coupling.
- Composition over inheritance, where appropriate.
- GRASP principles for assigning responsibilities in object-oriented designs.

**Recommended resources:**

- [Clean Code, Second Edition](https://www.pearson.com/en-us/subject-catalog/p/clean-code-a-handbook-of-agile-software-craftsmanship-2nd-edition/P200000013239/9780135398579) by Robert C. Martin.
- [Refactoring: Improving the Design of Existing Code](https://martinfowler.com/books/refactoring.html) by Martin Fowler, with Kent Beck.
- [Applying UML and Patterns](https://www.oreilly.com/library/view/applying-uml-and/0130925691/) by Craig Larman, particularly its treatment of GRASP.

These principles are tools for reasoning, not laws. Applying them without understanding the problem can produce unnecessary abstractions and complexity. A useful design is one that makes the important decisions and responsibilities clear.

#### 1.4 Testing and test automation

**Learning focus:** Foundation and continuous learning.

DDD models express business rules. Those rules need to be verified and protected as the software evolves.

Developers should understand:

- Unit testing and testing domain behavior.
- Integration testing and persistence boundaries.
- Acceptance testing and business scenarios.
- Test-Driven Development (TDD).
- Test doubles and the appropriate use of mocks, stubs, and fakes.
- Characterization testing for existing systems.
- Test readability and maintainability.

**Recommended books:**

- [Test-Driven Development: By Example](https://www.pearson.com/en-us/subject-catalog/p/test-driven-development-by-example/P200000000316) by Kent Beck.
- [Growing Object-Oriented Software, Guided by Tests](https://www.pearson.com/en-us/subject-catalog/p/growing-object-oriented-software-guided-by-tests/P200000003479) by Steve Freeman and Nat Pryce.
- [Working Effectively with Legacy Code](https://www.pearson.com/en-us/subject-catalog/p/working-effectively-with-legacy-code/P200000003479) by Michael Feathers.

**Testing framework documentation:**

- [.NET testing documentation](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- [xUnit.net](https://xunit.net/docs/getting-started/v3/getting-started)
- [JUnit 5 User Guide](https://docs.junit.org/current/user-guide/)
- [pytest documentation](https://docs.pytest.org/en/stable/)

A useful DDD test should communicate business behavior. Prefer a test that explains why an order cannot be shipped before payment is confirmed over one that merely checks whether a particular private method was called.

#### 1.5 Version control and collaboration

**Learning focus:** Foundation and continuous learning.

DDD involves continuous refinement of the model and code. Developers need to collaborate effectively, review changes, and preserve a reliable history of decisions and implementation.

Learn:

- Git fundamentals.
- Branching and merging.
- Code review and collaborative development.
- Managing conflicts.
- Continuous Integration.
- Working with shared repositories and automated build pipelines.

**Recommended resources:**

- [Pro Git](https://git-scm.com/book/en/v2)
- [Git documentation](https://git-scm.com/doc)

### Additional knowledge worth developing

These subjects are useful but should not become barriers to participating in DDD.

**Design patterns**

Learn patterns when they help you recognize recurring design problems. Do not begin by searching for a pattern to apply to every domain concept.

Recommended reading: *Design Patterns: Elements of Reusable Object-Oriented Software* by the Gang of Four.

**Databases and persistence**

Understand relational modeling, transactions, indexes, constraints, and persistence technologies. Learn how persistence concerns can influence implementation without allowing the database schema to become the domain model.

**Refactoring**

Develop the ability to improve existing code in small, behavior-preserving steps. This becomes particularly important when introducing a better domain model into an existing system.

Recommended resource: [Martin Fowler's Refactoring catalog](https://refactoring.com/catalog/).

### How developers contribute to DDD

Beyond technical implementation, developers should actively:

- Participate in domain exploration workshops.
- Ask questions when business rules are ambiguous.
- Challenge models that do not reflect observed behavior.
- Help refine the Ubiquitous Language.
- Identify concepts that have different meanings across contexts.
- Express domain behavior through meaningful code.
- Collaborate with domain experts when tests reveal unexpected behavior.
- Refactor the implementation when the model changes.

**Practical exercise:** Take an existing feature and identify the business rules hidden in its application services, controllers, database queries, and conditional statements. Work with a domain expert to validate those rules, then explore how the model could express them more clearly.

**You are ready to participate when:** You can discuss business behavior without immediately translating every concept into a database table, API endpoint, or class.

---

## 2. Business Roles: Product Managers, Business Analysts, and Domain Experts

Business roles are essential to DDD because they provide the knowledge that software teams cannot reliably derive from code or technical documentation alone.

These roles are not interchangeable, and their responsibilities differ across organizations. However, they share an important contribution: helping the team understand what the business does, why it does it, and what rules govern its behavior.

A domain expert may understand the operational details of a process. A product manager may understand business objectives and priorities. A business analyst may help investigate requirements, policies, and process variations.

DDD works best when these perspectives are brought into the same conversations.

### Key contributions to DDD

#### 2.1 Providing domain knowledge

The first responsibility is to help the team understand the actual business.

This includes:

- Explaining business processes and their variations.
- Describing rules, policies, and constraints.
- Identifying important business events and decisions.
- Explaining exceptional situations and edge cases.
- Describing relationships between people, organizations, products, and other domain concepts.
- Sharing the reasons behind existing business practices.

**Practical ways to contribute:**

- Describe important processes using concrete examples.
- Prepare examples of normal and exceptional scenarios.
- Explain why a particular business rule exists.
- Identify decisions that depend on external conditions.
- Highlight differences between official procedures and actual operational practices.
- Identify assumptions that the team should validate.

Do not assume that existing process documentation contains the complete truth. The most important knowledge is sometimes found in exceptions, workarounds, and decisions that people make without realizing they are applying a business rule.

#### 2.2 Developing the Ubiquitous Language

The Ubiquitous Language is a shared language based on the domain model and used consistently within a Bounded Context.

It is not simply a glossary created at the beginning of a project. It develops through conversations, examples, modeling, implementation, and continuous refinement.

**Practical ways to contribute:**

- Discuss the meaning of important business terms.
- Identify words that are ambiguous or overloaded.
- Explain differences between similar concepts.
- Challenge technical terminology that does not represent the business accurately.
- Participate in naming domain concepts.
- Review whether the language used in conversations and software still reflects the business.

A particularly important point: the same word may have different meanings in different Bounded Contexts. The objective is not to force a universal vocabulary across the organization, but to make meaning explicit within each model.

**Recommended reading:**

- [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html) by Martin Fowler.
- [Domain-Driven Design Distilled](https://www.informit.com/store/domain-driven-design-distilled-9780134434421) by Vaughn Vernon.

#### 2.3 Identifying business needs and priorities

Product managers and business analysts help connect domain understanding to business objectives.

Their contribution includes:

- Explaining business goals and expected outcomes.
- Identifying stakeholders and their concerns.
- Understanding the value of proposed capabilities.
- Clarifying priorities and constraints.
- Distinguishing essential business capabilities from supporting functionality.
- Explaining why a change matters and what happens if it is not delivered.

Useful techniques include:

- User Story Mapping.
- Impact Mapping.
- MoSCoW prioritization.
- Example Mapping.
- Opportunity and outcome analysis.

These techniques complement DDD. They are not DDD patterns, and no particular requirements format is mandatory.

**Recommended resource:**

- [Specification by Example: How Successful Teams Deliver the Right Software](https://www.manning.com/books/specification-by-example) by Gojko Adzic.

This book is particularly useful for understanding how business stakeholders, analysts, testers, and developers can collaboratively specify and validate expected behavior.

#### 2.4 Validating the domain model

A domain model should be continuously validated against business reality.

Domain experts should not be expected to review only diagrams or technical documentation. They should participate in discussions about real scenarios and help determine whether the model expresses the right concepts and rules.

**Practical ways to contribute:**

- Review domain concepts and their definitions.
- Validate examples against real business situations.
- Explain where a proposed model does not match actual behavior.
- Identify missing rules and exceptional scenarios.
- Challenge assumptions about business processes.
- Participate in reviewing important changes to the model.

A useful question is:

"Would someone who performs this business activity recognize this model as an accurate representation of what they do and the decisions they make?"

### DDD concepts worth understanding

Business roles do not need to master tactical implementation details, but understanding the following concepts makes collaboration much more effective.

| Concept | Why it matters |
|---|---|
| Domain Model | Understand what the software is trying to represent |
| Ubiquitous Language | Participate in developing a shared vocabulary |
| Subdomains | Understand the different areas of the business |
| Bounded Contexts | Understand why different parts of the business may need different models |
| Entities and Value Objects | Understand the distinction between identity and value |
| Aggregates | Understand how related business rules can be protected together |
| Domain Events | Recognize meaningful things that have happened in the business |
| Context Mapping | Understand relationships and dependencies between different models |

### Recommended resources for business roles

**Books**

- [Domain-Driven Design Distilled](https://www.informit.com/store/domain-driven-design-distilled-9780134434421) by Vaughn Vernon. A concise introduction to strategic and tactical DDD concepts.
- [Specification by Example](https://www.manning.com/books/specification-by-example) by Gojko Adzic. Particularly useful for collaborative requirements, examples, and validation.
- [Domain Storytelling](https://www.domainstorytelling.org/book) by Stefan Hofer and Henning Schwentner. A practical approach to discovering and communicating domain knowledge through stories.

**Articles and online resources**

- [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html) by Martin Fowler.
- [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html) by Martin Fowler.
- [Domain-Driven Design with Domain Storytelling](https://domainstorytelling.org/domain-driven-design).
- [Domain-Driven Design Quickly](https://www.infoq.com/minibooks/domain-driven-design-quickly/), an introductory book available online.

**Videos and talks**

- [Domain Storytelling: Videos and talks](https://domainstorytelling.org/videos).
- [Domain-Driven Design Europe](https://dddeurope.com/): Conference talks covering strategic design, modeling, collaboration, and implementation.
- [Explore DDD](https://www.youtube.com/results?search_query=domain+driven+design+event+storming+domain+storytelling): Recorded discussions and workshops on DDD practices.

### How business roles contribute in practice

A domain expert should not merely answer questions submitted by developers. They should be an active participant in discovering and refining the model.

A product manager should not simply translate business requests into a prioritized backlog. They should help the team understand the underlying business capabilities and outcomes.

A business analyst should not only document requirements. They can help expose inconsistencies, clarify rules, and connect examples to a coherent model.

**Practical exercise:** Choose one important business process and explain it to the development team using a concrete example. Include the normal flow, at least two exceptional situations, the decisions involved, and the business reasons behind those decisions.

**You are ready to participate when:** You can explain not only what the business does, but why it behaves that way, what constraints govern it, and which assumptions need further investigation.

---

## 3. UX/UI Designers

User experience and interface design are important parts of building domain-oriented software.

A system may have a well-designed internal model and still be difficult to use if its interfaces expose confusing concepts, hide important distinctions, or force users to work against the way the business operates.

UX/UI designers can help connect the domain model to the actual experience of people using the software.

Their contribution is not to reproduce the domain model visually. It is to understand the domain well enough to design interfaces, workflows, and interactions that make sense to users.

### Key contributions to DDD

#### 3.1 Understanding the domain behind the interface

Before designing screens and workflows, designers should understand what users are trying to accomplish and the business concepts involved.

This means exploring:

- The people and roles involved in a process.
- Their goals and responsibilities.
- The business objects they create, manage, and interact with.
- The decisions they make.
- The constraints and exceptions they encounter.
- The terminology they use in their daily work.

A user journey is not necessarily identical to a domain workflow. Users may move across several business processes and Bounded Contexts to accomplish one goal.

Understanding both perspectives helps designers avoid interfaces that reflect technical structures rather than real user needs.

#### 3.2 Using domain language in user interfaces

The language used in the interface should be understandable to its intended users and consistent with the meaning of the domain concepts.

Designers should collaborate with domain experts and developers to identify:

- Terms that users already understand.
- Terms that have precise business meanings.
- Similar terms that users might confuse.
- Labels that hide important business distinctions.
- Concepts that require contextual explanation.

For example, a system may distinguish between a payment that has been authorized, captured, settled, or refunded. Showing all these states as simply "Paid" may hide meaningful business differences.

However, the Ubiquitous Language should not be copied mechanically into every UI label. Internal model terminology may need to be translated into clearer user-facing language, especially when users have different levels of domain expertise.

The important thing is to preserve meaning rather than force identical wording.

#### 3.3 Designing experiences across Bounded Contexts

A user may interact with multiple parts of a system that represent different Bounded Contexts.

For example, an employee may work with customer information, sales orders, payments, and fulfillment through a single interface.

Each context may have its own model and terminology, but the user still needs a coherent experience.

Designers can help by:

- Mapping user journeys across different business capabilities.
- Identifying where users cross context boundaries.
- Making transitions between workflows understandable.
- Preserving important distinctions between context-specific concepts.
- Avoiding misleading assumptions that similarly named concepts are identical.
- Designing appropriate navigation and information architecture.

A coherent experience does not require a single universal domain model. It requires that the interface helps users understand and work with the different responsibilities of the system.

#### 3.4 Providing feedback on the domain model

UX research and design can reveal problems that may not be obvious during technical modeling.

For example:

- Users may distinguish between two concepts that developers have modeled as one.
- A business process may require information that the current model does not expose.
- A workflow may reveal an important decision or exception that was missed during domain exploration.
- Different user roles may use the same term with different expectations.

Designers should bring these observations into domain modeling discussions rather than treating them as purely visual or interaction problems.

### DDD concepts worth understanding

| Concept | Why it matters to UX/UI |
|---|---|
| Domain Model | Understand the business concepts behind the interface |
| Ubiquitous Language | Keep terminology meaningful and consistent |
| Subdomains | Understand the different business capabilities users interact with |
| Bounded Contexts | Recognize when concepts and workflows have different meanings |
| Context Mapping | Understand how users move across connected parts of a system |
| Domain Events | Understand meaningful changes that may affect user journeys and feedback |
| Business Rules | Design interactions that communicate constraints and prevent misleading actions |

### Recommended resources

**Books**

- [Domain-Driven Design Distilled](https://www.informit.com/store/domain-driven-design-distilled-9780134434421) by Vaughn Vernon. Particularly useful for understanding Ubiquitous Language and Bounded Contexts without requiring deep implementation knowledge.
- [Domain Storytelling](https://www.domainstorytelling.org/book) by Stefan Hofer and Henning Schwentner. Useful for understanding business activities and interactions through stories.
- [The Design of Everyday Things](https://www.basicbooks.com/titles/don-norman/the-design-of-everyday-things/9780465050659/) by Don Norman. Not a DDD book, but a valuable foundation for understanding how people interact with designed systems.

**Articles**

- [Platform UX: Domain-Driven Design Concepts in Enterprise UX](https://www.uxmatters.com/mt/archives/2019/03/platform-ux-domain-driven-design-concepts-in-enterprise-ux.php) by Amy Chess. Discusses how DDD concepts can support systems thinking and enterprise UX.
- [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html) by Martin Fowler.
- [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html) by Martin Fowler.

### How UX/UI designers contribute in practice

Designers should participate in domain exploration, not only receive completed requirements and models.

They can contribute by:

- Observing users performing real business activities.
- Mapping user journeys and identifying business concepts.
- Participating in Event Storming and Domain Storytelling workshops.
- Validating terminology through user research.
- Identifying differences between user expectations and the domain model.
- Reviewing important domain changes from a usability perspective.
- Working with developers to ensure that UI behavior reflects business rules.

**Practical exercise:** Select a user journey that crosses multiple business capabilities. Map the user's goals, actions, decisions, and information needs. Identify which Bounded Contexts are involved and where terminology or workflow differences may confuse the user.

**You are ready to participate when:** You can explain the business meaning behind the screens and workflows you design, and can identify when a usability problem may reveal a deeper domain-modeling problem.

---

## 4. Architects and Technical Leaders

Architects and technical leaders help ensure that the software structure supports the domain model and the way the organization works.

Their responsibility is not to impose a preferred architecture before the domain is understood. Instead, they help the team make deliberate decisions about boundaries, dependencies, integration, deployment, and technical constraints.

### Key contributions to DDD

- Facilitate discussions about Subdomains and Bounded Contexts.
- Help distinguish domain boundaries from deployment and infrastructure boundaries.
- Evaluate modularity, coupling, and ownership.
- Select architectural approaches that support model integrity.
- Guide integration strategies between contexts.
- Identify operational consequences of distributed designs.
- Support incremental modernization and architectural evolution.
- Ensure that technical decisions do not silently redefine business concepts.

### Concepts worth understanding

- Strategic Design and Context Mapping.
- Layered, Hexagonal, and Clean Architecture.
- Modular Monoliths and Microservices.
- Dependency management and modularity.
- Messaging and Event-Driven Architecture.
- CQRS and Event Sourcing, including their trade-offs.
- Distributed consistency and transaction boundaries.
- Team Topologies and organizational boundaries.

### Recommended resources

- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.domainlanguage.com/ddd/) by Eric Evans.
- [Implementing Domain-Driven Design](https://www.pearson.com/en-us/subject-catalog/p/implementing-domain-driven-design/P200000003479) by Vaughn Vernon.
- [Patterns, Principles, and Practices of Domain-Driven Design](https://www.wiley.com/en-us/Patterns%2C+Principles%2C+and+Practices+of+Domain-Driven+Design-p-9781118714706) by Scott Millett and Nick Tune.
- [Team Topologies](https://teamtopologies.com/book) by Matthew Skelton and Manuel Pais.

**Practical exercise:** Take an existing system and compare its technical modules, deployment units, team ownership, and business capabilities. Identify where these boundaries align and where they conflict. Propose one improvement and explain its trade-offs.

**You are ready to participate when:** You can explain how architectural choices support or constrain the domain model, and can justify those choices in terms of business, team, and operational needs.

---

## 5. Testers and Quality Engineers

Testers and quality engineers help validate whether the software behaves according to the business rules and whether important scenarios are represented correctly.

Their contribution goes beyond finding defects after implementation. They can participate in discovering domain behavior, identifying exceptional cases, and challenging incomplete or ambiguous specifications.

### Key contributions to DDD

- Explore business scenarios with domain experts.
- Identify missing examples and exceptional cases.
- Help turn business rules into verifiable specifications.
- Participate in Example Mapping and collaborative specification.
- Validate behavior across Bounded Contexts.
- Identify integration and consistency problems.
- Help protect existing behavior during model evolution.
- Improve the readability and reliability of automated tests.

### Concepts worth understanding

- Ubiquitous Language.
- Domain Events and business scenarios.
- Example Mapping.
- Specification by Example.
- Acceptance Testing.
- Characterization Testing.
- Contract Testing.
- Integration Testing.

### Recommended resources

- [Specification by Example](https://www.manning.com/books/specification-by-example) by Gojko Adzic.
- [Growing Object-Oriented Software, Guided by Tests](https://www.pearson.com/en-us/subject-catalog/p/growing-object-oriented-software-guided-by-tests/P200000003479) by Steve Freeman and Nat Pryce.
- [Working Effectively with Legacy Code](https://www.pearson.com/en-us/subject-catalog/p/working-effectively-with-legacy-code/P200000003479) by Michael Feathers.

**Practical exercise:** Choose one business rule and develop a set of concrete examples with a domain expert. Include normal behavior, boundary conditions, and exceptional scenarios. Work with developers to turn those examples into automated tests where appropriate.

**You are ready to participate when:** You can use examples to expose ambiguity in business rules and help the team verify behavior in terms that domain experts understand.

---

## Working together: DDD is a team activity

The roles above describe different areas of expertise, not separate stages in a hand-off process.

A common but ineffective approach is to let business stakeholders define requirements, analysts document them, architects design the system, developers implement it, and designers prepare the interface after the technical decisions have been made.

DDD encourages a more collaborative way of working.

Consider a simple example: a customer requests a refund.

- A domain expert explains the business conditions under which a refund is allowed.
- A product manager explains the customer and business outcomes that matter.
- A business analyst helps clarify the rules and exceptional cases.
- A UX designer explores how the customer and support agent understand and perform the process.
- A developer models the behavior and its invariants.
- A tester identifies scenarios that validate the rules.
- An architect considers the boundaries and integration responsibilities involved.

Each participant sees a different aspect of the same problem. The model becomes stronger when these perspectives inform one another.

## A practical checklist for every role

Regardless of your role, you should be able to contribute to the following activities:

- [ ] Understand the business problem before proposing a solution.
- [ ] Ask questions when terminology or behavior is ambiguous.
- [ ] Distinguish facts from assumptions.
- [ ] Use concrete examples to explain and validate business rules.
- [ ] Participate in developing the language of the model.
- [ ] Recognize that different Bounded Contexts may have different models.
- [ ] Communicate discoveries and disagreements openly.
- [ ] Treat the domain model as something that evolves.
- [ ] Respect the knowledge and expertise of other participants.

The purpose of role preparation is not to make everyone think alike. It is to make different kinds of expertise work together.

**The most valuable outcome is a team that can understand the domain together, challenge its own assumptions, and continuously improve the software model as its knowledge grows.**