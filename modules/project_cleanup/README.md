# Old Project Cleanup Utility Module

This module schedules a job to clean up GCP resources based on specific criteria, running every 5 minutes via Google Cloud Scheduled Functions. Its main capabilities include:

* **Project Deletion:** Deletes GCP projects that are older than a specified age and match a particular set of labels.
* **Organization-Level Cleanup:** Cleans up organization-level resources that are no longer in use, such as Tag Keys, Security Command Center notifications, and Cloud Asset Inventory feeds.
* **Billing Sink Cleanup:** Removes Billing Account Log Sinks that match certain criteria.
* **Empty Service Perimeter Cleanup:** Deletes Service Perimeters that are empty, obsolete, and marked with a specific flag for cleanup.

For more details on its operation and configuration, please see the [function's readme](./function_source/README.md).

## Requirements

### App Engine

Running this module requires an App Engine app in the specified project/region. More information is in the [root readme](../../README.md#app-engine).

### Enabled Services

The following services must be enabled on the project housing the cleanup function prior to invoking this module:

- Artifact Registry API (`artifactregistry.googleapis.com`)
- Cloud Functions (`cloudfunctions.googleapis.com`)
- Cloud Scheduler (`cloudscheduler.googleapis.com`)
- Cloud Resource Manager (`cloudresourcemanager.googleapis.com`)
- Compute Engine API (`compute.googleapis.com`)
- Cloud Asset API (`cloudasset.googleapis.com`)
- Security Command Center API (`securitycenter.googleapis.com`)
- Cloud Logging API (`logging.googleapis.com`)
- Access Context Manager API (`accesscontextmanager.googleapis.com`)

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| access\_policy\_name | The name of the Access Policy to clean resources from. | `string` | `""` | no |
| billing\_account | Billing Account used to provision resources. | `string` | `""` | no |
| clean\_up\_billing\_sinks | Clean up Billing Account Sinks. | `bool` | `false` | no |
| clean\_up\_empty\_perimeters | If true, the function will clean up empty or obsolete Service Perimeters. | `bool` | `false` | no |
| clean\_up\_org\_level\_cai\_feeds | Clean up organization level Cloud Asset Inventory Feeds. | `bool` | `false` | no |
| clean\_up\_org\_level\_scc\_notifications | Clean up organization level Security Command Center notifications. | `bool` | `false` | no |
| clean\_up\_org\_level\_tag\_keys | Clean up organization level Tag Keys. | `bool` | `false` | no |
| dry\_run | If true, the cleanup function will only log what it would delete without performing deletions. | `bool` | `true` | no |
| function\_docker\_registry | Docker Registry to use for storing the function's Docker images. Allowed values are CONTAINER\_REGISTRY (default) and ARTIFACT\_REGISTRY. | `string` | `null` | no |
| function\_timeout\_s | The amount of time in seconds allotted for the execution of the function. | `number` | `500` | no |
| job\_schedule | Cleaner function run frequency, in cron syntax | `string` | `"*/5 * * * *"` | no |
| list\_billing\_sinks\_page\_size | The maximum number of Billing Account Log Sinks to return in the call to `BillingAccountsSinksService.List` service. | `number` | `200` | no |
| list\_scc\_notifications\_page\_size | The maximum number of notification configs to return in the call to `ListNotificationConfigs` service. The minimun value is 1 and the maximum value is 1000. | `number` | `500` | no |
| max\_resource\_age\_in\_hours | The maximum age in hours that a resource (e.g., project, folder, perimeter) can exist before being considered for cleanup. | `number` | `6` | no |
| organization\_id | The organization ID whose projects to clean up | `string` | n/a | yes |
| perimeter\_cleanup\_flags | A list of string flags used to identify test perimeters for cleanup. The function will look for these flags in the perimeter's description field. | `list(string)` | `[]` | no |
| project\_id | The project ID to host the scheduled function in | `string` | n/a | yes |
| region | The region the project is in (App Engine specific) | `string` | n/a | yes |
| target\_billing\_sinks | List of Billing Account Log Sinks names regex that will be deleted. Regex example: `.*/sinks/sk-c-logging-.*-billing-.*` | `list(string)` | `[]` | no |
| target\_excluded\_labels | Map of project lablels that won't be deleted. | `map(string)` | `{}` | no |
| target\_excluded\_tagkeys | List of organization Tag Key short names that won't be deleted. | `list(string)` | `[]` | no |
| target\_folder\_id | Folder ID to delete all projects under. | `string` | `""` | no |
| target\_included\_feeds | List of organization level Cloud Asset Inventory feeds that should be deleted. Regex example: `.*/feeds/fd-cai-monitoring-.*` | `list(string)` | `[]` | no |
| target\_included\_labels | Map of project lablels that will be deleted. | `map(string)` | `{}` | no |
| target\_included\_scc\_notifications | List of organization Security Command Center notifications names regex that will be deleted. Regex example: `.*/notificationConfigs/scc-notify-.*` | `list(string)` | `[]` | no |
| target\_tag\_name | The name of a tag to filter GCP projects on for consideration by the cleanup utility (legacy, use `target_included_labels` map instead). | `string` | `""` | no |
| target\_tag\_value | The value of a tag to filter GCP projects on for consideration by the cleanup utility (legacy, use `target_included_labels` map instead). | `string` | `""` | no |
| topic\_name | Name of pubsub topic connecting the scheduled projects cleanup function | `string` | `"pubsub_scheduled_project_cleaner"` | no |

## Outputs

| Name | Description |
|------|-------------|
| name | The name of the job created |
| project\_id | The project ID |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
