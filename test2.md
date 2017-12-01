This Wiki is a knowledge base to help Composer folks having issues we class as  "_we've seen something similar before"_ :-)  :1st_place_medal:  :2nd_place_medal: :3rd_place_medal: :rocket: 

Its grouped by Topic. Simply click on one - there's a brief description and a list of related issues. Click the 'back to top' link to return at any time to the topic index.

Each topic area has links to suggested solutions (you can open these in a new window) sourced from:

* **Documented Resolutions** 
* **Stack Overflow threads**
* **Rocketchat threads**


If you still have an issue,  see :link:  [here ](#issue)    


<a name="top"></a>

### :basecamp:  TOPIC INDEX   :basecamp:

Colons can be used to align columns.

| Topic       | Topic         | Topic  |
| ------------- |:-------------:| -----:|
| :link:  [Blockchain Recap](#recap)   | :link:  [Business Network Cards](#bizcards)| :link:  [Issues](#issue) |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
| col 4 is      | centered      |   $12 |


LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT
LOTS OF TEXT



<a name="recap"></a>
### :information_source:  Blockchain Recap

There are two place which "store" data in Hyperledger Fabric:

    * the ledger
    * the state database ('World state')

The ledger is the actual **"blockchain"**. It is a file-based ledger which stores serialized blocks. Each block has one or more transactions. Each transaction contains a 'read-write set' which modifies one or more key/value pairs. The ledger is the definitive source of data and is immutable.

The **state database** (or 'World State') holds the last known committed value for any given key - an indexed view into the chain’s transaction log. It is populated when each peer validates and commits a transaction. The state database can always be rebuilt from re-processing the ledger (ie replaying the transactions that led to that state). There are currently two options for the state database: an embedded LevelDB or an external CouchDB.

As an aside, if you are familiar with Hyperledger Fabric 'channels', there is a separate ledger for each channel.

The chain is a transaction log, structured as hash-linked blocks, where each block contains a sequence of _N_ transactions. The block header includes a hash of the block’s transactions, as well as a hash of the prior block’s header. In this way, all transactions on the ledger are sequenced and cryptographically linked together.

The state database is simply an indexed view into the chain’s transaction log, it can therefore be regenerated from the chain at any time.

Source: http://hyperledger-fabric.readthedocs.io/en/release/ledger.html

:card_index: [back to top](#top)

<a name="bizcards"></a>

### :information_source:  Business Network Cards

A Business Network Card provides the means to connect to a Composer business network which runs in a Composer runtime container. It is only possible to access a Composer business network through a valid Business Network Card. It consists of a connection profile, some metadata for the identity using it, and ultimately, a set of credentials (certificate/private key). An identity can have one or more cards, to connect to one or more business networks.

The benefits are that once you export a card, it is a portable card to connect to the Composer business network running on the blockchain network, so can be issued/given to someone (usually that real identity in that Organisation) to then transact on business network, on the blockchain network.

:card_index: [back to top](#top)

<a name="issue"></a>

### :information_source:  Creating an Issue for a Composer problem

The best place to get it answered is actually on [Stack Overflow](https://stackoverflow.com/questions/tagged/hyperledger-composer). 

You simply create a problem with the tag 'hyperledger-composer'  and provide: 

* The problem title
* background context
* Composer version
* Operating System/version
* what steps you took to reach your current situation or state
* any supporting logs, script or model files - so as to help troubleshoot or identify where the problem lies. 

Thank you

:card_index: [back to top](#top)
