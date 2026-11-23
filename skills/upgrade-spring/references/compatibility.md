# Spring Boot ↔ Java ↔ Spring Cloud compatibility

Use this to order the combined plan. A step can start only when its "Java required" is met; insert the Java hop before it if not.

| Boot | Java required | Java max supported | Spring Framework | Spring Cloud train | Jakarta EE / Servlet | OpenRewrite recipe |
|---|---|---|---|---|---|---|
| 2.7.x | 8 | 21 (17 officially; works on 21 for most apps) | 5.3 | 2021.0.x (Jubilee) | javax / Servlet 4 | `org.openrewrite.java.spring.boot2.UpgradeSpringBoot_2_7` |
| 3.0.x / 3.1.x | 17 | 21 | 6.0 | 2022.0.x (Kilburn) | Jakarta EE 9/10, Servlet 5/6 | `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0` |
| 3.2.x / 3.3.x | 17 | 21+ | 6.1 | 2023.0.x (Leyton) | Jakarta EE 10 | `..._3_2`, `..._3_3` |
| 3.4.x | 17 | 23+ | 6.2 | 2024.0.x (Moorgate) | Jakarta EE 10 | `..._3_4` |
| 3.5.x | 17 | 25 | 6.2 | 2025.0.x (Northfields) | Jakarta EE 10 | `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_5` |
| 4.0.x | 17 | 25 | 7.0 | 2025.1.x (Oakwood) | Jakarta EE 11, Servlet 6.1 | `org.openrewrite.java.spring.boot4.UpgradeSpringBoot_4_0` |
| 4.1.x | 17 | 26 | 7.0.x | 2025.1.2+ | Jakarta EE 11, Servlet 6.1 | `..._4_1` if `discover` lists it; else run `_4_0`, then bump the version and fix manually |

Ordering rules:

1. Bring Boot 2.x to **2.7** before leaving Java 11 — 2.7 is the bridge release and runs on both 11 and 17.
2. Boot **3.0** needs Java **17**: do Java 11 → 17 first.
3. Go 2.7 → **3.0** as its own step (the Jakarta and Security 6 break), then 3.0 → **3.5** (cumulative minor changes). Don't jump 2.7 → 3.5 in one commit; failures are much harder to attribute.
4. Java **25** needs Boot **3.5+** (or 4.x). Boot 2.x / 3.0–3.4 must be upgraded before the Java 21 → 25 hop.
5. Go to **4.0** only from the latest **3.5.x**, with all 3.5 deprecation warnings fixed first (they become errors in 4.0).
6. Spring Cloud: always use the newest service release of the matching train; check Maven Central for the current SR.
7. External app servers (WAR deployments): Boot 3 needs Tomcat 10.1+/Jetty 12; Boot 4 needs Tomcat 11/Jetty 12.1 (Servlet 6.1). Flag this to the user — it is an infrastructure change, not just code.

Before relying on a "Java max" or Cloud train above for a new target, check spring.io's system requirements and Spring Cloud compatibility table for the target release; they move with each minor.
