---
title: Overview of Azure resource logs| Microsoft Docs
description: Understand the supported services and event schema for Azure resource logs.
ms.service:  azure-monitor
ms.subservice: logs
ms.topic: reference
author: rboucher
ms.author: robb
ms.date: 09/20/2019

---

# Azure Resource logs overview
Azure Resource logs are [platform logs](platform-logs-overview.md) emitted by Azure resources that describe their internal operation. All resource logs share a common top-level schema with the flexibility for each service to emit unique properties for their own events.

> [!NOTE]
> Resource logs were previously known as diagnostic logs.

## Collecting resource logs
Resource logs are automatically generated by supported Azure resources, but they aren't collected unless you configure them using a [diagnostic setting](diagnostic-settings.md). Create a diagnostic setting for each Azure resource to forward the logs to the following destinations:

| Destination | Scenario |
|:---|:---|:---|
| [Log Analytics workspace](resource-logs-collect-storage.md) | Analyze the logs with other monitoring data and leverage Azure Monitor features such as log queries and log alerts. |
| [Azure storage](archive-diagnostic-logs.md) | Archive the logs for auditing or backup. |
| [Event hub](resource-logs-stream-event-hubs.md) | Stream the logs to third-party logging and telemetry systems.  |

## Compute resources
Resource logs differ from guest OS-level logs in Azure compute resources. Compute resources require an agent to collect logs and metrics from their guest OS, including such data as event logs, syslog, and performance counters. Use the [Diagnostic Extension](agents-overview.md#azure-diagnostic-extension) to route log data from Azure virtual machines and the [Log Analytics agent](agents-overview.md#log-analytics-agent) to collect logs and metrics from virtual machines in Azure, in other clouds, and on-premises into a Log Analytics workspace. See [Sources of monitoring data for Azure Monitor](data-sources.md) for details.

## Resource logs schema
Each Azure service has its own schema when resource logs are written to one of the destinations, but they all share a the top level schema in the following table. See [Service-specific schemas](#service-specific-schemas) below for links to the schema for each service. 

| Name | Required/Optional | Description |
|---|---|---|
| time | Required | The timestamp (UTC) of the event. |
| resourceId | Required | The resource ID of the resource that emitted the event. For tenant services, this is of the form /tenants/tenant-id/providers/provider-name. |
| tenantId | Required for tenant logs | The tenant ID of the Active Directory tenant that this event is tied to. This property is only used for tenant-level logs, it does not appear in resource-level logs. |
| operationName | Required | The name of the operation represented by this event. If the event represents an RBAC operation, this is the RBAC operation name (eg. Microsoft.Storage/storageAccounts/blobServices/blobs/Read). Typically modeled in the form of a Resource Manager operation, even if they are not actual documented Resource Manager operations (`Microsoft.<providerName>/<resourceType>/<subtype>/<Write/Read/Delete/Action>`) |
| operationVersion | Optional | The api-version associated with the operation, if the operationName was performed using an API (eg. `http://myservice.windowsazure.net/object?api-version=2016-06-01`). If there is no API that corresponds to this operation, the version represents the version of that operation in case the properties associated with the operation change in the future. |
| category | Required | The log category of the event. Category is the granularity at which you can enable or disable logs on a particular resource. The properties that appear within the properties blob of an event are the same within a particular log category and resource type. Typical log categories are “Audit” “Operational” “Execution” and “Request.” |
| resultType | Optional | The status of the event. Typical values include Started, In Progress, Succeeded, Failed, Active, and Resolved. |
| resultSignature | Optional | The sub status of the event. If this operation corresponds to a REST API call, this is the HTTP status code of the corresponding REST call. |
| resultDescription | Optional | The static text description of this operation, eg. “Get storage file.” |
| durationMs | Optional | The duration of the operation in milliseconds. |
| callerIpAddress | Optional | The caller IP address, if the operation corresponds to an API call that would come from an entity with a publicly-available IP address. |
| correlationId | Optional | A GUID used to group together a set of related events. Typically, if two events have the same operationName but two different statuses (eg. “Started” and “Succeeded”), they share the same correlation ID. This may also represent other relationships between events. |
| identity | Optional | A JSON blob that describes the identity of the user or application that performed the operation. Typically this will include the authorization and claims / JWT token from active directory. |
| Level | Optional | The severity level of the event. Must be one of Informational, Warning, Error, or Critical. |
| location | Optional | The region of the resource emitting the event, eg. “East US” or “France South” |
| properties | Optional | Any extended properties related to this particular category of events. All custom/unique properties must be put inside this “Part B” of the schema. |

## Service-specific schemas
The schema for resource diagnostic logs varies depends on the resource type which is defined by the  `resourceId` property) and the `category` properties. This following list shows all Azure services that support resource logs with links to the service and category-specific schema where available.

| Service | Schema & Docs |
| --- | --- |
| Azure Active Directory | [Overview](../../active-directory/reports-monitoring/concept-activity-logs-azure-monitor.md), [Audit log schema](../../active-directory/reports-monitoring/reference-azure-monitor-audit-log-schema.md) and [Sign-ins schema](../../active-directory/reports-monitoring/reference-azure-monitor-sign-ins-log-schema.md) |
| Analysis Services | https://azure.microsoft.com/blog/azure-analysis-services-integration-with-azure-diagnostic-logs/ |
| API Management | [API Management Diagnostic Logs](../../api-management/api-management-howto-use-azure-monitor.md#diagnostic-logs) |
| Application Gateways |[Diagnostics Logging for Application Gateway](../../application-gateway/application-gateway-diagnostics.md) |
| Azure Automation |[Log analytics for Azure Automation](../../automation/automation-manage-send-joblogs-log-analytics.md) |
| Azure Batch |[Azure Batch diagnostic logging](../../batch/batch-diagnostics.md) |
| Azure Database for MySQL | [Azure Database for MySQL diagnostic logs](../../mysql/concepts-server-logs.md#diagnostic-logs) |
| Azure Database for PostgreSQL | [Azure Database for PostgreSQL diagnostic logs](../../postgresql/concepts-server-logs.md#diagnostic-logs) |
| Cognitive Services | [Diagnostic Logging for Azure Cognitive Services](../../cognitive-services/diagnostic-logging.md) |
| Content Delivery Network | [Azure Diagnostic Logs for CDN](../../cdn/cdn-azure-diagnostic-logs.md) |
| CosmosDB | [Azure Cosmos DB Logging](../../cosmos-db/logging.md) |
| Data Factory | [Monitor Data Factories using Azure Monitor](../../data-factory/monitor-using-azure-monitor.md) |
| Data Lake Analytics |[Accessing diagnostic logs for Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-diagnostic-logs.md) |
| Data Lake Store |[Accessing diagnostic logs for Azure Data Lake Store](../../data-lake-store/data-lake-store-diagnostic-logs.md) |
| Event Hubs |[Azure Event Hubs diagnostic logs](../../event-hubs/event-hubs-diagnostic-logs.md) |
| Express Route | Schema not available. |
| Azure Firewall | Schema not available. |
| IoT Hub | [IoT Hub Operations](../../iot-hub/iot-hub-monitor-resource-health.md#use-azure-monitor) |
| Key Vault |[Azure Key Vault Logging](../../key-vault/key-vault-logging.md) |
| Load Balancer |[Log analytics for Azure Load Balancer](../../load-balancer/load-balancer-monitor-log.md) |
| Logic Apps |[Logic Apps B2B custom tracking schema](../../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md) |
| Network Security Groups |[Log analytics for network security groups (NSGs)](../../virtual-network/virtual-network-nsg-manage-log.md) |
| DDOS Protection | [Manage Azure DDoS Protection Standard](../../virtual-network/manage-ddos-protection.md) |
| Power BI Dedicated | [Diagnostic logging for Power BI Embedded in Azure](https://docs.microsoft.com/power-bi/developer/azure-pbie-diag-logs) |
| Recovery Services | [Data Model for Azure Backup](../../backup/backup-azure-reports-data-model.md)|
| Search |[Enabling and using Search Traffic Analytics](../../search/search-traffic-analytics.md) |
| Service Bus |[Azure Service Bus diagnostic logs](../../service-bus-messaging/service-bus-diagnostic-logs.md) |
| SQL Database | [Azure SQL Database diagnostic logging](../../sql-database/sql-database-metrics-diag-logging.md) |
| Stream Analytics |[Job diagnostic logs](../../stream-analytics/stream-analytics-job-diagnostic-logs.md) |
| Traffic Manager | [Traffic Manager Log schema](../../traffic-manager/traffic-manager-diagnostic-logs.md) |
| Virtual Networks | Schema not available. |
| Virtual Network Gateways | Schema not available. |

## Next Steps

* [Learn more about other Azure platform logs](platform-logs-overview.md) that you can use to monitor Azure.
