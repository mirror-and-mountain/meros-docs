# Testing Meros

[Documentation home](../../README.md) | [Framework](../README.md) | [Theme development](../../theme/README.md)

Shared testing documentation for Foundations, Framework, consuming themes and
first-party packages.

## Contents

- [Choose a workflow](#choose-a-workflow)
- [Test tiers](#test-tiers)
- [Run repository suites](#run-repository-suites)
- [WordPress integration environment](#wordpress-integration-environment)
- [WP-CLI test commands](#wp-cli-test-commands)
- [Dependency notices](#dependency-notices)

## Choose a workflow

| Goal | Guide |
| --- | --- |
| Choose a base class or configure a new consumer suite | [Reusable test cases](test-cases.md) |
| Run supplied suites from a Meros theme or test your own theme | [Theme developers](theme-developers.md) |
| Test package discovery, enabled/disabled behavior, tables or blocks | [Package developers](package-developers.md) |

Testing implementation remains in the source repositories. This section is the
canonical location for shared instructions; package guides describe their
package-specific behavior.

## Test tiers

- **Unit:** isolated PHPUnit classes with disposable paths, WordPress doubles and
  SQLite in memory. Use them for application boot, provider lifecycle, register
  contracts, migration behavior and local service logic.
- **Integration:** official WordPress test library plus a dedicated MySQL/MariaDB
  database. Use it for full Framework/package boot, real tables, native block
  registration and WordPress-dependent behavior.

Do not load the isolated WordPress doubles and real WordPress in the same
process. Keep unit and integration configurations separate.

The current suites use PHPUnit **9.6**, because the WordPress **7.1** test library
calls APIs removed in PHPUnit 10. Use the same major for both tiers. Livewire's
testing hooks use the standalone `illuminate/testing` development dependency,
not Acorn or Laravel Framework.

## Run repository suites

From an individual source repository:

```sh
composer install
composer test
composer test -- --filter=ApplicationTest
composer test:integration
```

The filter above is a Foundations example; select a test present in the chosen
repository. Configure the integration environment below before running that tier.
Use the [theme workflow](theme-developers.md#run-the-supplied-workspace-suites)
to share one Composer runtime across local package checkouts instead of
installing a separate runner in each package.

Test classes live in `tests/Unit` and `tests/Integration`, with supporting
fixtures beneath `tests/Support` or `tests/Fixtures`. Do not treat those
fixtures as production WordPress replacements.

## WordPress integration environment

Use a standalone checkout of `WordPress/wordpress-develop` and a
**dedicated, empty MySQL/MariaDB test database**. WordPress's test installer
drops tables. Never point this suite at a development or production site's
database, even if its name happens to end in `_tests`.

Set these environment variables before `composer test:integration` or a CLI
integration run:

| Variable | Value |
| --- | --- |
| `WP_TESTS_DIR` | Absolute path to `wordpress-develop/tests/phpunit` |
| `WP_TESTS_ROOT` | Absolute path to its `src` directory containing `wp-settings.php` |
| `MEROS_TEST_DB_NAME` | Dedicated database ending in `_tests`, such as `meros_tests` |
| `MEROS_TEST_DB_HOST` | TCP hostname, optionally with a port, such as `127.0.0.1:3307` |
| `MEROS_TEST_DB_USER` | User restricted to the test database |
| `MEROS_TEST_DB_PASSWORD` | Password for that user, if required |

The bootstrap rejects an already-loaded WordPress process, missing required
configuration and database names without the test suffix. It always uses the
`wptests_` table prefix. These guards are not permission to use a shared database.
No site's `wp-config.php` is loaded by the test bootstrap.

Framework integration tests disable `MEROS_ENVIRONMENT` development hooks.
Custom Meros tables created outside WordPress's transactions must be tracked
for teardown; see [database isolation](test-cases.md#database-and-block-isolation).
Do not run integration suites concurrently against the same database.

## WP-CLI test commands

From a running Meros theme:

```sh
wp meros:test list
wp meros:test run meros-foundations
wp meros:test run meros-framework --suite=integration
wp meros:test run meros-crm --suite=integration --filter=PackageBootTest
```

`list` finds Composer-installed repositories containing a PHPUnit configuration,
plus the theme's own suite and shared workspace suite when present. Package names
can be full Composer names or unique short names.

`run` defaults to unit tests and starts PHPUnit in a **separate PHP process**.
It uses PHPUnit's TestDox output to show named tests with pass/fail results,
followed by the normal test/assertion summary. Subprocess output streams without
adding extra line breaks.
It prefers the installed `tools/testing` runner and its bootstrap, so local
packages share one dependency runtime. Otherwise it looks for a runner in the
selected repository or active theme. Missing configurations/dependencies and
child failures are explicit errors; the child exit status propagates.

Without a package argument, the command selects the theme's suite, or the shared
workspace suite when the theme has no PHPUnit configuration. Integration runs
still require the dedicated test environment variables.

Packages installed only in the development runner may not appear individually
in the live theme's CLI listing. Run their named suite through the workspace
script, for example:

```sh
bash .devcontainer/test.sh integration --testsuite EmailConnectors
```

## Dependency notices

The Mailchimp SDK currently emits PHP deprecation notices during its unit
tests. They remain visible and are not suppressed. Passing tests do not mean
those upstream notices have been fixed.
