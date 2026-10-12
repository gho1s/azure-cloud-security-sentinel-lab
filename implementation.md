# Implementation walkthrough and evidence

This walkthrough records the work performed in the Azure lab. Each image below comes from the actual lab session; no portal screenshots were fabricated. Where an image is unavailable, the step is described as configuration context rather than visual proof.

## 1. Establish the lab environment

Created the resource group `Azure-Cloud-SecurityLab` in East US and configured `SecurityLab-VNet`, subnet `SecurityLab-Subnet` (`10.0.1.0/24`), and `SecurityLab-NSG` associated with the subnet. This provided a scoped lab environment and basic network control plane.

**Evidence limitation:** A resource-group inventory and VNet/NSG configuration screenshot are not included in this version. These settings are documented from the completed lab workflow; add portal screenshots if you want independent visual proof.

## 2. Create and configure the location policy

Assigned the Azure Policy definition **Allowed locations** under `SecurityLab-Allowed-Locations` with **Audit** effect. The initial allowed location was East US; then updated the assignment to allow East US and East US 2. Audit reports compliance findings rather than denying resource creation.

![Policy compliance view](../screenshots/01-policy-compliance.png)

*Evidence 01 — Policy compliance view. Shows governance visibility; compliance at one point in time is not a guarantee of future compliance.*

![Policy assignment configuration](../screenshots/02-policy-assignment.png)

*Evidence 02 — Azure Policy assignment and its settings.*

![Policy assignment details](../screenshots/03-policy-assignment-details.png)

*Evidence 03 — Policy assignment details following the controlled configuration change.*

## 3. Confirm the administrative event was recorded

Opened the subscription Activity Log and located policy-assignment operations with a **Succeeded** status. This confirms Azure recorded administrative control-plane activity.

![Activity log](../screenshots/04-activity-log.png)

*Evidence 04 — Subscription Activity Log showing the policy-assignment operation.*

## 4. Confirm Log Analytics ingestion

Opened `SecurityLab-Logs` and queried `AzureActivity` for policy-assignment operations. The observed results contained four activity records, including two successful policy-assignment write operations and two corresponding start events. This is direct evidence that relevant activity reached the workspace.

![Log Analytics workspace](../screenshots/06-workspace.png)

*Evidence 05 — Log Analytics workspace used for security telemetry.*

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "POLICYASSIGNMENTS"
| order by TimeGenerated desc
```

![AzureActivity KQL results](../screenshots/07-azureactivity-kql-results.png)

*Evidence 06 — KQL results showing policy-assignment activity. The observed time range and counts are specific to the lab test.*

## 5. Implement the Sentinel detection rule

Configured a scheduled rule called **Azure Security Policy Change Detection** with **Medium** severity, running every **5 hours** and querying a **5-hour** lookback. The rule checks successful `Microsoft.Authorization/policyAssignments/write` and `.../delete` events. The threshold is more than zero returned rows.

```kusto
AzureActivity
| where OperationNameValue in~ (
    "MICROSOFT.AUTHORIZATION/POLICYASSIGNMENTS/WRITE",
    "MICROSOFT.AUTHORIZATION/POLICYASSIGNMENTS/DELETE"
)
| where ActivityStatusValue =~ "Success"
```

**Evidence limitation:** A direct screenshot of the analytics-rule settings is not included in the existing package. The rule's configuration is documented from the guided lab and its subsequent observed outcome.

## 6. Verify a Sentinel incident

The Sentinel overview showed **one new Medium-severity incident** in the test period. A `SecurityIncident` query independently confirmed the title **Azure Security Policy Change Detection**, severity **Medium**, and status **New**. This completes the observed detection chain from policy-assignment activity to an incident record.

![Sentinel overview](../screenshots/08-sentinel-overview.png)

*Evidence 07 — Sentinel overview showing the incident count and severity.*

```kusto
SecurityIncident
| where TimeGenerated > ago(24h)
| project TimeGenerated, Title, Severity, Status
| order by TimeGenerated desc
```

![Confirmed incident in KQL](../screenshots/09-sentinel-incident-kql.png)

*Evidence 08 — `SecurityIncident` results with the matching detection title.*

## 7. Document the outcome

Captured the screenshots, saved reusable `.kql` files, and wrote the findings, limitations, and next steps in this repository. The incident was a controlled positive test; it does not demonstrate a real intrusion or an automated containment action.
