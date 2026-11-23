# Spring Boot step checklists

Each step: run the recipe, then verify every item below; the recipes cover most but not all of it.

## Boot 2.x → 2.7

- Recipe: `org.openrewrite.java.spring.boot2.UpgradeSpringBoot_2_7`.
- Spring Cloud → 2021.0.x latest SR.
- Security: move off `WebSecurityConfigurerAdapter` to `SecurityFilterChain` beans now (deprecated in 2.7, removed in 3.0). Recipe: `org.openrewrite.java.spring.boot2.WebSecurityConfigurerAdapter` if not already included.
- `spring.factories` auto-configuration registration → `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
- Fix all deprecation warnings — 3.0 removes them.

## Boot 2.7 → 3.0 (biggest break)

- Recipe: `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0` (includes `javax` → `jakarta`, properties, Security 6).
- Requires Java 17.
- Every `javax.servlet/persistence/validation/annotation/transaction` import → `jakarta.*`. Third-party libs must have Jakarta versions (e.g. Hibernate 6, Hibernate Validator 8, Jersey 3, EhCache 3.10 `jakarta` classifier, Apache HttpClient 5 for `RestTemplate`).
- Security 6: `authorizeRequests` → `authorizeHttpRequests`, `antMatchers/mvcMatchers` → `requestMatchers`, `@EnableGlobalMethodSecurity` → `@EnableMethodSecurity`; lambda DSL.
- Trailing-slash matching disabled by default (`/api/x/` no longer matches `/api/x`) — flag for review.
- Hibernate 6: ID generator and type changes (`@Type`, `@TypeDef` removed), HQL stricter, sequence naming defaults changed — review entity mappings and run DB integration tests.
- Actuator/metrics: Sleuth → Micrometer Tracing; `spring.sleuth.*` gone.
- Properties: `spring.redis.*` → `spring.data.redis.*`, `server.max-http-header-size` → `server.max-http-request-header-size`, and others (properties recipe handles most).
- Spring Cloud → 2022.0.x latest SR.

## Boot 3.0 → 3.5

- Recipe: `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_5` (chains 3.1 → 3.5).
- Spring Cloud → 2025.0.x latest SR (Gateway starters renamed: `spring-cloud-starter-gateway` → `spring-cloud-starter-gateway-server-webflux`, properties `spring.cloud.gateway.*` → `spring.cloud.gateway.server.webflux.*`).
- 3.2: `RestClient`/`JdbcClient` available; parameter-name discovery needs `-parameters` (the parent POM sets it; custom compiler config must keep it).
- 3.4: `@MockBean`/`@SpyBean` deprecated → `@MockitoBean`/`@MockitoSpyBean` (do it now — removed in 4.0).
- Testcontainers `@ServiceConnection` available; not required.
- Fix every deprecation warning before moving to 4.0.

## Boot 3.5 → 4.0

- Recipe: `org.openrewrite.java.spring.boot4.UpgradeSpringBoot_4_0` (Framework 7, Security 7, Cloud 2025.1, Hibernate 7.1, Testcontainers 2, springdoc 3, properties, `@MockBean` replacement).
- Modular starters: some starters renamed/split (e.g. `spring-boot-starter-web` → `spring-boot-starter-webmvc`); test slices need their own `-test` starters. `spring-boot-starter-classic` is a temporary bridge only.
- **Jackson 3**: package `com.fasterxml.jackson` → `tools.jackson` (annotations stay in `com.fasterxml.jackson.annotation`); `ObjectMapper` → `JsonMapper` builder style; custom serializers/modules need review. A Jackson 2 compatibility module exists — temporary only.
- Undertow support removed → switch to Tomcat or Jetty.
- Jakarta EE 11 / Servlet 6.1: external servers need Tomcat 11 / Jetty 12.1.
- JSpecify null-safety annotations across Spring APIs — Kotlin code may get new nullability errors.
- Security 7: remaining deprecated DSL methods removed; `and()` chaining gone.
- Spring Cloud → 2025.1.x latest SR (Spring Cloud 2025.0 does not work with Boot 4.0.1+).

## Boot 4.0 → 4.1

- Recipe `..._4_1` if available; else bump the parent/BOM version.
- Spring Cloud ≥ 2025.1.2. Review the 4.1 release notes for property deprecations.

## Behaviour changes to always list in the summary

Trailing-slash matching, Hibernate SQL/ID changes, Security default changes, Jackson date/null handling, actuator endpoint exposure defaults, and any property the properties-migrator reported.
