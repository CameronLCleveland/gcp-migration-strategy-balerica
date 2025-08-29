# Balerica Inc. Cloud Migration Strategy

**Author:** [Your Name]
**Date:** [Date]
**Project:** Individual Armageddon - Cloud Migration Strategy

## Project Overview

Balerica Inc., an educational services provider headquartered in Sao Paulo, sought to modernize its IT infrastructure, expand globally, and enhance security. This project outlines a comprehensive migration strategy from their existing on-premises setup to a secure, automated, and scalable environment on Google Cloud Platform (GCP).

### Goals
- Automate deployment and management processes.
- Reduce or eliminate "click-ops" administration.
- Enable secure remote control capabilities for administrators.
- Drastically reduce desktop reimaging times.
- Develop a scalable, secure browser for certification tests.
- Establish a secure, redundant global network connecting testing centers in 5 countries.

## Current State Analysis

The following diagram illustrates Balerica's existing on-premises network topology, which is manual, fragile, and not built for scale.

![Balerica Current Network Topology](diagrams/balerica-current-topology.png)

### Diagram Explanation & Key Issues:
*   **Flat Network:** All devices reside on a single network segment connected via **unmanaged TP-Link switches**, creating a major security risk where a breach on one device could compromise all assets.
*   **Manual Processes:** The workflow is characterized by **"click-ops"**—manual administration via CMD scripts, best-effort backups, and manual deployment of the secure exam browser.
*   **Lack of Central Management:** There is no central codebase or configuration management. **International testing centers** have no standardized, secure connection to the Sao Paulo HQ, relying on insecure, ad-hoc links.
*   **Desktop Management Overhead:** Reimaging the 30+ Lenovo M90 desktops is a slow, manual process using the factory Lenovo image.

## Future State Recommendation: GCP Architecture

The proposed future state leverages Google Cloud Platform to create a resilient, automated, and globally scalable infrastructure that addresses all of Balerica's expressed goals.

![Balerica Future State on GCP](diagrams/balerica-future-topology-gcp.png)

### Diagram Explanation & Networking Choices:

This architecture is built on a **global hub-and-spoke model** using GCP's best-in-class services.

**1. Global Network & Security:**
*   **Shared VPC Host Project (`balerica-net-hub`):** This creates a centrally managed and secured network foundation in GCP, enforcing consistent firewall rules and policies across all projects.
*   **Hybrid Connectivity:** Each location connects securely to the nearest GCP region using **Cloud HA VPN** with BGP dynamic routing for automatic failover. The Sao Paulo HQ uses a **Partner Interconnect** for a high-performance, dedicated link. This meets the requirement for **secure and redundant communications**.
*   **Zero Trust Security:** **Identity-Aware Proxy (IAP)** is implemented for all remote administration. This ensures that **no VM is ever exposed to the public internet**, and access is granted based on user identity and context, not just network location. This fulfills the goal of "secure remote control capabilities for administrators only."

**2. Automation & Modernization:**
*   **GKE Autopilot:** The homegrown secure browser is **containerized** and runs on a fully managed Kubernetes service. This makes it lightweight, highly scalable, and automatically managed by GCP, directly rivaling commercial solutions like Pearson VUE.
*   **Infrastructure as Code (IaC):** The entire environment can be provisioned using tools like Terraform, eliminating manual "click-ops" and ensuring consistent, repeatable deployments.
*   **ChromeOS Flex:** The existing Lenovo desktops are repurposed with ChromeOS Flex, enrolled in the Google Admin console. This allows for **centralized policy management** and reduces reimaging from hours to minutes, addressing a key pain point.

**3. Resilience and Performance:**
*   **Global Load Balancer:** Users worldwide connect to a single, secure endpoint provided by the **Global HTTPS Load Balancer**, which automatically routes them to the closest healthy backend instance, minimizing latency.
*   **Managed Services:** Using **Cloud SQL** and **Filestore** ensures databases and file storage are highly available, scalable, and backed up automatically, moving away from the "best effort" policy.

## Conclusion

This migration strategy transforms Balerica Inc. from a manually-intensive, geographically-limited operation into a fully automated, secure, and global organization capable of scaling effortlessly to meet its business goals. The choice of GCP services provides a modern, cost-effective, and future-proof foundation.
