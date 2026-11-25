# Getting started with a Meros theme

[Documentation home](../README.md) | [Theme development](README.md) | [Framework](../framework/README.md)

## Contents

- [Create and activate a theme](#create-and-activate-a-theme)
- [Application boot](#application-boot)
- [Theme configuration](#theme-configuration)
- [Project layout](#project-layout)
- [Build assets](#build-assets)
- [Packages and testing](#packages-and-testing)

## Create and activate a theme

Create a new project in your WordPress themes directory:

```sh
composer create-project mirror-and-mountain/meros your-theme
cd your-theme
```

For an existing checkout, run `composer install` in the theme root. Use the PHP
version and extensions required by the selected Composer dependencies.
Activate the theme through WordPress's Appearance screen.

Choose compatible versions of the theme, Framework and Foundations. This
documentation describes the Foundations-backed workspace, not older Acorn-based
releases.

## Application boot

The theme's `functions.php` loads `vendor/autoload.php` and calls
`MM\Meros\Loader::boot()`. Framework delegates the application runtime to
Foundations and registers the Framework, discovered packages and theme
providers. Do not add a second manual application boot in your theme.

`App\Theme` extends `MM\Meros\App\BaseTheme`. Put theme feature configuration
inside its `configure()` method, using the provider's registers:

```php
namespace App;

use MM\Meros\App\BaseTheme;

class Theme extends BaseTheme
{
    public function configure(): void
    {
        $this->forms()->make([
            'name' => 'contact',
            'title' => 'Contact us',
        ]);
    }
}
```

This creates a PHP form component, not a stored WordPress form or a submission
endpoint. Follow the [forms guide](../framework/forms.md) to add fields, render
the component and choose the appropriate submission workflow.

## Theme configuration

`config/theme.php` selects the theme class and contains a `packages` array:

```php
return [
    'theme_class' => 'App\\Theme',
    'packages' => [],
];
```

Keep the class consistent with the theme's Composer PSR-4 mapping (`App\` to
`app/`). Meros packages are normally discovered automatically from Composer
metadata; list a provider here only when it is not discoverable.

Foundations layers its config stubs, additional registered stubs and finally
the theme's `config/` files. Keep environment-specific settings and credentials
out of committed source. Preserve and back up the application's encryption key
when using encrypted settings or MFA.

See [Foundations paths and storage](../foundations/README.md#paths-and-storage)
and [WP-CLI commands](../foundations/README.md#wp-cli) for configuration caches,
storage overrides and maintenance.

## Project layout

Paths are relative to the consuming theme root:

| Path | Purpose |
| --- | --- |
| `functions.php` | Composer loading and Framework boot |
| `app/Theme.php` | Theme feature configuration |
| `app/Livewire/` | Theme Livewire classes |
| `config/` | Theme configuration overrides |
| `routes/` | Application routes |
| `templates/`, `parts/`, `patterns/`, `theme.json` | WordPress block-theme structure and design settings |
| `resources/` | Theme views and source assets, including custom block sources |
| `assets/`, `public/` | Theme assets and build/public output, according to build configuration |
| `database/` | Theme database-related definitions |
| `storage/` | Runtime caches and logs |
| `vendor/` | Composer dependencies |
| `tools/testing/` | Optional shared workspace test runner |

The starter's Composer setup can generate application/configuration files.
Check the created project rather than assuming every generated file is tracked
in the starter repository.

## Build assets

Install the theme's JavaScript dependencies, then use the scripts in its
`package.json`:

```sh
npm install
npm run build
```

| Command | Scope |
| --- | --- |
| `npm run dev:vite` | Vite development server |
| `npm run build:vite` | Vite production build |
| `npm run build:assets` | WordPress assets using the theme's webpack configuration |
| `npm run build:blocks` | Custom blocks under `resources/blocks/src/` |
| `npm run build` | Block, WordPress asset and Vite builds |

These commands run from the theme root. A package's asset scripts run from that
package's root; rebuilding the theme does not automatically rebuild every
first-party package.

## Packages and testing

- [First-party packages](../packages/README.md): install and configure CRM,
  Blocks and Email Connectors.
- [Framework features](../framework/README.md): forms, blocks, integrations
  and security.
- [Theme developer testing](../framework/testing/theme-developers.md): run
  supplied suites and add isolated or WordPress-backed theme tests.
