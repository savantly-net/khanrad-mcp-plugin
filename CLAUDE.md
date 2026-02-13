# CLAUDE.md

## Best Practices

Follow SOLID principles in all code:
- **Single Responsibility**: Each module, class, or function should have one reason to change.
- **Open/Closed**: Design for extension without modifying existing code.
- **Liskov Substitution**: Subtypes must be substitutable for their base types.
- **Interface Segregation**: Prefer small, focused interfaces over large general-purpose ones.
- **Dependency Inversion**: Depend on abstractions, not concretions.

Follow 12-Factor App principles where applicable:
- Store config in the environment, never in code.
- Treat backing services as attached resources.
- Keep dev/prod parity as close as possible.
- Strictly separate build, release, and run stages.
- Export services via port binding; keep processes stateless.
- Treat logs as event streams, not files.

General engineering standards:
- Write clean, readable code that favors clarity over cleverness.
- Keep functions small and focused on a single task.
- Prefer composition over inheritance.
- Handle errors explicitly at system boundaries; don't swallow exceptions.
- Avoid premature optimization -- measure first, then optimize.
- Keep dependencies minimal and up to date.

## Headkey Memory

At session start, call `ask` with "What do I know about khanrad-mcp-plugin?" to load prior context before doing any work.
When you learn something significant about khanrad-mcp-plugin (skill design, template patterns, plugin configuration), call `remember` with source: "khanrad-mcp-plugin" and relevant tags.
When you make a decision or choose an approach, call `believe` with the decision, confidence 0.7-0.9, and subject: "khanrad-mcp-plugin".

## Khanrad

At session start, if `.khanrad.json` exists in the project root, read it to get the project
slug and default board slug. Resolve slugs to IDs via `list-projects` and `list-boards`.
Use these IDs for all subsequent Khanrad operations in this session.
At session start, read the `khanrad://agent/tasks` resource to check for assigned issues before doing any work.
When starting a coding session, check if there are open issues on the board that match the current task context.
