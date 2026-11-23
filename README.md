# java-lts-upgrader

A Claude plugin that upgrades Maven projects across **Java LTS releases (11 → 17 → 21 → 25)** and **Spring Boot generations (2.x → 2.7 → 3.0 → 3.5 → 4.0/4.1)**, separately or together. It works one step at a time, so every step builds, passes tests and lands as its own commit.

## Skills

| Skill | What it does |
|---|---|
| `/upgrade-java [17\|21\|25] [--spring <ver>] [--no-spring] [--no-jakarta] [--dry-run]` | Java LTS upgrade. On Spring projects it adds the Spring Boot steps the Java target needs, or goes to the `--spring` target. |
| `/upgrade-spring [2.7\|3.0\|3.5\|4.0\|4.1] [--java <17\|21\|25>] [--dry-run]` | Spring Boot upgrade, plus Spring Cloud train alignment, Security, Hibernate, Jackson 3 and Testcontainers. Upgrades Java first whenever a Spring step requires it. |
| `/assess-java-upgrade [17\|21\|25] [--spring <ver>]` | Read-only readiness report (`upgrade-reports/ASSESSMENT.md`) with blockers, outdated libraries, risky APIs, and the combined Java + Spring plan with an effort rating per step. |

### Example combined plan (Java 11 + Boot 2.5 → Java 25 + Boot 4.0)

1. Boot 2.5 → 2.7
2. Java 11 → 17
3. Boot 2.7 → 3.0 (Jakarta, Security 6)
4. Boot 3.0 → 3.5
5. Java 17 → 21
6. Java 21 → 25
7. Boot 3.5 → 4.0 (Framework 7, Jackson 3, modular starters)

Each step gets its own commit.

## Requirements

- Maven project with git; clean working tree.
- The JDK for each Java target installed (e.g. `sdk install java 25-tem`).
- Network access to Maven repositories.
- **OpenRewrite recipes:** newer builds of `rewrite-migrate-java` / `rewrite-spring` may require a free Code Genome Project token in `~/.m2/settings.xml` (server id `codegenome`). Without it, the plugin falls back to Claude applying the migration checklists directly.

## Safety

Works on a new branch, never skips or disables tests, never pushes. Asks before breaking changes beyond what the target strictly needs. Stops at the last green step if a dependency blocks the next one.
