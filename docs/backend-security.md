# Backoffice security

The Umbraco backoffice is an administrative plane and should not be exposed with the same controls as the public website.

This reference recommends a layered model:

```text
Network gate
   +
Central identity / SSO
   +
MFA / Conditional Access
   +
Least-privilege Umbraco roles
   +
Audit and break-glass procedures
```

## Recommended security tiers

| Tier | Network access | Authentication | Recommended use |
|---|---|---|---|
| Baseline | Trusted office/VPN IP allow-list | Umbraco credentials + strong MFA-capable policy where available | Small controlled teams |
| Recommended | Trusted office/VPN/ZTNA allow-list | Entra ID / OIDC SSO + MFA + Conditional Access | Most production enterprise deployments |
| High assurance | Private access through VPN/ExpressRoute/ZTNA | Entra ID / OIDC SSO + MFA + Conditional Access | High-security or regulated environments |

The **Recommended** tier is the default pattern for this repository.

## 1. Use a dedicated backoffice hostname

Example:

```text
admin.example.com
```

Keep the administrative hostname separate from the public frontend hostname. Do not expose internal resource names in public documentation.

## 2. Apply App Service access restrictions

App Service access restrictions are an outer network gate. They execute before application authentication and can reduce the number of clients that can reach the backoffice at all.

### Portal

1. Open the backend App Service.
2. Go to **Networking** -> **Access restrictions**.
3. Add explicit `Allow` rules for approved corporate/VPN/ZTNA CIDRs.
4. Use distinct priorities, for example 100, 110, 120.
5. Set the unmatched/default action to **Deny** only after validation.
6. Review the **SCM site** rules separately.

### Azure CLI example

Use placeholders only in public documentation:

```bash
RG="<RESOURCE_GROUP>"
APP="<BACKEND_APP_NAME>"
CORP_CIDR="<CORPORATE_OR_VPN_CIDR>"

az webapp config access-restriction add \
  --resource-group "$RG" \
  --name "$APP" \
  --rule-name "Allow-Corporate-VPN" \
  --action Allow \
  --priority 100 \
  --ip-address "$CORP_CIDR"

az webapp config access-restriction set \
  --resource-group "$RG" \
  --name "$APP" \
  --default-action Deny
```

Validate from an approved network and from a non-approved network before closing the change.

## 3. Protect Kudu / SCM separately

The SCM endpoint is administrative and deployment-sensitive.

Do **not** blindly select "use the same restrictions as the main site" until you know how CI/CD reaches Kudu/SCM.

Common deployment patterns:

- **Self-hosted Azure DevOps agent in an approved network/VNet** — preferred when strict SCM restrictions are required.
- **Microsoft-hosted Azure DevOps agents** — source IP ranges can change; a narrow static IP allow-list may break deployments.
- **Separate SCM rules** — allow only approved deployment/admin sources while keeping the main backend rules independent.

Example command to reuse main-site restrictions after the deployment path has been validated:

```bash
az webapp config access-restriction set \
  --resource-group "<RESOURCE_GROUP>" \
  --name "<BACKEND_APP_NAME>" \
  --use-same-restrictions-for-scm-site true \
  --scm-default-action Deny
```

If using a deployment slot, validate the slot's SCM behavior as well.

## 4. Use Entra ID / OpenID Connect SSO for Umbraco backoffice

For self-hosted Umbraco, prefer an Umbraco-supported external login provider using OpenID Connect rather than relying only on local passwords.

Recommended identity design:

1. Create an Entra ID app registration for the backoffice login flow.
2. Use an organization-controlled redirect URI for the OIDC callback.
3. Configure the external login provider in the Umbraco application.
4. Map Entra groups/claims to appropriate Umbraco user groups.
5. Avoid mapping all authenticated users to Administrators.
6. Enforce MFA and Conditional Access at the Entra tenant level.
7. Remove or disable departed users in the identity provider rather than managing lifecycle independently in Umbraco.
8. Keep exact configuration aligned with the deployed Umbraco major version.

> Umbraco supports external login providers for backoffice users. The exact code/configuration changes vary by Umbraco version and authentication package, so this repository deliberately avoids publishing a version-specific implementation as universal guidance.

## 5. Conditional Access and MFA

Typical enterprise policy controls include:

- require MFA for the backoffice application;
- restrict access to approved users/groups;
- optionally require compliant or managed devices;
- optionally restrict sign-in countries/locations according to organizational policy;
- block legacy authentication where applicable;
- apply stronger policies to administrators than content editors.

Conditional Access is an identity-policy decision and should be owned by the organization managing the Entra tenant.

## 6. Least-privilege Umbraco roles

Use role/group mappings such as:

```text
Entra group: CMS-Administrators -> Umbraco Administrators
Entra group: CMS-Editors        -> Umbraco Editors
Entra group: CMS-Writers        -> Umbraco Writers
```

Do not use a single broad group for every backoffice user.

Periodically review:

- inactive accounts;
- Administrator membership;
- access to sensitive data;
- permissions to install packages/change configuration;
- service accounts.

## 7. Break-glass access

SSO or network policy failures can lock all administrators out.

Maintain a documented emergency path appropriate to organizational policy, for example:

- one tightly controlled local Umbraco administrator account;
- credentials held in an approved enterprise password vault;
- use only during a declared incident;
- alert/audit when used;
- rotate after use;
- test at a defined interval.

Do not document the actual username, password location, or recovery secrets in a public repository.

## 8. Consider private backoffice access for higher assurance

Where administrators already use private connectivity, a stronger design is:

```text
Administrator
  -> Corporate VPN / ExpressRoute / ZTNA
  -> Private DNS
  -> Backend App Service private access
  -> Umbraco SSO
```

This removes direct public reachability of the administrative application. It is more operationally complex and requires explicit design for CI/CD, health monitoring, callbacks, DNS, and emergency access.

## 9. Easy Auth: optional outer gate, not the default Umbraco recommendation

Azure App Service Authentication (Easy Auth) can authenticate requests before they reach the application. It can be useful for some administrative apps, but adding it in front of Umbraco creates a second authentication layer.

Before using it, validate:

- `/signin-oidc` and other identity callbacks;
- Umbraco backoffice login behavior;
- health probes;
- scheduler/background operations;
- APIs/webhooks used by the backend;
- deployment/SCM endpoints;
- logout and session behavior.

For this reference architecture, the default is **App Service access restrictions + Umbraco external login provider + Entra MFA/Conditional Access**.

## 10. Verify self-calls and scheduled operations

A backend may make outbound requests to its own administrative hostname for licensing, callbacks, scheduled tasks, or custom code.

After applying an IP allow-list:

1. restart the backend;
2. log in;
3. publish content;
4. test scheduled publishing;
5. test forms/workflows;
6. review Application Insights dependencies and failures;
7. only allow the NAT/static egress IP back into the backend if a real, documented self-call requires it.

Do not add broad exceptions pre-emptively.

## 11. Logging and alerting

Monitor:

- Entra sign-in logs for the backoffice app;
- Conditional Access failures;
- Azure Activity Log changes to App Service access restrictions;
- failed backoffice authentication;
- administrator/group changes;
- suspicious Kudu/SCM access;
- repeated 403 responses to the backend hostname;
- break-glass account use.

## Validation checklist

- [ ] Backoffice works from approved corporate/VPN/ZTNA network.
- [ ] Backoffice is denied from an unapproved public network.
- [ ] SSO succeeds for an authorized editor.
- [ ] Unauthorized Entra user/group is denied.
- [ ] MFA/Conditional Access policy is enforced.
- [ ] Least-privilege group mapping is correct.
- [ ] Azure DevOps can still deploy to the backend.
- [ ] Kudu/SCM is not broadly exposed.
- [ ] Health checks continue to work.
- [ ] Scheduled publishing and workflows still work.
- [ ] Break-glass procedure is documented and tested.
