# Clinical IoMT Threat Model (STRIDE Methodology)

## 1. Scope & System Architecture
This assessment covers an outpatient molecular diagnostics laboratory operating networked instruments handling electronic Protected Health Information (ePHI).

### Assessed In-Scope Devices
* **Automated Hematology Analyzer:** Runs embedded Windows 10 IoT; outputs complete blood count (CBC) panels via HL7 v2 over local Ethernet.
* **Real-Time PCR System:** Connected via USB to an instrument-control PC running Windows; generates diagnostic cycle threshold (Ct) reports.
* **Digital Pathology Slide Scanner:** High-throughput whole-slide scanner streaming uncompressed DICOM image files to a local staging server.

---

## 2. Data Flow Diagram (Logical Path)

[Diagnostic Instrument]
│ (Raw Assay Metrics / Unencrypted HL7 v2 / DICOM)
▼
[Local VLAN 10 Switch Port]
│ (802.1Q Trunked Interface)
▼
[Next-Generation Firewall / Gateway]
│ (Enforces Default-Deny ACLs & TLS Encapsulation)
▼
[Laboratory Information System (LIS) / PACS Server] (VLAN 20)

---

## 3. STRIDE Threat Analysis Matrix

| Threat Category | Specific Attack Vector | Potential Clinical Impact | Mitigating Technical Control |
| :--- | :--- | :--- | :--- |
| **Spoofing** | Attacker spoofs the MAC/IP address of the Hematology Analyzer on the local subnet to inject forged diagnostic results. | False lab results committed to patient records; risk of contraindicated medical treatments. | 802.1X port-based Network Access Control (NAC); static DHCP reservations bound to hardware MACs. |
| **Tampering** | Man-in-the-Middle (MitM) modification of cleartext HL7 messages traversing the internal switch. | Altered test values (e.g., platelet counts, viral load status). | Wrap Minimal Lower Layer Protocol (MLLP) traffic in TLS 1.3 tunnels across VLAN boundaries. |
| **Repudiation** | Laboratory staff use shared generic workstation logins (`labuser01`) on the PCR terminal. | Inability to audit individual accountability for test initiation, modification, or deletion. | Enforce Active Directory / LDAP authentication with individual credentialing and session audit logs. |
| **Information Disclosure** | Passive packet sniffing on the local segment capturing unencrypted DICOM pathology scans containing patient identifiers. | Severe HIPAA violation; direct leak of patient biopsy records and identity. | Enforce DICOM-TLS (port 2762) encapsulation; isolate diagnostic devices into non-broadcast VLANs. |
| **Denial of Service (DoS)** | Network broadcast storms or SYN flooding targeting the legacy TCP/IP stack of the hematology analyzer. | Instrument crash or freeze; delay of emergency STAT diagnostic orders. | Layer 2 broadcast containment; switchport ingress rate limiting; stateful inspection at the gateway. |
| **Elevation of Privilege** | Remote exploitation of unpatched SMBv1 / legacy service on an end-of-life instrument workstation. | Workstation compromise used as a foothold to pivot deeper into core healthcare IT infrastructure. | Host-level hardening: disable unused legacy protocols (SMBv1, NetBIOS); zero direct internet egress. |
