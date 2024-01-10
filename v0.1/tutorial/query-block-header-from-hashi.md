# Query block header from Hashi

To query a block header from Hashi, navigate to [ShoyuBashi](https://gnosisscan.io/address/0x31a8e89d6f98454d38c03eca3dc543f6581d607c#readContract) contract on gnosisscan.\
Click `getThresholdHash` function.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Enter the domain (chain ID, i.e. 5 for Goerli) and id (block number from domain) for the block you want to query.\
The result will be the block hash that has passed the threshold from adapters (i.e. 2 out of 4 in Goerli <-> GC). Not every block will be stored by the adapters and reach threshold, if no threshold has been reached, the result will be bytes32(0). You can utilize [Adapters Dashboard](https://hashiadapters-dashboard-tvw47.ondigitalocean.app/) to check which block has been stored.

You may double check if the block hash is correct by searching your block hash in [Adapters Dashboard](https://hashiadapters-dashboard-tvw47.ondigitalocean.app/).

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>getThresholdHash</p></figcaption></figure>
