# Spring upgrade troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `ClassNotFoundException: javax.servlet...` at startup on Boot 3+ | A dependency still compiled against `javax` | Find it with `dependency:tree -Dincludes=javax.servlet`; upgrade to its Jakarta version or `jakarta` classifier. |
| `NoSuchMethodError` / `ClassNotFoundException` in `org.springframework.*` | Mixed Spring versions — a hard-coded `spring-*` version or old Cloud train | Remove explicit Spring versions; align the Cloud BOM per `compatibility.md`; check `dependency:tree -Dincludes=org.springframework`. |
| Spring Cloud fails with "Spring Boot [x] is not compatible with this Spring Cloud release train" | Cloud train mismatch | Set the matching train from `compatibility.md`. Don't disable `spring.cloud.compatibility-verifier.enabled` except as a short diagnostic. |
| `Failed to bind properties` / app starts but feature silently off | Renamed property | Add `spring-boot-properties-migrator` (runtime scope) temporarily, start the app, apply the reported renames, remove it. |
| `HttpSecurity` method not found (`antMatchers`, `authorizeRequests`, `and()`) | Security 6/7 removals | Rewrite to `authorizeHttpRequests(a -> a.requestMatchers(...))` lambda DSL. |
| 404 for URLs ending in `/` after Boot 3 | Trailing-slash matching disabled | Fix the client/test URLs; only as a last resort add explicit trailing-slash mappings. Note it in the summary. |
| Hibernate `Unknown type`, `@Type` errors, sequence `..._seq` missing | Hibernate 6/7 mapping changes | Replace `@Type(type=...)` with `@JdbcTypeCode`/`@Convert`; set `@SequenceGenerator(allocationSize=...)` and names explicitly to match the existing DB. |
| `@MockBean` cannot be resolved (Boot 4) | Removed | `@MockitoBean` / `@MockitoSpyBean` from `org.springframework.test.context.bean.override.mockito`. |
| `package com.fasterxml.jackson.databind does not exist` (Boot 4) | Jackson 3 package move | Switch imports to `tools.jackson.databind`; update custom modules/serializers. Keep `com.fasterxml.jackson.annotation` imports. |
| `@WebMvcTest`/`@DataJpaTest` not found or context slice missing (Boot 4) | Test slices moved into modular test starters | Add the matching `spring-boot-starter-<module>-test` dependency. |
| Undertow classes missing (Boot 4) | Undertow removed | Replace `spring-boot-starter-undertow` with Tomcat (default) or Jetty; port Undertow-specific config. |
| Tests time out waiting for containers after Boot 4 | Testcontainers 2 API changes | Re-run `org.openrewrite.java.testing.testcontainers.Testcontainers2Migration`; update module artifact names. |
