---
name: upgrade-spring
description: This skill should be used when the user asks to "upgrade Spring Boot", "migrate to Spring Boot 3", "move to Spring Boot 4", "upgrade Spring Cloud/Security/Data", "upgrade Spring and Java together", or "go from Boot 2.x to the latest" in a Maven project. It upgrades Spring Boot one generation at a time (2.x → 2.7 → 3.0 → 3.5 → 4.0/4.1), aligns Spring Cloud and related libraries, upgrades Java first when a Spring step requires it, and commits after each verified step.
argument-hint: "[target: 2.7|3.0|3.5|4.0|4.1] [--java <17|21|25>] [--dry-run]"
---

# Upgrade Spring Boot (and the Java it needs) in a Maven project

Upgrade Spring Boot from the detected version to the target (default **latest 4.x GA**), one generation at a time. Each step must compile and pass tests before the next one. Spring steps and Java steps are interleaved in one plan when both are needed.

## Arguments

- `target` — 2.7, 3.0, 3.5, 4.0 or 4.1. Default: newest 4.x GA on Maven Central.
- `--java <N>` — also upgrade Java to LTS N (17, 21, 25). Without it, Java is upgraded only as far as a Spring step requires.
- `--dry-run` — produce patches in `upgrade-reports/` without keeping changes (same rules as `upgrade-java`).

## Step 0 — Preflight

1. Run the `upgrade-java` preflight (`${CLAUDE_PLUGIN_ROOT}/skills/upgrade-java/SKILL.md`, Step 0, items 1–8): Maven project, clean git tree, branch `spring-upgrade/<from>-to-<target>`, Maven command, current Java level, JDKs, green baseline build, OpenRewrite access mode.
2. Detect the Spring stack in every module: Boot version (parent `spring-boot-starter-parent` or imported `spring-boot-dependencies` BOM), Spring Cloud BOM (`spring-cloud-dependencies`) version, explicit overrides of managed versions (`<spring-framework.version>`, `<hibernate.version>`, `<jackson-bom.version>`, etc.), and which areas are used: Web MVC/WebFlux, Security, Data JPA/Mongo, Batch, Cloud (Gateway, OpenFeign, Config, Eureka), Actuator, Kafka, springdoc, Testcontainers, embedded container (Tomcat/Jetty/Undertow), WAR deployment to an external server.
3. Note non-Boot Spring projects (plain Spring Framework without Boot): use `org.openrewrite.java.spring.framework.UpgradeSpringFramework_6_0` / `_6_2` / `_7_0` in place of the Boot recipes and follow the same steps.
4. Build the **combined plan** with `references/compatibility.md` — the ordered list of Java hops and Spring steps, each gated by its requirements. Example from Java 11 + Boot 2.5 to Java 25 + Boot 4.0:
   1. Boot 2.5 → 2.7 (on Java 11)
   2. Java 11 → 17
   3. Boot 2.7 → 3.0 (Jakarta move)
   4. Boot 3.0 → 3.5
   5. Java 17 → 21
   6. Java 21 → 25
   7. Boot 3.5 → 4.0
5. Present the plan in plain language, with the breaking areas the user should know about (Jakarta, Security config rewrite, Jackson 3, removed Undertow, external server version). Ask once whether to proceed with the full plan or stop at an earlier step. If the user is not present, proceed with the full plan to the requested target.

## Step 1 — Execute the plan

For each item in order:

- **Java hop** → follow `upgrade-java` Step 1 for that hop (its references in `${CLAUDE_PLUGIN_ROOT}/skills/upgrade-java/references/`), but skip that hop's own Spring recipes; Spring is handled by the Spring steps in this plan.
- **Spring step** → read `references/spring-steps.md` for that step, then:
  1. Run the step's OpenRewrite recipe (module `org.openrewrite.recipe:rewrite-spring:RELEASE`; command form in `${CLAUDE_PLUGIN_ROOT}/skills/upgrade-java/references/openrewrite-setup.md`). In `manual` mode, apply the step checklist by hand.
  2. Align the Spring Cloud release train and other non-Boot-managed libraries to the versions in `references/compatibility.md`.
  3. Remove managed-version overrides that the new Boot version makes obsolete or incompatible; keep only overrides the project clearly needs and note them.
  4. Build on the hop's JDK: `<mvn> -B clean verify`. Log to `upgrade-reports/spring-<from>-<to>-build.log`.
  5. Fix loop — use `references/spring-troubleshooting.md` first, then `${CLAUDE_PLUGIN_ROOT}/skills/upgrade-java/references/troubleshooting.md`. Same rules as `upgrade-java`: root error first, no skipped or disabled tests, max 8 iterations without progress, then stop and report.
  6. Also start the application context once if the project has a `@SpringBootTest` or the user gave a start command; property and auto-configuration problems often appear only at startup.
  7. Commit: `chore: upgrade Spring Boot <from> → <to>` with recipes and manual fixes in the body.

## Step 2 — Finish

Write or extend `upgrade-reports/SUMMARY.md` with a Spring section: Boot/Cloud/Framework/Security/Hibernate/Jackson versions before → after, per-step commit hashes, property renames, security configuration changes, behaviour changes needing review (from `references/spring-steps.md`), and temporary aids that must not ship (e.g. `spring-boot-properties-migrator`, `spring-boot-starter-classic`, Jackson 2 compatibility modules) — remove them before the final commit unless the user wants to keep them. Reply with the final versions, what needs review, and that the branch is ready to push.
