---
name: flink-contributor
description: Apache Flink contribution conventions. Use when working on the apache/flink repo or any Flink subproject. Covers the Jira-first process, PR naming, code style, stability annotations (@Public, @PublicEvolving, @Internal, @Experimental), checkstyle, test patterns, and FLIP process. Pairs with oss-codebase-explorer for the general workflow.
---

# Apache Flink Contributor Conventions

Project-specific rules for contributing to Apache Flink. These override or supplement the general OSS contribution workflow.

## 1. Contribution Process (Jira-First)

Flink requires consensus **before** you open a PR:

1. **Create or find a Jira ticket** at https://issues.apache.org/jira/projects/FLINK
2. **Discuss the approach** on the Jira ticket or the dev@flink.apache.org mailing list.
3. **Get assigned** — only committers can assign tickets. Comment expressing interest and wait.
4. **Implement** only after you are assigned and there is consensus on approach.
5. **Open a PR** referencing the Jira ticket.

> ⚠️ PRs for unassigned Jira tickets or not authored by the assignee will not be reviewed or merged.

**Exceptions**: `[hotfix]` PRs for trivial fixes (typos in JavaDocs, documentation) do not need a Jira ticket.

**Starter issues**: Filter by the `starter` label in Jira for good first contributions.

## 2. PR Naming Convention

```
[FLINK-XXXX][component] Short imperative description
```

- `FLINK-XXXX` is the Jira issue number
- `component` matches the Jira component (e.g. `runtime`, `table`, `connectors`, `docs`)
- Hotfixes: `[hotfix][docs] Fix typo in event time introduction`

## 3. PR Structure

- **Fill out the PR template** completely — the description must answer "What / Why / How" without looking at the code.
- **Separate commits**: cleanup and refactoring in one commit, core behavioral changes in another. This is a critical requirement, not optional.
- **Reference open architecture questions** explicitly in the PR description if you made design choices not covered by the Jira discussion. See https://github.com/apache/flink/pull/8290 as an example.

## 4. Stability Annotations

All classes in `flink-core` and public-facing modules **must** have a stability annotation:

| Annotation | Can change in | Meaning |
|---|---|---|
| `@Public` | Major releases only | Stable API, guaranteed backward compatible within major version |
| `@PublicEvolving` | Major or minor releases | Public use, but interface may change across minor versions |
| `@Experimental` | Any release | New feature, may change or be removed at any time |
| `@Internal` | Any release | Not for public use; internal implementation detail |

**Rules**:
- New public APIs should start as `@Experimental` or `@PublicEvolving`, never `@Public` directly.
- Promotion path: `@Experimental` → `@PublicEvolving` (after 2 releases) → `@Public` (after 2 more releases).
- When modifying an existing `@Public` API, ensure backward compatibility.
- When deprecating: add `@Deprecated` with a `Since X.X.X` comment and create a follow-up Jira for removal.
- The annotation goes on the class/interface, and optionally on methods within `@Public` classes that have different stability.

```java
import org.apache.flink.annotation.PublicEvolving;

@PublicEvolving
public class MyNewProcessor {
    // ...
}
```

## 5. Code Style & Formatting

Flink uses **Spotless** and **Checkstyle**. Set up your IDE following https://flink.apache.org/how-to-contribute/code-style-and-quality-formatting/

### Hard rules:
- **Apache license header** in every file — the RAT plugin checks this on build.
- **No wildcard imports** — they cause merge conflicts and ambiguity.
- **No unused imports**.
- **Import order** — alphabetical, grouped into blocks separated by blank lines:
  ```
  org.apache.flink.*

  org.apache.flink.shaded.*

  <third-party>

  javax.*

  java.*

  scala.*
  ```
- **Empty line** before and after package declaration.
- **Package names**: lowercase, no underscores, no uppercase letters.
- **Constants** (`static final`): `UPPER_SNAKE_CASE`.
- **Fields/methods** (non-static): `lowerCamelCase`.

### Code quality:
- Prefer **immutable types** for APIs, messages, identifiers, configuration.
- **Avoid deep nesting** — flip conditions and exit early.
- **JavaDocs**: meaningful content, not just restating the method name to satisfy Checkstyle.
- **Keep methods short** — long methods hurt code cache locality and readability.
- Use `org.apache.flink.core.testutils.CheckedThread` if you need threads that propagate exceptions in tests.

## 6. Build & Test

```bash
# Full build (takes a while)
mvn clean install -DskipTests

# Build specific module
mvn clean install -pl flink-runtime -DskipTests

# Run tests for a module
mvn verify -pl flink-runtime

# Run a single test
mvn verify -pl flink-runtime -Dtest=MyTest

# Check formatting
mvn spotless:check

# Apply formatting fixes
mvn spotless:apply
```

### Test conventions:
- Test class name: `<ClassUnderTest>Test.java` or `<ClassUnderTest>ITCase.java` for integration tests.
- Use JUnit 5 (`@Test`, `@BeforeEach`, etc.) for new tests.
- **Never ignore a test without a comment** explaining why.
- Use `MiniCluster` for integration tests that need a running Flink environment.
- Use `assertThat` (AssertJ) over plain `assertEquals` where possible.

## 7. FLIP Process

For significant changes, a **Flink Improvement Proposal (FLIP)** is required:
- FLIPs live at https://cwiki.apache.org/confluence/display/FLINK
- A FLIP needs discussion and a vote on the dev mailing list.
- Don't start implementing before the FLIP is accepted.

## 8. Mailing Lists

- **dev@flink.apache.org** — design discussions, FLIPs, release planning
- **user@flink.apache.org** — usage questions
- Always prefer mailing list + Jira over private channels.

## 9. Common Gotchas

- **Checkstyle failures block merge** — run `mvn spotless:check` locally.
- **CI is slow** (~2-3 hours) — validate locally before pushing.
- **Flaky tests exist** — if CI fails on an unrelated test, check if it's a known flaky test before re-running.
- **Don't add dependencies lightly** — Flink is careful about dependency management. Discuss on Jira first.
- **Shaded dependencies** — Flink shades many libraries (Guava, Netty, etc.). Use the shaded versions, not direct dependencies.
- **Python contributions** — PyFlink has its own stability decorators mirroring Java annotations. Ensure parity.
