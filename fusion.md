## Product overview

Organizations must quickly adjust to the changing business and outside influences that are causing rapid change, resulting in a need for business agility and faster business insights. Clients need applications and data to adjust and shift in response to dynamic market demands. They also need diverse and simplified tools and data services to build anywhere, at any pace, and for applications and data to scale dynamically, achieve peak performance, and adhere to security requirements.

Companies need a consistent way to deploy applications across on-premises infrastructure and public clouds, and not all of them are deploying containers to create that portability and consistency between cloud and on-premises environments.

IBM Storage Fusion is a container-native hybrid cloud data platform that offers simplified deployment and data management for Kubernetes applications on Red Hat® OpenShift® Container Platform. IBM Storage Fusion is designed to meet the storage requirements of modern, stateful Kubernetes applications and to make it easy to deploy and manage container-native applications and their data on Red Hat OpenShift Container Platform.

It is an advanced storage and backup solution that is designed to simplify data accessibility and availability across hybrid clouds. Companies can expand data availability across complex hybrid clouds for greater business performance and resilience. With the IBM Storage Fusion solutions, organizations manage only a single copy of data. They need not create duplicate data when moving application workloads across the enterprise, easing management functions when you streamline analytics and AI.

IBM Storage Fusion provides a streamlined way for organizations to discover, secure, protect, and manage data from the edge, to the core data center, to the public cloud.

IBM Storage Fusion is available as two deployments, namely IBM Storage Fusion and IBM Storage Fusion HCI System.

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
