# Umbraco-specific design

## One application, two roles

Build the application once and deploy the same artifact to frontend and backend App Services. Role-specific behavior is controlled through configuration.

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

## Health checks

Expose a lightweight readiness endpoint suitable for Azure Front Door/App Service health probing. The endpoint should validate application readiness without performing expensive work.
