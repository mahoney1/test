This Wiki is a knowledge base to help Composer folks having issues we class as  "_we've seen something similar before"_ :-)  :1st_place_medal:  :2nd_place_medal: :3rd_place_medal: :rocket: :recycle: :sos: :face_with_head_bandage:. The idea is we help steer you towards a resolution :ok_man: 

Using is easy-peasy :last_quarter_moon_with_face: - find your topic area, and check out related issues. 


<a name="top"></a>
***
### :basecamp:  TOPIC INDEX   :basecamp:

| ~ [**Blockchain Recap**](#recap) | ~ [**Business Network Cards**](#bizcards) | ~ [**Creating issues**](#issue) | ~ [**Filters**](#filters) 
| :---------------------- | :-----------------------| :----------------------- | :-------------------- 
| ~ [**Event Hub Problems**](#event) | ~ [**Help with IBM Cloud / Kubernetes**](#cloud) | ~ [**Multi Org Setup**](#multiorg) | ~ [**Sample Networks**](#samples) 
| ~ [**Upgrading Composer**](#upgrade) | ~ [**Runtime Install Help**](#runtime-install) | ~ [**Topic Name**](#bizcards) | ~ [**Topic Name**](#samples) | ~ [**Topic Name**](#bizcards) 

***
Each topic area has links to suggested solutions (you can open these in a new window) sourced from:

* **Documented Resolutions** 
* **Stack Overflow threads**
* **Rocketchat threads**


If you still have an issue,  see :link:  [here ](#issue)    


<a name="recap"></a>

### :information_source:  Blockchain Recap

There are two place which "store" data in Hyperledger Fabric (the underlying blockchain infrastructure used by Composer):

    * the ledger
    * the state database ('World state')

The ledger is the actual **"blockchain"**. It is a file-based ledger which stores serialized blocks. Each block has one or more transactions. Each transaction contains a 'read-write set' which modifies one or more key/value pairs. The ledger is the definitive source of data and is immutable.

The **state database** (or 'World State') holds the last known committed value for any given key - an indexed view into the chain’s transaction log. It is populated when each peer validates and commits a transaction. The state database can always be rebuilt from re-processing the ledger (ie replaying the transactions that led to that state). There are currently two options for the state database: an embedded LevelDB or an external CouchDB.

As an aside, if you are familiar with Hyperledger Fabric 'channels', there is a separate ledger for each channel.

The chain is a transaction log, structured as hash-linked blocks, where each block contains a sequence of _N_ transactions. The block header includes a hash of the block’s transactions, as well as a hash of the prior block’s header. In this way, all transactions on the ledger are sequenced and cryptographically linked together.

The state database is simply an indexed view into the chain’s transaction log, it can therefore be regenerated from the chain at any time.

Source: http://hyperledger-fabric.readthedocs.io/en/release/ledger.html

#### :card_index: [back to base camp :camping: ](#top)  

<a name="bizcards"></a>

### :information_source:  Business Network Cards

A Business Network Card provides the means to connect to a Composer business network which runs in a Composer runtime container. It is only possible to access a Composer business network through a valid Business Network Card. It consists of a connection profile, some metadata for the identity using it, and ultimately, a set of credentials (certificate/private key). An identity (linked to a participant in Composer) can have one or more cards, to connect to one or more business networks.

The benefits are that once you export a card, it is a portable card. So it can be issued to a new user/given to someone (usually that real identity in that Organisation) to then connect and transact on business network, on the blockchain network. Yes, of course - they should be handled with care. We recommend that you only send identity cards that have been encrypted.

Best practices with cards:

When you create a card **file** on disk (eg. user4.card, networkAdmin.card) for the first time (eg, via CLI, a Playground export or using the APIs) - the file will most likely just have a single-use enrolment id and secret (as opposed to the certificate/key combo when its gets used later on). To populate the **card store** (ie in your or the other user's credentials vault)  with the certificate/key (from the Certificate of Authority or CA server) - follow the sequence below:

* Import the card **file** (eg. via Playground, or use the CLI eg. composer card import --file networkAdmin.card` or `composer card import -f user4.card`) into your CARD STORE. The only means to import a business network card into a user's cardstore is to use either the CLI, Playground or the Composer APIs - to actually import it.

* Connect to the business network using that **imported** card in Playground (or ping it via command line eg. composer network ping -c admin@tutorial-network` or `composer network ping -c user4@trade-network` - notice it refers to the business network) so that it gets 'used' and therefore it requests credentials (certificate/key combo) from the CA server and imports them into the card in the card store.



Note: If you export a Business Network Card that has never been used, it will contain just the one-time enrollment ID and enrollment secret only. The first time that exported card gets used,  it connects with the one-time secret and the identity's certificate/key combo is downloaded to the user's local wallet(credentials vault) - thus the enrol secret is no longer valid. If you then attempt to re-use the same card file elsewhere with only the secret still in it, it registers a different identity and this can be a cause of major pain for many (this is how security and certificates work, its not a 'Composer' thing).

More technical:

Together, both `cards` and `client-data` directories in $HOME/.composer are an integral part of the whole card structure. `client-data` is where the hlfv1 composer connector will store the identity credentials for a card (either by importing or when it enrolls an identity). eg for `admin@tutorial-network` it will have the usual client crypto artifacts eg. 'admin' xx-priv, xx-pub . Meanwhile, the imported cards get persisted to the `cards` subdirectory. If a card contains only an enrollment secret, this will get used on 'first use' to obtain the certificate, which will get stored initially only in`client-data`. It includes the certificate and private key for the user identity (which come either directly from the imported business network card, or are retrieved from the CA using the enrollment secret). If the card is subsequently exported then any certificate data gets retrieved from `client-data` and added to the exported card. If you like, one subdirectory acts as the card store including business network metadata elements and the other (client-data) is the working cache/store for client credentials.. 

#### :card_index: [back to base camp :camping: ](#top)   


<a name="event"></a>


### :information_source:  Event Hub Issues

Event Hub issues can vary in the kind of error reported - for example 'Unhandled Promise Rejection' is a case in point.
See below for suggested resolutions and follow the link in a new window.

* Have you checked the listening Fabric Event Hub is actually up and running (listening for events) ?
* Are you configuring a Multi-Org environment ? EventURLs are only defined for 'this' Org's peers (and not other Org's peers)

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| Unhandled Promise Rejections  | Your connection profile info (in your card) has been incorrectly defined
| #1 | See https://chat.hyperledger.org/channel/composer?msg=wnz6YZpvFMdCrgHZJ
| #2| https://stackoverflow.com/questions/46270080/node8232-unhandledpromiserejectionwarning-error-could-not-find-chaincode-wit

#### :card_index: [back to base camp :camping: ](#top)   


<a name="cloud"></a>


### :information_source:  IBM Cloud / Kubernetes Support / IBM Container service


| Message encountered | Resolution 
| :---------------------- | :-----------------------
| IBM Sandbox / Kubernetes support  |for support with your particular environment on IBM Cloud you should go to this page https://console.bluemix.net/docs/support/index.html#contacting-support


#### :card_index: [back to base camp :camping: ](#top)  


<a name="runtime-install"></a>

### :information_source:  Runtime install errors

The following are a selection of answers, to help understand what you may be encountering:

| Message encountered | Resolution 
| :---------------------- | :-----------------------
|:infornation_source: Error: No valid responses from any peers  | see below
| Error: Error: Endpoint read failed |  This is likely to be a connection (.json) config issue -  it needs to be configured for TLS (or non-TLS) communications 



#### :card_index: [back to base camp :camping: ](#top)  

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

#### :card_index: [back to base camp :camping: ](#top)   
