# Balerica Inc. Cloud Migration Strategy to GCP

**Author:** Cameron Cleveland
**Date:** 8\29\25
**Project:** Individual Armageddon - Cloud Migration Strategy

## Project Overview

Balerica Inc., an educational services provider headquartered in Sao Paulo, sought to modernize its IT infrastructure, expand globally, and enhance security. This project outlines a comprehensive migration strategy from their existing on-premises setup to a secure, automated, and scalable environment on **Google Cloud Platform (GCP)**.

### Goals
- Automate deployment and management processes.
- Reduce or eliminate "click-ops" administration.
- Enable secure remote control capabilities for administrators.
- Drastically reduce desktop reimaging times.
- Develop a scalable, secure browser for certification tests.
- Establish a secure, redundant global network connecting testing centers in 5 countries.

## Current State Analysis

The following diagram illustrates Balerica's existing on-premises network topology, which is manual, fragile, and not built for scale.

![Balerica Current Network Topology](diagrams/Cloud-Migration-HW-Before.jpeg)

### Diagram Explanation & Key Issues:
*   **Flat Network:** All devices reside on a single network segment connected via **unmanaged TP-Link switches**, creating a major security risk where a breach on one device could compromise all assets.
*   **Manual Processes:** The workflow is characterized by **"click-ops"**—manual administration via CMD scripts, best-effort backups, and manual deployment of the secure exam browser.
*   **Lack of Central Management:** There is no central codebase or configuration management. **International testing centers** have no standardized, secure connection to the Sao Paulo HQ, relying on insecure, ad-hoc links.
*   **Desktop Management Overhead:** Reimaging the 30+ Lenovo M90 desktops is a slow, manual process using the factory Lenovo image.

## Future State Recommendation: GCP Architecture

The proposed future state leverages Google Cloud Platform to create a resilient, automated, and globally scalable infrastructure that addresses all of Balerica's expressed goals.

![Balerica Future State on GCP](diagrams/GCP-HW-After.jpeg)

### Diagram Explanation & Networking Choices:

This architecture is built on a **global hub-and-spoke model** using GCP's best-in-class services.

**1. Global Network & Security:**
*   **Shared VPC Host Project (balerica-net-hub):** This creates a centrally managed and secured network foundation in GCP, enforcing consistent firewall rules and policies across all projects.
*   **Hybrid Connectivity:** Each location connects securely to the nearest GCP region using **Cloud HA VPN** with BGP dynamic routing for automatic failover. The Sao Paulo HQ uses a **Partner Interconnect** for a high-performance, dedicated link. This meets the requirement for **secure and redundant communications**.
*   **Zero Trust Security:** **Identity-Aware Proxy (IAP)** is implemented for all remote administration. This ensures that **no VM is ever exposed to the public internet**, and access is granted based on user identity and context, not just network location. This fulfills the goal of "secure remote control capabilities for administrators only."

**2. Automation & Modernization:**
*   **GKE Autopilot:** The homegrown secure browser is **containerized** and runs on a fully managed Kubernetes service. This makes it lightweight, highly scalable, and automatically managed by GCP, directly rivaling commercial solutions like Pearson VUE.
*   **Infrastructure as Code (IaC):** The entire environment can be provisioned using tools like Terraform, eliminating manual "click-ops" and ensuring consistent, repeatable deployments.
*   **ChromeOS Flex:** The existing Lenovo desktops are repurposed with ChromeOS Flex, enrolled in the Google Admin console. This allows for **centralized policy management** and reduces reimaging from hours to minutes, addressing a key pain point.

**3. Resilience and Performance:**
*   **Global Load Balancer:** Users worldwide connect to a single, secure endpoint provided by the **Global HTTPS Load Balancer**, which automatically routes them to the closest healthy backend instance, minimizing latency.
*   **Managed Services:** Using **Cloud SQL** and **Filestore** ensures databases and file storage are highly available, scalable, and backed up automatically, moving away from the "best effort" policy.

## Task 2: Pain Point Analysis & Solutions

### **1. Pain Point: Single Points of Failure and Lack of Redundancy**

The most critical pain point is the environment's fragility due to multiple single points of failure, particularly the standalone database server and the lack of redundant network paths. The current configuration means a hardware failure, software crash, or localized power outage in the São Paulo data center would cause complete, prolonged downtime for all global testing centers. This directly threatens business continuity, leading to canceled exams, lost revenue, and significant damage to Balerica's reputation as a reliable testing provider. The best-effort backup strategy further exacerbates this risk, potentially leading to data loss.

Our solution is to architect for high availability on Google Cloud Platform. We will migrate the database to **Cloud SQL configured for high availability**, which automatically provisions a synchronous standby replica in a separate zone within the same region, enabling automatic failover in the event of an outage. For connectivity, each global testing center will establish **Cloud HA VPN tunnels with BGP routing** to their nearest cloud region, ensuring automatic failover between connections. This transformation eliminates critical single points of failure, reducing potential downtime from hours to seconds. The primary drawback is a predictable increase in operational costs due to the doubled database instance and egress traffic costs. However, this is a net positive; the business cost of downtime far outweighs these cloud expenditures, directly achieving the goal of secure and redundant communications and ensuring business continuity.

### **2. Pain Point: Manual, Inefficient Desktop Management ("Click-Ops")**

The second critical pain point is the operational inefficiency and inconsistency of desktop management. The current process of manually reimaging each Lenovo desktop with a factory image via CMD scripts is incredibly time-consuming for IT staff. This "click-ops" approach leads to long resolution times for testing centers, creates non-standardized environments that are difficult to secure and troubleshoot, and directly contributes to examiner and candidate frustration during technical issues, undermining the testing experience.

Our solution is to repurpose the existing hardware into centrally managed, stateless thin clients. By installing **ChromeOS Flex** on the Lenovo M90 desktops and enrolling them in the **Google Admin console**, we can define and push a standardized, kiosk-mode policy that locks the device to only run the secure browser. "Reimaging" is reduced from a manual, multi-hour process to a simple reboot that takes seconds, instantly restoring a pristine, compliant state. The potential drawbacks include a minor learning curve for IT staff to use the web-based admin console and the need to validate hardware compatibility with ChromeOS Flex. Despite this, the solution is a definitive net positive; it directly and completely accomplishes the goal of drastically reducing desktop reimaging times and occurrences while also significantly reducing manual "click-ops."

### **3. Pain Point: Insecure Administrative Access and Flat Network**

The third pressing pain point is the severe security vulnerability created by the flat network architecture and insecure administrative practices. The unmanaged switches allow unfettered lateral movement, meaning a compromise of one device could lead to a full network breach. Furthermore, current remote administration methods likely rely on insecure protocols like RDP with public-facing ports, creating a direct attack vector for threat actors targeting Balerica's critical infrastructure and sensitive exam data.

Our solution is to implement a defense-in-depth, Zero-Trust security model within GCP. We will replace the flat network with a **segmented VPC** architecture using stringent firewall rules to control traffic between web, application, and database layers. Most critically, we will eliminate all public management ports by leveraging **Identity-Aware Proxy (IAP)** for all remote administrative access to Compute Engine instances. IAP gates access behind IAM permissions and two-factor authentication, ensuring only authorized users can establish secure tunnels to resources, which are never exposed to the open internet. The primary drawback is the need for administrators to adopt a new workflow using the GCP console or command-line interface instead of traditional RDP/SSH clients. This is a clear net positive; it is the most secure method to achieve the goal of "remote control capabilities on desktops from administrators only" and fundamentally transforms the security posture from vulnerable to robust.

## Conclusion

This migration strategy transforms Balerica Inc. from a manually-intensive, geographically-limited operation into a fully automated, secure, and global organization capable of scaling effortlessly to meet its business goals. The choice of **Google Cloud Platform** services provides a modern, cost-effective, and future-proof foundation for this transformation. By directly addressing the three most critical pain points with targeted GCP solutions, we ensure the migration not only meets but exceeds Balerica's stated goals for security, efficiency, and reliability.
