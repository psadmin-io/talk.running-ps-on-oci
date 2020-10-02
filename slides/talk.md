class: center, middle, black

# Pre-Cloud and Migration

---
# On Premise Environment

* Running on 28 servers
* Out of warranty
* Expensive to replace servers

???

When we got involved, the servers were extremely outdated. The replacement for those servers had been pushed back due to cost and effort to replace the physical boxes. 
---

# Why Move to Hybrid System

* Oracle provided a study on a Hybrid system (2016)
--
* ODA on site for production
  * 28-core system to 16-core system (ODA)
??? 
ODA, or Oracle Database Appliance, is an engineered system (like Exadata and Exalogic), but is two node Oracle Linux RAC cluster in a single box with shared storage and networking all included in the box. It's smaller than an Exadata (size and price), but can also run application VMs.
--
* OCI-Classic for non-prod and DR
  * Migrated to OCI-Classic (Q1/2017)
???
At the time, OPC was Oracle's main cloud. Shortly after we migrated to OPC (then renamed OCI-C), OCI was launched.
---

# Moving to the Cloud

* Migrated non-prod and DR to OCI (Q1/2019)
???
OCI-C solved a number of problems for SPPS, but it wasn't "great". OCI was mature, the clear direction for Oracle's cloud offering, and had many more features for customers. Oracle agreed to let us migrate from OCI-C to OCI.
--
* Exadata as a Service for Database
???
With the migration to OCI, we upgraded from DB Systems to ExaCS. The plan was to eventually put production on OCI and get the full performance benefits of an Exadata.
--
* Migrated Production to OCI (Q1/2020)
  * Use ODA for DR
???
After a number of DR tests, and a week of running production in OCI, we made the switch. To users, it was seamless.

In general, people were okay moving production to OCI. But for the few that had concerns, we kept the ODA (still under support) and used that as the DR system. If there were cloud issues, we can fall back to the ODA. Outside of a DR test, we haven't touched the ODA.

---

# OCI Benefits

* Cost savings!
--
* Utilize new technology
--
* Stay on top of hardware upgrades
--
* World-class data center and security
--
* Flexibility in sizing for future projects (9.2 Upgrade)

---

class: center, middle, black

# Life in the Cloud

---

# Cloud Technologies

Networking

* Virtual Cloud Networks
* Network Security Lists/Groups
* Gateways

---

# Cloud Technologies

High Availability

* Regions
* Availability Domains
* Fault Domains
* Load Balancing
* Instance Pools

---

# Cloud Technologies

Storage

* Object Storage
* File Storage System
* Block Storage

---

# Architecting PeopleSoft

Networking

* Single VCN for PeopleSoft
* Extend on-prem network to OCI via VPN
  * Use 10. /24 CIDR block
* Subnets for DMZ, Application, DR, Database
* Common Security Lists applied to subnets
  * Application, NFS, Database
* Use Service Gateways for private access to OS
* Use NAT Gateways for all non-DMZ traffic

???



---

# Architecting PeopleSoft

Networking

<graphic from OKIT here>

# Architecting PeopleSoft

High Availability

* Leverage Fault Domains
  * Non-regional Subnets
  * Exadata tied to an Availability Domain
* Use Data Guard/rsync for replication to ODA
* Running HA Proxy for Load Balancer
  * OCI Load Balancer Limitations

???

---

# Architecting PeopleSoft

Storage

* Object Storage for database backups
* File Storage Service for
  * Shared files
  * PS_APP_HOME, PS_CFG_HOME
  * Documentation

???




---
class: center, middle, title
# Title
---
class: center, middle, black
# Black
---
class: center, middle, white
# White
---
class: center, middle, gray
# Gray