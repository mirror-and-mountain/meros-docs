# Integrations

[Documentation home](../README.md) | [Framework](README.md) | [First-party packages](../packages/README.md)

## Contents

- [Using integrations](#using-integrations)
- [OAuth connection settings](#oauth-connection-settings)
- [Package-specific guides](#package-specific-guides)

## Using integrations

Resolve a registered integration and check its local readiness before use:

```php
use MM\Meros\Facades\Integrations;

$integration = Integrations::connect('mailchimp');
if ($integration->ready('marketing')) {
    $audiences = $integration->getAudiences();
}
```

`connect()` accepts a registered identifier, an integration class, or a unique
snake-case class name. It returns the integration even when it is not ready and
throws an `InvalidArgumentException` for unknown or ambiguous names. Readiness
checks the enabled setting and integration-specific prerequisites in the current
environment; OAuth integrations require a connected record and access token for
the selected audience. It does not contact the remote service or guarantee that
the next request will succeed. API request boundaries must enforce readiness too;
existing integration-specific facades remain available but are deprecated.

Integration authors can override `isReady(?string $capability): bool` for
additional local checks. Call `parent::isReady($capability)` when extending an
integration with its own prerequisites. The base implementation checks only the
enabled state and rejects unsupported capabilities. A readiness check is
side-effect-free apart from reading settings and connections; API failures should
still be logged and surfaced according to the integration's return contract.

## OAuth connection settings

OAuth integrations require saved, non-empty values for their `UsesBaseUrl`,
`UsesClientId` and `UsesClientSecret` settings before connecting. Values must be
saved for the current environment. Override `requiredConnectionSettings()` to
add custom setting names or exclude optional credentials:

```php
protected function requiredConnectionSettings(): array {
    return [...parent::requiredConnectionSettings(), 'tenant_id'];
}
```

Use setting names without environment suffixes. The same check applies when
starting the OAuth flow directly.

## Package-specific guides

- [Salesforce readiness and operations](../packages/meros-crm/README.md)
- [Microsoft 365 and Mailchimp](../packages/meros-email-connectors/README.md)
- [Testing integrations and package boot](testing/package-developers.md)
