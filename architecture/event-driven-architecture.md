# Event-Driven Architecture

Event-driven architecture uses events to communicate that something has happened or to trigger downstream work. It can reduce temporal coupling and support integration across Bounded Contexts.

Distinguish Domain Events inside a model from Integration Events published as external contracts. Define ownership, schema evolution, delivery guarantees, idempotency, retries, ordering expectations, and failure handling. Asynchronous communication does not remove coupling; it changes its form.
