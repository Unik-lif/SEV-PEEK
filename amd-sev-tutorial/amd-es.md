---
description: Whitepaper digest of AMD-ES.
---

# AMD-ES

### **AMD-Encrypted State**

### **Reasons:**

When interrupts occurs, register info will be loaded into hypervisor, Registers may leak some info. It should also be protected like guest memory.

Registers can be vulnerable, for example, the XMM registers are typically used to hold AES keys when using the x86 AES instructions.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-02 093223.png" alt=""><figcaption><p>AES stealing</p></figcaption></figure>

### Security Goals:

Reading registers states are inevitable when HV are used to perform some tasks. Instead of preventing reading registers, the idea to protect register state is similar to what SEV did for memory encryption. The SEV-ES technology enables the guest VM to decidewhat information it chooses to expose to the hypervisoron a per-case basis.

### Technical Overview:

In the legacy AMD-V architecture, saving and restoring guest VM register is a multi-step, which make it easy to leak data.

**What AMD-ES does is simply atomically do the steps above in only one instruction VMRUN.**&#x20;

_When VMRUN is executed for a guest with SEV-ES enabled, the CPU hardware loads all guest register information including system registers, GPRs,and floating point registers. Similarly when the VM stops running (a so-called “VMEXIT”), all of this state is saved automatically by hardware back to memoryand hypervisor state is loaded._

**The register will also be encrypted using the same key as memory encryption. Besides, an intergrity check will be lanuched when register states are saved.** It is computed by CPU, and the check-value will be saved in protected DRAM. This check is done when VMRUN takes effects.

相较于SEV对于内存的防护，此处对于寄存器防护加装了完整性，一定程度上可以抵抗重放攻击。那么为什么内存不加装完整性保护？

我个人认为，一方面寄存器的完整性保护依赖于被保护好的内存，对于内存整体来说提供这样的完整性保护开销是很大的。另一方面，重放与篡改类的攻击就寄存器和内存而言，显然是前者实现起来更加容易。在没有掌握密钥的情况下，想要直接对内存进行peek找到对应的代码.text段并加以利用，攻击的门槛是很高的。

可信本质上是相对安全，所以这样的设计在我看来还是算相对自然的事情。

但是即便有很高的攻击门槛，想要干点坏事儿还是很容易的。所以我们在之后看到了SEV-SNP的防护加装，其主要做的就是内存完整性的防护。

If th integrity-check value is not correct, CPU will refuse to resume a guest VM. So the only state to which a guest VM can be resumed is to the exact same state it was last in.

When using Hypervisor, **VMCB** should be maintained to store info of the VM. It is divided into 2 sections here.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-02 141000.png" alt=""><figcaption><p>VMCB</p></figcaption></figure>

The control Area is unencrypted while the save area is encrypted. Sensitive data is encrypted and has intergrity checks here.

### Revised VM Exit:

Different from legacy AMD-V VMEXIT, when AMD-ES is enabled, VMEXIT will be divided into two groups:&#x20;

* **AE:** automatic exits. AE events **do not require any hypervisor emulation** and include asynchronous interrupts, shutdown events, and cerain types of page faults. **When AE occurs, this will cause a full world switch and transfer control back to the hypervisor. As we mentioned above, with AMD-ES, the exit process is also atomic.**
  * The events involved here do not require guest register state.
* **NAE:** Non-automatic exits. This occur when the guest VM does something that will require hypervisor emulation. **When AMD-ES is enabled, this won't cause the world switch back to HV.** However, it will send a new exception **#VC**(VMM communication exception), which must be handled by the guest VM.
  * The events involved here do require guest register state.
  * Guest determines what register state to share in the GHCB.
  * Guest issues VMGEXIT instruction which causes an AE with exit code 0x403.

### [#vc](amd-es.md#vc "mention"): this is an exception send by CPU

vc exception informs the guest VM operating system that it performed an event which requires hypervisor emulation. The vc exception handler must respond and request services from the hypervisor.

A block called **GHCB(Guest Hypervisor Communication Block)** is established. This block resides in shared memory so it is accessible to both the guest VM and the hypervisor. (But not other Guest VMs).

**Notice:** GHCB is not a necessary thing. Follow the traditional process of HV emulation you can still get what you need.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-02 153604.png" alt=""><figcaption><p>NAE Flow</p></figcaption></figure>

Take the Above flow for example: you can refer to OSTEP chapter hypervisor as well, the key concept is the same.

1. The exception should be sent by CPU, so the Guest should first notify the CPU this info. **I trigger a NAE event, which is out of my scope, which should be emulated with the help of hypervisor.**
2. \#VC exception is sent to the Guest. #VC handler is ready. **Since GHCB is the shared part, the handler can simply copies state to GHCB as needed.**
3. **VMGEXIT**
4. **Registers State should be stored, as mentioned before, it will automatically and atomically be encrypted, then be sent to Hypervisor.**
5. **Hypervisor receive VMGEXIT, fetch what guest need, store it in GHCB. Then VMRUN to resume.**
6. **Load and decrypt guest state. etc..**

**A more detailed graph is shown below:**

<figure><img src="../.gitbook/assets/Screenshot 2023-03-18 164707.png" alt=""><figcaption><p>Clear Explaination</p></figcaption></figure>

****

