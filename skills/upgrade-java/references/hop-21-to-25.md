# Hop: Java 21 → 25

## Gate

If the project is still on Spring Boot 2.x, do not start this hop: Boot 2.x does not support Java 25. Tell the user the project stays on Java 21 until the Boot 3 upgrade is approved, and offer to run it now.

## Recipes

1. `org.openrewrite.java.migrate.UpgradeToJava25` — release 25, `UpgradePluginsForJava25` (Maven plugin bumps), Spring Boot 3 parent bump for 25, deprecated API replacements.
2. Spring Boot 3.x below 3.5 → `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_5` (minimum for Java 25 support).
3. Spring Boot 4 (only if the user approved it in the plan): `org.openrewrite.java.spring.boot4.UpgradeSpringBoot_4_0` — brings Spring Framework 7, Security 7, Jackson 3, Hibernate 7.1, modular starters. Separate commit after the Java 25 build is green: `chore: upgrade Spring Boot 3.5 → 4.0`.
4. Jakarta EE 11 if the runtime supports it and the user approved: `org.openrewrite.java.migrate.jakarta.JakartaEE11`.

## Manual checklist

Build:
- Maven ≥ 3.9 (update the wrapper if older), `maven-compiler-plugin` ≥ 3.14, surefire/failsafe ≥ 3.5.
- Lombok ≥ 1.18.40 (earlier versions crash javac 25 with `NoSuchFieldError`/`TypeTag :: UNKNOWN`).
- ByteBuddy ≥ 1.17.7 and Mockito ≥ 5.19 (older ByteBuddy rejects class file version 69: "Java 25 (69) is not supported"). As a stopgap only: `-Dnet.bytebuddy.experimental=true`.
- JaCoCo ≥ 0.8.14 (older versions fail instrumenting class file 69).
- ASM-based tools (Spotbugs, shade plugin, ArchUnit, Byte Buddy agents, Kotlin compiler) need versions that know class file major 69. Kotlin modules: Kotlin 2.3+ for JVM target 25.
- Where an exact minimum above is uncertain, check Maven Central for the newest release and use it.

API/behaviour changes:
- `SecurityManager` permanently disabled since 24: `System.setSecurityManager` throws. Flag and remove usage with the user.
- 32-bit x86 port removed; irrelevant for most builds.
- `sun.misc.Unsafe` memory-access methods warn by default (JEP 498) — upgrade libraries (Netty, Guava, protobuf, etc.) that print the warning.
- JNI usage warns unless `--enable-native-access` is set (JEP 472) — add it to runtime args for apps that load native libraries, list in summary.
- Finalization still deprecated for removal — flag `finalize()` overrides.
- New features (compact source files, scoped values, flexible constructor bodies, module imports, compact object headers via `-XX:+UseCompactObjectHeaders`): do not adopt automatically; list as opportunities.
