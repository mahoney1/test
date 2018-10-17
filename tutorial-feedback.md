**Item 1**

Under 'Enabling development mode'

The instructions should say as below:  (ie correct the .yml filename in 2 places, and basic-network location is from `fabric-samples`)

The `basic-network` sample is located under the `basic-network` directory of `fabric-samples` clone. The `docker-compose.yml` file must be edited to enable development mode. 

(FYI Matthew - ie the docker-compose.yml found under 'commercial-paper/infrastructure' is old - doesn't reflect what you'd expect (and you can cross check this with the docker-compose.yml in 'fabric-samples/basic-network' fyi which DOES reflect what you would expect) - just pointing this out - you want them to always get the edition from github.com/fabric-samples not infrastructure (in your tutorial)


**Item 2**

Under 'starting the Hyperledger Fabric basic network ' 

 - you don't need step 1 (you're already in that directory)
 - Step 3 - the command is  ../monitordocker.sh net_basic  (no underscore required)


**Item 3**

Under 'Starting the smart contract locally'

Navigate to the contracts/javascript directory under the `commercialpaper' folder

bullet 2: npm install 

the package.json in the .tgz downloaded has local "file:" libraries, they should get them from NPM (ie 1.4 snapshot editions in particular, matching the application below) - I've replaced with snapshots (below)  
 
```
"fabric-contract-api": "unstable",
        "fabric-shim": "unstable",
        etc
 ```
 
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

Bullet 4:  you need to replace ':' with '.' (fullstop) before `instantiate` -  as in below (also: see earlier comment about removing leading $ too)


`$ docker exec cli peer chaincode instantiate -n papernet -v 0 -l node -c '{"Args":["org.papernet.commercialpaper:instantiate"]}' -C papernet`

(as it exists now - becomes):

`$ docker exec cli peer chaincode instantiate -n papernet -v 0 -l node -c '{"Args":["org.papernet.commercialpaper.instantiate"]}' -C papernet`

**Item 5**

Under 'Setting up the client application'

Bullet 3: setting up local idwallet - you require an extra '../' in `application.js` to get the correct relative path to the crypto-config directory (this was (correctly) the case in a previous edition of addToWallet.js FYI )

`const fixtures = path.resolve(__dirname,'../../infrastructure/basic-network');`

Other than that, all good!


