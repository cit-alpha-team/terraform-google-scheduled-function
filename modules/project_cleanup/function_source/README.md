# Project Cleanup Utility

This is a simple utility that scans a GCP organization for resources matching certain criteria and enqueues them for deletion.

### Cleanup Capabilities

* **Project Deletion:**
    A project is deleted if **all** of the following conditions are met:
    * It is inside the folder specified by `TARGET_FOLDER_ID`.
    * It is older than the `MAX_PROJECT_AGE_HOURS`.
    * Its labels match at least one key-value pair in `TARGET_INCLUDED_LABELS` (if the map is not empty).
    * Its labels do not match any key-value pair in `TARGET_EXCLUDED_LABELS`.

* **Organization-Level Resource Cleanup:**
    * **Tag Keys:** A Tag Key is deleted if it's not in the `TARGET_EXCLUDED_TAGKEYS` list and is older than `MAX_PROJECT_AGE_HOURS`. All associated Tag Values are deleted first.
    * **SCC Notifications:** A Security Command Center notification config is deleted if its name matches a regex in `TARGET_INCLUDED_SCC_NOTIFICATIONS` and its associated Pub/Sub topic belongs to a project that is already marked for deletion.
    * **Cloud Asset Inventory Feeds:** A CAI feed is deleted if its name matches a regex in `TARGET_INCLUDED_FEEDS` and its associated Pub/Sub topic belongs to a project that is already marked for deletion.

* **Billing Sink Cleanup:**
    A Billing Account Log Sink is deleted if **all** of the following conditions are met:
    * It is older than the `MAX_PROJECT_AGE_HOURS`.
    * Its name matches at least one regex in the `TARGET_BILLING_SINKS` list.
    * It is not one of the default sinks (`_Required`, `_Default`).

* **Stale Access Level Cleanup:**
    An Access Level is deleted if it meets either of the following conditions:
    * It is not in use by any Service Perimeter.
    * It has no members defined in its conditions.


## Environment Configuration

The following environment variables may be specified to configure the cleanup utility:

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| `ACCESS_POLICY_NAME` | The name of the Access Policy to clean resources from. | `string` | n/a | no |
| `BILLING_ACCOUNT` | Billing Account used to provision resources. | `string` | n/a | no |
| `BILLING_SINKS_PAGE_SIZE ` | The maximum number of Billing Account Log Sinks to return in the call to `BillingAccountsSinksService.List` service. | `number` | n/a | yes |
| `CLEAN_UP_BILLING_SINKS` | Clean up Billing Account Sinks. | `bool` | n/a | yes |
| `CLEAN_UP_CAI_FEEDS`| Clean up organization level Cloud Asset Inventory Feeds. | `bool` | n/a | yes |
| `CLEAN_UP_SCC_NOTIFICATIONS` | Clean up organization level Security Command Center notifications. | `bool` | n/a | yes |
| `CLEAN_UP_STALE_ACCESS_LEVELS` | Clean up stale Access Levels based on their configuration and usage. | `bool` | n/a | no |
| `CLEAN_UP_TAG_KEYS` | Clean up organization level Tag Keys. | `bool` | n/a | yes |
| `DRY_RUN` | If true, the cleanup function will only log what it would delete without performing deletions. | `bool` | `true` | yes |
| `MAX_PROJECT_AGE_HOURS` | The project age, in hours, at which point deletion should be considered. | integer | n/a | yes |
| `SCC_NOTIFICATIONS_PAGE_SIZE` | The maximum number of notification configs to return in the call to `ListNotificationConfigs` service. The minimun value is 1 and the maximum value is 1000. | `number` | n/a | yes |
| `TARGET_BILLING_SINKS` | List of Billing Account Log Sinks names regex that will be deleted. Regex example: `.*/sinks/sk-c-logging-.*-billing-.*` | `list(string)` | n/a | no |
| `TARGET_EXCLUDED_LABELS` | Labels to match on for identifying projects to avoid deletion. | string | n/a | no |
| `TARGET_EXCLUDED_TAGKEYS` | List of organization Tag Key short names that won't be deleted. | `list(string)` | n/a | no |
| `TARGET_FOLDER_ID` | Folder ID to delete projects under. | string | n/a | yes |
| `TARGET_INCLUDED_FEEDS` | List of organization level Cloud Asset Inventory feeds that should be deleted. Regex example: `.*/feeds/fd-cai-monitoring-.*` | `list(string)` | n/a | no |
| `TARGET_INCLUDED_LABELS` | Labels to match on for identifying projects to delete. | string | n/a | no |
| `TARGET_INCLUDED_SCC_NOTIFICATIONS` | List of organization Security Command Center notifications names regex that will be deleted. Regex example: `.*/notificationConfigs/scc-notify-.*` | `list(string)` | n/a | no |
| `TARGET_ORGANIZATION_ID` | The organization ID whose projects to clean up. | `string` | n/a | yes |

## Required Permissions

This Cloud Function must be run as a Service Account with the following roles:

* `Organization Administrator` (`roles/resourcemanager.organizationAdmin`)
* `Access Context Manager Editor` (`roles/accesscontextmanager.policyEditor`)

If `CLEAN_UP_BILLING_SINKS` is enabled, the Service Account running the Cloud Function needs the `Logs Configuration Writer` (`roles/logging.configWriter`) role on the billing account specified in `BILLING_ACCOUNT`.
