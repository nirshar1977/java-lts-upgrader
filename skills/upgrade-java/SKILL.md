---
name: upgrade-java
description: This skill should be used when the user asks to "upgrade Java", "migrate to Java 17/21/25", "move my project to the next LTS", "upgrade from Java 11", "bump the JDK", or "upgrade Spring Boot along with Java" for a Maven project. It upgrades the project one LTS hop at a time (11 → 17 → 21 → 25), running OpenRewrite recipes, fixing remaining build and test failures, and committing after each verified hop.
argument-hint: "[target-version: 17|21|25] [--no-spring] [--no-jakarta] [--dry-run]"
---

# Upgrade a Maven Java project across LTS versions

Upgrade the current Maven project from its detected Java version to the target LTS (default **25**), one LTS hop at a time. Never skip a hop: 11 → 17 → 21 → 25. Each hop must compile and pass tests before the next one starts.

## Arguments

- `target-version` — 17, 21 or 25. Default 25. Reject anything that is not an LTS above the current version.
- `--no-spring` — upgrade Java only; leave Spring Boot versions alone.
- `--no-jakarta` — do not run the `javax.*` → `jakarta.*` migration for non-Spring projects.
- `--dry-run` — run every OpenRewrite recipe with `rewrite:dryRun`, write the patch files into `upgrade-reports/`, and change nothing else.

## Step 0 — Preflight (stop on any failure)

1. Confirm a `pom.xml` exists at the project root. If it is a Gradle project, stop and say this plugin supports Maven only.
2. Confirm a git repository with a clean working tree (`git status --porcelain` empty). If dirty, ask the user to commit or stash. Create a branch `java-upgrade/<from>-to-<target>`.
3. Pick the Maven command: `./mvnw` if present, else `mvn`. Record the Maven version (`-v`); Java 25 builds need Maven ≥ 3.9 — if the wrapper is older, update `.mvn/wrapper/maven-wrapper.properties` to the latest 3.9.x as part of hop 21→25.
4. Detect the current Java level, in this order: `maven.compiler.release`, `maven.compiler.source`/`target`, `java.version` property, `maven-compiler-plugin` `<release>`/`<source>`, toolchains. Check every module in a multi-module build. Report mixed levels.
5. Detect frameworks: Spring Boot parent/BOM version, `javax.*` imports (`grep -rl "import javax\.\(servlet\|persistence\|validation\|annotation\|ws\|xml\.bind\|inject\|transaction\)" --include=*.java`), Lombok, Mockito, JaCoCo, ByteBuddy, Kotlin.
6. Locate the JDKs needed for each hop. Check `JAVA_HOME`, `/usr/libexec/java_home -V` (macOS), `~/.sdkman/candidates/java/`, `/usr/lib/jvm/`, `~/.jdks/`. For every hop the **target** JDK must exist. If one is missing, tell the user the exact install command (e.g. `sdk install java 25-tem`) and stop before that hop.
7. Run a baseline build on the current JDK: `<mvn> -B -q clean verify`. If the baseline already fails, stop and report — do not upgrade a broken build. Save the list of failing tests if the user chooses to proceed anyway.
8. Choose the OpenRewrite access mode — read `references/openrewrite-setup.md` and follow it. The result is one of `rewrite` (recipes available) or `manual` (Claude performs the edits from the hop checklists).

Present a short plan (current version, hops, frameworks found, rewrite mode, JDK per hop). In the same message, ask once (AskUserQuestion when available) about the changes that are breaking beyond the JDK itself, and only those that apply:
- Spring Boot major upgrades (2.x → 3.5 during hop 11→17; 3.5 → 4.0 after hop 21→25). Minor bumps needed for JDK support (→ 2.7, → 3.5) happen without asking.
- `javax` → `jakarta` for non-Spring projects (the runtime/app server must support it).
- JUnit 4 → 5.
If the user is not present, take the conservative choice (no Spring major upgrade, no Jakarta move, no JUnit move), say so at the top of the summary, and continue.

## Step 1 — Execute each hop

For each hop in order, read the matching reference before touching code:

| Hop | Reference | Java recipe |
|---|---|---|
| 11 → 17 | `references/hop-11-to-17.md` | `org.openrewrite.java.migrate.UpgradeToJava17` |
| 17 → 21 | `references/hop-17-to-21.md` | `org.openrewrite.java.migrate.UpgradeToJava21` |
| 21 → 25 | `references/hop-21-to-25.md` | `org.openrewrite.java.migrate.UpgradeToJava25` |

Within a hop:

1. **Java recipe.** Run the hop's Java recipe (command form in `references/openrewrite-setup.md`). In `manual` mode, apply the hop checklist by hand instead.
2. **Framework recipes.** Run the framework recipes the hop reference lists for what Step 0 detected (Spring Boot, Jakarta, testing libraries). Run them as separate invocations so each diff is reviewable. Skip Spring with `--no-spring`, Jakarta with `--no-jakarta`.
3. **Pin the version.** Make sure every module ends with a single source of truth: `<maven.compiler.release>N</maven.compiler.release>` (remove stale `source`/`target`). Update toolchains, `.java-version`, `.sdkmanrc`, `.tool-versions`, Dockerfiles (`FROM eclipse-temurin:N`), and CI files (`.github/workflows/*.yml` `java-version`, `.gitlab-ci.yml`, `Jenkinsfile`) to N.
4. **Build on the target JDK.** Set `JAVA_HOME` to the hop's target JDK and run `<mvn> -B clean verify`. Capture output to `upgrade-reports/hop-<from>-<to>-build.log`.
5. **Fix loop.** While the build fails, read `references/troubleshooting.md`, diagnose the first root error (not cascading ones), apply the smallest correct fix, and rebuild. Prefer upgrading a plugin/dependency over changing code; prefer changing code over suppressing warnings. Never delete or `@Disabled` a test, never add `-DskipTests`, never lower the target version to make it pass. Stop after **8** fix iterations without progress and report the remaining errors to the user with your diagnosis.
6. **Commit.** When `clean verify` passes, commit with message `chore: upgrade Java <from> → <to>` and a body listing recipes run and manual fixes. One commit per hop keeps every hop bisectable.

In `--dry-run` mode, run steps 1–2 with `rewrite:dryRun`, copy `target/rewrite/rewrite.patch` to `upgrade-reports/hop-<from>-<to>.patch`, `git checkout .`, and move to the next hop without building.

## Step 2 — Finish

1. Run one last `<mvn> -B clean verify` on the final JDK.
2. Write `upgrade-reports/SUMMARY.md`: start/end versions, each hop's commit hash, recipes applied, manual fixes and why, dependency/plugin versions changed, warnings worth a human look (e.g. removed `SecurityManager` usage, `finalize()` overrides, behavioural changes from the hop references), and any tests that were already failing at baseline.
3. Tell the user, in plain language: which version the project is on now, how many hops succeeded, what needs their review, and that the branch is ready to push. Do not push or open a PR unless asked.

## Rules

- Keep each hop's changes inside its own commit; never mix hops.
- Do not reformat unrelated code. If a recipe produces large formatting-only churn, mention it in the summary.
- Treat `--release` as authoritative; do not leave `--enable-preview` flags behind unless the project already used preview features deliberately.
- If the project depends on an internal library that is not yet compatible with the target JDK, stop at the last good hop and explain which library blocks the next one.
