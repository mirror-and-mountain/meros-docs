# Testing a Meros package

[Documentation home](../../README.md) | [Testing](README.md) | [First-party packages](../../packages/README.md)

## Contents

- [Set up package suites](#set-up-package-suites)
- [Isolated contract and service tests](#isolated-contract-and-service-tests)
- [Package boot and table migrations](#package-boot-and-table-migrations)
- [Disabled packages](#disabled-packages)
- [Block registration and rendering](#block-registration-and-rendering)
- [External integrations](#external-integrations)

## Set up package suites

Follow the [consumer bootstrap setup](test-cases.md#bootstrapping-a-consumer-suite)
with your package's own `autoload-dev` namespace. Development dependencies are
not inherited from Framework or Foundations.

Use `tests/Unit` for isolated tests and `tests/Integration` for real WordPress
behavior. Run `composer test` and `composer test:integration` separately, or
select the package through the [workspace runner](theme-developers.md#select-suites-and-tests).

## Isolated contract and service tests

Extend `MM\Meros\Testing\TestCase` for real Framework services/registers with
isolated WordPress doubles and SQLite. Use
`MM\Meros\Foundations\Testing\TestCase` when Framework is not needed.

Exercise the contract's observable behavior: provider ownership, successful and
rejected registrations, supported feature configuration, migration effects,
readiness and explicit failures. Use `FeatureProvider` for lightweight provider
fixtures and check out registers before making or attaching features.

Use migrations that belong under the fixture provider's root. Verify actual
create/update/rollback/drop behavior and migration tracking rather than only
checking that a migration class can be instantiated.

## Package boot and table migrations

Extend `MM\Meros\Testing\PackageTestCase`, supplying the main package class and
its stable handle. For example, the CRM package:

```php
namespace MM\Meros\Crm\Tests\Integration;

use MM\Meros\Crm\MerosCrm;
use MM\Meros\Testing\PackageTestCase;

final class PackageBootTest extends PackageTestCase
{
    protected function packageClass(): string
    {
        return MerosCrm::class;
    }

    protected function packageHandle(): string
    {
        return 'meros_crm';
    }

    public function test_package_tables_install(): void
    {
        $this->assertTrue($this->app->isBooted());
        $this->installProviderTables($this->package());
    }
}
```

The harness reads the package's Composer metadata, enables it in fixture
settings before registration, and boots through the actual providers.
`package()` asserts that the expected package was discovered.

The default root lookup assumes the main class is directly under the package's
`src/` directory, as in the first-party packages. Override `packageRoots()` if
your main class has a different layout or the test requires additional package
metadata.

Core Framework tables are ready before packages boot. The migration helper
installs registered package tables and checks errors/installed state; add your
own assertions for package-specific columns, data, update and rollback behavior.
Track any additional custom tables as explained in
[database isolation](test-cases.md#database-and-block-isolation).

## Disabled packages

Use a separate test class that overrides:

```php
protected function packageEnabled(): bool
{
    return false;
}
```

Assert that the package remains discoverable but does not configure enabled-only
services or features. Do not reuse enabled-package expectations in that class.
The CRM
[DisabledPackageTest](https://github.com/mirror-and-mountain/meros-crm/blob/main/tests/Integration/DisabledPackageTest.php)
is a concrete example.

## Block registration and rendering

In an enabled block package's integration test, fire WordPress's `init` hook
and inspect native registrations, checking out the provider's register first:

```php
use MM\Meros\Facades\Components\DynamicBlocks;

$package = $this->package();
do_action('init');
$blocks = DynamicBlocks::checkout($package)->all();
$this->assertNotEmpty($blocks);
foreach ($blocks as $block) {
    $registered = $this->assertBlockRegistered($block->getName());
}
```

For a rendering test, use the returned `WP_Block_Type` with appropriate test
attributes or render a block through WordPress. Assert the required HTML/data,
not just presence in the register. Teardown unregisters new native block types.
Meros Blocks uses the `DynamicBlocks` register, not the ordinary `Blocks`
register.

## External integrations

Keep routine suites independent of live Salesforce, Microsoft 365 or Mailchimp
accounts. Fake request boundaries and test payloads, response/error contracts,
enabled/disabled state and capability-specific readiness.

Readiness is a local prerequisite check, not proof that a remote API will
succeed. Verify the request boundary enforces it, and that errors remain visible.
Test queue dispatch and job behavior without accidentally running real remote
work or sending email.

The first-party package guides document the contracts to cover:

- [CRM writes, flows and queue behavior](../../packages/meros-crm/README.md)
- [Blocks and query/menu behavior](../../packages/meros-blocks/README.md)
- [Email connector operations](../../packages/meros-email-connectors/README.md)
