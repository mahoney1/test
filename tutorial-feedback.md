**Item 1**

Under 'Enabling development mode'

The instructions should say as below:  (ie correct the .yml filename in 2 places, and clarifying that the basic-network location is from `fabric-samples`)

The `basic-network` sample is located under the `basic-network` directory of `fabric-samples` clone. The `docker-compose.yml` file must be edited to enable development mode. UNLESS you explicitly mean to use the `docker-compose.yml` provided under `infrastructure/basic-network` of the gerrit .tgz file (in which case, the `fabric-chaincode-node` 'error' I mention below, may not occur).

As suggested - ignored basic-network stuff in gerrit tgz and used edition from github.com/fabric-samples - but 


**Item 2**

Under 'starting the Hyperledger Fabric basic network ' 

 - you don't need step 1 (you're already in that directory)
 - Step 3 - the command is  ../monitordocker.sh net_basic  (no underscore required)


**Item 3**

Under 'Starting the smart contract locally'

Navigate to the contracts/javascript directory under the `commercialpaper` folder


bullet 3 - Should remove the (leading) extra dollar symbol ? (I know you're showing its 'command line' but I think it may confuse, would suggest to remove (especially as previous command is not using that 'style')


This DOESN'T work for me: 

`$(npm bin)/fabric-chaincode-node start --peer.address=localhost:7052 --chaincode-id-name=papernet:0`

you get:
ERROR [lib/chaincode.js] uncaughtException: Illegal value for namevalue element of type string: undefined (not a string) 

This DOES work for me:

`CORE_CHAINCODE_ID_NAME=papernet:0 $(npm bin)/fabric-chaincode-node start --peer.address=localhost:7052`

- as shown in https://github.com/mahoney1/docs/blob/master/Running-Commercial-Paper-Contract.md

**Item 4**

Under:  'Installing and instantiating the smart contract'

General - remove leading '$' symbols for command line and keep consistent throughout

Bullet 2 - not required (CLI container already been started by the start.sh script earlier - ie remove it)

`docker-compose -f ./docker-compose.yml up -d cli`

Bullet 4:  (see earlier comment about removing leading $ too)


**Item 5**

Under 'Setting up the client application'

Bullet 3: setting up local idwallet - you require an extra '../' in `application.js` to get the correct relative path to the crypto-config directory (this was (correctly) the case in a previous edition of addToWallet.js FYI )

`const fixtures = path.resolve(__dirname,'../../infrastructure/basic-network');`

Other than that, all good!


