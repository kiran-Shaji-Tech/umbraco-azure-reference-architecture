# Operations and troubleshooting

## Routine validation

Check:

- public site health;
- backoffice login from an approved network;
- backoffice denial from an unapproved network;
- Entra SSO and MFA behavior;
- Front Door origin health;
- SQL connectivity;
- media read/write;
- Redis connectivity;
- Key Vault references;
- Forms workflows;
- SMTP/email;
- dependency failures in Application Insights.

## Useful Kudu checks

### Static outbound IP

```powershell
curl.exe https://api.ipify.org
```

### Private DNS

```powershell
nslookup <storage-hostname>
nslookup <redis-hostname>
nslookup <key-vault-hostname>
```

### Port connectivity

```powershell
Test-NetConnection <host> -Port 443
Test-NetConnection <redis-host> -Port <redis-port>
```

## Backoffice access request runbook

1. Confirm the requester has a business-approved role.
2. Add the user to the approved Entra group rather than granting direct broad permissions where possible.
3. Confirm Conditional Access/MFA applies.
4. If a new corporate/VPN CIDR is required, process it through network-change approval.
5. Never paste production CIDRs into a public issue or repository.
6. Validate login and effective Umbraco role.
7. Record who approved the access and its review/expiry date if applicable.

## Backoffice lockout runbook

If all administrators are denied:

1. determine whether the failure is network restriction, Entra authentication, Conditional Access, Umbraco role mapping, or application startup;
2. use Azure management-plane access to inspect App Service access restrictions;
3. validate from the approved VPN/corporate network;
4. check Entra sign-in logs and Conditional Access results;
5. use the approved break-glass process only if necessary;
6. avoid changing multiple security controls simultaneously;
7. revert the most recent access change if required;
8. rotate/resecure emergency credentials after use.

## Deployment failure after SCM hardening

Symptoms:

- Azure DevOps deployment cannot reach Kudu/SCM;
- HTTP 403 from deployment endpoint;
- public site remains healthy but release fails.

Checks:

1. inspect SCM access restrictions independently of the main site;
2. determine whether the pipeline uses Microsoft-hosted or self-hosted agents;
3. confirm the agent source is allowed;
4. if using `use same restrictions for SCM`, verify that this was intentional;
5. restore the previous SCM rule set if the deployment path was not accounted for.

## Common failure patterns

| Symptom | Likely area |
|---|---|
| External API rejects requests after VNet change | NAT/Route All/static IP allow-list |
| Media disappears on one tier only | Local filesystem or storage-provider mismatch |
| Private Endpoint exists but traffic still public | Private DNS zone/link/resolution issue |
| App fails after Key Vault public access disabled | Private DNS, identity/RBAC, or VNet routing |
| Forms data differs between backend/frontend | Forms storage still local |
| Front Door origin unhealthy | health endpoint, host header, access restrictions, app startup |
| Backoffice returns 403 only outside office/VPN | Access restrictions working as designed or CIDR missing |
| SSO loops or callback fails | OIDC redirect URI/reserved path/cookie/proxy configuration |
| Azure DevOps backend deployment fails after hardening | SCM/Kudu access restrictions |
| Scheduled publishing fails after backend allow-list | required self-call or callback blocked |

## Change discipline

Perform one networking or identity hardening change at a time. Validate before disabling the old/public path, then retain a short rollback window.
