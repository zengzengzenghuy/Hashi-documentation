# Safe with Hashi

<figure><img src="../../.gitbook/assets/diagram (1).png" alt=""><figcaption><p>Safe with Hashi</p></figcaption></figure>

### Overview

Safe with Hashi is a mechanism crafted to empower users with the ability to control a Safe across different blockchains using Hashi. The idea is to have a main Safe which can be controlled by peripheral Safes using storage proofs verification against a block header propagated using Hashi.



### How it works

1. Begin by deploying two Safes - the main Safe and the Secondary Safe. Attach a module called `ControllerModule` to the main Safe.
2. Users wishing to execute a cross-chain transaction should call `execTransaction` on the `Peripheral` Safe which just writes in storage a commitment corresponding to the used function parameters.
3. After the transaction is included in a block, a relayer propagates the corresponding block header to the chain where the main Safe is deployed.
4. Once all bridges have stored the block header in their corresponding adapters, the user can call `execTransaction` on the `ControllerModule`, providing the storage proof. The `ControllerModule` verifies the proof against the block header propagated with Hashi, and subsequently calls `execTransaction` on the Main Safe using `execTransactionFromModule`.



### Reference

1. [**https://github.com/crosschain-alliance/safe-crosschain/tree/**](https://github.com/crosschain-alliance/safe-crosschain/tree/)
