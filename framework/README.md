# Meros Framework

[Documentation home](../README.md) | [Theme development](../theme/README.md) | [Foundations](../foundations/README.md)

The Framework underpins Meros starter themes for WordPress. Foundations supplies
the application runtime and Illuminate services; Framework supplies the
provider/register/feature model and WordPress-facing services used by themes and
packages.

Start a new theme from the [Meros theme](../theme/getting-started.md), not the
Meros Blocks package. First-party packages extend the Framework rather than
replace the starter theme.

## Contents

- [Forms](forms.md): PHP components, visual builder, templates, conditions,
  repeaters, pagination, submissions and action extension points.
- [Integrations](integrations.md): local readiness checks, request boundaries
  and OAuth connection settings.
- [Dynamic blocks](dynamic-blocks.md): nested blocks and inspector controls.
- [Roles and capabilities](security/roles-and-capabilities.md): code-defined
  permissions and user-configurable role grants.
- [Multi-factor authentication](security/mfa.md): native WordPress login,
  enrollment, recovery and package identity adapters.
- [Testing](testing/README.md): unit and WordPress integration suites.
  - [Reusable test cases](testing/test-cases.md)
  - [Theme developers](testing/theme-developers.md)
  - [Package developers](testing/package-developers.md)
- [First-party packages](../packages/README.md)
- [License](#license)

## License

Meros Framework is open-sourced software licensed under the MIT license.
