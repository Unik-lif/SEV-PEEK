---
description: Designed to protect a VM from a malicious hypervisor in specific way.
---

# SEV-SNP Report.

**Stronger attack models:**

benign but vulunerable hypervisor. The speaker: David Kaplan also note that the Guest in the threat model is not able to trigger a denial of service in whole process.

Enforcing Integrity: **Reverse mapped table.**

**One entry for every page. -> Page Onwership is written in.**

**Companioned with Page Validation.**

**VMPLs -> Optional features.**

**DoorBell #HV. -> See Hecate.**

**Specific capabilities:**

* Virtual VTOM. -> Virtual Top of memory.
* Reflect-VC. -> take #VC exceptions and turn them into VMEXIT events.

**For side channels**, some optional protections are offered by AMD.

\----------------------------------------

**Attestation: Establishing Trust in Guests.**

**太抽象了，润润子。**
