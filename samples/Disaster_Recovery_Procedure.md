# Disaster Recovery Procedure

**For Internal Use Only**  
**Date:** May 24, 2026  
**Organization:** Acme  
**Address:** 123 Example Street, Example City, ST 00000

---

## Table of Contents

- [Acronyms](#acronyms)
- [Document Revision History](#document-revision-history)
- [Overview](#overview)
  - [Making the Decision to Failover](#making-the-decision-to-failover)
  - [Prerequisites and Expectations](#prerequisites-and-expectations)
- [Identifying the Acme Databases](#identifying-the-acme-databases)
  - [Acme SQL Engine Databases and SQL Analysis Services Cubes](#acme-sql-engine-databases-and-sql-analysis-services-cubes)
  - [Databases to Include in Backup Plan](#databases-to-include-in-backup-plan)
  - [Databases to Be Synched via Log Shipping, Database Mirroring, or AlwaysOn](#databases-to-be-synched-via-log-shipping-database-mirroring-or-alwayson)
- [Identifying Your Environment](#identifying-your-environment)
  - [Shared Alias Name Examples](#shared-alias-name-examples)
  - [Dedicated Alias, FQDN, IP, and Hostname-Based Environments](#dedicated-alias-fqdn-ip-and-hostname-based-environments)
  - [Dedicated Alias Name Examples](#dedicated-alias-name-examples)
- [Pre-Failover Procedures](#pre-failover-procedures)
- [Failing Over](#failing-over)
  - [Failover Procedure](#failover-procedure)
  - [List of Acme Databases](#list-of-acme-databases)
  - [List of SQL Permissions Required by the SQL and Windows Service Accounts Used by Acme](#list-of-sql-permissions-required-by-the-sql-and-windows-service-accounts-used-by-acme)
- [Post-Failover Procedures](#post-failover-procedures)
- [Testing and Troubleshooting](#testing-and-troubleshooting)
  - [Acme Support Contact Information](#acme-support-contact-information)

---

## Acronyms

| Term | Definition |
|---|---|
| AlwaysOn AG | Availability Groups |
| AlwaysOn FCI | Failover Clustering Instances |
| BI | Business Intelligence |
| DBA | Database Administrator |
| DNS | Domain Name Servers |
| DR | Disaster Recovery |
| FQDN | Fully Qualified Domain Name |
| IIS | Internet Information Services |
| LDAP | Lightweight Directory Access Protocol |
| SQL | Structured Query Language |
| SSO | Single Sign On |
| UI | User Interface |

## Document Revision History

| Version | Date | Author | Comment |
|---|---|---|---|
| 0.1 | 05/16/2026 | John Doe | Document creation |
| 0.2 | 05/17/2026 | Documentation Specialist | Format, proof document |
| 1.0 | 05/24/2026 | Documentation Specialist | Finalize and PDF |

---

## Overview

The purpose of this document is to provide the Acme Administrator with the needed steps required to switch/failover from the Acme PROD environment to the Acme disaster recovery (DR) environment (or vice versa). The purpose, reason, or necessity to perform a switch-over is typically caused by a planned or unplanned event.

| Event | Example |
|---|---|
| Planned event | Acme installations, upgrades, and patches |
| Unscheduled event | Unscheduled system outages affecting access to, from, and between Acme servers, services, and users. |

### Making the Decision to Failover

Failing over can be an extensive process and may use a lot of time and resources. Establishing rules and procedures for "If and When" to failover is recommended. For example, performing a complete failover may take 30 minutes from start to completion. This may not be the best option if the outage is only scheduled for approximately 10 minutes or has a resolution time of approximately 10 minutes (accidental reboot of the primary SQL or IIS server). Establishing a well-balanced set of rules and procedures is encouraged.

### Prerequisites and Expectations

The Acme Failover guide relies on several prerequisites and expectations from the customer.

Keep in mind the following information:

- Log Shipping, Database Mirroring, and/or AlwaysOn (AG or FCI) database syncing from the primary environment's (PROD) SQL/business intelligence (BI) servers to the secondary environment's (DR) SQL/BI servers are already set up/in-place and in operational status.
- The Acme Failover guide does not include instructions on how to set up, disconnect, reconnect/re-establish, or bring online databases when set up for Log Shipping, Database Mirroring, or AlwaysOn (AG or FCI) database syncing.
- The Acme Failover guide does not provide instructions on SQL Server Database Backup and Restore procedures.
- A Database Administrator or System Administrator with knowledge of managing database backups/restores, Log Shipping, Database Mirroring, and/or AlwaysOn (AG or FCI) is recommended to set up, manage, and monitor the primary and secondary SQL server operations.
- Both the PROD and DR environments have their own dedicated Acme IIS/App server.
- A full/complete failover is the only recommended procedure.
- A Database Backup Plan is a requirement to protect data in the event of data corruption. A failover environment will not protect against data corruption in most cases and therefore should not be a substitute for a Database Backup Plan.

---

## Identifying the Acme Databases

Developing and implementing a backup plan/strategy is recommended.

Backup procedures, schedules, and plans vary from one customer to the next and even between database administrator (DBA) teams. Therefore, it is up to the customer (DBA) to implement a backup plan/strategy. In the event there are no established backup plans or procedures, consider the following minimum recommendations:

- A comprehensive backup plan consists of full daily/nightly backups or full native backups taken at least once a week with incremental backups taken daily.
- As previously mentioned, a database backup plan is a requirement to protect data in the event of data corruption. A Failover Environment will not protect against data corruption in most cases and should not be a substitute for a database backup plan.

### Acme SQL Engine Databases and SQL Analysis Services Cubes

The following table displays a list of databases and service cubes.

| MS SQL Engine Databases | MS SQL Analysis Service Cube |
|---|---|
| Acme | Acme |
| AcmeContent | |
| AcmeDataWarehouse | |
| AcmeDashboard | |
| AcmeDashboardWarehouse | |
| AcmeAudit | |

### Databases to Include in Backup Plan

Although not listed below, the Acme SQL analysis cube may be optionally backed up. The Acme cube can be rebuilt by running the Acme Business Intelligence Installer and is, therefore, not as critical as the databases listed below.

> **Note:** Some databases listed below may not exist depending on the customer's Acme license.

- Acme
- AcmeContent
- AcmeAudit (if Acme Audit module is used)
- AcmeDataWarehouse (if Acme Business Intelligence module is used)
- AcmeDashboard (if Acme Dashboards module is used)
- AcmeDashboardWarehouse (if Acme Dashboards module is used)

### Databases to Be Synched via Log Shipping, Database Mirroring, or AlwaysOn

The following databases should be synched via log shipping, database mirroring, or AlwaysOn:

- Acme
- AcmeContent
- AcmeAudit (if Acme Audit module is used)
- AcmeDataWarehouse (if Acme Business Intelligence module is used)
- AcmeDashboard (if Acme Dashboards module is used)
- AcmeDashboardWarehouse (if Acme Dashboards module is used)

---

## Identifying Your Environment

For load balancer and shared alias-based environments, proceed to **Failing Over** and then **Testing and Troubleshooting** if the Acme environments (PROD and DR) use shared domain name servers (DNS) aliases for all connections between the Acme servers (Acme Web UI, IIS server, SQL server, BI server, etc.).

### Shared Alias Name Examples

The following table represents an environment where a Load Balancer and/or manual DNS switchover redirects from PROD to DR systems or vice versa. If such an environment applies to your Acme Environment, go to **Failing Over**.

| Component | Alias |
|---|---|
| PROD and DR Acme Web UI Alias | `http://ACME/acme` |
| PROD and DR IIS Server Alias name | `ACME` |
| PROD and DR SQL Server Alias name | `ACMESQL` |
| PROD and DR BI Server Alias name | `ACMEBI` |

### Dedicated Alias, FQDN, IP, and Hostname-Based Environments

Proceed to **Pre-Failover Procedures** and review all sections of this guide if the Acme environments (PROD and DR) use fully qualified domain name (FQDN), hostname, IP, or dedicated DNS aliases for all connections between the Acme servers (Acme Web UI, IIS server, SQL server, BI server, etc.).

### Dedicated Alias Name Examples

The following table is an example of aliases; however, it is also applicable to FQDN, hostname, and IP.

| Component | Alias |
|---|---|
| PROD Acme Web UI Alias | `http://ACMEPROD/acme` |
| DR Acme Web UI Alias | `http://ACMEDR/acme` |
| PROD IIS Server Alias | `ACMEPROD` |
| DR IIS Server Alias | `ACMEDR` |
| PROD SQL Server Alias | `ACMEPRODSQL` |
| DR SQL Server Alias | `ACMEDRSQL` |
| PROD BI Server Alias | `ACMEPRODBI` |
| DR BI Server Alias | `ACMEDRBI` |

The scenario above represents an environment where each server-to-server connection is using a dedicated connection name. A Load Balancer may still exist in the above example to disable access to the primary environment and bring online the secondary environment. Users connect to PROD and DR using unique/different Acme URLs when accessing PROD vs. DR. This section also applies to an environment where a simpler URL redirects the user to the more complex URL normally used by Acme.

For example, the user types `http://ACME` in a web browser and it redirects the user to `http://ACMEPROD.domain.local/acme`.

---

## Pre-Failover Procedures

Perform the steps in this section on PROD and DR prior to setting up Log Shipping/DB Mirroring/AlwaysOn (AG or FCI) between PROD and DR prior to a failover.

If Log Shipping/DB Mirroring/AlwaysOn (AG or FCI) are set up and running, replace the server name (FQDN) in the URL from PROD's IIS/App server name to DR's IIS/App server name (or vice versa). The information captured in this section will be used during a failover from PROD to DR or vice versa.

**FQDN URL example:** `http://AcmePRODServerName.domain.local/Acme`

> **Note:** If an alias (DNS/Load Balancer) is used to switch to/from PROD and DR to access the Acme URLs, skip this section and continue to **Failing Over**.  
> **Alias URL example:** `http://DNSAliasName/Acme`

1. Log in to the PROD Acme Web UI.
2. Open the Acme Administration module (gear icon).
3. Navigate to **Configuration**.
4. Click **General**.
5. Note the Acme Vir URL for later use once switching back to PROD from the DR server.
   - Verify the end of the URL ends with `/`.

6. In **Configuration**, go to **LDAP**.
7. From **LDAP Authentication**, note the preferred Acme UI login method selected to access the Acme Application.
   - **Off** = Acme Managed/FORMS Authentication
   - **Mixed** = Windows Authentication (SSO) primary and Acme Managed/FORMS Authentication secondary
   - **On** = Windows Authentication (SSO) only
   - **External** = Third Party Authentication (example = CA SiteMinder)

> **Note:** If not using Windows Authentication or External Authentication with Acme, go to Step 13.

8. If using the Acme GroupSync Module to Authenticate (SSO), go to **Permissions**.
9. Go to **Group locations**.
10. Click **Edit user groups**.
11. Note each Acme Group Name and LDAP URL string for that group.
12. Verify each URL points to the correct Security Group and Active Directory/LDAP server, as it may differ if the PROD and DR systems are in different domains or communicate to different LDAP server(s).

> **Note:** If using the GroupSync module (Steps 8-12) or not using Windows Authentication or External Authentication with Acme, go to Step 16.

13. If using the older LDAP sync Module in Acme to use Windows or External Authenticate (SSO), click **LDAP** in the **Configuration** section.
14. In **LDAP root**, note the LDAP URL string.
15. Verify that the URL points to the correct Active Directory server (if PROD LDAP server differs from DR LDAP server).

16. Navigate to **Configuration**.
17. Click **Dashboards and reports**.
18. In **Reports URL**, note the Reports URL.
19. The default SSRS instance for report server services URL ends in `ReportServer`.

> **Note:** If not using Acme Reporting Module, go to Step 20.  
> **Note:** If not using Acme Dashboards Module, go to Step 22.

20. From **Configuration**, go to **Dashboards and reports**.
21. In **Dashboards URL**, note the Dashboards URL.

22. In **Configuration**, go to **Discover**.
23. In **Service Location**, verify the URL points to the Discover Service.

> **Note:** If not using Acme Discover, skip Steps 22 and 23.

---

## Failing Over

Depending on the type of failover event, follow the procedure below:

- If the failover event is planned, perform Steps 1-4.
- If the failover event is unplanned, skip Step 1 and perform Steps 2-4.

### Failover Procedure

1. The DBA disables/disconnects the Log Shipping/DB Mirroring/AlwaysOn (AG or FCI) connection from PROD to DR (or vice versa).
2. The DBA brings online the databases on the DR environment (or PROD if failing over back to PROD from DR).
3. From the active environment, navigate to SQL Server Agent (or Task Scheduler if SQL Server Agent is not used) on the server where the Acme ETL jobs reside (SQL or BI).
4. Enable the ETL main job if planning to use the DR environment for longer than a day.

> **Note:** For Dedicated Alias, FQDN, IP, and Hostname-Based Environments only, go to Step 5.

5. Connect to the SQL/BI Server where the `AcmeDataWarehouse` database resides.
6. Locate the `dbo.SSIS Configuration` table.
7. Right-click the table and select **Edit Top 200 Rows**.
8. Modify the following:

| Table | Action |
|---|---|
| `ConnSource` | Modify the Data Source to point to the SQL Server hosting the `Acme` Database |
| `ConnDW` | Modify the Data Source to point to the SQL Server hosting the `AcmeDataWarehouse` Database |
| `ConnAcmeCube` | Modify the Data Source to point to the SQL Analysis Server hosting the `Acme` Cube |
| `ConnSourceGDCMWorkflow` | Modify the Data Source to point to the SQL Server hosting the `Acme` Database |
| `ConnArchive` | Modify the Data Source to point to the SQL Server hosting the `AcmeDataWarehouse` Database |
| `ETL` | Modify the Data Source to point to the SQL Integration Server hosting the `Acme` SSIS Package Files |
| `ConnReportServer` | Modify the Data Source to point to the SQL Server hosting the `ReportServer` Database |
| `ConnDundasBI` | Modify the Data Source to point to the SQL Server hosting the `AcmeDashboard` Database |

### List of Acme Databases

- Acme
- AcmeContent
- AcmeAudit (if Acme Audit module is used)
- AcmeDataWarehouse (if Acme Business Intelligence module is used)
- AcmeDashboard (if Acme Dashboards module is used)
- AcmeDashboardWarehouse (if Acme Dashboards module is used)

### List of SQL Permissions Required by the SQL and Windows Service Accounts Used by Acme

- `db_owner` role for SQL and Service accounts used for running the Acme Application, Acme Services, IIS Application Pools, and Proxy Accounts.
- `db_reader`, `db_writer`, and `execute` for accounts used by Acme Data Sources in SSRS.

---

## Post-Failover Procedures

Follow the post-failover procedures after successfully completing the **Failover Procedure**.

1. Log in to the Acme Web UI.
2. Open the Acme Administration module (gear icon).
3. Navigate to **Configuration**.
4. Click **General**.
5. Verify the Acme Vir URL points to the correct alias/web server name.
   - Ensure the end of the URL ends with `/`.
6. If it does not, click **Edit**, type the correct link, and click **Save**.

7. In **Configuration**, go to **LDAP**.
8. From **LDAP Authentication**, verify the preferred Acme UI login method is selected to access the Acme Application.
   - **Off** = Acme Managed/FORMS Authentication
   - **Mixed** = Windows Authentication (SSO) primary and Acme Managed/FORMS Authentication secondary
   - **On** = Windows Authentication (SSO) only
   - **External** = Third Party Authentication (example = CA SiteMinder)

> **Note:** If not using Windows Authentication or External Authentication with Acme, go to Step 13.

9. If using the Acme GroupSync Module to Authenticate (SSO), go to **Permissions**.
10. Go to **Group locations**.
11. Click **Edit user groups**.
12. Verify the LDAP URL settings are correct for each Acme Group. Each URL should point to the correct Security Group and Active Directory/LDAP server.

> **Note:** If using the GroupSync module or not using Windows Authentication or External Authentication with Acme, skip Step 13.

13. If using the older LDAP sync module in Acme to use Windows or External Authenticate (SSO), go to **LDAP** in **Configuration**.
14. In **LDAP root**, verify LDAP URL settings are pointing to the correct Active Directory server.

> **Note:** If not using Acme Reporting Module, go to Step 18.

15. Navigate to **Configuration**.
16. Click **Dashboards and reports**.
17. In **Reports URL**, verify the URL points to the correct SQL reporting services server reports service URL.
   - The default SSRS instance for report server services URL ends in `ReportServer`.
> **Note:** If not using Acme Dashboards Module, go to Step 20.

18. From **Configuration**, go to **Dashboards and reports**.
19. In **Dashboards URL**, verify the URL points to the correct dashboards URL.

> **Note:** If not using Acme Discover, go to Step 22.

20. From **Configuration**, go to **Discover**.
21. In **Service Location**, verify the URL points to the Discover Service.
22. After all changes have been made and saved, restart IIS and retest accessing the Acme Application. Acme should function with no errors.

---

## Testing and Troubleshooting

1. Log in to Acme and perform a simple use test of all commonly accessed/used modules.
2. If errors appear, review the guide again and verify all steps were followed.
3. If all steps were followed and errors are still present, contact Acme Support.
4. On the server where the Acme ETL jobs reside (SQL or BI), monitor the ETL main job and verify it runs successfully the next day.
5. If the job fails or failed during its scheduled run time, contact Acme Support.

### Acme Support Contact Information

For questions or concerns, contact Acme Support via email or phone.

- **Email:** support@acme.example
- **UK:** +44 20 0000 0000
- **US:** +1 (555) 010-0002

---

**Acme**  
For Internal Use Only  
Contact information intentionally generalized for sanitization.
