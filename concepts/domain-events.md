# Domain Events

A Domain Event represents something meaningful that has happened in the domain. It is usually named in the past tense, such as `OrderPlaced` or `PaymentReceived`. It captures a fact rather than a request.

Domain Events help make business behavior explicit and can communicate changes to other parts of a model. When events cross a Bounded Context or process boundary, they often become Integration Events with a deliberately designed public contract. Do not expose internal domain objects as external event schemas by default.

Consider event ownership, naming, immutable facts, timestamps and identifiers where relevant, duplicate delivery, ordering, retries, and schema evolution.
