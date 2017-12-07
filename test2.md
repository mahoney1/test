## :sos: :eyes: COMPOSER KNOWLEDGE BASE :eyes: :sos:

This Wiki is a knowledge base to help Composer folks having issues we class as  "_we've seen something similar before"_ :-)  :1st_place_medal:  :2nd_place_medal: :3rd_place_medal: :rocket: :recycle: :sos: :face_with_head_bandage:. The idea is we help steer you towards a resolution :ok_man: :thumbsup:

Using is easy-peasy :last_quarter_moon_with_face: - find your  [**TOPIC AREA**](#top), and check out related answers/resolutions. If you don't see an answer - you can 1) check our [**Open Composer Issues**](https://github.com/hyperledger/composer/issues) list and 2) search [**Stack Overflow list**](https://stackoverflow.com/questions/tagged/hyperledger-composer) for answers. Failing that, go to [S/O]((https://stackoverflow.com/questions/tagged/hyperledger-composer)) to create a problem report - see guidance on ~ [**Creating Stack Overflow issues**](#issue) below

Our [**Documentation**](https://hyperledger.github.io/composer) page should be the 'first port of call'  :ship:  :boat: to understand concepts/examples/usage. Also check out the [**Reference**](https://hyperledger.github.io/composer/reference/reference-index.html) :books: :books:  section to understand how to use the CLI/ACLs/APIs and a glossary of terms used. :relieved:
Our [**Tutorials**](https://hyperledger.github.io/composer/tutorials/tutorials.html) should be used to get going - and consolidate your learning :smiley_cat:

<a name="top"></a>
***
###  :o: TOPIC INDEX  :o:

| ~ [**Blockchain Recap**](#recap) | ~ [**ACLs**](#acls) | ~ [**Business Network Cards**](#bizcards)  | hfh
| :---------------------- | :-----------------------| :----------------------- | :-------------------- 
| ~ [**Event Hub Problems**](#event) | ~ [**Filters**](#filters) | ~ [**Help with IBM Cloud / Kubernetes**](#ibmcloud) | [**Historian Queries **](#historian)
| ~ [**Multi Org Setup**](#multiorg) | ~ [**Passport Strategies**](#passport-strategy) | ~ [**Queries**](#queries)  | ~ [**Sample Networks**](#samples) 
| ~ [**Upgrading Composer**](#upgrade) | ~ [**Runtime Install Help**](#runtime-install) | ~ [**Topic Name**](#bizcards) | ~ [**Topic Name**](#samples) 
| ~ [**Topic Name**](#bizcards) | test | test | test

***
Each topic area has links to suggested solutions (you can open these in a new window) sourced from:

* **Documented Resolutions** 
* **Stack Overflow threads**
* **Rocketchat threads**


If you still have an issue,  see :link:  [here ](#issue)    

<a name="acls"></a>


### :information_source:  Access Control Lists (ACLs)

The following are a selection of answers, to help understand what you may be encountering:

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| ACL that target > 1 transaction  | see Rocketchat thread for target definition [here](https://chat.hyperledger.org/channel/composer?msg=HaxJg3sHESrPCvzcd)


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

jump to [review card FAQs](#cardfaq) to see more on current errors & resolutions

A Business Network Card provides the means to connect to a Composer business network which runs in a Composer runtime container. It is only possible to access a Composer business network through a valid Business Network Card. It consists of a connection profile, some metadata for the identity using it, and ultimately, a set of credentials (certificate/private key). An identity (linked to a participant in Composer) can have one or more cards, to connect to one or more business networks.

The benefits are that once you export a card, it is a portable card. So it can be issued to a new user/given to someone (usually that real identity in that Organisation) to then connect and transact on business network, on the blockchain network. Yes, of course - they should be handled with care. We recommend that you only send identity cards that have been encrypted. See separate note on use of cards with a multi-user REST server environment.

Best practices with cards:

When you create a card **file** (first time) on disk (such as: user4.card, networkAdmin.card) - eg. via CLI, a Playground export or using the APIs) - the file will most likely just have a single-use enrolment id and secret (as opposed to the certificate/key combo present,  when a previously imported business network card gets used). The next task is to import the**new** BN card into the user's wallet (whether, say, its 'admin' or 'user4') in the **user's wallet** - then 'use' it - see summary steps below:

* Import the card **file** (eg. via Playground, or use the CLI eg. composer card import --file networkAdmin.card` or `composer card import -f user4.card`)` into your wallet (in the CARD STORE). The only means to import a business network card into a user's wallet in the cardstore (wherever that may be)  is to use either the CLI, Playground or the Composer APIs.

* Connect to the business network using that **imported** card either via:  Playground, CLI or using the APIs. From the CLI you can ping it via command line eg. `composer network ping -c admin@tutorial-network` or `composer network ping -c user4@trade-network` so that it gets 'used' and therefore it requests the identity's credentials (certificate/key combo) from the CA server and imports them into the user's wallet in the card store. Once you've done this, then you can export it to a card file (if you want to share).


Important: If you export a Business Network Card that **has never been used** (eg in Playground for example), it will still just contain the one-time enrollment ID and enrollment secret only. The first time that exported card gets used,  it connects with the one-time secret and the identity's certificate/key combo is downloaded to the user's local wallet. You should then export this card to a new file, if you plan on using that user's business network card elsewhere  - ie don't re-use the 'old' / original card file (use the new file !) - otherwise re-using the 'old' will just register a different identity (because it still has an enrol id remember) and this can be a cause of major pain for many (this is how security, identity and certificates work, its not a 'Composer' thing).

More technical:

Together, both `cards` and `client-data` directories in $HOME/.composer are an integral part of the whole wallet structure in the card store (credentials vault). `client-data` is where the hlfv1 composer connector will store the identity credentials for a card (either by importing or when it enrolls an identity). eg for `admin@tutorial-network` it will have the usual client crypto artifacts eg. for a user admin it has : 'admin' xx-priv, xx-pub . Meanwhile,an imported card get persisted to the `cards` subdirectory. If that card contains only an enrollment secret, on 'first use' it will retrieve the cert/key combo from the CA server and get stored in`client-data`. If the card is subsequently exported then the certificate/key gets retrieved from `client-data` too and added to the exported card. 

<a name="cardfaq"></a>

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| Authorization errors Playground local | See **answer** at https://stackoverflow.com/questions/47617442/authorization-failure-when-creating-new-business-network-in-local-playground

#### :card_index: [back to base camp :camping: ](#top)   


<a name="byfn"></a>

### :information_source:  BYFN Issues

We only use the Fabric BYFN network as a means to demonstrate the Multi-Org tutorial. See [Multi-Org](#multiorg) for more info.


#### :card_index: [back to base camp :camping: ](#top)  
<a name="event"></a>


### :information_source:  Event Hub Issues

Event Hub issues can vary in the kind of error reported - for example 'Unhandled Promise Rejection' is a case in point.
See below for suggested resolutions and follow the link in a new window.

* Have you checked the listening Fabric Event Hub is actually up and running (listening for events) ?
* Are you configuring a Multi-Org environment ? EventURLs are only defined for 'this' Org's peers (and not other Org's peers)
* grpc or grpcs (http or https for CA) protocol defined correctly in your connection info ?

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| Unhandled Promise Rejections  | Your connection profile info (in your card) has been incorrectly defined
| #1 | See https://chat.hyperledger.org/channel/composer?msg=wnz6YZpvFMdCrgHZJ
| #2| https://stackoverflow.com/questions/46270080/node8232-unhandledpromiserejectionwarning-error-could-not-find-chaincode-wit

#### :card_index: [back to base camp :camping: ](#top)   


<a name="filters"></a>


### :information_source:  Filters

The syntax examples below are Loopback Filter syntax FYI

Examples of stringified JSON Filters 

| Filter Type  | Example
| :---------------------- | :-----------------------
| Two condition 'and' filter | `{ "where":{ "and":[ { "isAccepted":true }, { "isClosed":false }] } }`
| Three condition 'and' filter | ` { "where":{ "and":[ { "isAccepted":true }, { "isClosed":false }, { "providertype":"Broker"} ] } } `

Examples of non-stringified JSON Filters 

| Filter Type  | Example
| :---------------------- | :-----------------------
| Three Condition 'and' filter | `[where][and][0][isAccepted]=1&filter[where][and][1][isClosed]=0&filter[where][and][2][providerType]=Broker"" `

eg. `curl -g -X GET "http://127.0.0.1:3000/api/Commodity?filter[where][and][0][isAccepted]=1&filter[where][and][1][isClosed]=0&filter[where][and][2][providerType]=Broker"" `

Example of Filter resolve: (meaning: Search on a modeled asset and resolve the data for the related model (eg. Commodity owner below)

| Filter goal  | Example
| :---------------------- | :-----------------------
| resolve relationships | `{"where":{"":"resource:org.acme.biznet.Commodity#ABC"}, "include":"resolve"}`



#### :card_index: [back to base camp :camping: ](#top)   

<a name="historian"></a>

### :information_source:  Historian Queries - common asks

Some typical examples of historian queries asked are below (obviously you would build this query per the docs) :

| Filter goal  | Example
| :---------------------- | :-----------------------
| SELECt ALL | `SELECT org.hyperledger.composer.system.HistorianRecord `
| Select Transaction Type | ` SELECT org.hyperledger.composer.system.HistorianRecord  WHERE (transactionType == 'myTranType' `
| with Order By  | `SELECT org.hyperledger.composer.system.HistorianRecord  WHERE (transactionType == 'myTranType') ORDER BY [transactionTimestamp DESC `
| Select by Txn ID | SELECT org.hyperledger.composer.system.HistorianRecord WHERE (transactionId==_$trxID)

eg.  `SELECT org.hyperledger.composer.system.HistorianRecord  WHERE (transactionType == 'ShipmentReceived') ORDER BY [transactionTimestamp DESC `


<a name="ibmcloud"></a>


### :information_source:  IBM Cloud / Kubernetes Support / IBM Container service

Please seek support through the official channels - see link below for more info

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| IBM Sandbox / Kubernetes support  |for support with your particular environment on IBM Cloud you should go to this page https://console.bluemix.net/docs/support/index.html#contacting-support


#### :card_index: [back to base camp :camping: ](#top)  

<a name="multiorg"></a>

### :information_source:  Multi Org / BYFN Composer tutorial - issues.

The following are a selection of answers, to help understand what you may be encountering. Check also [Runtime Install errors](#runtime-install) for runtime issues

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| Multi-Org on separate physical machines | See Rocketchat here ->  https://chat.hyperledger.org/channel/composer?msg=JtwfHPJmbSgLG5dr5
| As above - other things to consider | https://chat.hyperledger.org/channel/composer?msg=f8yyrCfaBKXwt9qKH



#### :card_index: [back to base camp :camping: ](#top)  


<a name="passport-strategy"></a>

### :information_source:  Passport Strategy Info

The following are a selection of answers, to help understand what you may be encountering:

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| Is Passport-local supported?  |Not tested - see Rocketchat thread [here](https://chat.hyperledger.org/channel/composer?msg=jP6znqHXa6fChLiAX)
| Passport-local (custom)  | as-is - see Rocketchat thread [here](https://chat.hyperledger.org/channel/composer?msg=uruWP9jJbCEQcQqNo)
| Passport-jwt info | Not test - see Rocketchat thread [here](https://chat.hyperledger.org/channel/composer?msg=etkJ7wzdbdFnSXW79)
| Custom Passport strategy |  Useful Rocketchat thread (as-is) [here](https://chat.hyperledger.org/channel/composer?msg=KW4DbESMZKkPRWmPQ)


#### :card_index: [back to base camp :camping: ](#top)  


<a name="runtime-install"></a>

### :information_source:  Runtime install errors 

The following are a selection of answers, to help understand what you may be encountering:

| Message encountered | Resolution 
| :---------------------- | :-----------------------
| Error: No valid responses from any peers  | see below
| Error: Error: Endpoint read failed |  This is likely to be a connection (.json) config issue -  it needs to be configured for TLS (or non-TLS) communications depending in your setup


#### :card_index: [back to base camp :camping: ](#top)  


#### Sample Networks (issue)

The link below provides the link to our Sample Networks - to clone it simply do a `git clone https://github.com/hyperledger/composer-sample-networks.git` from the command line or from the same Github repo in a browser.

<a name="samples"></a>


### :information_source:  Composer Sample Networks

| Description | Link
| ----------- | ---------
| Sample Networks | https://github.com/hyperledger/composer-sample-networks
| Sample Applications | https://github.com/hyperledger/composer-sample-applications
| Sample Models | https://github.com/hyperledger/composer-sample-models

#### :card_index: [back to base camp :camping: ](#top)  

#### Creating a problem report (issue)

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
