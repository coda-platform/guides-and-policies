# Data Security Framework

### Overview
 
- **Description:** This file describes the general framework under which the various entities affiliated with the CODA project have accepted to collaborate.
- **Primary author(s):** Michaël Chassé, Maxime Lavigne, Bruno Lavoie, Louis-Antoine Mullie
- **Contributors:** 

### Introduction

This document outlines the data security policies governing the use of CODA, a platform for distributed analysis of real-world healthcare data. The overarching principle of the CODA platform is that no individual patient data will be exchanged between hospital sites. The objective of this document is to document the procedures that will be deployed in order to protect confidential patient information across all aspects of the CODA project.

### Definitions

"CODA network" refers to the entirety of the devices and systems deployed as part of the CODA project.

"CODA computation node" refers to a virtual machine deployed on a physical server at one of the participating sites, which provides computing resources for decentralized analyses, and is accessible through the CODA standard interface.

"CODA storage node" refers to a virtual machine deployed on a physical server at one of the participating sites, which stores de-identified data meeting the inclusion criteria for the cohort.

"CODA application server" refers to a virtual machine deployed on a physical server at the computing administration site (CHUM), which communicates with the CODA network and serves the applications that are built as part of the CODA network.

"CODA dashboard" refers to a web-facing application, which is deployed on the CODA application server, and enables authenticated and authorized users to obtain a high-level overview of key statistics that are computed by the CODA network.

"CODA standard format" refers to the standard format used to represent and store the data. In this document, the Fast Healthcare Interoperability Resources (FIHR) format will be assumed.

### Authentication system

The authentication component will be responsible for:

- Keeping an up-to-date list of user accounts who will be authorized to access certain CODA applications and other system components;
- Managing creation, update and revocation of authentication credentials associated with the aforementioned users;
- Recording a timestamped log of successful and unsuccessful authentication attempts, and notifying the system administrator of any suspicious login attempts.

End-users will be authenticated via the use of the following combination of credentials:

- An institutional e-mail, associated with a whitelisted e-mail domain or sub-domain at one of the participating institutions (e.g. @chum.ssss.gouv.qc.ca).
- An account password, which will be supplied by the user at the time of account creation. Standard rules for the creation of secure passwords will be enforced.
- A 2-factor authentication code, which will be sent to the user's phone via Short Message Service (SMS) at the time of authentication.

The creation of user passwords will be according to the NIST digital identity guidelines:

- Minimum of 8 characters and maximum of 128 characters
- Ability to use all special characters, but no special requirement to use them
- Restriction of context-specific passwords (e.g. name of the site)
- Restriction of commonly used passwords and dictionary words
- Restriction of passwords from previously breached corpuses

### Authorization system

A central authorization system will be implemented to limit what operations users can perform in the CODA network. This system will be responsible for:
 
- Defining access roles, and the permissions associated with each role;
- Granting or revoking one or more roles to a user account;
- Recording a timestamped log of any data accessed by a user.
 
Note that the central authorization system does not govern access control outside of the CODA network (e.g. existing servers and storage infrastructures at participating sites). Access control for these external components remains under the local responsibility of each site.

Roles will be granted on a per-site basis.

#### Protection of data at rest

"Data at rest" refers to information that is stored in a permanent or semi-permanent fashion on any component of the CODA network. The protection of data at rest in the context of the CODA project is based on the following principles:

- Data isolation: CODA storage nodes, inside which de-identified data will be stored, will be implemented as virtual machines that are used solely for the purpose of data storage and retrieval.
- Access limitation: only a restricted subset of users is allowed to access environments where data at rest is present, and only a restricted set of interfaces is enabled for access. CODA storage nodes may only be accessed from a whitelisted list of IP addresses.
- Server hardening: CODA storage nodes will run AppArmor, a Linux application security system that protects the operating system and applications by providing mandatory access control.
- Data preservation: data collected as part of the CODA will be preserved for 10 years, unless otherwise specified by an ERB-approved study protocol.
- Access logs: Recording a timestamped log of any data accessed by a user.

### Protection of data in transit

"Data in transit" refers to any information exiting a component of the CODA network over a network interface. Such exchanges may occur either:

- Between multiple CODA computation nodes, e.g. when a federated learning query is performed, and a trained machine learning model is passed between nodes.
- Between a computation node in the CODA network and an authorized device in the local area network of one of the participating sites, e.g. when an authorized researcher programmatically submits an API query to the local CODA computation node.
- Between the application server and the CODA network, e.g. when an authorized researcher performs a distributed analysis via the CODA dashboard.
- Between the application server and an authorized device in the local area network of one of the participating sites, e.g. when an authorized researcher consults the CODA dashboard from their personal workstation at one of the participating sites.
- All data in transit will be protected using Secure Sockets Layer/Transport Layers Security (SSL/TLS), with 4096-bit RSA keys. Standard SSL/TLS will be used for applications deployed over a web interface (scenario 4), which only display a highly restricted subset of aggregate statistics. Two-way SSL/TLS, aslo known as "TLS with client certificate authentication," will be used for all other situations of data in transit (scenarios 1-3).

### Data de-identification

The CODA project is committed to applying the highest standards for the protection of the patient's privacy and confidentiality. No personally identifying information shall, under any circumstance, be made available on any component of the CODA network. This section describes the data de-identification policies that will be enforced in order to ensure that confidential information is handled with the utmost rigor.
 
All patient-level identifying information (e.g. patient identifiers, address of residence, day of birth) is removed during data extraction, prior to storage of the data in the CODA repository. Elements of a unique type (e.g. a specific laboratory test) with a count less than five in the overall output are censored. 

Database row identifiers (non-sensitive identifiers that uniquely identify a database record in local source systems) are hashed using PBKDF2 with 100,000 iterations of SHA512, with a secret pepper (per-row) and a secret salt (per-site) each consisting of 128-character hexadecimal strings (512 bits) generated using secure cryptographic random number generators.

For DICOM files, following the whitelist approach to de-identification,14 only technical, non-identifying metadata headers are retained. Pixel data is extracted only from imaging slices, corresponding to the following modalities in the DICOM standard: CR, CT, US. Free text data (such as imaging reports, textual clinical observations or nursing notes) is not currently included in the CODA standard format.

Textual data (such as imaging reports, free text clinical observations or nursing notes) is not currently included in the CODA standard format. If such data is included at a later date, a formal process will be agreed upon to identify the optimal de-identification strategy, and the data security policy will be amended accordingly.

### Application security

The CODA application server will be responsible for serving web-facing applications, such as the CODA dashboard. Applications deployed on this server will implement 2-factor authentication, as described in section 3, and be served over SSL/TLS, as described in section 6.
 
In addition to these measures, CODA applications will be subject to review by the data security committee in order to ensure that they follow guidelines for secure coding practices[1]. This includes, but is not limited to, the following:
 
- Ensuring that appropriate validation of all user-supplied inputs is performed;
- Ensuring that session management is performed using strong random tokens;
- Ensuring that logout functionality fully terminates the associated session or connection;
- Ensuring that server-side components segregate privileged logic from other code;
- Ensuring that error handling does not disclose sensitive information;
- Ensuring that the SSL/TLS keys are stored in a secure location.
 
"Scripting" will not be permitted through the dashboard. Researchers who build custom statistical and/or machine learning models using scripts, and wish to train them across sites, will be required to submit their applications for code review; such models will

### Site responsibilities

De-identification of data will be under the ultimate responsibility of the site lead data scientist, who will seek assistance from the data security committee in the event of any questions. In addition, each site will nominate a privacy auditor, who will receive a summary report of each "build" of the site's CODA database. This summary report will contain descriptive statistics for each table and column, and implement automated rules to flag potentially problematic information. The privacy auditor will review each summary report in order to identify any potential anomalies in the data de-identification process.

### Formal auditing

Independent security audit: this document will be formally reviewed by an independent security consultant, and a security audit will be conducted once the network is in place and multicenter analyses are ready to be performed. Subsequent audits may be necessary after significant changes to the security framework, as deemed relevant by the data security committee.

Duty to report: all members of the CODA project have a duty to report any security anomalies to the data security committee: for example, if a user was given an incorrect role, or if a site found multiple users were using the same credentials.
Data security committee

A data security committee will be created in order to coordinate and supervise the implementation of the data security framework described in this document. This committee will contain at least one independent consultant with formal expertise in information security. In addition, each site will nominate a privacy auditor, who will report to the data security committee.
