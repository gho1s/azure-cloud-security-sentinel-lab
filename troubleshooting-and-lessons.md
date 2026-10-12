# Troubleshooting, lessons learned, and next improvements

## Challenges encountered

### Defender portal access

The Microsoft Defender portal returned an account/tenant access error during the lab. Rather than claiming direct access to the Sentinel Incidents interface, incident creation was validated using the `SecurityIncident` table in the Log Analytics workspace.

**Lesson:** An inaccessible UI does not necessarily mean the detection pipeline is broken. Check authorized alternate evidence sources and distinguish portal permissions from telemetry availability.

### Locating the Activity Log event

The subscription Activity Log groups and expands entries, which made locating the correct operation and details less intuitive. The successful policy-assignment operation was eventually identified.

**Lesson:** Confirm operation name, result, resource, and timestamp; do not assume any nearby event is the test event.

### Data versus detection

Activity Log entries in the portal and records in `AzureActivity` are not the same verification step. An Azure portal event must be ingested into the workspace before a Sentinel KQL rule can analyze it.

**Lesson:** Troubleshoot in stages: source event → ingestion → KQL query → rule execution → incident.

### Scheduled detection delay

The configured analytics rule runs every five hours. An immediate incident is not guaranteed after a change; query windows and ingestion delays matter.

**Lesson:** Treat scheduled rules as asynchronous and confirm actual results rather than promising immediate alerts.

## Production-minded improvements

- Use a dedicated least-privilege identity and scope roles carefully.
- Document the exact diagnostic-setting or Azure Activity connector configuration.
- Configure an actionable incident response playbook with explicit approvals and safeguards.
- Add detection tuning and suppression/grouping to reduce false positives.
- Use Azure budgets and alerts; stop or delete unused billable resources.
- Consider infrastructure-as-code for repeatable deployment and change review.
- Add negative tests and repeated positive tests to evaluate coverage and reliability.

## Portfolio integrity

All screenshots in this repository are from the user's lab. No claim of actual compromise, production deployment, or automatic remediation is made. Documented but unpictured network setup should be backed by fresh screenshots if needed.
