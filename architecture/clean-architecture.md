# Clean Architecture

Clean Architecture organizes code so that business rules are not controlled by frameworks, databases, or delivery mechanisms. Its central concern is dependency direction: outer details depend on inner policy, not the other way around.

It can support DDD, but it is not DDD itself. A project can have a clean dependency graph and still have a weak domain model.
