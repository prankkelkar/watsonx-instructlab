## IBM Fusion HCI Product overview

Organizations must quickly adjust to the changing business and outside influences that are causing rapid change, resulting in a need for business agility and faster business insights. Clients need applications and data to adjust and shift in response to dynamic market demands. They also need diverse and simplified tools and data services to build anywhere, at any pace, and for applications and data to scale dynamically, achieve peak performance, and adhere to security requirements.

Companies need a consistent way to deploy applications across on-premises infrastructure and public clouds, and not all of them are deploying containers to create that portability and consistency between cloud and on-premises environments.

IBM Storage Fusion is a container-native hybrid cloud data platform that offers simplified deployment and data management for Kubernetes applications on Red Hat® OpenShift® Container Platform. IBM Storage Fusion is designed to meet the storage requirements of modern, stateful Kubernetes applications and to make it easy to deploy and manage container-native applications and their data on Red Hat OpenShift Container Platform.

It is an advanced storage and backup solution that is designed to simplify data accessibility and availability across hybrid clouds. Companies can expand data availability across complex hybrid clouds for greater business performance and resilience. With the IBM Storage Fusion solutions, organizations manage only a single copy of data. They need not create duplicate data when moving application workloads across the enterprise, easing management functions when you streamline analytics and AI.

IBM Storage Fusion provides a streamlined way for organizations to discover, secure, protect, and manage data from the edge, to the core data center, to the public cloud.

IBM Storage Fusion is available as two deployments, namely IBM Storage Fusion and IBM Storage Fusion HCI System. IBM Storage Fusion is now IBM Fusion and IBM Storage Fusion HCI is not IBM Fusion HCI.

## IBM Storage Fusion
It is a software-defined storage management software with protection, backup, and caching elements and can be run on existing hardware resources.
To know more about IBM Storage Fusion, see https://ibm.com/docs/en/storage-fusion-software/2.8.x.

## IBM Storage Fusion HCI System
It is a purpose-built, hyper-converged architecture that is designed to deploy bare metal Red HatOpenShift container management and deployment software alongside IBM Storage Fusion software. For consistent and rapid deployment and management, it features an appliance form-factor, hyper-converged infrastructure along with integrated software-defined storage to meet the storage requirements of modern, stateful Kubernetes applications. Built with a storage platform that includes the essential elements necessary for mission-critical containers and hybrid cloud, the IBM Storage Fusion provides a comprehensive infrastructure with compute, networking, and storage resources, including a data platform and global data services for Red Hat OpenShift.
Benefits of IBM Storage Fusion
Benefits offered by IBM Storage Fusion.

## IBM Storage Expert Care
IBM Storage Expert Care is a simplified method of selecting services and support for systems at the time of the purchase. IBM Storage Expert Care is designed to simplify and standardize the support approach for IBM Storage Fusion HCI Systemwith simple straightforward pricing and selection of services.

## Security in IBM Storage Fusion.
IBM Storage Fusion is a secure platform to deploy your applications.
The following security measures were followed for IBM Storage Fusion product:
* Penetration testing to check for exploitable vulnerabilities
* Threat modelling and threat assessment to ensure that threats are assessed and resolved
* Static scan to maintain high code quality and to ensure that no security vulnerabilities are left in the code
* Dynamic scan to detect and remove runtime vulnerabilities
* Open Source Scan to detect and remove open source vulnerabilities in open source libraries that are used by IBM Storage Fusion
* Security and Privacy by Design (SPbD) compliance certified by IBM BISO to ensure security and privacy compliance
* Container Software certifications (IBM Certification and Red Hat Certifications) to ensure adherence to container security standards.

### User Interface


The Log Collector panel shows the 6 tiles which refers to 6 collection sets predefined in above configmap `isf-serviceability-operator-collection-sets` and allow the user to choose the collection sets desired.
1. Backup and restore
2. Compute Nodes
3. Network switches
4. Storage controller
5. System Health check
6. Administration

Each collectionset already collects relevant k8 resources and namespaces. 
For e.g. storage includes scale must gather logs and storage related logs, backup restore collects spp agent server logs and other spp related logs, system health collectset include imm logs, appliance hardware related logs, Network switches covers nw switches related to logs and Administration contains audit logs and OpenShift configuration related logs
Please refer collectionset configmap for latest details.

After log collection is complete you will see one of following three statuses for each request in the **UI**.
* `Complete` : log collection was completed without any erros and is ready to download.
* `Partial` : The log collection was complete however there were some errors while collecting few items from the list of requested logs. Download the logs and check for `status.json` file  in the zip to understand more about errors. 
*(Note: Logcollector is designed in such a way that it does not stop log collection upon encountering an error. It will instead log the error and proceed with collecting rest of the requested items.)*
* `Error`: The log collection failed or timed out. Please try again.

### Eventmanager severity

Event Manager will look for a `severity` field in the `labels` map. Event Manager processing will be based on the value in this field:
* INFO - A Kubernetes event with type=Normal will be created from this alert. Alerts of this type should convey general expected occurences. For example, "A physical volume was allocated for XXX at time YYY." By default, this event will be in the OCP event queue up to 3 hours after its last duplicate was encountered.
* WARNING - A Kubernetes event with type=Warning will be created from this alert. Alerts of this type should convey suspected conditions that will not open a ticket but should be investigated by the user. For example, "Available disk space on physical volume XXX has dropped below 20%." By default, this event will be in the OCP event queue up to 7 days after its last duplicate was encountered.
* CRITICAL - A Kubernetes event with type=Warning will be created from this alert, and a ticket will be opened if this alert is in the allow-tickets config map (see below). Alerts of this type should convey definite problems that need immediate resolution. By default, this event will be in the OCP event queue up to 14 days after its last duplicate was encountered.


## Verifying image signatures
Digital signatures provide a way for consumers of content to ensure that what they download is both authentic (it originated from the expected source) and has integrity (it is what we expect it to be). All images for IBM Storage Fusion are signed. This page describes how to verify the signatures on those images.

## Planning and prerequisites for IBM Fusion HCI deployment
There are prerequisites, preparations and planning required before you can deploy IBM Fusio HCI in data centre. Here is list of planning items
### Hardware overview of a single rack
Review the overall system layout and hardware configuration. Ensure that IBM Storage Fusion is installed in a restricted access location, such that the area is accessible only to skilled and instructed persons with proper authorization.
#### Single rack (9155-C00, 9155-C01, 9155-C04, 9155-C05, 9155-G02, and 9155-F01)
In this appliance, you cannot configure your racks with any servers in positions 29 to 32.
##### Rack (Model R42)
42U rack with 2 to 6 PDUs, cabling, and components.
##### Compute-only server (Model C00, C04) / Compute-storage server (Model C01, C05, C10, C14)
The available compute-only and compute-storage server model servers are 9155-C00, 9155-C01, 9155-C04, 9155-C05, 9155-C10, and 9155-C14. The 9155-C00 and 9155-C04 are compute-only servers. The 9155-C01, 9155-C05, 9155-C10 (32 core node), and 9155-C14 (64 core node) are compute-storage servers.

You can have either six model C10 nodes with storage or model C14 nodes with storage as a minimum configuration. Each of these servers have a minimum of two storage drives that can be increased up to a maximum of ten storage drives on each server.

It is possible to expand IBM Storage Fusion HCI System beyond the minimum six servers with combinations of models C00, C01, C04, C05, C10, and C14.
Important: In the rack model that supports 9155-C00, 9155-C01, 9155-C04, 9155-C05, you can include compute-only (C10) servers on need basis. To include C10 server (compute-only), contact IBM Support for upgrade steps.
In the rack model that supports 9155-C10 and 9155-C14, you cannot add 9155-C00, 9155-C01, 9155-C04, and 9155-C05.

Servers can be added to a maximum of 16 for both racks. If a client chooses to have only one GPU server, then they can have 15 compute-storage servers. Each of the servers added to IBM Storage Fusion HCI System is also added to the storage cluster, increasing the total storage capacity. The servers are also combined into a Red Hat® OpenShift® Container Platform. All applications that run on IBM Storage Fusion HCI System are deployed and run within the OpenShift cluster.

###### Hardware configuration for 9155-C10 (32 core compute-only node)
1. Lenovo SR630 V3
2. 2x 16-core Intel Gold 6426Y 2.5GHz 185W processors (“Sapphire Rapids”)
3. 256GB RAM (16x 16GB DDR5 DIMMs), expandable to 512GB RAM
4. 0-10x 7.68TB or 3.84TB PCIe generation 5 NVMe drives
5. 1x NVIDIA ConnectX-6 DX dual-port 100GbE PCIe NIC
6. 1x NVIDIA ConnectX-6 LX dual-port 25GbE PCIe NIC
7. 1x Intel I350 1GbE RJ45 4-port OCP Ethernet Adapter
8. 2x 960GB M.2 NVMe OS drives (RAID 1)
###### Hardware configuration for 9155-C14 (64 core compute-storage node) 
1. Lenovo SR630 V3
2. 2x 32-core Intel Gold 6438N 2.0GHz 205W processors (“Sapphire Rapids”)
3. 1024GB RAM (16x 64GB DDR5 DIMMs) expandable to 2048GB RAM
4. 0-10x 7.68TB or 3.84TB PCIe generation 5 NVMe drives
5. 1x NVIDIA ConnectX-6 DX dual-port 100GbE PCIe NIC
6. 1x NVIDIA ConnectX-6 LX dual-port 25GbE PCIe NIC
7. 1x Intel I350 1GbE RJ45 4-port OCP Ethernet Adapter
8. 2x 960GB M.2 NVMe OS drives (RAID 1)
###### Hardware configuration for 9155-C00 (compute-only nodes)
1. Lenovo SR645 server
2. 2x AMD EPYC 7302 16 Cores (32 Cores total), 32 Threads (64 Threads total), 3.0 GHz or 3.3 GHz CPU
3. 256 GB RAM (16x 16 GB DIMMs)
4. 2x 960 GB M.2 OS drives (RAID 1)
5. 1x NVIDIA ConnectX-6 dual-port 100 GbE network adapter
6. 1x NVIDIA ConnectX-4 dual-port 25 GbE network adapter
7. 1x 1 GbE RJ45 4-port OpenShift adapter
8. Same specifications as the compute-storage server but with zero NVMe disks
10. 1U height
###### Hardware configuration for 9155-C04 (compute-only nodes) 
1. Lenovo SR645 higher density
2. 2x AMD EPYC 7543 32C (64C total) 256MB L3, 2.8 GHz / 3.7 GHz, CPU 225W
3. Base RAM: 16x 64GB DIMMs (16 GB/core, 1024GB total)
4. Upgraded RAM: 32x 64GB DIMMs (32 GB/core, 2048GB total)
5. 2x 480GB M.2 OS drives (RAID 1)
6. 1x NVIDIA ConnectX-6 dual-port 100GbE network adapter
7. 1x NVIDIA ConnectX-4 dual-port 25GbE network adapter
8. 1x 1GbE RJ45 4-port OCP adapter
9. Zero NVMe drives
10. 1U height
###### Hardware configuration for 9155-C01 (compute-storage node)
1. Lenovo SR645 server
2. 2x AMD EPYC 7302 16 Cores (32 Cores total), 32 Threads (64 Threads total), 3.0 GHz or 3.3 GHz CPU
3. 2x960 GB M.2 OS drives (RAID 1)
4. 1x NVIDIA ConnectX-6 dual-port 100 GbE network adapter
5. 1x NVIDIA ConnectX-4 dual-port 25 GbE network adapter
6. 1x 1 GbE RJ45 4-port OpenShift Container Platform adapter
7. 2-10x Samsung PM1733 7.68 TB NVMe PCIe 4.0 disks
8. NVMe disks are added in pairs
9. All compute-storage servers must have the same number of NVMe drives
10. The maximum number of compute-storage servers is 20, reduced by the number of GPU servers installed
11. 1U height
###### Hardware configuration for 9155-C05 (compute-storage node) 
1. Lenovo SR645 higher density
2. 2x AMD EPYC 7543 32C (64C total) 256MB L3, 2.8 GHz / 3.7 GHz, CPU 225W
3. Base RAM: 16x 64GB DIMMs (16 GB/core, 1024GB total)
4. Upgraded RAM: 32x 64GB DIMMs (32 GB/core, 2048GB total)
5. 2x 480GB M.2 OS drives (RAID 1)
6. 1x NVIDIA ConnectX-6 dual-port 100GbE network adapter
7. 1x NVIDIA ConnectX-4 dual-port 25GbE network adapter
8. 1x 1GbE RJ45 4-port OCP adapter
9. 2-10x Samsung PM1733 7.68TB NVMe PCIe 4.0 drives
10. 1U height
#### Network layout and configuration
IBM Storage Fusion HCI System has two physical networks defined within it: a high-speed network for use by the storage cluster and applications, and a management network that is used for controlling the servers and monitoring the health of the servers.

##### 100GbE High Speed Network Switch (Model S01)
The high-speed network is built around a pair of 32-port, 100Gb Ethernet switches. The switches are configured together using MLAG to create a redundant pair. All of the compute-storage servers and the GPU servers have a 2-port, 100Gb Ethernet adapter. One port on the adapter is connected to the first high-speed switch and the second port is connected to the second high-speed switch. This 100GbE connections are reserved for use by the storage cluster. All of the compute-storage servers and the GPU servers also have a 2-port, 25Gb Ethernet adapter. Using breakout cables that split the 100GbE ports on the switch into four 25GbE ports, one port on the server’s 25 GbE network adapter is connected to the first high-speed switch and the second port is connected to the second high-speed switch. The AFM servers do not have a 2-port, 100 GbE network adapter. Instead, these servers have two of the 2-port 25GbE network adapters. Again, using breakout cables, each of the adapters has one port that is connected to the first high-speed switch and the other port that is connected to the other high-speed switch. The 25GbE network that is connected are intended for use by the Red Hat® OpenShift® cluster and the applications that are deployed within that cluster.
1. RU20 to RU21
##### 1GbE Management Network Switch (Model S02)
The management network is built around a pair of 48-port, 1Gb Ethernet switches. The IMM port of every IBM Storage Fusion HCI System server is connected to the first of the management switches using CAT5e cables with RJ45 connectors. The alternate IMM port, on either the LOM or an OCP adapter, is configured for all IBM Storage Fusion HCI System servers and it is connected to the second management switch. These connections are also made using CAT5e cables with RJ45 connectors. This is all done so that there is redundancy to support management functions even if one of the management switches fail or one of the cables become disconnected. Each management port from the high speed switches (S01s) as well as each of the 6 rack PDUs are connected to the management network switches.
1. RU18 to RU19

### Multiple racks
The multiple rack topology offers two variants: high-availability multi-rack and expansion rack.
### Power consumption by configuration
Before you up size the nodes, consider the power consumption data that is depicted in the following Power consumption tables.
### Site-readiness
The selection of a site for information technology equipment is the first consideration in planning and preparing for the installation.
### Resource utilization
The resource utilization and software distribution of IBM Storage Fusion and its services.
### Supported PDU power cords
Find out which power distribution unit (PDU) power cords are supported for your system.
### Power consumption specification for client-provided PDUs
The following power consumption information for IBM Storage Fusion HCI System components is provided as a planning aid for you to connect these components to PDUs that you provide.
### Network planning
As you plan for installing systems in your data center, review information about the network resources and your configuration options. Your network administrator is the intended audience for this network planning. The initial setup of IBM Storage Fusion HCI System involves connecting the appliance to your data center’s network. The appliance includes two high-speed switches that are connected to your core network through one port channel. This connection acts as the gateway between the IBM Storage Fusion appliance and your network. It enables administration of the appliance and OpenShift®, and is also used for network traffic in and out of the cluster. Your network team must prepare your network before the installation of the IBM Storage Fusion appliance. An IBM Systems Service Representative (SSR) does the initial configuration of the appliance, and as a final step, they connect the appliance to your pre-configured network by using the information you provided. As such, it is important that you complete the network configuration before SSR visit.

Complete the planning worksheets to provide IBM Service Support Representative (SSR) with the IBM Storage Fusion HCI System network setup plan.
To download the worksheets, see IBM Storage Fusion HCI Installation worksheets. When you fill the worksheet, check with your network team about whether the CIDR ranges that you plan to use are free on your network. The network planning involves three key steps:

#### DNS and DHCP.
Domain Name System (DNS) and Dynamic Host Configuration Protocol (DHCP) must be configured so that each node in the appliance has a hostname and IP address. Each node comes with a pre-configured MAC address, and also a MAC address for the bootstrap VM that provides a temporary control plan to orchestrate the installation of OpenShift. IBM provides a list of all MAC addresses so that your server or network teams can configure DNS and DHCP entries for each node. Create a DHCP entry for each MAC address, and then create forward and reverse DNS entries for each subsequent IP address. For a full list of steps about setting your DHCP and DNS, see Setting up the DNS and DHCP for IBM Storage Fusion appliance. Ensure that your DHCP server can provide infinite leases. Your DHCP server must provide a DHCP expiration time of 4294967295 seconds to properly set an infinite lease as specified by rfc2131. If a lesser value is returned for the DHCP infinite lease time, the node reports an error and a permanent IP is not set for the node. In RHEL 8, dhcpd does not provide infinite leases. If you want to use the provisioner node to serve dynamic IP addresses with infinite lease times, use dnsmasq rather than dhcpd.
##### Configure DNS and DHCP
#### Configure NTP server
Ensure that the network administrator configured an NTP server that is connected to your network. IBM Storage Fusion HCI System requires a connection to an NTP server so that time can be coordinated across the nodes in the OpenShift cluster. Ensure that the NTP server is accessible on the network where the IBM Storage Fusion HCI System is connected. Provide the IP address of the NTP server in the planning worksheet. It is needed by the IBM SSR to complete the initial configuration of the appliance.

#### Planning the connection between the IBM Storage Fusion HCI appliance and your network switches.
The IBM Storage Fusion HCI System appliance contains two high-speed switches that are used to connect to your data center network. The storage and compute nodes that make up the appliance are connected to the data center network via the high-speed switches in the appliance. The nodes are not connected directly to the data center switches. As such, the connection between the appliance and the data center network is a switch-to-switch connection, not a node-to-switch connection. The appliance switches must be treated as leaf switches within your network, and so it is recommended to connect them to core switches.

The IBM Storage Fusion HCI System switches should be connected to two data center switches for redundancy. Those switches must be configured to look like one logical switch via VPC (or equivalent stacking technology). The stacking must be used to allow redundancy, upgradability, and higher bandwidth. The IBM Storage Fusion HCI System high speed switches use MLAG technology to stack the switches, meaning they appear as a single logical switch to your network.
It is recommended to use an LACP topology to connect the IBM Storage Fusion HCI System switches to the data center switches.
Note: LACP is the preferred choice. Support is not available for non-LACP.
Link Aggregation Control Protocol (LACP) is an IEEE standard that is defined in IEEE 802.3ad to dynamically build an Etherchannel. For IBM Storage Fusion HCI System, the recommended LACP settings are as follows:
1. Rate setting is Fast
2. Mode is Active-Active setting

#### LACP topology
Two ports (31 and 32) on each of the IBM Storage Fusion HCI System high-speed switches are used to connect to the data center switches. Link aggregation is used to group multiple links into a single port channel with a bandwidth of all of the individual links combined. This configuration also provides redundancy and high availability, meaning that if any of the links or switches fail, traffic is automatically balanced between the remaining links.

It is recommended to use both ports 31 and 32 on each high-speed switch to connect to the data center switches as this results in a total of four links. The four links are aggregated into a single port channel with four times the bandwidth. The recommendation is to have two switches and four ports. If you want one switch, then combine Data center switch 1 and Data center switch 2 with four ports.

#### Planning for Hosted Control Plane
In the same CIDR range (number of clusters), you must have a set of free available IPs (no DHCP or DNS is required). It is based on the number of Hosted Control Plane clusters that you plan to create in the rack.

#### Planning and prerequisites for remote mount support
1. The storage VLAN (default 3201) must be available and configured on the customer network. If the default storage VLAN 3201 is not available on the customer network, contact IBM Support to change the default VLAN on IBM Storage Fusion HCI System.
2. The default gateway from the storage VLAN must be configured and reachable from IBM Storage Fusion HCI System.
3. The routing must be in place on the customer network from the storage gateway to the external IBM Elastic Spectrum System
4. The IBM Elastic Spectrum System must also be configured to support MTU 9000 along with other devices en route router, switches. The default MTU of IBM Storage Fusion HCI System is 9000.
#### Provide network configuration for IBM SSR setup
After the IBM Storage Fusion HCI System appliance is shipped to your data center, an IBM Systems Service Representative (SSR) visits on site to setup the appliance. As part of this process, the SSR configures IBM Storage Fusion HCI System to connect to your internal network. To complete this task, the SSR need information about your network.

Fill out the information for IBM SSR in the following Planning Worksheet, and then share the worksheet with your IBM representative:

https://www.ibm.com/support/pages/node/6491629.

#### Firewall requirements for IBM Storage Fusion HCI System
IBM Storage Fusion HCI System requires outbound access to external sites for accessing image registries and sending telemetry data to IBM and Red Hat®. This outbound access can be optionally routed through a proxy server. Configure your data center's firewall rules or proxy access control list (ACL) to meet the requirements for IBM Storage Fusion HCI System.
#### Proxy configuration
Proxy configuration is needed only in a connected install through proxy.

### Metro Disaster recovery deployment in IBM Fusion HCI
Metro-DR (Disaster Recovery) provides two-way synchronous data replication between IBM Storage Fusion HCI System clusters installed at two sites. When site disaster occurs, applications can be failed over to the second site. Because replication is synchronous, the Metro-DR solution is only available for metropolitan distance data centers with 40-millisecond latency or less.

You can deploy IBM Storage Fusion HCI System in data center site 1 and data center site 2, resulting in Red Hat® OpenShift® Container Platform clusters that run at each site. Two clusters are configured such that they share a synchronously replicated storage layer, allowing for data mirroring with zero Recovery Point Objective (RPO) between the two clusters. Both the Red Hat OpenShift clusters can host active workloads at the same time rather than keeping one cluster to be a cold standby. The IBM Storage Fusion HCI System storage network is stretched between both Red Hat OpenShift clusters so that data written to one cluster is automatically mirrored to the second cluster. A special tier breaker node is hosted at a third site and is used to determine which cluster is in charge of the data when the network between the two clusters is severed. Configuring a Metro-DR topology requires several network connections to be made between the two clusters and the tie breaker.

Metro-DR, the terminology scale admin network refers to the OpenShift Container Platform pod network, and scale daemon network refers to the IBM Storage Fusion HCI System storage network. The Metro-DR setup includes two sets of networks, namely internal storage network and Red Hat OpenShift network. In Metro-DR, the internal storage network of one site must connect to the other site and tiebreaker, that is, externalize your storage network. Similarly, the Red Hat OpenShift network of one site needs to connect to the other site.
#### OpenShift management network
The Red Hat OpenShift management network provides communication between the two OpenShift clusters, and is used by IBM Storage Fusion HCI System to coordinate replication activities, such as determining which applications are mirrored or triggering application failover. Every node in both clusters has an IP address in this network. The Red Hat OpenShift network across the sites must not be connected over Network address translation (NAT). You can decide to have the Red Hat OpenShift network across the sites in the same or different subnet.

#### IBM Storage Fusion storage network
The IBM Storage Fusion HCI System storage network is used for storage communication between nodes. In a Metro-DR configuration, the storage network must be connected between the two clusters to allow data replication. The storage network must also be connected to a tie breaker and the tie-breaker needs connectivity only on the storage network. This allows the tie breaker to choose which cluster is in charge of the data when the two clusters cannot talk to each other. The IBM Storage Fusion HCI System Storage network uses the 100G interface of each node and requires a high-bandwidth connection across the sites. The IBM Storage Fusion HCI System Storage network between the sites must not be connected through Network Address Translation (NAT). The IBM Storage Fusion HCI System storage network for each site must be in a different subnet than the other site and the tiebreaker.

#### General Metro-DR prerequisites
NOte: Metro-DR solution is only supported on Global Data Platform storage.
##### Setting up the tiebreaker
Install IBM Storage Fusion HCI System in two separate data centers within a metropolitan distance from each other. A third site must be identified to host the tiebreaker, and a VM is needed to host the tie breaker. For the actual procedure to setup, see Preparing the tiebreaker.
##### Setting up a network
1. Network connectivity must be available between all three sites for both the Red Hat® OpenShift network and the IBM Storage Fusion HCI System storage network. The IBM Storage Fusion HCI System storage network is responsible for the synchronous data replication between both sites, and a high-bandwidth L3 connection must be used that has less than 40 milliseconds of latency and within 150 km. It is recommended that all switches used to connect the two clusters support jumbo frames as it improves performance.
2. The IBM Storage Scale core pod that runs on every IBM Storage Fusion HCI System node at both sites is assigned an IP address on the storage network. As the number of nodes in a cluster can grow over time, a default IP address range is automatically generated for each cluster. This default IP address range is displayed during the installation GUI for each cluster. If needed, you can provide a different range of IP addresses  
4. Jumbo frames:
   4.5 Configure all the network devices through which the traffic flows from one site to another with Maximum Transmission Unit (MTU) size 9216 or higher. If your network does not support jumbo frames all the way from site 1 to site 2, then do the following steps:
      4.5.1 Do not enable Jumbo frames during the installation of site 2.
      4.5.2 If you configured site 1 with jumbo frames, reconfigure site 1 to disable it.
   4.6 For optimal performance, IBM recommends that you enable the jumbo frames.
   4.7 If you enable jumbo frames on the site 1, then you must enable it on the site 2 and on all network devices in between.
   4.8 Both the client and IBM Storage Fusion HCI System switches must have matching packet sizes.

##### Setting up communication between the clusters
The Metro-DR solution involves network traffic between two IBM Storage Fusion HCI System clusters and the tie breaker. Update the firewall for the three sites to allow traffic between each of the three endpoints in the solution.
1. Site one and site two must allow communication between the IP addresses of the two IBM Storage Fusion HCI System clusters on both Red Hat OpenShift network and the IBM Storage Fusion HCI System storage network. On the Red Hat OpenShift, the network communications across sites take place over UDP port 4500. On the storage network, communication across sites takes place over TCP ports 1191 and 12345.
2. Site one and the tie breaker site must allow communication between the IP address on the storage network of site one cluster and the IP address of the tie breaker virtual machine. This communication takes place over 1191 and 12345.
3. Site two and the tie breaker site must allow communication between the IP address on the storage network of the site two clusters and the IP address of the tie breaker virtual machine. This communication takes place over 1191 and 12345.
4. For Metro-DR allowed ports, see Firewall requirements for IBM Storage Fusion HCI System.

##### Storage network
1. A default IP range is provided out of the box for the storage network.
2. If needed, you can provide a different range of IP addresses for the storage network; however, it must be different across sites.
3. Storage VLANs and storage networks are customizable for a single rack and must use only a private network for the storage network.

##### Admin network
Each cluster has a pod network that needs to communicate with each other in a Metro-DR setup. To achieve this communication, make sure that the pod network is different across the sites. By default, different values are available for site 1 and site 2 during installation, but you can override them.

##### Preparing the tiebreaker
Visit https://www.ibm.com/docs/en/sfhs/2.8.x?topic=recovery-preparing-tiebreaker

### Enterprise registry for IBM Storage Fusion HCI System installation
During the installation process, IBM Storage Fusion HCI System installs Red Hat® OpenShift® Container Platform and IBM Storage Fusion software using images that are hosted in the Red Hat and IBM entitled registries. If you want to use your enterprise registry, you can install both OpenShift and IBM Storage Fusion HCI System software from images that are maintained in a container registry that you manage.
### Backup and restore planning
Plan resource requirements for backups.
### Activating Red Hat OpenShift Container Platform subscriptions purchased from IBM
If you purchased your Red Hat OpenShift Container Platform subscription from IBM, then you must activate your OpenShift subscription with Red Hat and activate OpenShift support with IBM.
### Activating IBM Storage Fusion HCI System Software to be downloaded
You need IBM Storage Fusion HCI System software entitlement to access IBM Storage Fusion HCI System images that are used to install your IBM Storage Fusion HCI System appliance.
### Installation prerequisites
Go through these general prerequisites and ensure that they are all met before you go ahead with your installation.

## Troubleshooting IBM Fusion HCI problems

This section lists known problems and solutions for Fusion HCI.

### IBM Fusion HCI node becomes NotReady
This can happen due to couple of reasons. When there are CSRs waiting to be approved or node connection is lost or core os is hung.
#### Recommended actions
To resolve problem check if there are pending CSRs. If found any then approve those. If no pending CSRs are found then check if node is pinging on IPv4 and IPv6. If either IP is not pinging then reboot node from IMM. 

### Nodes is added with localhost or local.domain to Red Hat OpenShift Container Platform cluster
This can happen when node does not receive valid hostname from DHCP server.
#### Recommended actions
To resolve this problem, verify your DHCP has correct mac address to IP/host mapping. If DHCP is correct, then set hostname on node manually using command `sudo hostnamectl set-hostname hostname.subdomain` and restart kubelet service using command `sudo systemctl restart kubelet`

### While upgrading Global data platform, received an error event with description "Pidslimit on the nodes is less than 12228."
The node Pidslimits is set to a value less than 12228, which is the minimum required value for the Global Data Platform to run on IBM Storage Fusion HCI System. It results in a non-functional storage cluster that impacts the applications that require storage.
#### Recommended actions
Follow steps [1](https://www.ibm.com/docs/en/error?originalUrl=2.6/sf_ocp_upgrade.html#sf_ocp_upgrade__step_rlb_25q_2bc) to [4](https://www.ibm.com/docs/en/error?originalUrl=2.6/sf_ocp_upgrade.html#sf_ocp_upgrade__step_t4p_25q_2bc) to apply kubeletconfig. It ensures the pidslimits are as expected on nodes. After the Pidslimit is set correctly, the upgrade precheck detects the fix and resumes upgrade automatically.

### Received an event/error that DNS resolution for image registry failed.
It indicates that the hostname resolution specified for the image registry failed for one or more cluster nodes. It blocks the upgrade because of image pull failure. It can also cause other issues outside of the upgrade because the newly scheduled or rescheduled pods cannot access the images. The registries can have a non-empty auth specified in the pull-secret of the openshift-config namespace. Check the connectivity for successful image pulls of IBM Storage Fusion components and the OpenShift® Container Platform. Failure to address the registry access can result in an ImagePullbackOff error of the pod.
#### Recommended actions
Check the DNS settings for the image registry. A firewall can prevent the access to the registry. After the DNS resolution issue is resolved, the upgrade precheck detects the fix and resumes upgrade automatically.

