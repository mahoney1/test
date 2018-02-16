---
layout: default
title: Configuring OAUTH2 Authentication with a persistent Composer REST server instance
category: tutorials
section: tutorials
index-order: 307
sidebar: sidebars/accordion-toc0.md
---

# Configuring OAUTH2 Authentication with a persistent Composer REST server instance

This tutorial provides an insight into configuring the OAUTH2 authentication strategy (eg. Google, Facebook, Twitter etc) for end users of a blockchain network that will consume or interact with a deployed smart contract in the form of a Commodity Trading business network. You will run the REST server in multi user mode and test interacting with the network as different blockchain identities.

The REST Server is configured to persist the business network cards (required to connect to the network) and the OAUTH 2.0 user authentication tokens using MongoDB store. Administrators of any REST server must select Passport strategies to authenticate clients such as applications etc in multi user mode (there are a wide range of strategies (300+ at the time of writing). 

In a business organisational sense, enterprise  strategies such as SAML, JSON Web Tokens (JWT) or LDAP are more appropriate obviously, a classic example for the latter being authentication to an organisational Active Directory server. We use Google as the authentication provider for this tutorial, as its easy for anyone to setup a Google account (see Appendix on how to achieve this) and carry out the tutorial without worrying about prereqs to be installed.

Note that for a production environment goes, an organisation would typically deploy multiple instances of a REST server to increase resiliency of the system. When doing this, all instances would share a data source (eg. network storage) so that the user doesn’t have to be authenticated on each individual instance.

Once authenticated, application users obtain an access token and, once defined as a participant in the business network, can consume the REST APIs. As a Blockchain developer, you wouldn't ordinarily be required to do this, but gives the developer an insight into how application users can be authenticated and from there,how to transact on the blockchain using their blockchain identity.

You should carry out this tutorial as a non-privileged user (sudo or root privileges are not required)


### Set up the persistence DB Credentials Store using MongoDB

As mentioned, we will store credentials in a persistent data store once the appropriate business network cards are imported to the REST Wallet. 

#### Start the MongoDB Instance

1. Open a terminal window and enter the following command:

    docker run -d --name mongo --network composer_default -p 27017:27017 mongo
    
 It should output that a docker image has been downloaded and provide a SHA256 message. An instance of the MongoDB docker container has been started
 
 ### Build the REST Server Docker Image with OAUTH2 module
 
 1. In your $HOME directory, create a docker file `Dockerfile` in an editor and paste into the following sequence (including special characters below):
 
     FROM hyperledger/composer-rest-server:next
RUN npm install --production loopback-connector-mongodb passport-google-oauth2 && \
    npm cache clean –force && \
    ln -s node_modules .node_modules
    
This Docker file will pull the Docker image located at /hyperledger/composer-rest-server and additionally install two more npm modules:

•	**loopback-connector-mongodb** – This module provides a MongoDB connector for the LoopBack framework and allows our REST server to use MongoDB as a data source. For more information: https://www.npmjs.com/package/loopback-connector-mongodb
•	**passport-google-oauth2** – This module lets us authenticate using a Google account with our REST server. For more information: https://www.npmjs.com/package/passport-google-oauth-2

2. From the same directory where the `Dockerfile` resides, build the custom Docker REST Server image:

    docker build -t myorg/composer-rest-server .
    
The parameter given the –t flag is the name you want to give to this Docker image, this can be up to you to name - but for this guide the image will be called ‘myorg/composer-rest-server’.

You should see output similar to the following with the bottom 2 lines indicating it was 'Successfuly built':

    docker build -t myorg/composer-rest-server .
    Sending build context to Docker daemon  4.203GB
    Step 1/2 : FROM hyperledger/composer-rest-server:next
     ---> e682b4374837
    Step 2/2 : RUN npm install --production loopback-connector-mongodb passport-google-oauth2 &&     npm cache clean  --force &&     ln -s node_modules .node_modules
     ---> Running in 7a116240be21
    npm WARN saveError ENOENT: no such file or directory, open '/home/composer/package.json'
    npm WARN enoent ENOENT: no such file or directory, open '/home/composer/package.json'
    npm WARN composer No description
    npm WARN composer No repository field.
    npm WARN composer No README data
    npm WARN composer No license field.

    + passport-google-oauth2@0.1.6
    + loopback-connector-mongodb@3.4.1
    added 114 packages in 7.574s
    npm WARN using --force I sure hope you know what you are doing.
     ---> a16cdea42dac
    Removing intermediate container 7a116240be21
    Successfully built a16cdea42dac
    Successfully tagged myorg/composer-rest-server:latest
  
  
INFO: Don’t worry about seeing the 'npm warn messages' as shown on the console as per above. This can be ignored.

### Define Environment variables for REST Server instance configuration

Create a file called `envvars.txt` in your $HOME directory and paste in the following configuration settings:

    COMPOSER_CARD=restadmin@trade-network
    COMPOSER_NAMESPACES=never
    COMPOSER_AUTHENTICATION=true
    COMPOSER_MULTIUSER=true
    COMPOSER_PROVIDERS='{
    "google": {
	    "provider": "google",
	    "module": "passport-google-oauth2",
	    "clientID": "854054453281-uc8lnej4bmkqstbcubhcvbt305677ee6.apps.googleusercontent.com",
		    "clientSecret": "jTINazEegjuRBn4fbNNYC5NP9d",
		    "authPath": "/auth/google",
		    "callbackURL": "/auth/google/callback",
		    "scope": "https://www.googleapis.com/auth/plus.login",
		    "successRedirect": "/",
		    "failureRedirect": "/"
      }
    }'
    COMPOSER_DATASOURCES='{
        "db": {
            "name": "db",
            "connector": "mongodb",
            "host": "mongo"
        }
    }'


The environment variables defined here will indicate that we want a multi user server with authentication using Google OAuth2 along with MongoDB as the persistent data source.

The first line indicates the name of the business network card we will start the network with - a specific REST Administrator against a defined business network. You will also see that in this configuration we also define the data source the REST server will use and the authentication provider we are using. These can be seen with the COMPOSER_DATASOURCES and COMPOSER_PROVIDERS variables respectively.


### Load environment variables in current terminal and launch the persistent REST Server instance

From the same directory as the `envvars.txt` file you created containing the environment variables, run the following command:

     source envvars.txt
     
INFO	No output from command? - this is expected. If you did have a syntax error in your `envvars.txt` file then this will be indicated by an error, after running this command.

Let’s now confirm that environment variables are indeed set by checking a couple of them using “echo” command as shown below

    echo $COMPOSER_CONFIG
    echo $COMPOSER_ENROLLMENT_ID

### Deploy our sample Commodities Trading Business network to query from REST client

If you've not already done so - download the `trade-network.bna` for the Trade-network from https://composer-playground.mybluemix.net/

(In Playground, connect to the network as `admin` and export the trade-network.bna to your home directory

To deploy it, run the following sequence:

    composer runtime install -c PeerAdmin@hlfv1 trade-network

    composer network start -c PeerAdmin@hlfv1 -A admin -S adminpw -a trade-network.bna -f  networkadmin.card

You should get confirmation that the Commodities Trading Business Network has been started and an 'admin' networkadmin.card file has been created and you're all set

Next import and download the certs for the admin card:

    composer card import -f networkadmin.card

    composer network ping -c admin@trade-network

You should get confirmation that the connectivity was successfully tested. We're now ready to work with the business network.

    
###  Create the REST server administrator for the Composer REST server instance 


First, we need to create our REST adninistrator identity and business network card - used to launch the REST server later.

    composer participant add -c admin@trade-network -d '{"$class":"org.hyperledger.composer.system.NetworkAdmin", "participantId":"restadmin"}'

     composer identity issue -c admin@trade-network -f restadmin.card -u restadmin -a "resource:org.hyperledger.composer.system.NetworkAdmin#restadmin"
    
    
    composer card import -f  restadmin.card

    composer network ping -c restadmin@trade-network

Because we are hosting our REST server in another location with its own specific network IP information, we need to update the connection.json - so that the docker hostnames (from within the persistent REST server instance) can resolve each other's IP addresses.

The one liner below will substitute the 'localhost' addresses with docker hostnames and create a new connection.json - which goes into the card of our REST administrator. We will also use this custom connection.json file for our 'test' authenticated user later on in the OAUTH2 REST authentication sequence nearer the end of this tutorial. To quickly change the hostnames - copy-and-paste then run this one-liner (below) in the command line from the $HOME directory..

sed -e 's/localhost:/orderer.example.com:/' -e 's/localhost:/peer0.org1.example.com:/' -e 's/localhost:/peer0.org1.example.com:/' -e 's/localhost:/ca.org1.example.com:/'  < $HOME/.composer/cards/restadmin@trade-network/connection.json  > /tmp/connection.json && cp -p /tmp/connection.json $HOME/.composer/cards/restadmin@trade-network/
    
###  Launch the persistent REST server instance 
    
Next, run the following docker command to launch a REST server instance (with the `restadmin` business network card)    

    docker run \
    -d \
    -e COMPOSER_CONNECTION_PROFILE=${COMPOSER_CONNECTION_PROFILE} \
    -e COMPOSER_BUSINESS_NETWORK=${COMPOSER_BUSINESS_NETWORK} \
    -e COMPOSER_ENROLLMENT_ID=${COMPOSER_ENROLLMENT_ID} \
    -e COMPOSER_ENROLLMENT_SECRET=${COMPOSER_ENROLLMENT_SECRET} \
    -e COMPOSER_NAMESPACES=${COMPOSER_NAMESPACES} \
    -e COMPOSER_AUTHENTICATION=${COMPOSER_AUTHENTICATION} \
    -e COMPOSER_MULTIUSER=${COMPOSER_MULTIUSER} \
    -e COMPOSER_CONFIG="${COMPOSER_CONFIG}" \
    -e COMPOSER_DATASOURCES="${COMPOSER_DATASOURCES}" \
    -e COMPOSER_PROVIDERS="${COMPOSER_PROVIDERS}" \
    --name rest \
    --network composer_default \
    -p 3000:3000 \
    myorg/composer-rest-server
    

This will output the ID of the Docker container eg . 690f2a5f10776c15c11d9def917fc64f2a98160855a1697d53bd46985caf7934 and confirm that the REST server has been indeed started. You can see that it is running using the following command:

    docker ps |grep rest
    

### Test the REST APIs for the business network

Open a browser window and launch the REST API explorer by going to to http://localhost:3000/explorer to view and use the available APIs.

INFO	Admin identity `restadmin` is used as an initial default - The REST server uses “restadmin” identity until a specific identity e.g. trader1 is set as a default identity in the REST client wallet

Go to the “System: general business network methods” section

Go to the “/system/historian” API and click on “Try it out!” button as shown below:

<need image>


You should get an Authorized error. In the next Section, you will notice that the extent of records that are visible in Composer's Historian are granted by granular ACL rules. By default, ACLs work such that all records in a business network are denied due to the secured REST server restrictions.

### Create some Participants and Identities for testing OAUTH2 authentication

Next, you will need to create a participant and issue an identity for them. This is because when the REST server is configured to allow multiple REST client users and allow authentication with their different IDs at the REST client


We will be using the composer CLI commands to add participants and identities.

    composer participant add -c admin@trade-network -d '{"$class":"org.acme.trading.Trader","tradeId":"trader1", "firstName":"Jo","lastName":"Doe"}
     
    composer identity issue -c admin@trade-network -f jdoe.card -u jdoe -a "resource:org.acme.trading.Trader#trader1"

    composer card import -f jdoe.card 

    composer network ping -c jdoe@trade-network
    
Once again, because we will use this identity to test inside the persistent REST docker container - we will need to change the hostnames to represent the docker resolvable hostnames - once again run this one-liner to carry out those changes quickly:

    sed -e 's/localhost:/orderer.example.com:/' -e 's/localhost:/peer0.org1.example.com:/' -e 's/localhost:/peer0.org1.example.com:/' -e 's/localhost:/ca.org1.example.com:/'  < $HOME/.composer/cards/jdoe@trade-network/connection.json  > /tmp/connection.json && cp -p /tmp/connection.json $HOME/.composer/cards/jdoe@trade-network/
    
 Lastly, we want to export the card - to use for import on a 'remote application' - this is the card that we will use to authenticate to the REST server (ie if it was located remote to the REST server)

     composer card export -f jdoe.card -n jdoe@trade-network

This card can now be used in the REST client


### Authenticating from the REST API Explorer and testing using specific identities 

You need to add an identity to a REST client wallet and then set this identity as the default one to use for making API calls.

Go to http://localhost:3000/auth/google - this will direct you to the Google Authentication consent screen. 

Login using the following credentials: (example - as advised, you should set up your own per the instructions in the appendix):

Email: blockchainworkshop1@gmail.com
Password: bcworkshop123

You will now be redirected back to the secured REST server (http://localhost:3000/explorer) which shows the access token message as follows: 

<need image>

While we have authenticated to Google - we have not actually set any cards as default in our wallet, to interact with our business network - we will do that next using the new identity we created earlier

### Retrieve the Default Wallet and Import the card and set a default Identity

There is a default wallet that is already create, which we will use to add our identity to

Call the REST endpoint under Wallets:

    GET /wallets
 
It should show a default Wallet and return a Wallet ID in JSON form (and which changes for each use). Copy the **ID** field value / contents **only** (ie the alpha-numeric code inside the quotes) and go to the following REST API endpoint:

    POST /wallets/{id}/identities

In the authenticated browser - go to the POST system operation under /Wallets - its called /Wallets/Import endpoint

Choose to import the file trader1.card  - and provide the name of the card as trader1@trade-network  and click 'Try it Out'

You should you should get an HTTP Status code 204 (request was successful) 

Next, go back to 

    GET /wallets
    
You should see that trader1@trade-network is imported into the wallet. - Next let's set this as the default identity (all we've done so far is Import the business network card)

Go to the POST endpoint for  /Wallets{name}/setDefault and use  trader1@trade-network as the default card and click on Try It Out

You should get an HTTP Status code 204 (request was successful)

Revisting the GET  /Wallet operation  - you should see that trader1@trade-network is now set as the default user ('true'))
    
Then paste the wallet ID parameter (from earlier) in the 'id' field and click on 'Try it Out'

### Test interaction with the Business Network as the default ID

Go to System REST API  Methods section and expand the /GET System/Historian section

Click on 'Try It Out' - you should now see results from the Historian Registry, as the blockchain identity 'trader1'

Go to the Person methods and expand the /GET Person endpoint

You should now be able to see the results of the participants that user 'trader1' is allowed to see in that participant Registry, which is subject to any ACLs that have been set. 'Trader1' can only see and edit their own participant records (according to the ACLs) - this merely shows that the REST APIs are subject to access control like any other operation.

<image>
 


    
    
    


