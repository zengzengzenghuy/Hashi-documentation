# Message Dispatching using Yaru and Yaho

For relaying arbitrary message between Ethereum/Goerli and Gnosis Chain, several contracts need to be deployed on top of the [current AMB deployments](https://docs.gnosischain.com/bridges/hashi/#goerli---gnosis-chain): Yaho, Yaru, AMB Message Relay, AMB Adapter, and Hashi. The addresses from this article are for Goerli <-> Gnosis Chain.

## Prerequisite

For Goerli->Gnosis Direction:

1. Deploy [Yaho](https://github.com/gnosis/hashi/blob/main/packages/evm/contracts/Yaho.sol) on Goerli, and [Yaru](https://github.com/gnosis/hashi/blob/main/packages/evm/contracts/Yaru.sol) on Gnosis Chain, with constructor `_hashi: Hashi address on Gnosis Chain, _yaho: Yaho address on Goerli, _chainId: 5`.
2. Optional: Deploy [Hashi Module](https://github.com/gnosis/hashi/blob/main/packages/evm/contracts/zodiac/HashiModule.sol) for Safe that will be called by Yaru, i.e. Safe on Gnosis Chain.
3. Deploy [AMB Message Relay](https://github.com/gnosis/hashi/blob/main/packages/evm/contracts/adapters/AMB/AMBMessageRelayer.sol) on Goerli, with constructor `_amb: AMB address on Goerli, _yaho: Yaho address on Goerli`.
4. Deploy [AMB Adapter](https://github.com/gnosis/hashi/blob/main/packages/evm/contracts/adapters/AMB/AMBAdapter.sol) on Gnosis Chain, with constructor `_amb: AMB address on Gnosis Chain, _reporter: AMB Message Relay on Goerli, _chainId: bytes32(5)` .

> &#x20;In the current deployment, `gas` is added as a parameter in `dispatchMessagesToAdapters()` to justify the [amb's gas requirement](https://github.com/gnosischain/tokenbridge-contracts/blob/master/contracts/upgradeable\_contracts/arbitrary\_message/MessageDelivery.sol#L40). It is not added into the official hashi code at the moment as [it might raise incompatibility issue for other adapters](https://github.com/gnosis/hashi/pull/19#discussion\_r1278769527). Please be aware that these contracts might be changed anytime.

## Workflow: Ethereum -> Gnosis Chain

* User calls `Yaho.dispatchMessagesToAdapters`. [Example](https://goerli.etherscan.io/tx/0x659145934c8eeff82a574d2e8bcb1cf8edd67bef24b69c22906ddfd250287f7f)
* Yaho calls AMB Message Relay `relayMessages` which will eventually calls `storeHashes` from AMB Adapter on Gnosis Chain.
* AMB Message Relay calls AMB `requireToPassMessage`, and the message will be relayed to AMB on Gnosis Chain.
* Once the AMB Adapter `storeHashes` is called, `MessageDispatched` event will be emitted and user need `messageId` data from the event. [Example](https://gnosisscan.io/tx/0x3c1355dea3c1afc3d01b3d9667a22c3d0d2dbe1c2c9a5dc95a7fa0625960468c/advanced#eventlog)
* User calls `Yaru.executeMessages` with `messageId` and `message` as parameters. [Example](https://gnosisscan.io/tx/0x81ba4eba5108bfa974a168d2aa533f8b682a99e3f4bbbf9b7e7cd1c1d994f17b)

![](https://hackmd.io/\_uploads/Bk0Gd1Bi2.png)

#### Initiate Transaction

1. Create function calldata for the contract you want to call on Gnosis Chain.

```
const calldata = contractInterface.encodeFunctionData(function_name, parameters)
```

2. Optional: Create transaction calldata for Hashi Module.

This step is only needed when [Safe](https://safe.global/) is used.&#x20;

```
const txData = hashi_module_interface.encodeFunctionData("executeTransaction", [
      ${contract_to_call_on_Gnosis_Chain},
      0,
      calldata,
      0,
]);
```

3. Create message with format below: With Safe:

```
const message =
{
    to: ${Hashi Module address},
    toChainId: 100 // Gnosis Chain,
    data: ${tx_data}
}
```

Without Safe:

```
const message =
{
    to: ${Contract address on Gnosis Chain},
    toChainId: 100 // Gnosis Chain,
    data: ${calldata}
}
```

4. Call `Yaho.dispatchMessagesToAdapters([message],[AMB_Message_Relay_Address],[AMB_Adapter_Address])` using Safe or EOA.
5. Once the transaction is created, you need to collect `messsage Id` from [`MessageDispatched` event](https://github.com/gnosis/hashi/blob/main/packages/evm/contracts/Yaho.sol#L27) emitted from Yaho.

#### Claim Transaction

After the message is relayed to Gnosis Chain by AMB bridge, you can proceed to claim your transaction. Make sure that your message is stored by checking if the AMB Adapter contract emits `HashStored` event with the correct `message Id`.

1. Call `Yaru.executeMessages([message],[messageId],[Safe_from_Goerli or EOA from Goerli that calls Yaho],[AMB_Adapter_Address on Gnosis Chain])`

#### Implement your hook

For a successful message execution to happen, the contract specified as message recipient on the destination chain must implement a hook to process correctly the `message.data`. The hook must revert unless the `msg.sender` matches the expected `Yaru` contract, `Yaru.sender()` matches the expected sender, and `Yaru.adapters()` matches the desired set of oracles that must have agreed on the reported hash.

| Contract                      | Address                                                                                                                           | Chain  |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------ |
| Yaho                          | [0xC1289f49A1972E2C359a0647707a74E24ce73F8b](https://goerli.etherscan.io/address/0xc1289f49a1972e2c359a0647707a74e24ce73f8b#code) | Goerli |
| AMB Message Relay             | [0x2433c921152B3dE133F96762a9b359D02dB34c93](https://goerli.etherscan.io/address/0x2433c921152b3de133f96762a9b359d02db34c93#code) | Goerli |
| AMB Adapter for Message Relay | [0xbA5B3f0643582E75AF252e7631dE62c046970167](https://gnosisscan.io/address/0xbA5B3f0643582E75AF252e7631dE62c046970167#code)       | GC     |
| Yaru                          | [0x171C1161bCde7adB32a9Ca92c412d39bE6F97C59](https://gnosisscan.io/address/0x171C1161bCde7adB32a9Ca92c412d39bE6F97C59#code)       | GC     |

## Workflow: Gnosis Chain -> Ethereum

![](https://hackmd.io/\_uploads/HyZ7u1rj2.png)

* User calls `Yaho.dispatchMessagesToAdapters`. [Example](https://gnosis.blockscout.com/tx/0x2621ab8e7666645e347a017032e1dbb65459c13a9cff152ca8e35dcbb46eb699?tab=index)
* Yaho calls AMB Message Relay `relayMessages` which will eventually calls `storeHashes` from AMB Adapter on Gnosis Chain.
* AMB Message Relay calls AMB `requireToPassMessage`, and `UserRequestsForSignature` event will be emitted.
* User calls AMB Helper contract's `getSignature` with the `encodedData` as parameters obtained from step3, the function will return signature.[Tutorial](https://docs.gnosischain.com/bridges/tutorials/using-amb#submitting-amb-confirmations-manually)
* User calls AMB on Goerli's `executeSignature` with `encodedData` and `signature` as parameters. The `storeHashes` from AMB Adapter will be called, `MessageDispatched` event will be emitted and user need `messageId` data from the event. [Example](https://goerli.etherscan.io/tx/0xb5d71b13a07e206ad553ac0695e3e26c3ef19a97c561a0d7e284eae9b6fb2597)
* User calls `Yaru.executeMessages` with `messageId` and `message` as parameters. [Example](https://goerli.etherscan.io/tx/0xd08ab3ca71bc4c349cb4f9ddf93b4ffb478836f88f57a177ff64fe45d33b4b15)

| Contract                      | Address                                                                                                                                   | Chain  |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| Yaho                          | [0x619360550f337bdA5A3A4709fEff2140c9577593](https://gnosisscan.io/address/0x619360550f337bdA5A3A4709fEff2140c9577593)                    | GC     |
| AMB Message Relay             | [0x6796E267aa898464417BcD4D81690242bD39D68e](https://gnosisscan.io/address/0x6796E267aa898464417BcD4D81690242bD39D68e)                    | GC     |
| AMB Helper                    | [0xc7118ecF785871F08Feff5DE07d5F884b6199477](https://gnosisscan.io/address/0xc7118ecF785871F08Feff5DE07d5F884b6199477#readContract)       | GC     |
| AMB Adapter for Message Relay | [0xBB448c841b1Ce6636aa804Bf97B1eBf9DF940f08](https://goerli.etherscan.io/address/0xBB448c841b1Ce6636aa804Bf97B1eBf9DF940f08#code)         | Goerli |
| Yaru                          | [0x6Cdfa83b5b3e4eBc603fF12B8aAD893f7d7231d8](https://goerli.etherscan.io/address/0x15d5678FF44883444264f7947c1c5C31d07b4482#readContract) | Goerli |
| Hashi                         | [0x6a948572432818DeBbb04A0b82b6c12ec5Ca15B5](https://goerli.etherscan.io/address/0x6a948572432818DeBbb04A0b82b6c12ec5Ca15B5)              | Goerli |
