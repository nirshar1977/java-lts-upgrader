---
name: assess-java-upgrade
description: This skill should be used when the user asks "how hard is it to upgrade to Java 25", "assess my Java upgrade", "can we move to Spring Boot 4", "what blocks moving to Java 21", "check upgrade readiness", or wants a report before running a Java LTS or Spring Boot upgrade on a Maven project. It inspects the project read-only and produces an upgrade-readiness report without changing code.
argument-hint: "[java-target: 17|21|25] [--spring <2.7|3.0|3.5|4.0|4.1>]"
---

# Assess Java LTS upgrade readiness (read-only)

Produce a readiness report for upgrading the current Maven project to the target LTS (default 25). Change nothing in the project except writing the report file.

## Steps

1. Run Step 0 items 1, 3, 4, 5 and 6 of the `upgrade-java` skill (detect build tool, Maven version, current Java level per module, frameworks, installed JDKs). Do not create a branch.
2. List the hops required and, for each hop, read its reference in `${CLAUDE_PLUGIN_ROOT}/skills/upgrade-java/references/`.
3. Inventory risks, using `<mvn> -B -q dependency:tree -DoutputType=text` and source greps:
   - Dependencies/plugins below the hop minimums (Lombok, Mockito, ByteBuddy, JaCoCo, compiler/surefire, Spotbugs, Kotlin, MapStruct).
   - Usage of removed or restricted APIs: `javax.xml.bind`, `javax.annotation`, Nashorn (`ScriptEngineManager` with "nashorn"/"js"), `SecurityManager`, `Thread.stop`, `finalize()`, `sun.misc.Unsafe`, `sun.*`/`com.sun.*` internals, boxed-primitive constructors, `new URL(`, `new Locale(`.
   - `--add-opens`/`--illegal-access` flags in poms, `.mvn/jvm.config`, Dockerfiles and start scripts.
   - Spring stack (see `upgrade-spring` Step 0 item 2): Boot and Spring Cloud versions, train mismatches, managed-version overrides, `WebSecurityConfigurerAdapter`/`antMatchers`/`authorizeRequests`, `@MockBean`, `spring.factories` auto-config, Sleuth, Undertow, Jackson custom serializers/modules, Hibernate `@Type`, external WAR server versions.
   - Build the combined Java + Spring plan from `${CLAUDE_PLUGIN_ROOT}/skills/upgrade-spring/references/compatibility.md` (e.g. Boot 2.x cannot reach Java 25 without Boot 3.5).
   - `javax.*` EE imports and the runtime that would need to support Jakarta.
   - JUnit 4 tests.
   - Optional: if the OpenRewrite mode check (`references/openrewrite-setup.md`) succeeds, run `rewrite-maven-plugin:dryRun` with `UpgradeToJava<target>` and count changed files from `target/rewrite/rewrite.patch`; then `git checkout .` to discard any build output side effects.
4. Rate each Java hop and Spring step **Low / Medium / High** effort with one line of reasoning.
5. Write `upgrade-reports/ASSESSMENT.md` with: current state, hop table with ratings, blocker list (must fix), warnings (should review), JDKs to install, and the exact command to run the upgrade (`/upgrade-java <target> --spring <version>` or `/upgrade-spring <version>`).
6. Reply with the headline: current → target, number of hops, top 3 blockers, and where the full report is.
