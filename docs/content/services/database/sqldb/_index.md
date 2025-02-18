+++
title = "SQL DB"
description = "Best Practices and Resiliency Recommendations for Azure SQL Database"
date = "2023-05-30"
author = "temalo"
msAuthor = "temalo"

draft = false
+++

The presented resiliency recommendations in this guidance include Azure Database Services

## Summary of Recommendations

{{< table style="table-striped" >}}
| Recommendation                                                                                                                                                                  | Impact  |  State  | ARG Query Available |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----:  | :-----: | :-----------------: |
| [SQLDB-1 - Use Active Geo Replication to Create a Readable Secondary in Another Region](#sqldb-1---use-active-geo-replication-to-create-a-readable-secondary-in-another-region) | High    | Preview |         No          |
| [SQLDB-2 - Use Auto Failover Groups that can include one or multiple databases, typically used by the same application](#sqldb-2---use-auto-failover-groups-that-can-include-one-or-multiple-databases-typically-used-by-the-same-application)                                                           |  High   | Preview |         No          |
| [SQLDB-3 - Use a Zone-Redundant database](#sqldb-3---use-a-zone-redundant-database)                                                                                             | Medium  | Preview |         No          |
| [SQLDB-4 - Implement Retry Logic](#sqldb-4---implement-retry-logic)                                                                                                             |  High   | Preview |         No          |
| [SQLDB-5 - Monitor your Azure SQL Database in near-real time to detect reliability incidents](#sqldb-5---monitor-your-azure-sql-database-in-near-real-time-to-detect-reliability-incidents)                                                                                    |  High   | Preview |         No          |
| [SQLDB-6 - Back up your keys](#sqldb-6---back-up-your-keys)                                                                                                                     | Medium  | Preview |         No          |
| [SQLDB-7 - Test application fault resiliency](#sqldb-7---test-application-fault-resiliency)                                                                                     | Medium  | Preview |         No          |
{{< /table >}}

{{< alert style="info" >}}

Definitions of states can be found [here]({{< ref "../../../_index.md#definitions-of-terms-used-in-aprl">}})

{{< /alert >}}

## Recommendations Details

### SQLDB-1 - Use Active Geo Replication to Create a Readable Secondary in Another Region

#### Impact: High

#### Recommendation/Guidance
If your primary database fails, perform a manual failover to the secondary database. Until you fail over, the secondary database remains read-only. Active geo-replication enables you to create readable replicas and manually failover to any replica if there is a datacenter outage or application upgrade. Up to four secondaries are supported in the same or different regions, and the secondaries can also be used for read-only access queries. The failover must be initiated manually by the application or the user. After failover, the new primary has a different connection end point.

#### Resources
- [Active Geo Replication](https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview)

### SQLDB-2 - Use Auto Failover Groups that can include one or multiple databases, typically used by the same application

#### Impact: High

#### Recommendation/Guidance
You can use the readable secondary databases to offload read-only query workloads. Because autofailover groups involve multiple databases, these databases must be configured on the primary server. Autofailover groups support replication of all databases in the group to only one secondary server or instance in a different region.

#### Resources
- [AutoFailover Groups](https://learn.microsoft.com/en-us/azure/azure-sql/database/auto-failover-group-overview?tabs=azure-powershell)
- [DR Design](https://learn.microsoft.com/en-us/azure/azure-sql/database/designing-cloud-solutions-for-disaster-recovery)

### SQLDB-3 - Use a Zone-Redundant Database

#### Impact: Medium

#### Recommendation/Guidance
By default, the cluster of nodes for the premium availability model is created in the same datacenter. With the introduction of Azure Availability Zones, SQL Database can place different replicas of the Business Critical database to different availability zones in the same region. To eliminate a single point of failure, the control ring is also duplicated across multiple zones as three gateway rings (GW). The routing to a specific gateway ring is controlled by Azure Traffic Manager (ATM). Because the zone redundant configuration in the Premium or Business Critical service tiers doesn't create extra database redundancy, you can enable it at no extra cost.

#### Resources
- [Azure Traffic Manager](https://learn.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
-[Zone Redundant Databases](https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla)

### SQLDB-4 - Implement Retry Logic

#### Impact: High

#### Recommendation/Guidance
Although Azure SQL Database is resilient when it concerns transitive infrastructure failures, these failures might affect your connectivity. When a transient error occurs while working with SQL Database, make sure your code can retry the call.

#### Resources
- [How to Implement Retry Logic](https://learn.microsoft.com/en-us/azure/azure-sql/database/troubleshoot-common-connectivity-issues)

### SQLDB-5 - Monitor your Azure SQL Database in Near Real-Time to Detect Reliability Incidents

#### Impact: Medium

#### Recommendation/Guidance
Use one of the available solutions to monitor SQL DB to detect potential reliability incidents early and make your databases more reliable. Choose a near real-time monitoring solution to quickly react to incidents.

#### Resources
- [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/insights/azure-sql#analyze-data-and-create-alerts)
- [Azure SQL Database Monitoring](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-sql-database-azure-monitor)
- [Monitoring SQL Database Reference](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-sql-database-azure-monitor-reference)

### SQLDB-6 - Back Up Your Keys

#### Impact: Medium

#### Recommendation/Guidance
It is highly recommended to use Azure Key Vault (AKV) to store encryption keys related to Always Encrypted configurations, however it is not required. If you are not using AKV, then ensure that your keys are properly backed up.

#### Resources
- [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
- [Getting Started with Always Encrypted](https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-landing?view=azuresql)

### SQLDB-7 - Test application fault resiliency

#### Impact: Medium

#### Recommendation/Guidance

High availability is a fundamental part of the SQL Database platform that works transparently for your database application. However, we recognize that you may want to test how the automatic failover operations initiated during planned or unplanned events would impact an application before you deploy it to production. You can manually trigger a failover by calling a special API to restart a database, or an elastic pool. In the case of a zone-redundant serverless or provisioned General Purpose database or elastic pool, the API call would result in redirecting client connections to the new primary in an Availability Zone different from the Availability Zone of the old primary. So in addition to testing how failover impacts existing database sessions, you can also verify if it changes the end-to-end performance due to changes in network latency. Because the restart operation is intrusive and a large number of them could stress the platform, only one failover call is allowed every 15 minutes for each database or elastic pool.

#### Resources

- [Test application fault resiliency](https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla?view=azuresql&tabs=azure-powershell#testing-application-fault-resiliency)
