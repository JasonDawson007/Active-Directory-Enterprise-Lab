# Enterprise Active Directory & Security Engineering Lab

**Author:** [Your Name] | [LinkedIn Profile Link] | [GitHub Profile Link]

---

## Executive Summary

Designed, deployed, secured, and audited an on-premises enterprise Active Directory Domain Services (AD DS) environment (dawsontech.local) built on Windows Server 2022 Datacenter (DC01) and Windows 11 Enterprise (CLIENT01). The project established a secure identity and access management foundation operating under the Principle of Least Privilege (PoLP) and Role-Based Access Control (RBAC).

Key accomplishments include:
- Automated identity lifecycle operations via PowerShell and CSV-based bulk provisioning.
- Designed a tiered departmental Organizational Unit (OU) structure isolating standard users, privileged administrative accounts, and endpoints.
- Hardened domain endpoints using Group Policy Objects (GPOs), deploying interactive compliance banners and restricting standard user administrative actions.
- Configured centralized departmental file shares utilizing NTFS inheritance stripping, security group access control, and Group Policy Preference (GPP) drive mapping.
- Implemented Advanced Security Audit Policies to capture actionable authentication telemetry, validating Event IDs 4624 (Logon Success), 4625 (Logon Failure), and 4740 (Account Lockout).
- Simulated a credential-stuffing/brute-force attack, analyzed forensic security telemetry, and executed administrative account containment and unlocking via PowerShell.

---

## Environment Architecture & Topology

| System / Hostname | Operating System | Domain Role | Primary Function |
| :--- | :--- | :--- | :--- |
| **DC01** | Windows Server 2022 Datacenter | Domain Controller (dawsontech.local) | AD DS, DNS, Group Policy Management, Central File Share (\\DC01\Departmental) |
| **CLIENT01** | Windows 11 Enterprise | Domain-Joined Workstation | Endpoint policy enforcement, RBAC verification, security telemetry generation |

### Organizational Unit (OU) Design Hierarchy

- **dawsontech.local**
  - **DawsonCorp**
    - **Admins**
    - **Employees**
      - Executive
      - Finance
      - IT
      - SecurityOps
    - **Groups**
    - **WorkStations**

---

## Technical Implementations & Engineering Phases

### Phase 1: Directory Architecture & Account Management
- **Bulk Provisioning Automation:** Executed Provision-ADUsers.ps1 to ingest employee data from CSV into Active Directory, standardizing sAMAccountNames and automating departmental OU placement.
- **Tiered OU Hierarchy:** Created a production-modeled directory tree under OU=DawsonCorp, partitioning objects into Admins, Employees, Groups, and WorkStations to enforce granular policy application.
- **Role-Based Identity Segregation:** Structured departmental sub-OUs (Finance, SecurityOps, IT, Executive) to maintain clean security boundaries between standard staff and domain administration.

### Phase 2: Endpoint Hardening & Policy Enforcement
- **Legal Compliance Notice:** Enforced an interactive logon banner via GPO (AUTHORIZED USE ONLY - DAWSONCORP) requiring user acknowledgment prior to authentication.
- **Workstation Isolation & RSoP Verification:** Linked GPO-Sec-WorkstationHardening to OU=WorkStations. Verified policy application on CLIENT01 using gpresult /r, confirming scope assignment and filtering behavior.
- **Least Privilege Enforcement:** Validated standard user privilege restrictions by preventing unprivileged accounts from modifying system hardware within Device Manager without administrative credential elevation.

### Phase 3: Centralized Storage & RBAC File Permissions
- **Share Architecture:** Deployed a central departmental SMB share (\\DC01\Departmental) containing dedicated subdirectories for Finance and SecurityOps.
- **NTFS Permission Hardening:** Disabled permission inheritance on confidential departmental folders, stripped default Users read permissions, and delegated access exclusively to dedicated Global Security Groups (SG_Finance_RW, SG_SecurityOps_RW).
- **Automated Drive Mapping:** Configured Group Policy Preferences (GPP) to map drive letter S: to \\DC01\Departmental upon logon.
- **Access Boundary Validation:** Confirmed that authorized security staff (mvance) possessed read/write capability in SecurityOps, while unauthorized access to Finance resulted in an explicit Windows Access Denied block.

### Phase 4: Advanced Security Auditing & Threat Telemetry
- **Advanced Audit Policy Configuration:** Created and enforced GPO-Sec-AdvancedAuditing, enabling granular auditing subcategories:
  - Logon/Logoff: Audit Logon (Success and Failure)
  - Logon/Logoff: Audit Account Lockout (Success and Failure)
  - Account Management: Audit User Account Management (Success and Failure)
  - Detailed Tracking: Audit Process Creation (Success)
- **Subcategory Enforcement:** Enforced the local security option to force audit policy subcategories over legacy audit categories to prevent suppression.
- **Command-Line Telemetry:** Enabled command-line logging within process creation events to ensure administrative and process execution visibility.

### Phase 5: Incident Response & Account Containment Playbook
- **Brute-Force Threshold Enforcement:** Defined domain account lockout thresholds in the Default Domain Policy set to 3 invalid logon attempts with a 30-minute lockout duration.
- **Attack Simulation:** Executed repeated invalid credential submissions against standard user account rgreen on CLIENT01, triggering immediate workstation lockout enforcement.
- **Forensic Event Ingestion:** Analyzed the Security log on DC01, identifying Event ID 4740 with the caller computer (CLIENT01) and target account (rgreen).
- **PowerShell Containment & Remediation:** Queried directory state using Get-ADUser to confirm LockedOut: True, executed Unlock-ADAccount, and re-verified return to LockedOut: False.

---

## Evidence & Verification Gallery

| # | Artifact Link | Test Description & Technical Finding |
| :-: | :--- | :--- |
| **1** | [Provision-ADUsers.ps1.jpg](images/Provision-ADUsers.ps1.jpg) | **PowerShell Provisioning Automation:** Automated bulk user onboarding execution in PowerShell ISE parsing CSV input and mapping accounts to respective OUs. |
| **2** | [Directory architecture.png](images/Directory%20architecture.png) | **Active Directory Structure:** ADUC console confirming tiered OU structure under DawsonCorp with departmental user segregation. |
| **3** | [Proper OU level inheritance.png](images/Proper%20OU%20level%20inheritance.png) | **Interactive Logon Banner:** Enforcement of organizational legal notice on CLIENT01 prior to desktop session access. |
| **4** | [Workstation RSoP Verification.jpg](images/Workstation%20RSoP%20Verification%20%26%20Hardening%20GPO%20Enforcement%20on%20CLIENT01.jpg) | **Resultant Set of Policy (RSoP):** Terminal output verifying CLIENT01 membership in WorkStations OU and enforcement of GPO-Sec-WorkstationHardening. |
| **5** | [Device Manager Elevation.jpg](images/Standard%20User%20Privilege%20Restriction%20%20Elevation%20Requirement%20in%20Device%20Manager.jpg) | **Privilege Boundary Enforcement:** System dialog blocking standard user modifications in Device Manager without elevated administrative credentials. |
| **6** | [Centralized file sharing and RBAC.png](images/Centralized%20file%20sharing%20and%20RBAC%20enforcemen.png) | **RBAC Read/Write Authorization:** Authenticated access on drive S: confirming file write permissions inside the authorized SecurityOps directory. |
| **7** | [Modal Access Denied.png](images/Modal%20Access%20Denied%20%2816%29.png) | **NTFS Access Termination:** Explicit Windows security denial preventing unauthorized personnel from accessing restricted departmental folders (\\DC01\Departmental\Finance). |
| **8** | [Event log logon success_failure.png](images/Event%20log%20logon%20success_failure.png) | **Authentication Telemetry Correlation:** Event Viewer Security log detailing consecutive Event ID 4625 (Logon Failure) entries followed by Event ID 4624 (Logon Success) for rgreen. |
| **9** | [Account lockout GPO configuration.png](images/Account%20lockout%20GPO%20configuration.png) | **Domain Lockout Policy Configuration:** Group Policy Management Editor confirming the 3-attempt lockout threshold and 30-minute duration window. |
| **10** | [Account locked out prompt.jpg](images/The%20referenced%20account%20is%20currently%20locked%20out%20and%20may%20not%20be%20logged%20on%20to..jpg) | **Endpoint Lockout Enforcement:** Workstation authentication interface confirming active account lockout after repeated invalid password submissions. |
| **11** | [Event Viewer Event ID 4740.png](images/Event%20Viewer%20details%20pane%20showing%20Event%20ID%204740.png) | **Brute-Force Detection (Event 4740):** Forensic log on DC01 isolating account lockout event, source workstation (CLIENT01), and targeted account (rgreen). |
| **12** | [Execution of Unlock-ADAccount.png](images/Execution%20of%20Unlock-ADAccount.png) | **PowerShell Triage & Remediation:** Verification of account lockout state (LockedOut: True), execution of Unlock-ADAccount, and validation of successful account restoration (LockedOut: False). |

---

## Engineering Hurdles & Troubleshooting

- **GPP Drive Mapping UNC Syntax Error:** Initial deployment of the shared departmental drive failed to mount on CLIENT01 due to a malformed UNC path (\\\DC01). Identified the syntax error within Group Policy Preferences, re-configured the path to \\DC01\Departmental, set the action to Replace, and verified successful mapping.
- **Security Event Log Access Restrictions:** Standard user accounts attempting to open the Security log in Event Viewer encountered an Access is denied (5) error. Identified that the Windows Local Security Authority (LSA) restricts Security.evtx access to administrators, resolving the issue by elevating Event Viewer via Run as administrator.
- **GPO Directory Collision Resolution:** Deleting a policy link left an unlinked GPO object in Active Directory, preventing recreation under the same name. Resolved the conflict by locating and removing the orphaned object directly from the root Group Policy Objects container in gpmc.msc.
- **Audit Policy Subcategory Overrides:** Addressed potential subcategory suppression by explicitly enabling the local security option to force advanced audit policy subcategories over legacy audit categories.

---

## Key Skills Demonstrated

- **Active Directory Administration:** Domain architecture design, OU hierarchy, user and security group management, Default Domain Policy configuration.
- **Identity & Access Management (IAM):** Role-Based Access Control (RBAC), Principle of Least Privilege (PoLP), inheritance stripping, NTFS and share permissions.
- **Endpoint Hardening & Governance:** Group Policy Objects (GPO), Group Policy Preferences (GPP), User Account Control (UAC) enforcement, Resultant Set of Policy (RSoP) analysis.
- **Security Telemetry & Auditing:** Advanced Audit Policy deployment, Windows Event Viewer forensic analysis (Event IDs 4624, 4625, 4740), brute-force detection.
- **Incident Response & Automation:** Account lockout investigation, caller machine identification, PowerShell incident triage and account remediation (Unlock-ADAccount).
