# Umbraco-specific design

## One application, two roles

Build the application once and deploy the same artifact to frontend and backend App Services. Role-specific behavior is controlled through configuration.

## Backoffice authentication

The backoffice should use stronger authentication than password-only public access.

For self-hosted Umbraco, external login providers can be used for backoffice users. A common enterprise pattern is Microsoft Entra ID through OpenID Connect.

Design goals:

- central user lifecycle in Entra ID;
- MFA and Conditional Access;
- group/claim mapping to least-privilege Umbraco groups;
- no automatic Administrator role for every authenticated user;
- controlled local break-glass access;
- explicit callback/reserved-path configuration according to the deployed Umbraco version.

The exact implementation varies by Umbraco major version and authentication provider/package. Validate against the official Umbraco documentation for the version in use.

## Shared media

Use Azure Blob-backed media storage when multiple App Services/instances must see the same media library.

## Data Protection

ASP.NET Core Data Protection keys must be shared across frontend and backend when authentication/cookies/protected state must remain consistent across instances.

A common pattern uses:

- shared Blob location for key persistence;
- Key Vault key for encryption-at-rest;
- identical `ApplicationName` across all participating App Services.

## Umbraco Forms

Any Forms data that defaults to local filesystem storage should be reviewed. If both tiers require it, move it to shared storage or implement the relevant storage abstraction.

## `UmbracoApplicationUrl`

Treat this setting carefully. It is application-wide and may be used by Umbraco or packages for absolute URLs, callbacks, licensing, or scheduled processing. Set it only after confirming the behavior for the exact Umbraco/Forms versions in use.

When the backend hostname is IP-restricted, test any feature that generates or calls absolute backend URLs.

## Health checks

Expose a lightweight readiness endpoint suitable for Azure Front Door/App Service health probing. The endpoint should validate application readiness without performing expensive work.

Do not accidentally force interactive SSO on a health endpoint that must be consumed by a platform probe. If an authentication layer is placed in front of the entire backend, explicitly design probe and callback behavior.
