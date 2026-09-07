# Architecture

## Application roles

The reference topology uses one Umbraco installation deployed into two App Services from the same build artifact.

| Tier | Role | Responsibilities |
|---|---|---|
| Frontend | Subscriber | Public rendering, scalable read traffic, public Forms execution where applicable |
| Backend | SchedulingPublisher | Backoffice, editorial workflows, publishing, scheduled publishing, administrative operations |

Both tiers share the same durable application state.

## Shared services

- **Azure SQL Database** — Umbraco relational state.
- **Azure Blob Storage** — media and other shared blob-backed data.
- **Azure Managed Redis** — distributed cache/state when required by the implementation.
- **Azure Key Vault** — secrets, keys, and certificates.
- **Application Insights** — telemetry and dependency visibility.

## Why not share App Service local storage?

App Service instances and separate App Services must not be treated as a shared filesystem. Any state that must be identical across frontend and backend should be moved to a suitable shared service.

## Availability model

The frontend can scale horizontally while the backend can remain intentionally constrained. Health probes and deployment slots should be used where appropriate.

Cross-region disaster recovery is not part of this base reference architecture. Add it only after defining business RTO/RPO requirements.
