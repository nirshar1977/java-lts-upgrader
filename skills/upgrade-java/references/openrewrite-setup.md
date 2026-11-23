# OpenRewrite setup for Maven

Run recipes from the command line so the project's `pom.xml` never needs a permanent rewrite plugin entry.

## Command form

```bash
<mvn> -B -U org.openrewrite.maven:rewrite-maven-plugin:run \
  -Drewrite.recipeArtifactCoordinates=<module-coords>[,<more-coords>] \
  -Drewrite.activeRecipes=<recipe-id>[,<more-ids>] \
  -Drewrite.exportDatatables=true
```

- Use `rewrite-maven-plugin:dryRun` instead of `:run` for `--dry-run`; the patch lands in `target/rewrite/rewrite.patch` (per module in multi-module builds).
- Run rewrite with the **target** JDK of the hop on `JAVA_HOME` when that JDK is installed; the recipes parse sources with the running JDK, and parsing Java 21+ syntax on an older JDK fails.
- Large projects: add `-Drewrite.pomCacheEnabled=true` and give Maven memory via `MAVEN_OPTS="-Xmx4g"`.

## Recipe modules

| Module coordinates | Used for |
|---|---|
| `org.openrewrite.recipe:rewrite-migrate-java:RELEASE` | `UpgradeToJava17/21/25`, Jakarta (`jakarta.JavaxMigrationToJakarta`, `jakarta.JakartaEE10`, `jakarta.JakartaEE11`) |
| `org.openrewrite.recipe:rewrite-spring:RELEASE` | `boot2.UpgradeSpringBoot_2_7`, `boot3.UpgradeSpringBoot_3_5`, `boot4.UpgradeSpringBoot_4_0` |
| `org.openrewrite.recipe:rewrite-testing-frameworks:RELEASE` | JUnit 4 → 5, Mockito upgrades |

Recipe IDs are fully qualified with the prefix `org.openrewrite.java.migrate.` or `org.openrewrite.java.spring.` respectively. Confirm an ID exists with `rewrite-maven-plugin:discover` if a run reports "recipe not found".

## Access mode (decide during preflight)

Recent builds of these recipe modules are published under the Moderne Source Available License, and new binaries may be hosted only on the Code Genome Project repository, which needs a sign-in token. Decide the mode like this:

1. **Credentials present?** If `~/.m2/settings.xml` has a server/repository with id `codegenome` (URL `https://artifacts.codegenomeproject.org/maven`), use mode `rewrite` with `RELEASE` coordinates.
2. **Try public resolution.** Otherwise run a cheap probe: `<mvn> -B -U org.openrewrite.maven:rewrite-maven-plugin:discover -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-migrate-java:RELEASE -q`. If it succeeds and lists `UpgradeToJava25`, use mode `rewrite`. If it resolves but lacks the needed recipe, the public copy is too old for that hop — use it for earlier hops and `manual` for the rest.
3. **Ask once.** If resolution fails, tell the user in plain words: the automated rewrite recipes need a free Code Genome Project token (or a locally built copy of the recipe modules). Offer: (a) they add the token to `settings.xml` and you retry, or (b) continue in `manual` mode.
4. **Manual mode.** Apply each hop checklist yourself: build-file changes first, then source changes guided by compiler errors and the deprecation list in the hop reference. Expect more fix-loop iterations.

Never write the user's token into the project, the commit, or the reports.
