# Network Segmentation & Access Control Architecture

## 1. Network Zoning Strategy
To enforce zero-trust containment for clinical devices handling electronic Protected Health Information (ePHI), all assets are segregated into distinct Virtual Local Area Networks (VLANs). 

Each zone maintains a dedicated Class C private subnet with explicit boundaries enforced at the Layer 3 Next-Generation Firewall (NGFW).

### Subnet Allocations
* **VLAN 10 — Clinical IoMT Diagnostic Zone (`10.10.10.0/24`)**
  * **Assigned Assets:** Automated Hematology Analyzers, PCR Instrument PCs, Digital Pathology Slide Scanners.
  * **Traffic Profile:** High-volume telemetry, HL7 v2 messaging, DICOM streaming.
  * **Security Baseline:** No direct outbound internet access. Isolated broadcast domain.

* **VLAN 20 — Core Healthcare IT & Database Zone (`10.10.20.0/24`)**
  * **Assigned Assets:** Laboratory Information System (LIS) application server, PostgreSQL/MSSQL database cluster, local PACS archive.
  * **Traffic Profile:** Encrypted database transactions, centralized log ingestion.
  * **Security Baseline:** Ingress strictly restricted to approved clinical ports; database access restricted to app tier.

* **VLAN 30 — Administrative & Clinician Workstations (`10.10.30.0/24`)**
  * **Assigned Assets:** Clinical staff desktops, nurse station laptops, barcode label printers.
  * **Traffic Profile:** Web browsing, LIS client access, EHR portal access.
  * **Security Baseline:** Standard enterprise endpoint detection and response (EDR); internet access through secure web gateway.

* **VLAN 90 — Secure Out-of-Band Vendor Management (`10.10.90.0/28`)**
  * **Assigned Assets:** Hardened Jump Host / Bastion server for medical device vendor remote diagnostics.
  * **Traffic Profile:** Encrypted administrative sessions (SSH, RDP).
  * **Security Baseline:** Mandatory multi-factor authentication (MFA); session recording; zero lateral routing.

---

## 2. Firewall Access Control List (ACL) Matrix
All inter-VLAN routing is governed by an explicit **Default Deny** security posture. Traffic is blocked unless matching a specific whitelist rule.

| Rule ID | Source Zone / Subnet | Destination Zone / Subnet | Service / Protocol | Port | Action | Justification / Operational Scope |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **FW-01** | VLAN 10 (`10.10.10.0/24`) | VLAN 20 (`10.10.20.15`) | TCP (MLLP / TLS) | 2575 | **ALLOW** | Transmit HL7 diagnostic orders and lab results to LIS interface engine. |
| **FW-02** | VLAN 10 (`10.10.10.0/24`) | VLAN 20 (`10.10.20.20`) | TCP (DICOM-TLS) | 2762 | **ALLOW** | Transfer whole-slide imaging files from slide scanner to PACS server. |
| **FW-03** | VLAN 30 (`10.10.30.0/24`) | VLAN 20 (`10.10.20.15`) | TCP (HTTPS) | 443 | **ALLOW** | Lab technicians and clinicians access the LIS web interface. |
| **FW-04** | VLAN 90 (`10.10.90.5`) | VLAN 10 (`10.10.10.0/24`) | TCP (RDP / SSH) | 3389, 22 | **ALLOW** | Vendor telemetry and patch deployment via authenticated Bastion Jump Host. |
| **FW-05** | ALL VLANs | VLAN 20 (`10.10.20.50`) | UDP (Syslog) | 514 | **ALLOW** | Forward switch, firewall, and endpoint access logs to central SIEM. |
| **FW-06** | VLAN 10 (`10.10.10.0/24`) | WAN (Internet) | ANY | ANY | **DENY** | Prohibit medical devices from reaching external endpoints or command-and-control servers. |
| **FW-07** | VLAN 30 (`10.10.30.0/24`) | VLAN 10 (`10.10.10.0/24`) | ANY | ANY | **DENY** | Prevent compromised staff workstations from scanning or exploiting IoMT hardware. |
| **FW-DEFAULT** | ANY | ANY | ANY | ANY | **DENY & LOG** | Catch-all rule dropping and logging all unauthorized inter-subnet packets. |
