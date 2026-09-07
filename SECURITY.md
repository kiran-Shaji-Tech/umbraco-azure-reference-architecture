# Security policy

This repository is a public reference architecture and must not contain real production credentials, customer identifiers, connection strings, private keys, administrative allow-lists, tenant-specific SSO values, Front Door profile IDs, NAT IPs, or environment-specific secrets.

## Reporting a security issue

If you discover a secret or sensitive production value committed to this repository:

1. do not open a public issue containing the value;
2. rotate/revoke the credential or security value first;
3. notify the repository owner through a private channel;
4. remove the value from current files;
5. remove it from Git history using an appropriate history-rewrite process when required;
6. review forks, releases, Actions artifacts, and caches for copies.

## Reference security model

The repository recommends:

- WAF/CDN + Azure Front Door for public frontend traffic;
- App Service origin restrictions for the frontend;
- restricted network access for the backoffice;
- Entra ID/OpenID Connect SSO with MFA/Conditional Access for backoffice users;
- separate protection for SCM/Kudu;
- managed identities and Key Vault for secrets;
- Private Endpoints for sensitive PaaS data-plane services;
- NAT Gateway for stable controlled outbound egress.

These are reference patterns and must be adapted to the security policy of the implementing organization.
