# Domain-Driven Design Roadmap

A practical, progressive guide to understanding and applying Domain-Driven Design. It brings together domain exploration, strategic design, tactical modeling, architecture, implementation, and the continuous evolution of domain models.

This repository is based on the author's learning, experience, and interpretation of DDD literature and practice. It is a learning aid, not a definitive specification. Some topics are core DDD; others are complementary techniques or optional specializations. Those distinctions are made explicit throughout.

## Start here

- [The learning roadmap](roadmap/README.md): the recommended sequence, learning outcomes, and practical milestones.
- [DDD cheatsheet](cheatsheet/README.md): concise comparisons, reminders, and decision prompts.
- [Concepts](concepts/README.md): a reference for the vocabulary and building blocks.
- [Practices](practices/README.md): workshops and hands-on modeling techniques.
- [Architecture](architecture/README.md): architectural styles and integration approaches that can support DDD.
- [Checklists](checklists/README.md): practical prompts for workshops and design reviews.
- [Resources](resources/README.md): books, articles, talks, tools, and communities.

## How to use this repository

If you are new to DDD, follow the roadmap from foundations through strategic and tactical design. If you already work with DDD, use the cheatsheets and concept pages as a reference. If you are applying DDD to an existing system, start with domain exploration and the legacy-system material rather than attempting a full rewrite.

The roadmap is intentionally not a strict certification path. Revisit earlier stages as your understanding of the domain changes. Some subjects, such as Event Sourcing, Wardley Mapping, Team Topologies, and Data Mesh, are optional or adjacent disciplines rather than prerequisites for DDD.

## What DDD is useful for

DDD is most valuable when business rules, language, collaboration, and change are significant sources of complexity. It helps teams develop a shared understanding of the domain, make model boundaries explicit, and shape software around meaningful concepts. It is not automatically the right investment for every application. A small CRUD application with little domain complexity may not need the full set of DDD practices.

## Repository map

| Area | Purpose |
|---|---|
| `roadmap/` | Progressive learning path and stage-by-stage outcomes |
| `concepts/` | Definitions and explanations of DDD concepts |
| `practices/` | Domain exploration, collaborative modeling, and refactoring |
| `architecture/` | Architectural options that complement domain modeling |
| `cheatsheet/` | Short comparisons and design prompts |
| `checklists/` | Reusable workshop and review checklists |
| `resources/` | Curated further reading and tools |

## About the material

The content reflects a practical interpretation of established DDD ideas and related software design practices. Where a topic is an interpretation, an optional technique, or an adjacent discipline, it should not be mistaken for a rule prescribed by DDD itself. Validate recommendations against your domain, constraints, team, and business goals.

## Contributing

Corrections, examples, useful resources, and improvements are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

If this repository helps you, consider starring it on GitHub so others can discover it.
