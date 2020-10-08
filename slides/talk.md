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
* Migrated Production to OCI (Q2/2020)
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

### Compartments and Tags

---

# Architecting PeopleSoft

### Compartments and Tags

* Single compartment for PeopleSoft
* Free-form tags for tier and app

???

Sub compartments were not a feature when we started out build out. While they have been added, and you can easily move resources to new compartments, we opted to leave it as-is for now. We use some tagging, but it's a collection of free form tags that are loosely enforced.

---

# Cloud Technologies

### Networking

* Virtual Cloud Networks
* Network Security Lists/Groups
* Gateways

---

# Architecting PeopleSoft

### Networking

* Single VCN for PeopleSoft
* Extend on-prem network to OCI via VPN
  * Use 10. /24 CIDR block
  * Site-to-site VPN
* Subnets for DMZ, Application, DR, Database
* Common Security Lists applied to subnets
  * Application, NFS, Database

???

Our deployment to OCI was designed to extend our on-prem network. This is common for many deployments but it can impose some restrictions on your network. We were given a /24 network to extend the private IP range from our internal network to OCI. We connected on-prem to OCI via a site-to-site VPN.

We built 5 subnets, app for non-prod, DMZ, DR, and 2 for the database (client and backup subnets). We built a number of security lists to try and standardize our port openings. 

Additionally, we used the NAT Gateways and Service Gateways to keep our network traffic private.

---

# Cloud Technologies

### High Availability

* Regions
* Availability Domains
* Fault Domains
* Load Balancing
* Instance Pools

---

# Architecting PeopleSoft

### High Availability

* Leverage Fault Domains
  * Non-regional Subnets
  * Exadata tied to an Availability Domain
* Use Data Guard/rsync for replication to ODA
* Running HA Proxy for Load Balancer
  * OCI Load Balancer Limitations

???

At the time we built our network, subnets were limited to one AD. This means that our networking limited us to working with FDs for HA. Still way better than our previous setup :)

We need to sync back to the ODA, so we use Active Data Guard for the database, and (reliable) rsync for the PeopleSoft servers. 

At the time of our build, we found the OCI load balancer to be missing some features and not up to our standards (slow health check). We are using HA Proxy for now.

---

# Cloud Technologies

### Storage

* Object Storage
* File Storage System
* Block Storage

---

# Architecting PeopleSoft

### Storage

* Object Storage for database backups
* File Storage Service for
  * Shared files
  * PS_APP_HOME, PS_CFG_HOME
  * Documentation
* Attached Block Storage on all instances
  * Backup Policies
  * Configurable Performance

???

Object Storage is great - it can store and serve large files very easily. Database backups are sent to object store.

The FSS is excellent. It's NFS v3 compatibale file storage that you can mount on your servers. Backups are super simple and incremental. We leverage this for storing our DPK files, other shared files, Documentation, and a few HOMEs. 

We attach block storage to all our instance, and all PS resources are installed there. We use a terraform module to create all of our instances, attach block storage, mount the drives to a consistent location, and associate a backup policy for boot and block drives.

---

# Cloud Technologies

### Infrastructure as Code

* Terraform
* Resource Manager

---

# Architecting PeopleSoft

### Infrastructure as Code

* Used Terraform to build most resources
* Exceptions:
  * Users and some IAM policies
  * Exadata/DBs
* Resource Manager only for Cloud Manager

???

Terraform is the foundation for IaC. We used TF to deploy the network, route tables, security lists, instances, Cloud Manager and more. A few exceptions - the Exadata (but that is supported in TF), and some of the inital Tenancy configuration.

Resource Manager was released after we did our build, so we haven't used it for much of the current setup. The exception was cloud manager; we leveraged the CM Marketplace stack for that.

---

# Cloud Technologies

### Management

* Monitoring and Notifications
* Management Cloud
* OS Management Service
* Cloud Shell and CLI

---

# Architecting PeopleSoft

### Management

* Instances added to OMC for server monitoring
* Adopting OSMS to replace scripts
* Using Instance Principals and CLI with Rundeck

???

In addition to the basic OCI monitoring, we licenced Oracle Management Cloud for an additional layer of monitoring and analytics. As part of our provisioning process, we install the OMC agent.

The OS Management Service was released and we are working on adopting it for OS patching. With moving the cloud, this is one area that we are responsible for that is often handled by IT.

---
class: center, middle, black

# Second Chances

???

It's not very often you get to re-do a large migration like this. SPPS is in a unique situation. They migrated their PeopleSoft 9.1 application to OCI and are upgrading to 9.2 now.

With this, we will be building out their 9.2 system and get to learn from our own lessons.

A number of this are in our favor:

* The timing is better; OCI has fixed a number of the limitations that stopped us from using a feature (regional subnets, LB shapes/hosts, NSGs)
* We are not dependent on IT anymore so we can build out our own network; less "over the wall" type issues when troubleshooting
* We can use Cloud Manager as it doesn't support 9.1

---

# Re-architecting PeopleSoft

### Compartments

* New PeopleSoft 9.2 Compartments
    * PeopleSoft9.2
        * Non-production 
        * Production
        * DR (?)
* Structured tags to support 
    * Dynamic Groups (OSMS)
    * Patching
    * Budgeting

???

A question I am still working through - Do we need a DR compartment?

The plan is build DR in another region. Do we need separate policies for DR when they really are production servers, just offline/standby in another region? I don't think so.

Use structured tags and namespaces. Using pre-defined tags and enforcing them will help us build reports and dynamic groups to cover our resources. 

---

# Re-architecting PeopleSoft

### Networking

* Two VCNs for PeopleSoft
  * Separate VCN for Database
* Build our own network space
  * Use 10. /16 CIDR block
* Regional Subnets for DMZ, Application, DR
* Limit Security Lists to common rules per subnets
  * Network Security Groups for fine-grained access

???

Moving to two VCNs is a function of running Running ExaCS than it is a stratetic design. Moving ExaCS between compartments is easy, but not VCNs. OCI has LPGs (local peering gateways) that let you connect two VCNs very easily. We will build a new network for 9.2 that support a bigger IP CIDR block, use regional subnets, and we will use NSGs for anything application specific. Security Lists will only be used for things that are common for a subnet (SSH, RDP, TNS for DB, etc).

---

# Re-architecting PeopleSoft

### High Availability

* Leverage Availability Domains and Fault Domains
  * Regional Subnets
  * Exadata tied to an Availability Domain
* Use Data Guard/FSS/OS/rsync for data replication to DR
* Use Instance Pools and OCI LB for HA and scalability

???

Regional subnets will make it easy to use ADs for HA. One thing that does hold us back a bit; the ExaCS is tied to an AD. As we build out our DR in the cloud (move away from an ODA on prem), we will look at other database options that will help with HA.

Instead of rsync as the primary replication tool for DR, we will look at FSS/OS to replicate data between servers. FSS and OS is HA in a region as a service, so we don't worry about that. But we still will use rsync (or similar tool) for syncing to other regions.

We are looking instantiating Instance Pools for every environment, even if they don't need more than a single server. You get a lot of automatic connection with Instance Pools - LB pool and attachments, easier to spin up new/patched systems, etc. (See Kyle's talk on the subnet).

---

# Re-architecting PeopleSoft

### Storage

* Expand File Storage Service for
  * PS_HOME/Middleware
  * Leverage Cloud Manager Repository
* Backup Policies
  * Specify custom backup policies to match environments

???

We love FSS, and it's been very reliable. I'm more open to running PS_HOME and the middleware on FSS for simplicity (CPU patching is as easy as mounting a new path). BUT, I still have my reservations; running the binaries locally eliminates a whole class of performance and stability issues by not depending on the network.

We will leverage the cloud manager repository for deploying new environments (tied in with `ioco` in a few slides).

For the most part, we are happy with the delivered Gold, Silver, Bronze backup policies. But for some environments we may look at a custom policy as that's something we couldn't do before.

---

# Re-architecting PeopleSoft

### Infrastructure as Clode

* Integration Terraform with Resource Manager
    * Gitlab integration
    * Drift Reports
* Use the psadmin.io Terraform module for instances
    * https://github.com/psadmin-io/oci-tf-ps-instance

???

We want to use Resource Manager more for two main reasons:
* You can import repos from GitLab! Each Stack can point back to repo and branch which makes managing RM stack easy.
* Drift Detection will compare what your TF code said to build with what exists in OCI. It will generate a report to tell where the configuraiton has changed. This is excellent for things like Security Lists and NSGs to see if undocumented or unnecessary holes exist.

For our 9.1 environment, I built a TF module to standardized our PS instances. We have rewritten that module and published it on GitHub for anyone to use. It will create an an instance the way we build them, set up the attached storage, backup policies, and more.

---

# Re-architecting PeopleSoft

### Management

* Push notifications to WebHooks/PagerDuty
* Add OMC log agent to 9.2 instances
* Expand use of Rundeck for OCI management
* Leverage Cloud Manager for PeopleSoft Images

???

With 9.2 we can use more of Oracle Management Cloud's features, like log analytics.

We want to use Rundeck for more things

I haven't talked much about CM yet. I think CM is great for installing new PIs and we plan to use it for that. We also have Phire installed in our CM environment (TDB on that being a good/bad thing). A reason we aren't looking to use it much more than that is it doesn't understand shared infrastructure well. We opted to use a pre-paid billing model, so we want to utilize our infrastructure but we don't want to overbuild. CM can lead to server sprawl. We also run HR/FS on the same server for each tier, something that CM can't do either. We'll keep evaluating CM and providing feedback to PS, but I'm not as

---
class: center, middle, white

# psadmin.io Cloud Operations

???

* Announcing psadmin.io Cloud Operations, or ioCloudOps
* We have been building our own preferred toolset and methodology for a modern PeopleSoft deployment
* New methodologies, using a Cloud first approach
* Leverage cloud APIs and tooling

---

# psadmin.io Cloud Operations

### Methodology

* Align with Agile, DevOps and IaC
* Leverage cloud tools and APIs
* Monitoring and Notifications
* Compartments, Tags, Budget
* [psadmin.io/oci](https://psadmin.io/oci)

???

* We spent a lot of time doing this
* Putting what we learned with DPK, OCI in one place
* NOT a one size fits all, but lays out examples of choices YOU can make
* If you are interested in working with us, checkout out website
    * OCI
    * Automated Deployments, Patching
    * Tools upgrades

---

# psadmin.io Cloud Operations

### Toolset

* Terraform and OCI Resource Manager
* Cloud Manager
* DPK, psadmin.io Puppet modules
* `psadmin-plus`
* `ioco`

---

# ioco
### psadmin.io Cloud Operations Utility

A python utility for common tasks related to administering PeopleSoft in the Cloud

[github.com/psadmin-io/ioco](https://github.com/psadmin-io/ioco)

---

# Examples

* `ioco oci block --make-file-system --mount`
* `ioco cm attach-dpk-files`
* `ioco dpk deploy --dpk-type=FSCM`

???

* Create a default partition and file system on new block storage, then mount it
* Attach the CM dpk files repo
* Get DPK files from CM dpk files repo, then deploy a midtier setup