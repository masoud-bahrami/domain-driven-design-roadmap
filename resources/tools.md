# Tools for DDD Work

Domain-Driven Design does not depend on specific tools, frameworks, or development environments. It is a way of understanding a business, developing models collaboratively, and expressing those models in software.

Nevertheless, the right tools can make domain exploration, modeling, communication, documentation, and implementation considerably more effective.

The tools below are grouped by the kind of work they support. They are recommendations, not requirements. Choose them according to your team's workflow, project constraints, budget, security requirements, and existing infrastructure.

## 1. Collaborative Domain Exploration

Domain exploration techniques such as EventStorming, Domain Storytelling, and Example Mapping require people to communicate, share knowledge, challenge assumptions, and make their understanding visible.

The primary requirement is a workspace where participants can contribute and change the model easily.

### Online Whiteboarding Tools

**[Miro](https://miro.com/)**

A collaborative online whiteboard with templates, sticky notes, diagrams, and integrations.

Useful for:
- Big Picture and Process-Level EventStorming.
- Domain Storytelling.
- Example Mapping.
- Collaborative workshops and remote brainstorming.

**[Mural](https://www.mural.co/)**

A collaborative visual workspace with workshop-oriented features, templates, and facilitation capabilities.

Useful for structured discovery sessions, collaborative modeling, and remote workshops.

**[FigJam](https://www.figma.com/figjam/)**

A collaborative whiteboarding tool within the Figma ecosystem.

Particularly useful when developers, product people, and UX/UI designers already use Figma.

**[Microsoft Whiteboard](https://www.microsoft.com/en-us/microsoft-365/microsoft-whiteboard)**

A digital whiteboard integrated with Microsoft's productivity ecosystem, including Microsoft Teams.

A practical choice for organizations already using Microsoft 365.

**[Excalidraw](https://excalidraw.com/)**

A lightweight, open-source virtual whiteboard with a hand-drawn visual style.

Useful for quick sketches, informal modeling, architecture discussions, and collaborative exploration without requiring a complex workspace.

**[tldraw](https://www.tldraw.com/)**

A lightweight collaborative canvas for sketching, diagramming, and visual exploration.

Useful for quickly expressing ideas and exploring relationships during design discussions.

### Physical Whiteboards and Sticky Notes

For in-person workshops, physical whiteboards, wall space, and sticky notes remain excellent tools.

They offer several advantages:
- Participants can move around and interact naturally.
- Large models can remain visible throughout a workshop.
- Sticky notes can be rearranged as understanding changes.
- Drawing and writing are immediate and accessible.
- The physical environment encourages active participation.

For EventStorming, the available wall space and visibility of the entire model can matter more than the sophistication of the digital tool.

Photographs can preserve workshop outcomes, but a photograph of a workshop is not a maintained domain model.

<text color="secondary" size="xs">Note: Google Jamboard was discontinued in 2024 and should no longer be recommended as an available whiteboarding product.</text>

## 2. Diagramming and Modeling

After exploring the domain, teams often need to communicate concepts, relationships, boundaries, workflows, and architectural decisions through diagrams.

Different diagrams represent different aspects of a system. A Context Map, a UML class diagram, and a deployment diagram are not interchangeable.

### Text-Based Diagramming

Text-based diagramming allows diagrams to be maintained as source files, reviewed through version control, and updated alongside software documentation.

**[PlantUML](https://plantuml.com/)**

A text-based diagramming tool supporting UML and other diagram types.

Useful for:
- Class and sequence diagrams.
- Component and deployment diagrams.
- Architecture documentation.
- Diagrams maintained alongside source code.

**[Mermaid](https://mermaid.js.org/)**

A text-based diagramming and visualization tool that renders diagrams from a simple textual syntax.

It integrates particularly well with Markdown, documentation repositories, GitHub, and technical wikis.

Useful for:
- Context Maps and relationship diagrams.
- Flowcharts and sequence diagrams.
- State diagrams.
- Architecture documentation.

Both PlantUML and Mermaid are useful when diagrams need to be version-controlled and reviewed as part of normal development workflows.

### Visual Diagramming Tools

**[diagrams.net (Draw.io)](https://www.diagrams.net/)**

A free diagramming application supporting flowcharts, UML, architecture diagrams, network diagrams, and other visual representations.

Useful for architecture documentation, Context Maps, process diagrams, and general technical communication.

**[Visual Paradigm](https://www.visual-paradigm.com/)**

A commercial modeling platform supporting UML, BPMN, enterprise architecture, and other modeling techniques.

Useful for teams that need more formal modeling capabilities and integrated modeling workflows.

**[Lucidchart](https://www.lucidchart.com/)**

A collaborative diagramming platform supporting flowcharts, UML, architecture diagrams, and process visualization.

Useful for teams that need shared diagrams and integration with existing collaboration platforms.

### Lightweight Diagramming and Sketching

**Excalidraw** and **tldraw** also work well for lightweight diagramming.

They are particularly useful when the objective is to communicate an idea rather than maintain a formal model.

For example, a quick sketch showing how Ordering depends on Inventory may be more effective during a discussion than a detailed UML diagram.

**A useful rule:** Start with the simplest representation that communicates the idea. Introduce more formal notation only when it adds clarity.

## 3. Knowledge Management and Documentation

DDD produces knowledge that should remain accessible and evolve with the business.

This includes:
- Ubiquitous Language.
- Bounded Context definitions.
- Context Maps.
- Business rules and constraints.
- Domain decisions and unresolved questions.
- Integration contracts.
- Modeling workshop outcomes.
- Architectural decisions.

Documentation should help people understand and discuss the domain. It should not become a separate activity in which glossaries and diagrams are maintained independently of the actual business and software.

### Knowledge Management Platforms

**[Confluence](https://www.atlassian.com/software/confluence)**

A collaborative documentation platform that can be used to maintain domain knowledge, architectural decisions, Context Maps, and workshop outcomes.

Useful when the organization already uses Atlassian products.

**[Notion](https://www.notion.com/)**

A flexible workspace for organizing documentation, databases, notes, and project knowledge.

Useful for maintaining lightweight domain documentation and collaborative knowledge bases.

**[Obsidian](https://obsidian.md/)**

A local-first knowledge management application based on Markdown files and linked notes.

Useful for connecting domain concepts, decisions, definitions, and technical documentation while keeping content in portable files.

### Version-Controlled Documentation

**[GitHub](https://github.com/)** and **[GitLab](https://about.gitlab.com/)** can be used to maintain domain documentation alongside source code.

For example:

```text
docs/
├── domain/
│   ├── ubiquitous-language.md
│   ├── bounded-contexts.md
│   └── context-map.md
│
├── decisions/
│   ├── 001-modular-monolith.md
│   └── 002-shipping-integration.md
│
└── diagrams/
    ├── context-map.mmd
    └── shipment-workflow.mmd
```

This approach makes changes reviewable, preserves history, and allows documentation to evolve through the same workflow as the software.

### Shared Documents and Spreadsheets

**[Google Docs and Sheets](https://workspace.google.com/)**

Useful for collaborative notes, simple glossaries, workshop summaries, and tracking open questions.

**[Microsoft Word and Excel](https://www.microsoft.com/microsoft-365)**

Useful for teams already working within Microsoft 365, particularly when formal documents and spreadsheets are part of their existing workflow.

These tools can be sufficient for many projects. A dedicated knowledge management platform is not a prerequisite for DDD.

## 4. Communication and Team Collaboration

Ubiquitous Language develops through conversations between developers, domain experts, and other stakeholders.

Communication tools can help preserve those conversations and make collaboration easier, especially in distributed teams.

**[Slack](https://slack.com/)**

A communication platform supporting channels, discussions, integrations, and asynchronous collaboration.

Useful for discussing domain questions, sharing discoveries, and keeping conversations accessible to the wider team.

**[Microsoft Teams](https://www.microsoft.com/microsoft-teams/group-chat-software)**

A collaboration platform combining messaging, meetings, file sharing, and integrations.

Useful for organizations already operating within the Microsoft ecosystem.

**[Jira](https://www.atlassian.com/software/jira)**

A work management and issue tracking platform.

It can help connect domain knowledge with development work through user stories, business rules, acceptance criteria, and decisions.

However, Jira should not become the only place where the domain model exists. A ticket describing a business operation is not a substitute for a shared understanding of the business.

## 5. Implementation and Code Generation

Tools and frameworks can help implement domain models, manage persistence, and reduce repetitive work.

However, distinguish between tools that support implementation and tools that actually help discover or express the domain.

### Object-Relational Mapping and Persistence

**[Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)**

An ORM for .NET that provides code-first modeling, database migrations, relationship configuration, and persistence capabilities.

It can support DDD implementations through:
- Explicit mapping configurations.
- Value Object and owned-type mappings.
- Private constructors and encapsulated state.
- Backing fields.
- Aggregate persistence.
- Database migrations.

EF Core does not generate a meaningful DDD model from a database schema. The domain model should guide persistence mapping, not be dictated by the ORM.

### Code Generation

**[JHipster](https://www.jhipster.tech/)**

A development platform for generating application structures and code using Java and related technologies.

It can accelerate the creation of applications and CRUD-oriented structures, but generated code should be reviewed and adapted to the actual domain model. Generating entities and relationships does not automatically produce well-designed Aggregates or Bounded Contexts.

**T4 Text Templates (.NET)**

T4 templates can generate repetitive code based on conventions and metadata.

They may be useful when a team has stable, well-understood implementation conventions and wants to reduce repetitive work.

Custom generation should follow established domain and architectural decisions rather than attempt to discover them.

### IDEs and Development Environments

**[Visual Studio](https://visualstudio.microsoft.com/)** and **[JetBrains Rider](https://www.jetbrains.com/rider/)** provide development, debugging, testing, refactoring, and code navigation capabilities for .NET applications.

**[IntelliJ IDEA](https://www.jetbrains.com/idea/)** provides a comprehensive development environment for Java and other supported languages.

These tools help implement and evolve the model, but the quality of the model still depends on the understanding behind the code.

## 6. Choosing the Right Tools

Different stages of DDD work benefit from different tools.

| DDD activity | Useful tools | Main purpose |
|---|---|---|
| EventStorming | Miro, Mural, FigJam, physical whiteboards | Explore business processes and domain events collaboratively |
| Domain Storytelling | Miro, Excalidraw, physical whiteboards | Communicate actors, activities, and work objects |
| Example Mapping | Miro, FigJam, physical cards | Clarify business rules through concrete examples |
| Context Mapping | diagrams.net, PlantUML, Mermaid | Visualize Bounded Context relationships |
| Architecture modeling | PlantUML, Mermaid, Visual Paradigm | Communicate software structures and dependencies |
| Ubiquitous Language | Markdown, Confluence, Notion, shared documents | Maintain and refine domain terminology |
| Knowledge management | GitHub, GitLab, Obsidian, Confluence | Preserve decisions and domain knowledge |
| Implementation | Visual Studio, Rider, IntelliJ IDEA | Develop and maintain the domain model |
| Testing | xUnit, NUnit, JUnit, Jest | Validate business behavior and implementation |
| Code generation | T4, JHipster, IDE tooling | Automate repetitive implementation work |

These are suggestions, not prescribed combinations. A team can perform effective DDD with a whiteboard, sticky notes, a text editor, and a test framework.

## 7. Principles for Choosing Tools

**Prefer adoption over novelty.**

A tool that the team already understands is often more valuable than a sophisticated tool that introduces friction.

**Optimize for collaboration.**

Domain exploration requires participation from people with different perspectives. Choose tools that make it easy for everyone to contribute, not only developers.

**Keep knowledge portable.**

Where possible, prefer formats that can be exported, version-controlled, searched, and maintained independently of a specific vendor.

**Do not confuse documentation with understanding.**

A beautifully maintained Context Map does not guarantee that the team understands the relationships between its Bounded Contexts. Documentation should reflect actual conversations and validated decisions.

**Avoid unnecessary tool proliferation.**

Using separate platforms for every activity can fragment knowledge and create maintenance overhead.

**Choose tools according to constraints.**

Budget, internet access, security requirements, organizational policies, and the team's existing infrastructure all matter. A technically excellent tool may be unsuitable if the organization cannot reliably access or maintain it.

## 8. A Minimal DDD Toolset

A team starting with DDD does not need an extensive collection of products.

A practical starting setup might include:

- A whiteboard or collaborative canvas for domain exploration.
- Markdown files or a shared workspace for Ubiquitous Language and decisions.
- Mermaid or diagrams.net for Context Maps and architecture diagrams.
- Git for versioning models and documentation.
- An IDE and automated test framework for implementation and validation.

Add specialized tools only when a real need emerges.

**The best DDD toolset is not the one with the most features. It is the one that helps people discover the domain, communicate meaning, challenge assumptions, and preserve the understanding they gain.**