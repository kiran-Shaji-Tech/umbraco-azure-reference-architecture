# Security

Security is layered across edge, administrative access, identity, secrets, private networking, and auditability.

## Security objectives

1. Public users should reach the frontend only through approved edge controls.
2. The Umbraco backoffice should not be broadly reachable from the Internet.
3. Administrative identity should be centrally managed and MFA-capable.
4. Azure PaaS data-plane services should use private access where justified.
5. Secrets should not be stored in source control or plaintext App Service configuration.
6. Deployment and operational access must remain recoverable after hardening.

## Backoffice / administrative plane

Recommended baseline:

```text
Trusted corporate/VPN/ZTNA source
        -> App Service Access Restrictions
        -> Entra ID / OpenID Connect SSO
        -> MFA / Conditional Access
        -> Least-privilege Umbraco role
```

See [Backoffice security](backend-security.md) for detailed implementation guidance.

## Identity and secrets

Recommended pattern:

1. enable system- or user-assigned managed identity on App Services;
2. grant least-privilege Key Vault access;
3. reference secrets using Key Vault references;
4. never place secret values in source control;
5. separate deployment identities from human administrator identities;
6. periodically review role assignments and unused identities.

## Key Vault networking

Use a Private Endpoint for application data-plane access. If another trusted Azure service needs certificate/secret access, verify the service-specific networking exception before disabling public access.

Do not publish real vault names, secret names that reveal customer context, private IPs, tenant IDs, role assignment IDs, or certificate identifiers in a public repository.

## Edge hardening

Prevent origin bypass.

Recommended controls:

- App Service access restrictions that allow Azure Front Door backend traffic only on the **frontend** origin;
- validate `X-Azure-FDID` for the expected Front Door profile;
- set unmatched App Service frontend access to deny after validation;
- if an upstream WAF/CDN sits before Front Door, use an additional control such as a secret header or equivalent so clients cannot bypass that layer by calling the Front Door endpoint directly.

The `AzureFrontDoor.Backend` service tag alone is not sufficient to identify one specific Front Door profile; pair it with `X-Azure-FDID` validation.

## Example frontend origin restriction

```bash
az webapp config access-restriction add \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FRONTEND_APP_NAME>" \
  --rule-name "Allow-Expected-FrontDoor" \
  --action Allow \
  --priority 100 \
  --service-tag AzureFrontDoor.Backend \
  --http-header "x-azure-fdid=<FRONT_DOOR_ID>"

az webapp config access-restriction set \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<FRONTEND_APP_NAME>" \
  --default-action Deny
```

Use the exact syntax supported by the installed Azure CLI version and validate health probes before setting default deny.

## TLS

- redirect HTTP to HTTPS;
- set HTTPS-only on App Services where supported;
- use modern TLS versions supported by the platform;
- automate certificate rotation where possible;
- document certificate ownership and renewal paths;
- do not commit PFX files or certificate private keys.

## Kudu / SCM security

Treat `*.scm.azurewebsites.net` as an administrative endpoint.

- apply explicit SCM restrictions;
- test the deployment mechanism before making SCM deny-by-default;
- prefer self-hosted deployment agents in controlled networks when strict allow-lists are required;
- do not expose SCM merely because the main site is protected.

## Logging and audit

Enable/retain according to organizational policy:

- Azure Activity Log;
- App Service access logs/diagnostics where required;
- Front Door/WAF logs;
- Entra sign-in and Conditional Access logs;
- Key Vault audit logs;
- SQL auditing/Defender features according to policy;
- Application Insights telemetry;
- alerts on unexpected access-rule changes and authentication failures.

## Public repository safety

A public architecture repository should explain **patterns**, not disclose the actual security boundary.

Never publish:

- real trusted CIDRs;
- actual backoffice hostname if sensitive;
- tenant IDs or app registrations unless intentionally public;
- Front Door profile IDs;
- NAT IPs;
- security-group membership;
- break-glass details;
- screenshots that expose firewall/access-restriction rules.
