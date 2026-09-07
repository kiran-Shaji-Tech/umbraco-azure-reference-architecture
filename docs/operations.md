# Operations and troubleshooting

## Routine validation

Check:

- public site health;
- backoffice login;
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

## Common failure patterns

| Symptom | Likely area |
|---|---|
| External API rejects requests after VNet change | NAT/Route All/static IP allow-list |
| Media disappears on one tier only | Local filesystem or storage-provider mismatch |
| Private Endpoint exists but traffic still public | Private DNS zone/link/resolution issue |
| App fails after Key Vault public access disabled | Private DNS, identity/RBAC, or VNet routing |
| Forms data differs between backend/frontend | Forms storage still local |
| Front Door origin unhealthy | health endpoint, host header, access restrictions, app startup |

## Change discipline

Perform one networking hardening change at a time. Validate before disabling the old/public path, then retain a short rollback window.
