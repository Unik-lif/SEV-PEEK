---
description: 2019 KVM Forum.
---

# Secure Encrypted Virtualization - What's next?

Introduced the process for **LIVE MIGRATION** of SEV.

Have to use SP to perform these tasks. But it is not so quick -> becomes a bottleneck.

**SET UP a SEV MIGRATION HELPER.**

* Source Hypervisor uses a SEND API to migrate the MH.
* Destination hypervisor uses RECEIVE API to receive the MH.
* MH migrates the remainder of the guest
  * Uses CPU AES instruction, AMD SP not involved
  * Migration at multiple Gbps. (**relatively quick.**)
