# java-lts-upgrader

A Claude plugin that upgrades Maven Java projects across LTS releases — **11 → 17 → 21 → 25** — one hop at a time, so every step builds, passes tests and lands as its own commit.

## Skills

| Skill | What it does |
|---|---|
| `/upgrade-java [17\|21\|25] [--no-spring] [--no-jakarta] [--dry-run]` | Detects the current Java level, runs OpenRewrite migration recipes for each hop, fixes remaining compile/test failures, updates CI/Docker/toolchain files, commits per hop, and writes `upgrade-reports/SUMMARY.md`. |
| `/assess-java-upgrade [17\|21\|25]` | Read-only readiness report (`upgrade-reports/ASSESSMENT.md`): blockers, outdated plugins, risky APIs, effort per hop. |

Covers Spring Boot (2.7 / 3.5 / 4.0), `javax` → `jakarta`, Lombok, Mockito/ByteBuddy, JaCoCo, JUnit 4 → 5. Breaking framework upgrades are asked about first; JDK-compatibility bumps are applied automatically.

## Requirements

- Maven project with git; clean working tree.
- The JDK for each target hop installed (e.g. via SDKMAN: `sdk install java 25-tem`).
- Network access to Maven repositories.
- **OpenRewrite recipes:** newer builds of `rewrite-migrate-java` / `rewrite-spring` may require a free Code Genome Project token in `~/.m2/settings.xml` (server id `codegenome`). Without it, the plugin falls back to Claude applying the migration checklists directly.

## Safety

Works on a new branch `java-upgrade/<from>-to-<to>`, never skips or disables tests, never pushes. Stops at the last green hop if a dependency blocks the next one.
