---
name: opensearch-contributor
description: OpenSearch contribution conventions. Use when working on the opensearch-project/OpenSearch repo or any OpenSearch subproject (plugins, dashboards, data-prepper). Covers the issue-first model, PR template, gradle-check CI, changelog requirements, DCO sign-off, test randomization, and plugin conventions. Pairs with oss-codebase-explorer for the general workflow.
---

# OpenSearch Contributor Conventions

Project-specific rules for contributing to the OpenSearch Project. These override or supplement the general OSS contribution workflow.

## 1. Contribution Process (Issue-First)

OpenSearch follows an issue-first model:

1. **Open a GitHub issue** describing the problem or feature — even if you know the solution.
2. **Discuss** in the issue. Wait for maintainer feedback on approach before heavy implementation.
3. **Open a PR** referencing the issue.

> ⚠️ Exception: truly trivial changes (typo fixes) can skip the issue step.

**Good first issues**: Filter by `good first issue` label at https://github.com/opensearch-project/OpenSearch/contribute

## 2. Developer Certificate of Origin (DCO)

Every commit **must** be signed off with DCO:

```bash
git commit -s -m "Your commit message"
```

This adds a `Signed-off-by: Your Name <email@example.com>` line. The DCO check will block merge if this is missing. To retroactively sign off:

```bash
git commit --amend --signoff
# or for multiple commits:
git rebase --signoff HEAD~N
```

## 3. PR Conventions

### PR template
OpenSearch uses a standardized PR template at `.github/pull_request_template.md`. Fill out **every section**:
- Description of the change
- Issues resolved (link with `Resolves #XXXX`)
- Check list items (tests, docs, changelog, backwards compat)

### Commit messages
- Clear, imperative style: "Add retry logic to bulk indexer" not "Added retry logic"
- Reference the issue: "Resolves #12345" or "Related: #12345"

### PR size
- Keep PRs focused. Large PRs get slower reviews.
- If the change is big, break it into a stack.

## 4. Changelog Requirements

**Every user-facing change** must update `CHANGELOG.md`:

- Use **Keep A Changelog** format.
- Add entries under the `[Unreleased X.Y.0]` section.
- Categories: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.
- Format: `- Description of change ([#ISSUE_NUMBER](link))`.

On release, unreleased entries move to `./release-notes/opensearch.release-notes-{version}.md`.

## 5. Build & CI

OpenSearch uses **Gradle**. The `gradle-check` workflow is the primary CI gate.

```bash
# Full build
./gradlew assemble

# Run all tests
./gradlew test

# Run tests for a specific module
./gradlew :server:test

# Run a single test class
./gradlew :server:test --tests "org.opensearch.index.IndexSettingsTests"

# Run with specific Java version (OpenSearch requires JDK 21+)
export JAVA_HOME=/path/to/jdk21
./gradlew build

# Check for forbidden API usage
./gradlew forbiddenApis

# Precommit checks (formatting, forbidden APIs, etc.)
./gradlew precommit
```

### CI behavior:
- ✅ **Success**: green check, message includes commit SHA.
- ⚠️ **Flaky**: test failure summary with link to flaky test guidance — check if your change caused it or it's a known flaky test.
- ❌ **Failure**: examine the failure. Fix before requesting review.
- Use `skip-diff-analyzer` label to skip AI-powered code diff analysis if needed.

## 6. Code Style

### Java conventions:
- **Java 21** — use modern Java features where appropriate.
- Follow the existing style in the file you're editing.
- Use the project's Spotless/Checkstyle configuration — don't fight it.
- **No `@author` tags** — use `git blame` instead.
- **License header**: Apache 2.0 in every file.

### Naming:
- Follow existing patterns in the module.
- Plugin names: don't include the word "plugin" for official plugins within the org.
- Test classes: `<ClassUnderTest>Tests.java` (note: plural `Tests`, not `Test` — OpenSearch convention differs from Flink).

### Package structure:
- Core: `org.opensearch.*`
- Plugins: `org.opensearch.<plugin-name>.*`
- Note: If you see references to `org.elasticsearch`, open an issue — those are legacy references from the fork.

## 7. Test Conventions

### Randomized testing:
OpenSearch uses **extensive randomization** in tests to cover edge cases. This means:
- Tests may occasionally fail without code changes (flaky tests).
- Use `@Repeat` cautiously.
- Use `ESTestCase` (now `OpenSearchTestCase`) as the base class for unit tests.
- Use `OpenSearchIntegTestCase` for integration tests.
- Use `OpenSearchSingleNodeTestCase` for tests that need a running node.

### Handling flaky tests:
- If CI fails on a test unrelated to your change, check the flaky test list.
- Don't disable tests without an issue tracking re-enablement.
- When reporting a flaky test, include the seed: `./gradlew test --tests "TestClass" -Dtests.seed=DEADBEEF`

### REST tests:
- YAML-based REST tests live alongside integration tests.
- Use them for API-level testing of new endpoints.

## 8. Plugin Development

If contributing a plugin:
- Use the template: https://github.com/opensearch-project/opensearch-plugin-template-java
- Official plugins share the OpenSearch release cycle.
- Third-party plugins manage their own versioning but should document compatibility.
- Extension points: `Plugin` interface, `ActionPlugin`, `AnalysisPlugin`, `EnginePlugin`, etc.

## 9. Documentation

- **Code docs**: developer documentation lives in the main repo.
- **User docs**: maintained in https://github.com/opensearch-project/documentation-website
- If your PR adds a user-facing feature, add the `needs-documentation` label — it auto-creates a doc issue.
- Rendered docs: https://opensearch.org/docs

## 10. Labels & Triage

Key labels to know:
- `good first issue` — newcomer-friendly
- `enhancement` — feature request
- `bug` — confirmed bug
- `v3.0.0`, `v2.x` — version targeting
- `needs-documentation` — triggers doc issue creation
- `backport` — needs cherry-pick to release branch

## 11. Common Gotchas

- **Elasticsearch remnants** — the codebase was forked from ES 7.10.2. If you find ES references outside of attributions/copyrights, open an issue.
- **Gradle is slow** — incremental builds help. Use `--daemon` and `-x test` during development.
- **JDK version matters** — build requires JDK 21+. Check `JAVA_HOME`.
- **Don't add dependencies without discussion** — especially transitive ones that affect the dependency tree.
- **Backwards compatibility** — REST API changes must be backward compatible. Use `_cat/plugins` and version checks.
- **Security-sensitive areas** — changes to auth, crypto, or permission checks need extra scrutiny and possibly a security review.
