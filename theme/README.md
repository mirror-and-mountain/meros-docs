# Meros theme development

[Documentation home](../README.md) | [Framework](../framework/README.md) | [First-party packages](../packages/README.md)

## Contents

- [About](#about)
- [Getting started](#getting-started)
- [Framework features and packages](#framework-features-and-packages)
- [Testing](#testing)
- [License](#license)

## About
Meros is our first Wordpress starter theme powered by the [Meros Framework](https://github.com/mirror-and-mountain/meros-framework). Its goal is to provide a developer-friendly environment for creating block-based themes that benefit from a standardised approach for adding features through a Laravel-esque syntax and overall experience.

## Getting started
To start creating a new theme run the following command:
```console
composer create-project mirror-and-mountain/meros <YourThemeName>
```

Follow the [getting-started guide](getting-started.md) for application boot,
theme configuration, project layout and asset builds.

## Framework features and packages

Use the [Framework guides](../framework/README.md) for forms, dynamic blocks,
integration readiness and security. Feature registration belongs in the
theme/provider lifecycle, while runtime services come from
[Foundations](../foundations/README.md).

The [first-party package section](../packages/README.md) describes CRM, Blocks
and Email Connectors.

## Testing

Testing documentation is shared under Framework. See
[theme developer testing](../framework/testing/theme-developers.md) for the
Podman workspace runner and writing tests for your own theme, and
[package developer testing](../framework/testing/package-developers.md) for
reusable package boot and service checks.

## License
Meros Themes are open-sourced software licensed under the MIT license. Note that if you wish to publish any theme created from this project, you will need to license it in accordance with the [Wordpress guidelines.](https://developer.wordpress.org/themes/releasing-your-theme/)
