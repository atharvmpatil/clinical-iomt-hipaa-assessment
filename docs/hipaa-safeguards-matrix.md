# HIPAA Security Rule Compliance Matrix (45 CFR Part 164 Subpart C)

This matrix maps the engineering controls implemented in the Clinical IoMT network design to the technical and administrative safeguards required under the HIPAA Security Rule.

| Standard & Citation | Safeguard Specification | Requirement Type | Implemented Technical Architecture Control | Verification & Audit Evidence |
| :--- | :--- | :--- | :--- | :--- |
| **Access Control**<br>`§ 164.312(a)(1)` | Unique User Identification | **Required** | Deprecated generic local instrument logins. Integrated PCR and scanner workstation consoles with Active Directory; enforced individual named operator credentials. | Active Directory user provisioning exports and authentication event logs. |
| **Access Control**<br>`§ 164.312(a)(2)(ii)` | Emergency Access ("Break-Glass") | **Required** | Standalone emergency administrator credentials stored in an encrypted password vault requiring secondary administrative sign-off and immediate alerts. | Privileged Access Management (PAM) checkout records and alerting audit trails. |
| **Access Control**<br>`§ 164.312(a)(2)(iii)` | Automatic Logoff | **Addressable** | Enforced 10-minute session inactivity timeout via Group Policy (GPO) across all diagnostic instrument console workstations. | Registry configuration benchmarks and active GPO audit reports. |
| **Access Control**<br>`§ 164.312(a)(2)(iv)` | Encryption and Decryption | **Addressable** | BitLocker AES-256 volume encryption enabled on instrument control PCs; LIS database storage encrypted at rest using AES-256 transparent data encryption (TDE). | Volume encryption status reports and database encryption key rotation logs. |
| **Audit Controls**<br>`§ 164.312(b)` | Audit Controls | **Required** | Centralized Syslog forwarding from managed switch ports, firewalls, and LIS servers to an immutable SIEM instance; 6-year retention policy enforced. | SIEM ingestion dashboard, retention policies, and sample log query extracts. |
| **Integrity**<br>`§ 164.312(c)(1)` | Mechanism to Authenticate ePHI | **Addressable** | SHA-256 checksums calculated upon generation of raw instrument test files and validated by the LIS ingestion engine before database commit. | Checksum verification logs in LIS transaction pipelines. |
| **Transmission Security**<br>`§ 164.312(e)(1)` | Transmission Security & Encryption | **Addressable** | Disallowed unencrypted cleartext protocols across VLAN links. Mandated TLS 1.3 encapsulation for HL7 MLLP (TCP 2575) and DICOM (TCP 2762). | Wireshark packet capture analysis showing encrypted payloads. |
| **Device & Media Controls**<br>`§ 164.310(d)(1)` | Physical USB / Device Security | **Required** | Physical USB port blockers installed on instrument PCs; USB mass storage class drivers disabled via endpoint administrative policy. | Endpoint policy deployment logs and physical audit verification sheets. |

