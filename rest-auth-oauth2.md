---
layout: default
title: Configuring OAUTH2 Authentication with a persistent Composer REST server instance
category: tutorials
section: tutorials
index-order: 307
sidebar: sidebars/accordion-toc0.md
---

# Configuring OAUTH2 Authentication with a persistent Composer REST server instance

This tutorial provides an insight into configuring the OAUTH2 authentication strategy (eg. Google, Facebook, Twitter etc) for end users of a blockchain network that will consume or interact with a smart contract in the form of a deployed business network. You will run the REST server in multi user mode as well as store OAUTH 2.0 user authentication tokens in a persistent data store using MongoDB. Administrators of any REST server must select Passport strategies to authenticate clients in multi user mode (there are a wide range of strategies (300+ at the time of writing)). 

In a business organisational sense, enterprise  strategies such as SAML or LDAP are more appropriate obviously, a classic example being authentication against an Active Directory server. We will use Google as the authentication provider for this tutorial, as its easy for anyone to setup an account to carry out the tutorial without any real effort or prereqs to be installed (steps to set up a Google account are shown at the bottom).

Note that for a production environment, it is normal to deploy multiple instances of a REST server to increase resiliency of the system. When doing this, all instances would share a data source (eg. network storage) so that the user doesn’t have to be authenticated on each individual instance.

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

    COMPOSER_CARD=restadmin@digitalproperty-network
    COMPOSER_NAMESPACES=never
    COMPOSER_AUTHENTICATION=true
    COMPOSER_MULTIUSER=true
    COMPOSER_PROVIDERS='{
    "google": {
	    "provider": "google",
	    "module": "passport-google-oauth2",
	    "clientID": "854054473281-uc8lnej4bmkqstbcubhcvbt305677ee6.apps.googleusercontent.com",
		    "clientSecret": "jTINazEjuRBn4fbNNYC5NP9d",
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


#
