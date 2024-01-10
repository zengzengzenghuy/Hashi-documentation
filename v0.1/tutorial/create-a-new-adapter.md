# Create a new adapter



Integrating a new adapter into Hashi enhances the security of the overall architecture.

There are two contracts need to be created:

1. Header Reporter contract on source chain
2. Adapter contract on destination chain

#### Header Reporter contract

Header reporter contract will be called by user or bot to pass block header of certain block(s) from source chain.

1. Create a public function called `reportHeaders`.
2. In the `reportHeaders` function, define the bridging logic and make sure three requirements are met:\
   i. Call `storeBlockHeaders()` from HeaderStorage contract\
   ii. Bridge block headers returned from the above step with encoded data that will call the function to store block headers in adapter contract on destination chain\
   iii. Emit `HeaderReported()` event.
3. ```
   function reportHeaders(uint256[] memory blockNumbers) public {
       // return block header(s) of block number(s)
       bytes32[] memory blockHeaders = headerStorage.storeBlockHeaders(blockNumbers)
       // pass block header to bridge
       Bridge.passMessage(blockHeaders, storeHashesEncodedData)
       // emit event for each block header
       for (uint i = 0; i < blockNumbers.length; i++){
           emit HeaderReported(address(this),blockNumbers[i],blockHeaders[i])
       }
   }
   ```



#### Adapter contract

Create a new adapter contract for your oracle that can store block headers and fetched by Hashi.

1. Inherit `OracleAdapter.sol` and `BlockHashOracleAdapter.sol`.
2. Create a `storeHash` function that will call `_storeHash` with the following parameter:\
   `domain`(uint256): Identifier for the domain, i.e. chain id.\
   `id`(uint256): Identifier for the ID of the hash, i.e. block number.\
   `hash`(bytes3): block hash(block header).\
   Calling `_storeHash` willl emit `Hashstored(id,hash)` event.
3. ```

   contract Adapter is OracleAdapter, BlockHashOracleAdapter {
       function storeHashes(uint256[] memory ids, bytes32[] memory hashes)public{
           for(uint256 i = 0; i < ids.length; i++){
               _storeHash(sourceChainID, ids[i], hashes[i])
           }
       }
   }

   ```
