# Reusable test cases

[Documentation home](../../README.md) | [Testing](README.md) | [Theme developers](theme-developers.md) | [Package developers](package-developers.md)

## Contents

- [Available classes](#available-classes)
- [Development dependencies](#development-dependencies)
- [Bootstrapping a consumer suite](#bootstrapping-a-consumer-suite)
- [Application fixtures](#application-fixtures)
- [Database and block isolation](#database-and-block-isolation)

## Available classes

Testing classes are exported through the libraries' **normal autoload maps**, so
downstream themes and packages can extend them. Composer does not inherit a
dependency's `autoload-dev` mappings or install its `require-dev` dependencies.

| Class | Purpose |
| --- | --- |
| `MM\Meros\Foundations\Testing\TestCase` | Isolated application with disposable paths, WordPress doubles, array cache/session and SQLite in memory; call `createApplication()` in your test |
| `MM\Meros\Foundations\Testing\WordPressTestCase` | Official WordPress lifecycle, disposable theme/storage and the dedicated test database |
| `MM\Meros\Foundations\Testing\WordPress` | Integration bootstrap with database/configuration safeguards |
| `MM\Meros\Foundations\Testing\Bootstrap` | Loads a consumer's test namespaces into the active Composer runtime |
| `MM\Meros\Testing\TestCase` | Isolated Foundations application with real Framework services and registers; created during setup, without full Framework boot |
| `MM\Meros\Testing\FeatureProvider` | Lightweight feature-provider fixture for contract/register tests |
| `MM\Meros\Testing\WordPressTestCase` | Full Framework boot with a minimal fixture theme and installed core tables |
| `MM\Meros\Testing\PackageTestCase` | WordPress-backed package discovery and enabled/disabled package boot |

Framework's isolated case tests registration only. Use its WordPress case for
behavior depending on complete provider boot, WordPress hooks or native APIs.
Its default theme is the fixture `MM\Meros\Testing\TestTheme`, not your live
theme; the [theme guide](theme-developers.md#test-your-own-theme) shows how to
select a consuming theme class.

## Development dependencies

In a new consuming repository, merge the development dependencies and test
namespace into your existing Composer manifest:

```sh
composer require --dev phpunit/phpunit:^9.6 yoast/phpunit-polyfills:^3.1 illuminate/testing:^13.0
```

For a theme:

```json
{
    "autoload-dev": {
        "psr-4": {
            "App\\Tests\\": "tests/"
        }
    }
}
```

For a package, use its own test namespace instead. Run `composer dump-autoload`
after changing the autoload mapping. Framework-dependent tests also need
`meros-framework`; Foundations-only tests need `meros-foundations`.

## Bootstrapping a consumer suite

Create `tests/bootstrap.php`:

```php
<?php

use MM\Meros\Foundations\Testing\Bootstrap;

$root = dirname(__DIR__);
if (!class_exists(Bootstrap::class) && is_file($root . '/vendor/autoload.php')) {
    require $root . '/vendor/autoload.php';
}
if (!class_exists(Bootstrap::class)) {
    throw new RuntimeException('Install development dependencies or use the workspace runner.');
}
Bootstrap::load($root);
```

The guard avoids loading a second Composer runtime when the shared runner
already loaded the libraries.

Create `tests/bootstrap-integration.php`:

```php
<?php

require __DIR__ . '/bootstrap.php';
MM\Meros\Foundations\Testing\WordPress::bootstrap();
```

Use `phpunit.xml` for the unit directory:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="tests/bootstrap.php" cacheResultFile=".phpunit.result.cache">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

Use `phpunit.integration.xml` for WordPress:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="tests/bootstrap-integration.php" cacheResultFile=".phpunit.result.cache">
    <php>
        <env name="MEROS_ENVIRONMENT" value="false" force="true"/>
    </php>
    <testsuites>
        <testsuite name="Integration">
            <directory>tests/Integration</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

Add Composer scripts without replacing existing scripts:

```json
{
    "scripts": {
        "test": "phpunit -c phpunit.xml",
        "test:integration": "phpunit -c phpunit.integration.xml"
    }
}
```

Ignore `.phpunit.result.cache` in version control. The
[integration environment](README.md#wordpress-integration-environment) is
required before using the integration configuration.

## Application fixtures

Foundations' isolated `createApplication($providers, $config, $stubs)` accepts
providers, config overrides and optional config-stub directories. It uses a
non-secret test encryption key and a disposable environment-file path instead
of reading the live theme's `.env`.

`$this->directory`, `$this->files` and `writeFile($relativePath, $contents)`
support isolated fixtures. Container/facade state, publishing maps and
database connections are reset between tests.

Register contracts require a provider checkout before making or attaching
features:

```php
$provider = new MM\Meros\Testing\FeatureProvider($this->directory);
$feature = $register->checkout($provider)->make($properties);
```

Use real feature/register implementations appropriate to the behavior under
test. The Framework's
[FeatureRegistrationTest](https://github.com/mirror-and-mountain/meros-framework/blob/main/tests/Unit/FeatureRegistrationTest.php)
and [TableMigrationTest](https://github.com/mirror-and-mountain/meros-framework/blob/main/tests/Unit/TableMigrationTest.php)
provide concrete examples.

When overriding lifecycle methods, call the parent. Isolated cases use PHPUnit's
`setUp()` / `tearDown()`; WordPress cases use WordPress's `set_up()` / `tear_down()`.

## Database and block isolation

Framework's WordPress case installs real core tables after Framework
configuration and **before packages boot**, since OAuth integrations can query
connection records while configuring. It also creates fixture Composer
metadata, registers Livewire and supplies a fixture theme path.

`installProviderTables($provider)` initializes and installs a provider's
registered tables, checks installation errors and tracks tables for teardown.
For other custom DDL, call `trackTable('unprefixed_table_name')` before creating
the table. Cleanup refuses table prefixes outside `wptests_`.

WordPress and Illuminate use separate database connections. Do not assume a
WordPress write inside its test transaction is immediately visible through
Illuminate. Prefer fixtures written through the connection used by the
code under test; manage cross-connection fixtures explicitly rather than
pointing either connection at the live site.

New native block types are unregistered during teardown.
`assertBlockRegistered($name)` checks the WordPress registry and returns the
registered `WP_Block_Type`, so tests can also exercise rendering.
