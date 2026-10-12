# Architecture and security boundaries

## System diagram

![Azure cloud policy change detection architecture](../assets/architecture.svg)

## What each component does

1. **Azure Policy** — `SecurityLab-Allowed-Locations` is an **Audit** assignment. It records governance/compliance state; it does **not** block deployments. The allowed location list was updated to include `eastus2` alongside `eastus`.
2. **Azure Activity Log** — Subscription control-plane operations include policy-assignment write operations. The Azure portal showed a successful policy assignment event.
3. **Activity log ingestion** — The subscription's activity events were ingested into Log Analytics `SecurityLab-Logs`, verified by results in `AzureActivity`. The precise connector/diagnostic configuration was **not captured**; this document does not claim a particular ingestion mechanism.
4. **Log Analytics** — KQL queried policy-assignment write/delete operations and their status.
5. **Microsoft Sentinel** — A scheduled analytics rule named `Azure Security Policy Change Detection` queried the data every five hours, using a five-hour lookback. The lab verified an incident titled the same as the rule.
6. **Network resources** — `SecurityLab-VNet`, `SecurityLab-Subnet` (`10.0.1.0/24`), and `SecurityLab-NSG` were configured as supporting cloud-security resources. The evidence does not establish any VM-based attack simulation or NSG traffic-denial test.

## Trust boundaries and limitations

The test used an **authorized administrative change**, not an attack. The alert indicates a policy-assignment operation; it does not attribute malicious intent. The observed incident is a positive detection test, not proof of comprehensive security coverage. Policy-assignment audit detection is different from enforcing location restrictions.
