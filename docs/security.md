# Security

## Identity and secrets

Recommended pattern:

1. enable system- or user-assigned managed identity on App Services;
2. grant least-privilege Key Vault access;
3. reference secrets using Key Vault references;
4. never place secret values in source control.

## Key Vault networking

Use a Private Endpoint for application data-plane access. If another trusted Azure service needs certificate/secret access, verify the service-specific networking exception before disabling public access.

## Edge hardening

Prevent origin bypass.

Recommended controls:

- App Service access restrictions that allow Azure Front Door backend traffic only;
- validate `X-Azure-FDID` for the expected Front Door profile;
- if an upstream WAF/CDN sits before Front Door, use an additional control (for example a secret header or equivalent) so clients cannot bypass that layer by calling the Front Door endpoint directly.

## TLS

- redirect HTTP to HTTPS;
- use modern TLS versions supported by the platform;
- automate certificate rotation where possible;
- document certificate ownership and renewal paths.

## Logging and audit

Enable/retain:

- Azure Activity Log;
- App Service diagnostics as required;
- Front Door/WAF logs;
- Key Vault audit logs;
- SQL auditing/Defender features according to policy;
- Application Insights telemetry.
