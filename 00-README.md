# Azure Cloud Security Monitoring & Microsoft Sentinel Detection Lab

**Microsoft Azure · Azure Policy · Log Analytics · KQL · Microsoft Sentinel · SIEM**

> **Validated outcome:** An authorized Azure Policy assignment change was recorded in the subscription Activity Log, ingested into `SecurityLab-Logs`, and associated with a **Medium-severity** Sentinel incident titled **Azure Security Policy Change Detection**. This was a controlled security test, **not a real-world compromise**.

## Project overview

This lab explores how to monitor changes to cloud governance controls. I configured Azure networking and an auditing location policy, used Azure Activity Logs and Log Analytics to inspect administrative operations, and created a scheduled Sentinel detection rule. I tested the workflow by changing the policy assignment and confirming a matching incident record.

**Security problem:** Policy-assignment changes can alter an organization's governance posture. Security teams benefit from visibility into who changes those controls, when, and whether the change warrants investigation.

## Architecture

![Azure policy-change detection architecture](assets/architecture.svg)

The network resources support the lab environment; they are **not** claimed as the direct cause of the incident. The precise ingestion connector configuration was not captured, although successful `AzureActivity` ingestion was verified. See [Architecture and trust boundaries](docs/architecture.md).

## Lab environment

| Component | Configuration |
|---|---|
| Resource group | `Azure-Cloud-SecurityLab` |
| Region | East US |
| Virtual network | `SecurityLab-VNet` |
| Subnet | `SecurityLab-Subnet` (`10.0.1.0/24`) |
| Network security group | `SecurityLab-NSG` (subnet association) |
| Policy assignment | `SecurityLab-Allowed-Locations` |
| Policy definition and effect | Allowed locations · **Audit** |
| Allowed locations after test | `eastus`, `eastus2` |
| Log Analytics workspace | `SecurityLab-Logs` |
| Detection platform | Microsoft Sentinel |
| Scheduled analytics rule | `Azure Security Policy Change Detection` |
| Rule schedule/lookback | Every 5 hours / last 5 hours |
| Severity | Medium |

## Implementation and evidence

### 1. Configure governance with Azure Policy

Created an **Allowed locations** policy assignment using the **Audit** effect. Updated its allowed-region list to include East US 2. Audit provides compliance visibility; it does not prevent out-of-region deployments.

![Azure Policy assignment](screenshots/02-policy-assignment.png)

*Evidence: Azure Policy assignment configuration.*

![Azure Policy assignment details](screenshots/03-policy-assignment-details.png)

*Evidence: Policy assignment details after the authorized update.*

### 2. Verify Azure recorded the change

Reviewed the subscription Activity Log for successful policy-assignment operations.

![Azure Activity Log](screenshots/04-activity-log.png)

*Evidence: Policy assignment administrative event in Azure Activity Log.*

### 3. Query ingested security telemetry

Used KQL against the `AzureActivity` table in `SecurityLab-Logs`. The test query returned **four** policy-assignment records, including **two successful write** events and **two start** events.

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "POLICYASSIGNMENTS"
| order by TimeGenerated desc
```

![AzureActivity query results](screenshots/07-azureactivity-kql-results.png)

*Evidence: Relevant policy-assignment records were available in Log Analytics.*

### 4. Build a Sentinel detection rule

Configured a scheduled **Medium-severity** rule, running every **five hours**, to find successful policy-assignment write/delete events. The query is stored in [`queries/policy-change-detection.kql`](queries/policy-change-detection.kql).

```kusto
AzureActivity
| where OperationNameValue in~ (
    "MICROSOFT.AUTHORIZATION/POLICYASSIGNMENTS/WRITE",
    "MICROSOFT.AUTHORIZATION/POLICYASSIGNMENTS/DELETE"
)
| where ActivityStatusValue =~ "Success"
```

**Screenshot gap:** The analytics-rule configuration page was not included in the original captured evidence. The rule configuration is documented from the guided lab; no substitute screenshot was invented.

### 5. Validate the security incident

The Sentinel overview displayed one new **Medium** incident. Querying `SecurityIncident` confirmed the incident title **Azure Security Policy Change Detection**, severity **Medium**, and status **New**.

![Sentinel overview](screenshots/08-sentinel-overview.png)

*Evidence: Sentinel overview showing the newly reported incident.*

![SecurityIncident KQL validation](screenshots/09-sentinel-incident-kql.png)

*Evidence: KQL validation of the incident title, severity, and status.*

## Results

| Validation | Observed result |
|---|---|
| Azure Policy update | Authorized assignment edit performed |
| Subscription Activity Log | Successful policy-assignment operation visible |
| Log Analytics | Four relevant activity records returned |
| Sentinel incident | `Azure Security Policy Change Detection` · Medium · New |

**Conclusion:** The lab demonstrated a working positive detection path for Azure Policy assignment changes. It did not test an actual compromise, enforcement of location restrictions, or automated remediation.

## Troubleshooting and lessons learned

The Microsoft Defender portal returned a tenant/account access error during the session. Incident creation was verified through the `SecurityIncident` table instead. This reinforced the importance of validating each telemetry stage independently rather than relying on a single portal view.

Additional lessons: the subscription Activity Log and Log Analytics ingestion are separate checks; scheduled rules are not instantaneous; and auditing a policy is different from blocking a policy violation.

Read [Troubleshooting and lessons learned](docs/troubleshooting-and-lessons.md).

## Documentation and files

- [Full implementation walkthrough with all 8 evidence screenshots](docs/implementation.md)
- [Architecture and security boundaries](docs/architecture.md)
- [Detection test procedure and recorded results](docs/testing-and-results.md)
- [Troubleshooting, limitations, and improvements](docs/troubleshooting-and-lessons.md)
- [Publishing guide](docs/github-publishing.md)
- [`queries/`](queries/) — Detection and validation KQL
- [`screenshots/`](screenshots/) — Original selected lab evidence

## Next improvements

Capture VNet/NSG and analytics-rule settings; document the exact activity-ingestion configuration; add repeated tests, detection tuning, and incident response automation with explicit approvals.

## Resume-ready summary

**Azure Cloud Security Monitoring & Microsoft Sentinel Detection Lab** — Built an Azure security monitoring lab using Azure Policy, Log Analytics, KQL, and Microsoft Sentinel; verified successful ingestion of policy-assignment operations and validated generation of a Medium-severity security incident during controlled testing.

## Safety, privacy, and cost

Before publishing publicly, review all screenshots for identifiers, names, or sensitive metadata. Never commit credentials or secrets. Azure resources and Sentinel ingestion can generate charges; disable or delete unused resources after preserving evidence.
