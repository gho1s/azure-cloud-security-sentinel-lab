# Detection testing and results

## Test objective

Verify that a controlled change to an Azure Policy assignment produces an activity record in the Log Analytics workspace and can be detected by a Microsoft Sentinel scheduled analytics rule.

## Test method

- **Stimulus:** Edit `SecurityLab-Allowed-Locations` to include `eastus2` as an allowed location, retaining `eastus`.
- **Activity Log validation:** Inspect policy-assignment write operations and success status in the Azure subscription Activity Log.
- **Workspace validation:** Query `AzureActivity` for `POLICYASSIGNMENTS` operations during the last 24 hours.
- **Detection validation:** Query `SecurityIncident` for incidents in the last 24 hours and inspect title, severity, and status.

## Recorded observations

| Check | Observation | Evidence |
|---|---|---|
| Policy configuration | Audit policy allowed East US and East US 2 after edit | `screenshots/02-policy-assignment.png`, `03-policy-assignment-details.png` |
| Subscription Activity Log | Policy-assignment operation with Succeeded status | `screenshots/04-activity-log.png` |
| Log Analytics | Four matching activity records, including two successful write events | `screenshots/07-azureactivity-kql-results.png` |
| Sentinel overview | One new Medium incident | `screenshots/08-sentinel-overview.png` |
| SecurityIncident query | Incident titled `Azure Security Policy Change Detection`, severity Medium, status New | `screenshots/09-sentinel-incident-kql.png` |

## Result

**PASS for the observed positive detection test.** Policy assignment activity reached Log Analytics, and a matching Sentinel incident was observed.

## What this test does not prove

- The exact Activity Log diagnostic-setting/connector setup, since its settings were not captured.
- Detection latency or consistent triggering across repeated runs.
- Unauthorized actor attribution, prevention of policy changes, or automatic incident containment.
- Coverage for every possible Azure Policy modification scenario.

## Follow-up validation ideas

1. Capture the analytics-rule configuration screen and the Activity Log connector/diagnostic configuration.
2. Measure event time versus incident creation time over several controlled tests.
3. Add an incident triage checklist and a second detection case.
4. Narrow scope, tune noisy operations, and consider entity mapping and alert grouping.
