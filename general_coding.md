# General Coding

Language-neutral rules. Language- and framework-specific files build on these and win where they conflict.

## Code and comments

- Comments explain *why*: design decisions, trade-offs, non-obvious constraints. Don't write comments that restate what the code does.
- When code relies on a library or external dependency for a non-obvious reason, say why in a comment where it is used.
- Handle edge cases explicitly. Throw specific exceptions with messages that say what was wrong and with which value. Never swallow exceptions.
- Don't copy-paste code. When a second place needs the same logic, move it into a common class or common project and call it from both, instead of duplicating it. This applies to tests too.

## Code review

- Make only high-confidence suggestions when reviewing code changes.

## Project structure

- Organize code by feature (feature folders), not by technical type.
- Keep separation of concerns: models, services and data access are separate layers. Services don't talk to the database directly; data access goes through repository interfaces.
- When creating new files or folders, state their purpose in one line in the reply.

## Testing

- Changes to critical paths always come with tests.
- Mock external dependencies (I/O, network, time, randomness) in unit tests.
- Do not write "Arrange", "Act" or "Assert" comments.
- Name tests by the language-specific convention. Where none is given, copy the style of nearby test files.

## Performance

- Measure before optimizing. Don't add caching or other optimizations without a measured need.

## Inversion of Control

- Register services as transient by default. Use singleton or scoped only when the service has to keep state.
