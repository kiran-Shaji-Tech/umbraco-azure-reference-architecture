# Networking

## VNet layout

Example only:

| Subnet | Example CIDR | Purpose |
|---|---|---|
| `snet-appsvc-frontend` | `10.10.1.0/26` | Frontend App Service VNet Integration |
| `snet-appsvc-backend` | `10.10.2.0/26` | Backend App Service VNet Integration |
| `snet-private-endpoints` | `10.10.10.0/27` | Private Endpoints for PaaS services |

The App Service integration subnets are delegated to `Microsoft.Web/serverFarms`. Keep the Private Endpoint subnet separate.

## Public frontend inbound path

```text
Internet
  -> Public DNS
  -> Upstream WAF/CDN
  -> Azure Front Door
  -> Frontend App Service
```

For the frontend origin, use App Service access restrictions to permit only the expected Azure Front Door path and deny direct origin access. Combine the `AzureFrontDoor.Backend` service tag with `X-Azure-FDID` validation for the intended Front Door profile.

## Backoffice inbound path

The backoffice is an administrative surface and should not be treated as a normal public endpoint.

Recommended baseline:

```text
Editor / Administrator
  -> Corporate network / VPN / ZTNA
  -> Backend App Service access restrictions
  -> Entra ID / OIDC SSO + MFA
  -> Umbraco Backoffice
```

Alternative high-assurance pattern:

```text
Editor / Administrator
  -> VPN / ExpressRoute / ZTNA
  -> private name resolution
  -> private backend access
  -> Entra ID / OIDC SSO + MFA
```

Do not publish real administrative IP ranges in a public repository.

## Access restrictions versus NSGs

App Service **inbound access restrictions** protect the public App Service endpoint. The App Service VNet Integration subnet is primarily for outbound connectivity and Private Endpoint access; adding an NSG to that subnet is not a substitute for App Service inbound restrictions.

Use each control for its intended purpose.

## Outbound path

```text
Frontend/Backend
  -> VNet Integration
  -> NAT Gateway
  -> Static public IPv4
  -> External APIs / SMTP / SaaS
```

Validation example from Kudu:

```powershell
curl.exe https://api.ipify.org
```

The returned IP should match the NAT Gateway public IP.

## Private Endpoint traffic

Typical DNS zones:

| Service | Private DNS zone |
|---|---|
| Blob Storage | `privatelink.blob.core.windows.net` |
| Key Vault | `privatelink.vaultcore.azure.net` |
| Azure Managed Redis | verify against the current service documentation for the deployed SKU |

Validation pattern:

```powershell
nslookup <service-hostname>
Test-NetConnection <service-hostname> -Port <port>
```

Expected DNS result: RFC1918/private address reachable from both App Services.

## NSGs and route tables

Do not add NSGs or user-defined routes merely for completeness. Introduce them only when there is a specific traffic-control requirement and test App Service platform dependencies carefully.

## SQL networking

This reference may use Azure SQL over its public endpoint with server firewall rules allowing only the NAT Gateway egress IP. A SQL Private Endpoint can be added as a separate hardening step if required.

## Administrative self-call caveat

After restricting the backend to trusted source ranges, test whether the backend makes any required requests to its own public administrative hostname. If it does, the call may egress through NAT and return to the App Service access restriction boundary.

Do not pre-emptively allow the NAT IP. Add such an exception only if a documented, tested application dependency requires it.
