# Introduction

## What is Hashi

Hashi is an EVM Hash Oracle Aggregator, designed to facilitate [a principled approach to cross-chain bridge security.](https://ethresear.ch/t/hashi-a-principled-approach-to-bridges/14725)

The primary insight being that the vast majority of bridge-related security incidents could have had minimal impact if the systems relying on them had built in some redundancy. In other words, it's much more secure to require messages be validated by multiple independent mechanisms, rather than by just one.

Hashi aims to create “additive security” to cross-chain messages by aggregating block headers from different sources. A block header will be considered valid only when a number of block sources (oracles) above a certain threshold report the same result. Hashi is the first step towards a principled approached to bridges and will play a key role in the Gnosis Chain interoperability roadmap.

Apart from block header, Hashi also provides arbitrary message relaying option. Check out [Message Dispatching with Yaho and Yaru ](v0.1/application.md)for more details.

### Oracles

Oracles consists of the bridge solutions available in the market, such as AMB, Telepathy, Dendreth, etc. To provide an universal interface for Hashi, an adapter contract is designed specifically for each oracle. Some of the oracles require header reporter to report block header in certain slot.

Existing oracle adapters:

* [Telepathy ZK light client](https://docs.telepathy.xyz/)
* [Dendreth ZK light client](https://github.com/metacraft-labs/DendrETH)
* [Gnosis AMB bridge](https://docs.gnosischain.com/bridges/tokenbridge/amb-bridge)
* [Sygma protocol](https://medium.com/buildwithsygma)
* [Celer](https://cbridge.celer.network/1/100/SOS)
* [LayerZero](https://layerzero.network/)
* [Chainlink CCIP](https://docs.chain.link/ccip)
* [Connext](https://www.connext.network/)
* [Hyperlane](https://www.hyperlane.xyz/)
* [Optimism bridge](https://app.optimism.io/bridge/deposit)
* [Wormhole](https://docs.wormhole.com/wormhole/)
* [Zetachain](https://www.zetachain.com/)





### Resources

* Cross Chain Alliance: [https://github.com/crosschain-alliance](https://github.com/crosschain-alliance)
* Repository: [github.com/gnosis/hashi](https://github.com/gnosis/hashi)
* Intro discussion: [ethresear.ch/t/hashi-a-principled-approach-to-bridges](https://ethresear.ch/t/hashi-a-principled-approach-to-bridges/14725)
