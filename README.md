# From Shared Inboxes to Credential Exfiltration: An Empirical Security Analysis and Defense Architecture for Nextcloud Mail


This repository hosts the formal security research paper: **"From Shared Inboxes to Credential Exfiltration: An Empirical Security Analysis and Defense Architecture for Nextcloud Mail"**. 

The paper presents an empirical security audit of the [**Nextcloud Mail application**](https://github.com/nextcloud/mail) (`OCA\Mail`), focusing on authorization boundaries, protocol transitions, and parser differentials across multi-user delegation setups, server-side Sieve generation, client-side rich-text editing, and cryptographic verification pipelines.

---

## Direct Document Access

- **Primary Research Document**: [**`Nextcloud_Mail_Security_Research_Paper.pdf`**](./Nextcloud_Mail_Security_Research_Paper.pdf)

---

## Executive Summary

Modern enterprise webmail platforms bridge standard RFC 5322/IMAP/SMTP messaging with enterprise directory provisioning, mailbox delegation, server-side rule execution, and client-side web rendering. While this tight integration enhances team workflows, it introduces significant attack surface along authorization and parser boundaries.

This research systematically examines the [**Nextcloud Mail implementation (`v6.0.0-alpha.1`)**](https://github.com/nextcloud/mail), discovering **eight undocumented vulnerabilities** ranging from CVSS 4.3 to 8.8. The paper provides detailed execution traces, formal invariants, concrete defensive remediations, and test suite validations.

---

## Architectural Diagrams & Visualizations
All diagrams are provided in standalone, uncompressed SVG vector format with uniform Times New Roman typography:
| Figure | Diagram Title & File Link | Description & Attack Context |
| :---: | :--- | :--- |
| **Figure 1** | [**`architecture.svg`**](./architecture.svg) | **System Architecture & Attack Surface Boundaries**<br>Decomposes client controllers, domain services, QBMappers, and external protocol bridges across untrusted boundaries. |
| **Figure 2** | [**`managesieve.svg`**](./managesieve.svg) | **ManageSieve Host Hijacking & Credential Exfiltration (VULN-01)**<br>Execution trace showing fallback credential extraction and plain IMAP / master password capture on rogue listeners. |
| **Figure 3** | [**`crossaccount.svg`**](./crossaccount.svg) | **Cross-Account Attachment Hijacking (VULN-02)**<br>Sequence diagram showing unauthorized rebinding of private draft attachments across delegated accounts. |
| **Figure 4** | [**`sievefilter.svg`**](./sievefilter.svg) | **Sieve Filter Compilation & CRLF Breakout State Machine (VULN-03)**<br>UML state machine contrasting vulnerable `# ` comment breakout with hardened AST validation and allowlisting. |
| **Figure 5** | [**`csshtml.svg`**](./csshtml.svg) | **Dual-Stage CSS & HTML Sanitization Pipeline (VULN-04 & VULN-05)**<br>Contrasts flawed case-sensitive regex matching with tokenized AST proxy rewriting. |
| **Figure 6** | [**`dkim_alignment.svg`**](./dkim_alignment.svg) | **Decoupled Evaluation vs. Strict Domain Alignment (VULN-06)**<br>Illustrates how decoupled DKIM verification enables domain spoofing versus strict RFC 7489 alignment. |
| **Figure 7** | [**`formalentity.svg`**](./formalentity.svg) | **Relational Entity Model & SASI Boundaries**<br>Database entity relationships and ownership scoping enforcing the Strict Account Scoping Invariant. |
---

## Discovered Vulnerability Summary

| ID | Title | CVSS v3.1 | Severity | CWE | Primary Attack Impact |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **VULN-01** | ManageSieve Host Hijacking & Password Exfiltration | `8.8` | **High** | CWE-522, CWE-285 | Extraction of cleartext IMAP passwords and enterprise master credentials via fallback abuse. |
| **VULN-02** | Cross-Account Attachment Hijacking | `8.1` | **High** | CWE-639, CWE-863 | Unprivileged delegates reassign and exfiltrate private draft attachments from isolated accounts. |
| **VULN-03** | ManageSieve CRLF Script Injection | `7.6` | **High** | CWE-74, CWE-93 | CRLF breakout in filter metadata injects arbitrary Sieve commands (`redirect :copy`) on mail server. |
| **VULN-04** | Composer HTML Injection & Tracking Pixel Leak | `6.5` | **Medium** | CWE-79, CWE-200 | Unsanitized sender labels injected into CKEditor 5 bypass proxy and trigger immediate IP disclosures. |
| **VULN-05** | CSS Sanitization & Remote Image Proxy Bypass | `5.4` | **Medium** | CWE-200, CWE-184 | Case-sensitivity flaws and mixed-protocol inline CSS properties bypass the privacy proxy filter. |
| **VULN-06** | Decoupled DKIM Verification Domain Spoofing | `5.3` | **Medium** | CWE-347, CWE-290 | Cryptographically valid DKIM signatures from arbitrary domains display authentic validation badges. |
| **VULN-07** | Cross-Account S/MIME Key Signing via Drafts | `5.0` | **Medium** | CWE-639, CWE-285 | Delegate abuses owner identity to sign outbound correspondence using owner's private certificate. |
| **VULN-08** | Plaintext Autoconfig & Path Traversal Risks | `4.3` | **Medium** | CWE-319, CWE-22 | Unencrypted HTTP discovery fallback and unvalidated attachment filenames allow traffic interception. |

---

## Key Research Contributions

1. **Formal Threat Model**: Formulates boundaries between Account Owners ($U_{\text{owner}}$), Delegatees ($U_{\text{delegate}}$), Provisioned Enterprise Users ($U_{\text{prov}}$), and Remote Adversaries ($Adv_{\text{ext}}$).
2. **Strict Account Scoping Invariant (SASI)**: Formalizes database-level isolation guarantees:
   $$\forall R \in \text{Resources}, \quad \text{Account}(R) \equiv A_{\text{effective}} \quad \land \quad \text{Owner}(A_{\text{effective}}) \equiv U_{\text{owner}}$$
3. **Host-Bound Credential Lifecycle**: Proves that secondary protocol credentials (ManageSieve) must be cryptographically bound to primary inbound host origins to prevent egress exfiltration.
4. **Grammar-Constrained AST Compilation**: Replaces regex-based filters with AST transformations for both RFC 5228 Sieve script compilation and CSS URL tokenization.
5. **DKIM Alignment Enforcement**: Restructures DKIM validation to enforce strict domain alignment between the cryptographic signing identity (`d=`) and the envelope `From:` header per RFC 7489 (DMARC).

---

## Repository Structure
```text
.
├── Nextcloud_Mail_Security_Research_Paper.pdf  # Formal academic paper (7 pages, Times New Roman)
├── architecture.svg                           # Figure 1: System Architecture & Boundaries
├── managesieve.svg                            # Figure 2: ManageSieve Host Hijacking Trace
├── crossaccount.svg                           # Figure 3: Cross-Account Attachment Hijacking Trace
├── sievefilter.svg                            # Figure 4: Sieve Filter State Machine
├── csshtml.svg                                # Figure 5: CSS & HTML Sanitization Pipeline
├── dkim_alignment.svg                         # Figure 6: DKIM Verification & Alignment Flow
├── formalentity.svg                           # Figure 7: Relational Entity Model & SASI
└── README.md                                  # Repository overview and diagram navigation
```
---

## BibTeX Citation

If you reference this research, methodology, or vulnerability taxonomy in academic work, please cite:

```bibtex
@article{atankov2026nextcloudmail,
  title={From Shared Inboxes to Credential Exfiltration: An Empirical Security Analysis and Defense Architecture for Nextcloud Mail},
  author={Atanas, Tankov},
  journal={Preprint / Technical Security Report},
  year={2026},
  url={https://github.com/tankovatanas/nextcloud-mail-security-research}
}
```

---

## Responsible Disclosure & Disclaimer

The vulnerabilities documented in the paper were identified through static dataflow analysis, grammar differential audits, and isolated local reproduction. In accordance with coordinated vulnerability disclosure practices, technical details, proof-of-concept vectors, and remediations have been documented for review through appropriate security channels. This publication is provided for defensive engineering and academic purposes only.
