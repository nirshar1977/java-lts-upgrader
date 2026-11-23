# Hop: Java 17 → 21

## Recipes

1. `org.openrewrite.java.migrate.UpgradeToJava21` — release 21, plugin bumps, deprecated API replacements, SequencedCollection idioms (`getFirst()`/`getLast()`), etc.
2. Spring Boot 3.0–3.1 → `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_5` (Java 21 is first-class from Boot 3.2). Spring Boot 2.7 on Java 21 works for most apps but is out of OSS support — recommend the Boot 3 upgrade from hop 11→17 if it was skipped.
3. Jakarta EE 9 → 10 if the runtime supports it and the user approved: `org.openrewrite.java.migrate.jakarta.JakartaEE10`.

## Manual checklist

Build:
- `maven-compiler-plugin` ≥ 3.11, `<release>21</release>`.
- Lombok ≥ 1.18.30, Mockito ≥ 5 (needs Java 11+; inline mock maker default), ByteBuddy ≥ 1.14.9, JaCoCo ≥ 0.8.11, Error Prone/Checker Framework recent enough for 21.
- Mockito/ByteBuddy on 21 may print "dynamic agent loading" warnings (JEP 451). Fix by adding Mockito as a `-javaagent` in surefire `argLine` (resolve the jar path via `maven-dependency-plugin:properties` → `${org.mockito:mockito-core:jar}`), not by silencing.

API/behaviour changes:
- `Thread.stop()` now always throws `UnsupportedOperationException`.
- `new URL(String)` constructors deprecated (20) → `URI.create(..).toURL()`.
- `Locale` constructors deprecated (19) → `Locale.of(..)`.
- `SequencedCollection` added: custom `List`/`Deque` implementations that defined `getFirst()`/`reversed()` with incompatible return types now fail to compile — rename or align signatures.
- Virtual threads available (JEP 444): do not adopt automatically; mention as a follow-up opportunity in the summary.
