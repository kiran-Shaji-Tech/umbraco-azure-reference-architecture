# Networking

## VNet layout

Example only:

| Subnet | Example CIDR | Purpose |
|---|---|---|
| `snet-appsvc-frontend` | `10.10.1.0/26` | Frontend App Service VNet Integration |
| `snet-appsvc-backend` | `10.10.2.0/26` | Backend App Service VNet Integration |
| `snet-private-endpoints` | `10.10.10.0/27` | Private Endpoints for PaaS services |

The App Service integration subnets are delegated to `Microsoft.Web/serverFarms`. Keep the Private Endpoint subnet separate.

## Inbound path

```text
Internet
  -> Public DNS
  -> Upstream WAF/CDN
  -> Azure Front Door
  -> Frontend App Service
```

Backoffice/admin traffic should use a separate hostname and be restricted according to business requirements.

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
