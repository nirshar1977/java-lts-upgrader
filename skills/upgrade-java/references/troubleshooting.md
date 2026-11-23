# Troubleshooting build failures after a hop

Always start from the **first** error in the log. Map it below, fix, rebuild.

| Symptom | Likely cause | Fix |
|---|---|---|
| `Unsupported class file major version 61/65/69` | A build plugin/tool (ASM inside it) predates the JDK. 61=17, 65=21, 69=25 | Upgrade the plugin named in the stack trace (JaCoCo, Spotbugs, shade, Groovy/gmavenplus, Kotlin, ArchUnit). |
| `java.lang.NoSuchFieldError: ... JCTree` / `TypeTag :: UNKNOWN` during compile | Old Lombok or other annotation processor | Upgrade Lombok (see hop minimums); update `annotationProcessorPaths` too, not just the dependency. |
| `Java NN (NN) is not supported by the current version of Byte Buddy` | Old ByteBuddy via Mockito/Hibernate | Upgrade Mockito; pin `net.bytebuddy:byte-buddy` and `byte-buddy-agent` in `dependencyManagement`. |
| `InaccessibleObjectException: Unable to make field ... accessible: module java.base does not "opens"` | Library reflects into JDK internals | Upgrade the library first. Last resort: `--add-opens java.base/<pkg>=ALL-UNNAMED` in surefire `argLine` and runtime JVM options; record it. |
| `package javax.xml.bind / javax.annotation / javax.xml.ws does not exist` | APIs removed from the JDK (11) | Add `jakarta.xml.bind-api` + `org.glassfish.jaxb:jaxb-runtime`, `jakarta.annotation-api`, etc. Keep `javax` artifacts (`javax.xml.bind:jaxb-api` 2.3.x) only if the code still imports `javax`. |
| `package javax.servlet does not exist` after a Spring Boot 3/Jakarta step | Leftover imports or a dependency still on `javax` | Re-run the Jakarta recipe; replace the offending dependency with its `-jakarta` classifier or newer major. |
| `invalid target release: NN` / `release version NN not supported` | Maven running on an older JDK | Check `JAVA_HOME` and `mvn -v`; check toolchains.xml. |
| `error: reference to X is ambiguous` / `getFirst() in X cannot implement` | New JDK APIs clash (e.g. SequencedCollection in 21) | Rename the custom method or align its signature; qualify the call. |
| `warning: [removal]` turned into error by `-Werror` | Deprecated-for-removal API usage | Replace the API per the hop checklist; do not drop `-Werror`. |
| Tests fail only on the new JDK with date/number formatting differences | CLDR locale data changes (e.g. narrow no-break space before AM/PM since 20) | Fix the test expectation or use explicit patterns/locales; mention in summary. |
| `Mockito cannot mock this class` / inline mock maker errors | Mockito/ByteBuddy too old, or missing `-javaagent` on 21+ | Upgrade, then configure Mockito as a javaagent in surefire. |
| Spring context fails: `Failed to bind properties` / unknown property | Renamed Boot properties | Re-run the Boot properties recipe (`SpringBootProperties_3_x`/`_4_0`) or add `spring-boot-properties-migrator` temporarily to see renames; remove it before committing. |
| Surefire runs zero tests after JUnit 5 move | Old surefire or missing `junit-jupiter-engine`/vintage engine | Surefire ≥ 3.x; add `junit-vintage-engine` if JUnit 4 tests remain. |

If a failure is in a third-party library with no compatible release, stop at the last green hop and report the library, its version, and the upstream issue if you can find one.
