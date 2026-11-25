# Testing from a Meros theme

[Documentation home](../../README.md) | [Testing](README.md) | [Theme development](../../theme/README.md)

## Contents

- [Run the supplied workspace suites](#run-the-supplied-workspace-suites)
- [Podman integration setup](#podman-integration-setup)
- [Select suites and tests](#select-suites-and-tests)
- [Test your own theme](#test-your-own-theme)
- [Without the workspace tooling](#without-the-workspace-tooling)

## Run the supplied workspace suites

The Meros workspace has a development-only Composer project under
`tools/testing/`. It uses local path dependencies for Foundations, Framework,
CRM, Blocks and Email Connectors and does not change the theme's dependency
lockfile.

Inside the devcontainer, install the shared runner and official WordPress test
sources:

```bash
bash .devcontainer/setup-tests.sh
```

Run isolated suites:

```bash
bash .devcontainer/test.sh unit
```

The scripts and runner are supplied by the
[Meros theme repository](https://github.com/mirror-and-mountain/meros). A theme
created from an older starter may not contain them; use the standalone consumer
setup below or bring the matching development tooling into your project.

## Podman integration setup

On the host, from the Meros theme root, start **only** the testing database:

```bash
podman compose -p meros_devcontainer -f .devcontainer/docker-compose.yml --profile testing up -d test-db
```

`meros_devcontainer` is this workspace's Compose project name. Use your own
devcontainer's project name if it differs, so `test-db` joins the network used by
your WordPress container. Do not restart or replace the development database.

Then run inside the devcontainer:

```bash
bash .devcontainer/test.sh integration
```

The checked-in defaults are:

- WordPress tests: `$HOME/meros-testing/wordpress-develop/tests/phpunit`.
- WordPress source: `$HOME/meros-testing/wordpress-develop/src`.
- Database: `meros_tests`, host `test-db`.
- The isolated service's database user is `root`, with an empty password.
- The service has no published host port and keeps database data in memory.

That empty password is for the separate ephemeral service, not a recommendation
for a development or production database. The runner supports the environment
overrides listed in the [shared integration setup](README.md#wordpress-integration-environment).

`MEROS_WP_TEST_REF` selects the WordPress branch during initial setup (default
`7.1`). Keep it aligned with the development site's major/minor version.
Changing the variable does not switch an existing checkout; prepare the intended
checkout before running tests.

To stop only the test database, run on the host:

```bash
podman compose -p meros_devcontainer -f .devcontainer/docker-compose.yml --profile testing stop test-db
```

## Select suites and tests

The combined runner's named suites are `Foundations`, `Framework`, `CRM`,
`Blocks` and `EmailConnectors`:

```bash
bash .devcontainer/test.sh integration --testsuite CRM
bash .devcontainer/test.sh unit --filter TableMigrationTest
wp meros:test list
wp meros:test run meros-foundations
wp meros:test run meros-crm --suite=integration
```

The CLI inherits test environment variables from your shell; its integration
command does not supply database defaults itself. The devcontainer's configured
environment and `test.sh` provide the normal workspace values.

`wp meros:test run` without a package uses the theme's suite, or the combined
workspace suite if the theme has no PHPUnit configuration.

## Test your own theme

The combined workspace suites test the supplied packages, not your custom theme
automatically. Add your own unit/integration directories, Composer test namespace,
bootstrap files and PHPUnit configurations using the
[consumer setup](test-cases.md#bootstrapping-a-consumer-suite).

Run custom theme suites through the theme's own Composer runner. The stock
shared bootstrap loads the five package test namespaces, not your theme's
`App\` classes or test namespace. To include theme tests in that shared runner
(including CLI runs which prefer it), first add the theme's production/test
autoload mappings and any extra dependencies to the runner's Composer project,
then add its test directories to the shared configurations.

- Extend `MM\Meros\Testing\TestCase` for isolated Framework registration and
  service behavior. It does not fully boot your theme.
- Extend `MM\Meros\Testing\WordPressTestCase` for WordPress-backed behavior.
  By default it boots a minimal test theme rather than `App\Theme`.

To select your consuming theme class before the theme provider registers:

```php
namespace App\Tests\Integration;

use App\Theme as AppTheme;
use MM\Meros\Facades\Theme;
use MM\Meros\Foundations\Application;
use MM\Meros\Testing\WordPressTestCase;

final class ThemeBootTest extends WordPressTestCase
{
    protected function createApplication(array $providers = []): Application
    {
        add_action('meros_framework_configured', static function (): void {
            config(['theme.theme_class' => AppTheme::class]);
        });
        return parent::createApplication($providers);
    }

    public function test_the_theme_boots(): void
    {
        $this->assertTrue($this->app->isBooted());
        $this->assertInstanceOf(AppTheme::class, Theme::get());
    }
}
```

The hook runs before Framework registers the theme provider. It changes the
test app's class, not the live site's theme configuration. Paths remain
disposable: create/copy only the fixture configuration, routes or resources your
theme needs into `$this->directory` before boot. Do not redirect the harness to
live storage or a live database.

Add assertions for your theme's actual features, tables or rendered output;
boot success alone does not establish that every feature works.
Run your own configured suites from the theme root:

```sh
composer test
composer test:integration
```

## Without the workspace tooling

Install the [consumer development dependencies](test-cases.md#development-dependencies)
in the theme, use its own PHPUnit configurations and provision the
[standalone WordPress integration environment](README.md#wordpress-integration-environment).
WP-CLI can use the theme's PHPUnit executable when there is no shared runner.
