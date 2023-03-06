---
description: Original talk about the white paper of SME & SEV.
---

# x86 Memory Encryption

## From the talk in USENIX Security '16 & Xen projects.

SEV is an extension to the AMD-V architecture which supports running encrypted virtual machine under the control of KVM.

Each encrypted VM is associated with a unique encryption key.

**At least, I will go through all public files AMD offered.**

****

### AMD x86 Memory Encryption

#### Overview

**Motivation: Cloud usage**

1. Hypervisor must enforce full isolation between co-resident VMs.
2. Cloud users must fully trust the cloud hoster.

**Defend:**

can defend user access attacks and physical access attacks.

**How?**

Secure Memory Encryption & Secure Encrypted Virtualization

1. Hardware AES engine. -> for DRAM
2. Minimal performance impact. extra latency only taken for encrypted pages.
3. No applications changes required.
4. Encryption keys are managed by the AMD Secure Processor and are hardware isolated. They are not known to any softwares on the CPU.

To be detailed, OS/Hypervisor chooses pages to encrypt via page tables. This will prevent unpriviledged access to memory.

The Secure Memory Encryption process is shown below:

<figure><img src="../.gitbook/assets/Screenshot 2023-02-27 183841.png" alt=""><figcaption><p>SME tech</p></figcaption></figure>

**Briefly, it is the C-bit that counts.**

**Transparent SME**

enables memory encryption without OS modifications.

This will forces C-bit on all accesses, configured by AMD Secure Processor during boot x86 is released.

**SEV**

<figure><img src="../.gitbook/assets/Screenshot 2023-02-27 184639.png" alt=""><figcaption><p>SEV overview</p></figcaption></figure>

Hypervisor, guest VMs are seperateds cryptographically.

**SEV details**

1. Address Space ID(ASID) determines VM encryption key.
2. HW and Guest page tables determine if a page is private or shared
3. DMA must occur to shared pages

**The difference is shown below:**

<figure><img src="../.gitbook/assets/Screenshot 2023-02-27 192302.png" alt=""><figcaption><p>comparison</p></figcaption></figure>

#### Key management

This part is also included in the video, when we have to do remote attestation, we should refer to this part.
