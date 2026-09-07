# Public GitHub publication checklist

Before changing the repository visibility to **Public**, review every file, image, issue, pull request, commit, and Git history.

## Remove or replace

- [ ] customer/project names;
- [ ] Azure subscription IDs and tenant IDs;
- [ ] Entra application/client IDs unless intentionally public;
- [ ] real resource groups and resource names;
- [ ] real public/static IP addresses;
- [ ] real private IPs and exact production CIDRs;
- [ ] office/VPN/ZTNA allow-list ranges;
- [ ] App Service `azurewebsites.net` hostnames;
- [ ] Front Door endpoint hostnames/IDs;
- [ ] real DNS names, especially administrative hostnames;
- [ ] Key Vault secret names that disclose customer context;
- [ ] connection strings;
- [ ] API keys/passwords/tokens;
- [ ] certificates/private keys;
- [ ] break-glass usernames or recovery procedures containing secrets;
- [ ] security-group membership or privileged-user lists;
- [ ] screenshots containing account names, GUIDs, IDs, access keys, tokens, IP rules, or hidden browser data.

## Git history

Removing a secret from the latest commit is not enough if it exists in history.

- [ ] search the full Git history for production values;
- [ ] rotate/revoke exposed credentials first;
- [ ] rewrite history when necessary;
- [ ] invalidate cached artifacts/releases containing sensitive data;
- [ ] review GitHub Actions logs and uploaded artifacts if used.

## GitHub settings

Recommended:

- [ ] default branch `main`;
- [ ] require pull requests before merge;
- [ ] require at least one approval if maintained by a team;
- [ ] enable secret scanning and push protection where available;
- [ ] enable Dependabot alerts if code/dependencies are added;
- [ ] add a `SECURITY.md` if the repository contains executable reference code;
- [ ] optionally enable GitHub Pages for the `docs/` folder;
- [ ] avoid repository secrets containing any real customer credential unless the repository actually deploys infrastructure and there is a justified need.

## Security positioning

Clearly state that:

- the public frontend is designed to be Internet-facing;
- the backoffice is designed as a restricted administrative plane;
- real implementations should combine network restrictions with centralized SSO/MFA;
- the repository deliberately omits actual administrative allow-lists and identity details;
- public diagrams are illustrative and are not a complete map of any customer environment.

## Positioning

Clearly state that the repository is:

- a **reference architecture**;
- not an exact customer deployment;
- not guaranteed to fit every Umbraco version;
- subject to Azure SKU/region availability and organization policy.
