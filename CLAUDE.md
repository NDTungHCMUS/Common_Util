# CLAUDE.md

## Purpose

This is a **Java / Spring Boot** project for building small MVP demos — quick,
self-contained backend prototypes to explore an idea or pattern. **Nothing here
is production code.** Favor the simplest thing that works and demonstrates the
concept.

## How to work with me

1. **Think before do anything.** State assumptions. If a request has multiple
   interpretations, surface them — don't pick silently. If a simpler approach
   exists, say so. If something is unclear, stop and ask.
2. **Simplicity first.** Minimum code that solves the problem. No speculative
   features, abstractions, configurability, or error handling for impossible
   cases. If 200 lines could be 50, write 50.
3. **Surgical changes.** Touch only what the task needs. Match existing style.
   Don't refactor what isn't broken; flag unrelated dead code, don't delete it.
4. Ask enough questions, let me answer comprehensive before generate spec.md, plan.md

## Tech

- **Java 21**, **Spring Boot 3.x**, **Gradle** (`./gradlew`).
- Web: `spring-boot-starter-web`. Persistence: `spring-boot-starter-data-jpa`
- Tests: **JUnit 5** + Spring Boot Test; MockMvc for the web layer, Mockito for units.
- Build files (`build.gradle`, `gradlew`) are generated on first build — there is none yet.

## Code exploration

- The **codegraph** MCP server is registered globally (user scope), so it is
  available in this project for navigating and understanding the codebase.
  Each repo still needs its own `.codegraph/` index.
- Use codegraph's explore/search tools first for questions about how something
  works, where code lives, or before making edits — it returns relevant files
  and symbols in one call, faster than reading files blind.

## Commands

```bash
./gradlew bootRun                  # run the app
./gradlew test                     # run all tests
./gradlew test --tests ClassName   # run one test class
./gradlew build                    # full build + tests
```

## Conventions

- Layered structure: `controller` → `service` → `repository`; DTOs at the web edge,
  entities only inside the persistence boundary.
- Use **records** for DTOs, constructor injection (no field `@Autowired`),
  `Optional` for nullable returns.
- Validate request bodies with `jakarta.validation` (`@Valid` + annotations).
- Return clear contracts: proper status codes + a consistent error body; never
  leak stack traces or entities directly.
- Never catch bare `Exception`; use specific types. Use the project's SLF4J logger.
- No hardcoded secrets or magic numbers — constants and `application.yml`.
