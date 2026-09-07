# Public GitHub publication checklist

Before changing the repository visibility to **Public**, review every file and screenshot.

## Remove or replace

- [ ] customer/project names;
- [ ] Azure subscription IDs and tenant IDs;
- [ ] real resource groups and resource names;
- [ ] real public/static IP addresses;
- [ ] real private IPs and exact production CIDRs;
- [ ] App Service `azurewebsites.net` hostnames;
- [ ] Front Door endpoint hostnames/IDs;
- [ ] real DNS names;
- [ ] Key Vault secret names that disclose customer context;
- [ ] connection strings;
- [ ] API keys/passwords/tokens;
- [ ] certificates/private keys;
- [ ] firewall allow-lists and office/VPN IP ranges;
- [ ] screenshots containing account names, GUIDs, IDs, access keys, or hidden browser data.

## GitHub settings

Recommended:

- [ ] default branch `main`;
- [ ] require pull requests before merge;
- [ ] require at least one approval if maintained by a team;
- [ ] enable secret scanning and push protection where available;
- [ ] enable Dependabot alerts if code/dependencies are added;
- [ ] add a `SECURITY.md` if the repository contains executable reference code;
- [ ] optionally enable GitHub Pages for the `docs/` folder.

## Positioning

Clearly state that the repository is:

- a **reference architecture**;
- not an exact customer deployment;
- not guaranteed to fit every Umbraco version;
- subject to Azure SKU/region availability and organization policy.
