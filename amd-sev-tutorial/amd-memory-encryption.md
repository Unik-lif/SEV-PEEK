---
description: Digest for SME & SEV Whitepaper.
---

# AMD Memory Encryption

### **Basic Concepts:**

**SME: Secure Memory Encryption.**&#x20;

Before SME, data is carefully encrypted in disk, but not in memory, which makes it relative easy to leak some info for attackers. SME is basically a method to encrypt memory.

**SEV: Secure Encryption Virtualization.**

After intergration of SME in AMD-V tech, SEV makes it possible for VM isolated from not onlt other VMs, but the Hypervisor Adminstrator.

### **SME**

**SME largely depends on a 32-bit subsystem which can encrypt without the help of software in the host cores.**

<figure><img src="../.gitbook/assets/Screenshot 2023-02-28 160823.png" alt=""><figcaption><p>Memory Write with optional encryption</p></figcaption></figure>

The Graph above shows the basic process for memory encryption. A mux will using PTE C-bit as the selection bit, when 1 is input, then all the memory page entries will be encrypted before using.

The Subsystem on AMD SoC we talked above will randomly generate a key when resets, and this key will be stored in a confidential hardware, which the VM can trust.

What's more, since in 64-bit machine, we only have 46 wire for address encoding, so AMD SME simply using the 47th-bit as the place to hold C-bit. That's a very interesting thing.

<figure><img src="../.gitbook/assets/Screenshot 2023-02-28 161436.png" alt=""><figcaption><p>PTE address mapping</p></figcaption></figure>

The process is shown in Fig.2. C-bit is given by OS or HyperVisor.

### SME use models:

1. Full memory encryption: simply uses DRAM start from 0x8000 0000 0000，which naturly encrypted all of the page table entries.
2. Partial memory encryption: only encrypted a subset of the DRAM, this case, when used in VM models, can naturely establish an Isolation for VMs.

<figure><img src="../.gitbook/assets/Screenshot 2023-02-28 175032.png" alt=""><figcaption><p>Natural Isolation</p></figcaption></figure>

### **SME boundary cases:**

1. **coherency of encrypted & unecrypted part:** be careful when modifying the C-bit of the encrypted part. The Cache should be flushed beforehand to be consistent.
2. **32-bit legacy devices:** Use remap way to able 47th-bit set or reset.

**special case:** Transparent SME: TSME, all memory is encrypted regardless of C-bit. This mode can be set @ boot time.

### **SEV:**&#x20;

<figure><img src="../.gitbook/assets/Screenshot 2023-02-28 183514.png" alt=""><figcaption><p>SEV overview</p></figcaption></figure>

The threat model in AMD SEV is different from traditional model. The attacker here has higher priviledge, and can access the victim memory freely.&#x20;

The security here largely depends on cryptography.

### SEV use models:

Basically for Cloud & Sandbox.

### SEV Architecture

_昨天问了侯锐老师SoC与冯诺依曼内存瓶颈相关的问题，比如做一个球体的计算机什么的，我当时和老师说看来嵌入式会用的SoC多一些，得到肯定的答复，没想到今天就在文献看到了......_

With the help of SME technology, the architecture of SEV is relatively a very natural thing now.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-01 142749.png" alt=""><figcaption><p>SEV Architecture</p></figcaption></figure>

Every VM are cryptographically isolated.&#x20;

**Inside SoC:** an ASID tag will help to protects VM data.

**Outside SoC:** 128-bit AES encryption.

### Encrypted Memory:

**SEV** provides a more concrete use model for VMs, they give guest VM the ability to choose which data memory pages they would like to be private.

The whitpaper here says:

_This choice is done using the standard CPU page tables, and is fully controlled by the guest._

So we can imply that this choice-setting-bit might be the bit position smaller than 8. The PTE, specefically, should be paid more attention here.



private memory is encrypted with the guest-specific key, while shared memory may be encrypted with the hypervisor key. Which entitles the VM privacy as long as the convenience to using shared resources.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-01 145122.png" alt=""><figcaption><p>Communication Example</p></figcaption></figure>

Other memory part of Guest will be encrypted, thus isolated from HV.

### Key Management

The core security of SEV is dependent on the keys. Apparently we can't let HV to gain SEV, so it is SEV firmware that shoulder this great responsibility.

The HV uses this interface to enable SEV without compromising the security of Guest VMs. These activities include launching, running, snapshotting, migrating, and debugging a guest.

#### What does the firmware do?

1. **Authenticating the platform:** The authenticity of the platform is proven with its identity key. The key is signed by AMD to demonstrate the platform is not a masquerading one.
2. **Attestation of the guest launch:** guests securely launched with SEV enabled. A signature of various components of the SEV related guest state, including initial contents of memory, is provided by the firmware to the guest owner to verify that the guest is in the expected state. **This give guest VM assurance for the hypervisor not interfere with initialization of SEV. E.G.** SEV firmware provides a measurement back to the guest owner.
3. **Confidentiality of the guest:** encrypting memory with a memory encrpytion key that only the SEV firmware knows, **Which means the Key is safekept by the firmware.** During the setup operation, the guest's memory is kept encrypted during transmission. **Once the remote platform is authenticated, the SEV firmware sends key securely to the remote platform.**

<figure><img src="../.gitbook/assets/Screenshot 2023-03-01 152021.png" alt=""><figcaption><p>Attestation Example</p></figcaption></figure>

### **SEV Software Implications**

**Hypervisor:**

hypervisor communicate with AMD-Secure Processor to lanuch a new VM, perform guest attestation, perform migration, etc...

**Guest:**

Guest OS should know the new hardware feature and adjust its page table and other mechanism,  a similar way to do this is simply set C-bit to be 1.

DMA: -> must occur to shared guest memory. The Guest OS can choose to allocate memory pages for DMA as shared(C-bit 0), or copy data to/from **bounce buffer** for DMA purposes.

For multi-core -> same ASID for all virtual CPU for a particular guest.

### SEV Special Considerations

1. page flushed from cache prior to accessing it with a different C-bit. Besides, prior to replacing a hardware memory encryption key, the hypervisor software must perform a full cache flush.
2. besides PAE mode(in which the C-bit can be controlled), C-bit will always be set to 1.





****

