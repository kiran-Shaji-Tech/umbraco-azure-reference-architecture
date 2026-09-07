# Cost and trade-offs

This pattern favors predictable production behavior over the absolute minimum resource count.

## Main cost drivers

- App Service Plan tier and scale-out instance count;
- Azure Front Door traffic and WAF policy;
- Azure SQL compute/storage/backup retention;
- Azure Managed Redis tier;
- NAT Gateway hourly + data processing charges;
- Private Endpoint hourly/data processing charges;
- Log Analytics/Application Insights ingestion and retention;
- upstream WAF/CDN vendor charges.

## Trade-offs

### Separate frontend/backend App Services

**Benefit:** isolation and independent scaling.  
**Cost:** additional App Service resources and operational configuration.

### NAT Gateway

**Benefit:** one predictable egress IP.  
**Cost:** extra fixed and data-processing charges.

### Private Endpoints

**Benefit:** reduced public data-plane exposure.  
**Cost:** endpoint/DNS complexity and operational access considerations.

### Multi-region DR

Not included by default. It materially increases cost and operational complexity; justify it with business RTO/RPO requirements.
