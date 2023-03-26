---
description: Secured Execution with AMD's SEV.
---

# Enarx

David Kaplan gave a graph of Key Management, which can be used in Paper.

**Enrax is A project of Red Hat.**

### Gap: Need for privacy. -> very complicated Virtualization Stack which is hard for customer to trust.

On which technology do I build my application?

### **Introducing Enarx.**

**Can used both in Process-Based or VM-Based TEE cases.**

<figure><img src="../.gitbook/assets/Screenshot 2023-03-26 162218.png" alt=""><figcaption><p>Enarx Architecture</p></figcaption></figure>

**Using WASM and WASI.**

Enarx is a Deployment Framework. -> compile to WASM -> send to Enarx -> Deploy a secure environment for you.

<figure><img src="../.gitbook/assets/Screenshot 2023-03-26 163402.png" alt=""><figcaption><p>Process</p></figcaption></figure>

**The process is very similar to TCP handshaking to some extent.**
