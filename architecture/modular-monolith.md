# Modular Monolith

A modular monolith is one deployable application organized into explicit modules with well-defined responsibilities and dependencies. It can be a strong starting point for a system with multiple Bounded Contexts when independent deployment is not yet worth its operational cost.

Keep module interfaces explicit, prevent accidental cross-module database access, and make ownership visible. A modular monolith should make boundaries enforceable, not merely create folders with module names.
