### Multi-Tenant-PostgreSQL-as-a-Service in SAP Cloud PLatform

SAP Cloud Platform is an open platform-as-a-service (PaaS) product that provides core platform and backing services, for building and extending cloud applications on multiple cloud infrastructure providers.

    One of the core services provided by SCP is Multi-Tenant PostgreSQL as a Service (MT-PostgreSQL-as-a-Service). MT-PostgreSQL-as-a-Service is the virtualized view of a PostgreSQL service for a given tenant in a multi-tenant system. One or more of such MT-PostgreSQL-as-a-Service instances are multiplexed into a dedicated PostgreSQL-as-a-Service. Each dedicated PostgreSQL-as-a-Service instance consists of 5 VMs - Postgres-Master, Postgres-Standby and 3-PGPOOL VMs. Data is replicated asynchronously from Postgres-Master to Postgres-Standby.

SCP manages 2400+ MT-PostgreSQL-as-a-Service instances across multiple IAASs. 

MT-PostgreSQL-as-a-Service is materialized by defining database as the Multi-Tenant-Unit(MTU) whereby each tenant is encapsulated into a database. Logical dbs provides clear advantages like isolation, extension enablement, connection control, higher security which makes it the ideal choice as MTU.

### Cluster-Setup
![N|Solid](https://github.com/ankita0811/PostgresqlConf/blob/master/clustersetup.png?raw=true)

### Security-and-Isolation
Each dedicated-PostgreSQL-as-a-Service is associated with 1+ __security groups__, wihch acts as a virtual firewall that controls traffic to vms. Isolation among PostgreSQL-as-a-Service instances is achieved using iptables-manager module which ensures that only instance VMs will be able to communicate among themselves. 

Futher isolation in MT-PostgreSQL-as-a-Service is realized by providing unique membership role for each tenant-MTU. Applications are provided with separate username/password via MT-PostgreSQL-as-a-Service-binding. Tenant users are only entitled to connect to their own MTU and are stripped off any additional privileges. Multiple MT-PostgreSQL-as-a-Service-instances multiplexed into same dedicated-PostgreSQL-as-a-Service-instance cannot perform any operation on objects owned by one-another.

### Tenant-Admission-Control
 
Sever admission-control measures like connection_limits, disk_size_limits are built-in to MT-PostgreSQL-as-a-Service. For e.g. write requests are blocked when disk_size crosses a configured threshold to prevent postgress process crashes. Connection_limits are enforced at MTU level to prevent connection hogging.

### Disaster-Recovery/Data-Protection-using-Backup-and-Restore

#### Tenant-Based-Backup-And-Restore :

MT-PostgreSQL-as-a-Service supports tenant based scheduled backup and restore. Tenant backups are archived on cloud storage. On AWS/GCP/AZURE snapshots are used to archive the backups. In case of Openstack, backup is taken by copying  and compressing data directory.

#### Tenant-based-PITR : 
MT-PostgreSQL-as-a-Service also supports Point-In-Time-Recovery (PITR) using WAL archiving. Base backups along with WAL files are archived on cloud storage. Tenant data up to past 15 minutes can be restored.


### Tenant-Based-Monitoring

MT-PostgreSQL-as-a-Service supports tenant level health monitoring. Monitoring agent running on instance reports health of every tenant. It reports metrics like bulk reads, bulk writes, connections used etc. It also reports various system specific metrics like CPU, Memory, Disk usage etc.

The monitoring agent collects this information and reports it to centralized monitoring server, which stores in a time-series-database. A monitoring-web-application shows metrics via various charts so that devops can identify the tenant health at any given time-date range.

![N|Solid](https://github.com/ankita0811/PostgresqlConf/blob/master/database_scans_rows.png?raw=true)


![N|Solid](https://github.com/ankita0811/PostgresqlConf/blob/master/database_buffers.png?raw=true)

Instance Health Metrics

![N|Solid](https://github.com/ankita0811/PostgresqlConf/blob/master/instancehealth.png?raw=true)
### Tenant-Based-Logging

MT-PostgreSQL-as-a-Service supports tenant based logging for audit and troubleshooting purposes. Tenant database logs are pushed to a centralized system, enabling dev ops to debug any issue irrespective of instance availability.

### Alerting-System
Alerting-module raise alerts when some undesired state is reported, like primary-server-not-available, replication-down, disk-size-threshold-crossed, backup-failed etc.

![N|Solid](https://github.com/ankita0811/PostgresqlConf/blob/master/avs.png?raw=true)
![N|Solid](https://github.com/ankita0811/PostgresqlConf/blob/master/alert.png?raw=true) 
