### Domain Model Review Checklist

- [ ] Does the model use language understood by domain experts?
- [ ] Are ambiguous terms and context-specific meanings explicit?
- [ ] Does behavior live with the concepts responsible for it?
- [ ] Are invariants protected at clear boundaries?
- [ ] Are Entities and Value Objects distinguished by domain meaning?
- [ ] Are services coordinating behavior or hiding an anemic model?
- [ ] Are integration contracts explicit and owned?
- [ ] Can the team explain why each abstraction exists?
- [ ] Are tests written in terms of business behavior?
- [ ] What new insight would justify changing the model?

**Note:** A domain model review is not a compliance check against DDD patterns. The goal is to evaluate whether the model accurately represents the business, expresses its language, protects its rules, and helps the team reason about change. An abstraction is not valuable simply because it follows a pattern, and a design is not necessarily wrong because it does not use one. Be willing to challenge existing decisions, including decisions that once made perfect sense. The model should evolve as the team's understanding of the domain deepens.