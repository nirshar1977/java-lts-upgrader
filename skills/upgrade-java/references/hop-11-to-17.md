# Hop: Java 11 → 17

## Recipes (in this order, each its own invocation)

1. `org.openrewrite.java.migrate.UpgradeToJava17` — sets the release to 17, bumps build plugins, replaces deprecated APIs, adopts some 17 idioms.
2. Spring Boot below 2.7 → `org.openrewrite.java.spring.boot2.UpgradeSpringBoot_2_7` (Boot < 2.5 does not support Java 17). Commit this inside the hop.
3. Spring Boot 2.x → 3.x (only if the user approved the Spring major upgrade in the plan): `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_5`. It includes the `javax` → `jakarta` move. Make this a **separate commit** after the Java 17 build is green: `chore: upgrade Spring Boot 2.7 → 3.5`.
4. Non-Spring project with `javax.*` EE imports, and `--no-jakarta` not set, and the user approved it: `org.openrewrite.java.migrate.jakarta.JavaxMigrationToJakarta` — warn that the app server/runtime must also support Jakarta EE 9+ (e.g. Tomcat 10+, WildFly 27+). Separate commit.
5. JUnit 4 present → offer `org.openrewrite.java.testing.junit5.JUnit4to5Migration` (optional, separate commit).

## Manual checklist (also verify after recipes)

Build:
- `maven-compiler-plugin` ≥ 3.11 with `<release>17</release>`.
- `maven-surefire-plugin`/`failsafe` ≥ 3.x.
- Lombok ≥ 1.18.22, Mockito ≥ 4, ByteBuddy ≥ 1.12, JaCoCo ≥ 0.8.7, MapStruct ≥ 1.5, Spotbugs/PMD/Checkstyle versions that understand 17 syntax.
- Remove `--illegal-access=permit` (ignored since 17). Strong encapsulation is now enforced: reflective access into JDK internals fails with `InaccessibleObjectException`; fix by upgrading the library, last resort `--add-opens` in surefire `argLine` and runtime JVM args — note each one in the summary.

Removed/changed APIs:
- Nashorn JavaScript engine removed (15) → add `org.openjdk.nashorn:nashorn-core` or replace.
- RMI Activation removed; Applet API deprecated for removal.
- `SecurityManager` deprecated for removal (17) — flag usage, don't rewrite.
- `new Integer(..)` / boxed constructors deprecated for removal → `Integer.valueOf(..)`.
- Pack200 tools removed.
- Already removed in 11 but often hidden by old deps: JAXB, JAX-WS, `javax.annotation` (`@PostConstruct`), JavaFX — need explicit dependencies.

Behaviour to flag for review:
- Default TLS: TLS 1.0/1.1 disabled.
- `Thread.stop/suspend/resume` usage.
