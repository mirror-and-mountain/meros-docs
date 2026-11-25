# First-party packages

[Documentation home](../README.md) | [Framework](../framework/README.md) | [Theme development](../theme/README.md)

First-party packages extend a consuming Meros theme through the Framework's
package discovery and feature registration.

## Contents

- [Meros CRM](meros-crm/README.md)
  - [Readiness](meros-crm/README.md#salesforce-readiness)
  - [Record writes](meros-crm/README.md#salesforce-record-writes)
  - [Flow invocation and background queues](meros-crm/README.md#salesforce-flow-invocation)
  - [Action types](meros-crm/README.md#action-types)
- [Meros Blocks](meros-blocks/README.md)
  - [Searchable Query](meros-blocks/README.md#searchable-query)
  - [Masonry Query](meros-blocks/README.md#masonry-query)
  - [Dynamic mega menu](meros-blocks/README.md#dynamic-mega-menu)
- [Meros Email Connectors](meros-email-connectors/README.md)
  - [Microsoft 365 mailbox](meros-email-connectors/README.md#microsoft-365-mailbox)
  - [Shared mailbox](meros-email-connectors/README.md#microsoft-365-shared-mailbox)
  - [Mailchimp](meros-email-connectors/README.md#mailchimp)
- [Installation and enablement](#installation-and-enablement)
- [Developing and testing packages](#developing-and-testing-packages)

## Installation and enablement

Install compatible package versions through the consuming theme's Composer
project. Meros discovers package providers from Composer metadata; add manual
providers to `config/theme.php` only when discovery is unavailable.

Use the theme's Packages settings page to enable a discovered package, then
follow its guide for settings, optional tables and external connections.
Installing a package is not the same as enabling or connecting an integration.

## Developing and testing packages

Shared boot, migration and registration testing belongs in
[Framework testing](../framework/testing/README.md). The
[package developer guide](../framework/testing/package-developers.md) explains
the reusable `PackageTestCase`, disabled-package checks and native block tests.
