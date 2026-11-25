# Meros documentation

Canonical documentation for Meros themes, Foundations, Framework and first-party
packages. The source repositories keep short READMEs linking here.

These guides describe the Foundations-backed implementation in the current
workspace. Match your package versions to the APIs described here; older Acorn-based
releases can differ.

## Contents

- [Theme development](theme/README.md)
  - [Getting started, configuration and assets](theme/getting-started.md)
  - [Testing a consuming theme](framework/testing/theme-developers.md)
- [Foundations](foundations/README.md)
  - [Application boot and services](foundations/README.md#what-it-does)
  - [Paths and storage](foundations/README.md#paths-and-storage)
  - [WP-CLI commands](foundations/README.md#wp-cli)
- [Framework](framework/README.md)
  - [Forms](framework/forms.md)
  - [Integrations and OAuth readiness](framework/integrations.md)
  - [Dynamic blocks and inspector controls](framework/dynamic-blocks.md)
  - [Roles and capabilities](framework/security/roles-and-capabilities.md)
  - [Multi-factor authentication](framework/security/mfa.md)
  - [Testing](framework/testing/README.md)
    - [Reusable test cases](framework/testing/test-cases.md)
    - [Theme developer workflow](framework/testing/theme-developers.md)
    - [Package developer workflow](framework/testing/package-developers.md)
- [First-party packages](packages/README.md)
  - [Meros CRM](packages/meros-crm/README.md)
  - [Meros Blocks](packages/meros-blocks/README.md)
  - [Meros Email Connectors](packages/meros-email-connectors/README.md)
- [Source repositories](#source-repositories)
- [Maintaining documentation](#maintaining-documentation)
- [Licensing](#licensing)

## Source repositories

| Repository | Responsibility |
| --- | --- |
| [meros](https://github.com/mirror-and-mountain/meros) | Starter theme and consuming-theme tooling |
| [meros-foundations](https://github.com/mirror-and-mountain/meros-foundations) | Container, boot lifecycle, Illuminate services and WordPress runtime |
| [meros-framework](https://github.com/mirror-and-mountain/meros-framework) | Theme/package providers, feature contracts, registers and reusable testing |
| [meros-crm](https://github.com/mirror-and-mountain/meros-crm) | Salesforce integrations and CRM actions |
| [meros-blocks](https://github.com/mirror-and-mountain/meros-blocks) | First-party editor and front-end blocks |
| [meros-email-connectors](https://github.com/mirror-and-mountain/meros-email-connectors) | Microsoft 365 and Mailchimp integrations |

## Maintaining documentation

- Keep detailed documentation here; source READMEs should remain entry points.
- Link between documentation pages using relative paths so local and GitHub
  navigation work without a documentation-site build.
- Put shared testing setup and developer workflows under Framework, linking to
  them from theme, Foundations and package sections.
- Add new first-party packages beneath `packages/`, with an entry in both package
  and root contents.
- Keep source-code links pointed at the owning repository rather than assuming
  source files are present in this documentation repository.

## Licensing

The migrated Meros source documentation describes MIT-licensed projects.
Project-specific licensing notes remain in the relevant guides and source
repositories. Published WordPress themes must also follow
[WordPress's licensing guidelines](https://developer.wordpress.org/themes/releasing-your-theme/).
