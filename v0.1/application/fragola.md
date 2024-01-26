---
description: FRAGOLA, a FRAmework for GhO cross-Ledger Access
---

# FRAGOLA

<figure><img src="../../.gitbook/assets/diagram (2) (1).png" alt=""><figcaption><p>FRAGOLA Architecture</p></figcaption></figure>

## Project Description

FRAGOLA is a GHO facilitator framework helping GHO go cross-chain! FRAGOLA stands for "FRAmework for GhO cross-Ledger Access". This framework can be used to accept on different blockchains a variety of tokens as collateral to mint GHO, as long as the price oracle for assessing the assets’ worth is properly configured. For the hackathon-specific use case, FRAGOLA allows for the minting/burning of GHO on Ethereum while depositing/withdrawing collateral on a different chain, namely Gnosis Chain.

The following is a description of the minting flow. A user deposits liquidity in a pool and receives the corresponding LP token. This allows for the generalisation of our use case since the LP tokens can be of different types (aTokens, cTokens, Univ2 LP Tokens, ERC721…), making it possible to mint GHO with different kinds of collateral. For our example, the liquidity (ETH) is deposited on Gnosis chain. On Gnosis Chain, the user locks the tokenized collateral in a smart contract called “Vault”. Having the tokens locked in a Vault is necessary to ensure the position is over-collateralized. A storage proof of the deposit is created off-chain. The Vault authorizes the minting of GHO in Ethereum. Hashi is a chain-agnostic oracle aggregator that allows for additive security. It has adapters for several cross-chain rails (including oracles, bridges, CCIPs, zk light clients) so that multiple security mechanisms can be used to verify cross-chain transfers. On Gnosis, Hashi is used to propagate block headers to Ethereum. On Ethereum, Hashi waits for the majority of the oracles to reach a consensus on the block in which there is the minting authorization. FRAGOLA then verifies the storage proof against the block header propagated by Hashi, and, if matching, proceeds to mint GHO on Ethereum. It can easily be configured to use simple message passing via the underlying bridges rather than block header-based verification, if desired.

Conversely, in the burning flow, the burn function is called through the facilitator (which creates the storage proof of the request), the authorization request is sent to Hashi and the oracles verify the existence of the request on Ethereum and propagate the block headers to Gnosis. On Gnosis, once consensus is reached, the Vault verifies the storage proofs against the block header propagated by Hashi and, if matching, GHO is burned and the relative LP tokens are given back to the user.

In the case of liquidation, the lifecycle is very similar to the one for burning GHO. The only difference is that the liquidated user’s LP tokens are transferred to the liquidator.

### How it's Made

In building this project, we primarily focused on enabling GHO minting depositing collateral on different blockchains. The core technology we used is a combination of Hashi's block header propagation and the Chainlink Cross-Chain Interoperability Protocol (CCIP) through the Hashi adapter. This setup is crucial for verifying storage proofs, which confirm that a user has deposited the necessary collateral in the vault for GHO minting.

The project's architecture is centred around the FRAGOLA base framework and the reference facilitator, which mints GHO. The facilitator operates on a verification mechanism that uses storage proofs. These proofs are cross-checked against a block propagated through Hashi, ensuring the collateral deposit in the Vault is valid and on record. This approach simplifies the user experience, as the collateral is managed on various chains, not just Ethereum.

The most notable technical aspect of our project is how it simplifies the GHO minting process. By using cross-chain data verification, we've managed to facilitate GHO minting more directly, reducing complexity for users who wish to use assets from different blockchains as collateral. This directly addresses a key limitation in the current GHO framework, expanding its usability across diverse blockchain environments.

In summary, our project’s approach relies on efficiently utilising existing technologies like Hashi and Chainlink CCIP for a specific, practical purpose in the DeFi space – facilitating straightforward, cross-chain interactions for GHO minting.

### Resource

1. Live demo: [https://gho-fragola.eth.limo/](https://gho-fragola.eth.limo/)
2. Source Code: [https://github.com/crosschain-alliance/gho-fragola](https://github.com/crosschain-alliance/gho-fragola)
3. UI: [https://github.com/crosschain-alliance/gho-fragola-ui](https://github.com/crosschain-alliance/gho-fragola-ui)
4. Video demo: [https://ethglobal.com/showcase/fragola-oatod](https://ethglobal.com/showcase/fragola-oatod)

