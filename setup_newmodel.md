---
layout: default
title: Getting going with Fabric 1.3 chaincode development
category: tutorials
section: tutorials
index-order: 309
sidebar: sidebars/accordion-toc0.md
---

# Setting up and running a NodeJS Chaincode using the new Programming Model

![Smart Contract Example](./smartcontr.png)

This tutorial provides an End-to-End example of setting up, configuring and running a sample NodeJS chaincode, using the new {{site.data.conrefs.hlf_full}} programming model. The goal is to show a simple example of implementing Smart Contract chaincode using the new programming model. The tutorial is intended to be a full 'end-to-end' tutorial - it takes the developer from initial setup of a local Fabric 'dev' environment, to building, deploying and interacting with the smart contract, running on the blockchain. As part of this, we will show developing/testing a sample smart contract (chaincode) in 'Developer Mode' and in the appendix, provide instructions on standard deployment (no longer in 'developer mode') of the smart contract/chaincode to a 'remote' Fabric blockchain already configured. 


# Pre-requisites

- The new Fabric 1.3 `fabric-contract-apis` and `fabric-shim` dependencies in the `package.json` file for our chaincode example. 
- Docker and docker-compose installed - as part of this tutorial, you will pull the latest Fabric 1.3 docker images to set up your {{site.data.conrefs.hlf_full}}  environment.
- A {{site.data.conrefs.hlf_full}} Fabric 1.3 environment (dockerized images), configured for 'Chaincode dev mode' - this is a setting in a docker-compose YAML file - more on that later.
- multiple terminal windows
- a clone of this Fabric Samples Github repository from https://github.com/hyperledger/fabric-samples  - please note this repo has modifications to docker-compose YAML files and scripts to pull the {{site.data.conrefs.hlf_full}} 1.3 images mentioned earlier.
- Some sample code to use (provided as code blocks in this tutorial)


# A word about using Chaincode - Dev mode vs 'non-Dev' (deploy) mode

{site.data.conrefs.hlf_full}} - in its `basic-network` configuration, downloaded as part of Fabric samples - provides a simple docker-compose YAML file to start a simple network, and it starts the peer in "dev mode". Note that it also starts two additional containers (one is for the chaincode container and the other is a CLI container, so as to interact with the chaincode itself). In the configuration (`docker-compose-simple.yml`) the Chaincode container has volume mapping in place, the `chaincode` directory under `fabric-samples` is mapped to /opt/gopath/src/chaincode in the chaincode container. So our chaincode now can be deployed.

To start out with, you can deposit the chaincode to the directory $GOPATH/src directory to test the chaincode. Once the chaincode is tested and the developer is satisfied it works, it can be later be deployed (via the CLI container) to a  'test' or 'production' Fabric,   by attaching as a volume to our chaincode container,  so that this chaincode (and version of chaincode) can be installed/deployed and initialised on a channel on the {site.data.conrefs.hlf_full} blockchain network.

# Download the Fabric Samples

1. As per the Fabric [Getting Started guide](https://hyperledger-fabric.readthedocs.io/en/release-1.2/install.html) - you need to install the latest binaries and samples. The `curl` command to pull the 3 different images (3 parameters) is shown on that page.

eg. curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0 1.3.0 0.4.10

2. cd fabric-samples ; git checkout master  # need 'master' branch for latest changes

3. Set the PATH as follows (replace <workingdir> with where you've downloaded the sames, eg in $HOME for example):

   export PATH=<workingdir>/fabric-samples/bin:$PATH

4. change directory to the chaincode dev mode directory:
  
  cd chaincode-docker-devmode
  

# Download the correct Fabric Images 
 
 The docker images can be obtained from the Fabric Nexus docker repository: eg.
https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric-1.3.0-stable/

1. For the peer image - do the following:
    docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-peer:amd64-1.3.0-stable-1b2d58c

2. For the orderer image - do the following:
    docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-orderer:amd64-1.3.0-stable-1b2d58c

3. For the Fabric base image - do the following:
    docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-baseimage:amd64-1.3.0-stable-1b2d58c

2. For fabric tools, CLI - do the following:
   docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-tools:amd64-1.3.0-stable-1b2d58c
 
3. For Fabric ccenv - do the following:
   docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-ccenv:amd64-1.3.0-stable-1b2d58c

Find the latest date on the files listed under this directory (datestamp on right) for the platform you want to download the images eg. Mac, Linux etc
 

# Setup your Development environment

1. The first thing to do is to is to change directory to the Fabric chaincode dev environment inside your `fabric-samples` directory that was cloned earlier

    `cd chaincode-docker-devmode`
    
2. `docker-compose -f docker-compose-simple.yaml down`  # (to clear down any old environments)

3.  `docker ps -a ` to ensure you have no lingering containers  - if you do, stop them and remove them using `docker stop <container id> ` and `docker rm <container_id>

4. Now start the new Dev mode environment:

     `docker-compose -f docker-compose-simple.yaml up` 


<h2 class='everybody'>Conclusion</h2>

In this tutorial you have seen how to add an Organization to an existing blockchain network based on {{site.data.conrefs.composer_full}} . You've seen the important steps of an 'initiator' of the business network (the Org1 admin), admit an Org admin from Org3 to be able to perform business network operations, as an admin of the 3rd organization. Furthermore, you've seen how to add new Org3 aligned participants to the business network, and successfully submit a Trade transaction on the business network. 

We hope you found the tutorial useful :-) - thanks for completing it !


<h2 class='everybody'>Appendix - Setting up a deployable Fabric environments</h2>

1. In addition to the Fabric environment you deployed for `chaincode dev mode` earlier, you will need to download two additional images - the easiest way to do this is to copy the docker-compose yaml file below, and (ensuring you keep your chaincode src backed up to a safe place initially), tear down the chaincode dev environment Fabric (from earlier) and start up a 'fresh' Fabric environment, with the addition of the docker container instances for the CA server and CouchDB as the world state DB (so that rich queries can be performed). Go to the simple-network directory and type:
       ./startFabric.sh

2. Do a `docker ps` and ensure you see running containers for the fabric-tools, fabric-peer, fabric-orderer, fabric-ca, and fabric-couchdb
