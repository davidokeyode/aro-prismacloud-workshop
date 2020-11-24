<div align="center">
    <a href="./README.md">
        <img src="img/header.png"/>
    </a>
</div>

## Welcome

Welcome to the Prisma Cloud Azure RedHat OpenShift (ARO) workshop! This workshop was created as a walkthrough for an in person or virtual workshop, however you may feel free to run through at your own pace.

### Some Pre-requisites

* An Azure Subscription
  * Ideally this is a PAYG subscription where you have owner role assignment. We'll be performing some actions on Azure AD and other services that may require Application Administrator or Global Administrator level of access also.
  * You can sign up for a new Azure accout [here](https://azure.microsoft.com/free/?WT.mc_id=sentinelworkshop-github-debryen)
  * ARO requires a minimum of 40 cores to create and run an OpenShift cluster. The default Azure resource quota for a new Azure subscription does not meet this requirement. To request an increase in your resource limit, see [Standard quota: Increase limits by VM series](../azure-portal/supportability/per-vm-quota-requests.md)

* A Prisma Cloud Compute license
  * This can be either a Prisma Cloud Compute license only or a Prisma Cloud Enterprise license


### Agenda

The workshop is designed to take approximately 3-4 hours to complete. 

|    | Module                   | Format       |
|----|--------------------------|--------------|
| 01 | [Create an Azure Red Hat OpenShift cluster](../pull/1-create-aro-cluster.md)                         | Presentation |
| 02 | [Getting Started / Setup](02_getting_started.md) | Hands on Lab |
| 03 | [Kusto Query Language](03_kql.md)                | Presentation |
| 04 | [Data Connectors and Workbooks](04_workbooks.md) | Hands on Lab |
| 05 | [Analytics Alerts](05_analytics.md)              | Hands on Lab |
| 06 | [Incidents and Investigations](06_incidents.md)  | Hands on Lab |
| 07 | [Threat Hunting](07_hunting.md)                  | Hands on Lab |
| 08 | [Automation with Playbooks](08_playbooks.md)     | Hands on Lab |
| 09 | [Design Considerations](09_patterns.md)                | Presentation |
| 10 | [Wrap / Clean Up](10_cleanup.md)                 | Hands on Lab |

----

[![Next](img/get_started.png)](./01_intro.md)
