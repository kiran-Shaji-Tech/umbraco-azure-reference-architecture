# Lessons learned

These are reusable lessons from implementing this pattern.

1. **Separate App Services do not share local files.** Any shared Umbraco state must be externalized.
2. **Private Endpoints are mostly about DNS.** Creating the endpoint is easy; validating the correct hostname resolves privately is the critical step.
3. **NAT Gateway alone is not enough for App Service.** Route outbound internet traffic through VNet Integration for the NAT public IP to become the source address.
4. **Harden in stages.** Create private access, validate, then disable public access.
5. **Do not change unrelated dependencies during network cutover.** Keep rollback simple.
6. **Backend and frontend can share one NAT IP while using separate subnets.** This is useful for third-party allow-lists.
7. **Key Vault needs extra care when certificates are consumed by platform services.** Validate service-specific trusted access before lockdown.
8. **Build once and deploy the same artifact.** Role drift is harder to debug than configuration differences.
9. **Version-specific Umbraco behavior matters.** Document exact CMS and package versions instead of relying on generic configuration examples.
10. **A working architecture still needs operational proof.** Validate from Kudu and at the application layer, not only from the Azure portal.
