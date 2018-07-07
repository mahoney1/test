---
layout: default
title: Adding an organization to an existing multi-organization network
category: tutorials
section: tutorials
index-order: 309
sidebar: sidebars/accordion-toc0.md
---

# Adding an organization to an existing, running multi-organization network

![Overview of Three-Org blockchain network](../assets/img/tutorials/multi/three-org-network.png)

This tutorial provides an insight into adding/joining another organization to an existing multi-org blockchain network and the minimal steps needed - from a Composer perspective - to demonstrate interaction with a running business network as a participant from the 3rd organization. The conclusive test is being able to submit a simple Composer-based transaction as the newly created participant, which represents a blockchain identity from the 3rd organization.

It is worth emphasising here that this {{site.data.conrefs.hlf_full}} blockchain network (for three organizations)  will be deployed using docker containers, in a local environment and indeed in the same virtual machine ; obviously, in the real world, they'll be in separate IP networks or domains, with all the IP/name resolution that requires (or indeed perhaps, in secure Cloud environments). The task of configuring Fabric networking,  Docker networking and IP / Host / Container Name resolution is understandably, beyond the scope of this Composer tutorial. NOTE: If you choose to split your organisations across separate physical machines or separate virtual machines running on different IP networks, it is also outside the scope of this particular tutorial and you should consult the Fabric docs, or appropriate resources (such as the #fabric channel on [Rocketchat](https://chat.hyperledger.org/channel/fabric) or ask via Stack Overflow with the appropriate Fabric tag to seek assistance with any Fabric issues).


<h2 class='everybody'>Pre-requisites to this tutorial</h2>

You must first complete the [Multi-Org tutorial](./deploy-to-fabric-multi-org.html), as this sets up the {{site.data.conrefs.hlf_full}} 'BYFN' two-organization network with a running, established Composer business network (and associated metadata / identities that entails).

NOTE: If you've completed that tutorial and need a break first or are resuming the next day etc - you can use these commands to stop and start your BYFN environment, saving the current state ; these are equivalent to the `docker-compose up` and `docker-compose stop` commands to save you containers' state.

    ./byfn.sh -m stop    # stop current BYFN containers
    ./byfn.sh -m start   # start current BYFN containers from last 'stopped' state.

You must have completed the two-organization [Multi-Org tutorial](./deploy-to-fabric-multi-org.html)before doing this tutorial, and the runtime Fabric docker containers should be up and running, at the point where you had completed that tutorial. Do not do a teardown of the environment first - as this tutorial builds on that setup, which has a running, established Commodity Trading business network.

Firstly, to add the 3rd organization from a Fabric perspective, we use the [Add an Org to a channel](http://hyperledger-fabric.readthedocs.io/en/release-1.1/channel_update_tutorial.html) instructions, as the basis for adding the Fabric artifacts required - it extends the Fabric [Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html) 'BYFN' network. Note: we have incorporated the necessary steps for you in this tutorial - so no need to jump off to the Fabric docs :-).


The tutorial has colour-coded steps for convenience, to indicate 'which organization' should follow a particular step or sequence - or indeed, if steps are needed by all Orgs.

The first kind of step is for all organizations to follow (where applicable):

<h2 class='everybody'>Example Step: A step for Org1,  Org2 and Org3 to follow or a general instruction</h2>

The organization `Org1` is represented by Alice, the Green Conga Block:

<h2 class='alice'>Example Step: A step for Org1 to follow</h2>

The organization `Org2` is represented by Bob, the Violet Conga Block:

<h2 class='bob'>Example Step: A step for Org2 to follow</h2>

The organization `Org3` is represented by Mike, the Blue Conga Block

<h3 class='mike'>Example Step: A step for Org3 to follow</h2>

Let's get started!


<h2 class='everybody'>Step One: Verify your existing {{site.data.conrefs.hlf_full}} network</h2>

1. From the command line / terminal window - test the business network is active using an existing card, from the previous tutorial:

        composer network list -c dlowe@trade-network
        
This should output some business network information, proving connectivity to the `trade-network` is all fine and dandy.

<h2 class='everybody'>Step Two: Prepare to build additional artifacts for Org 3</h2>

1. Perform the following sequence to take a backup of the existing Composer profiles and security/crypto materials previously created in the 2-Org multi-org tutorial:

        cp -r /tmp/composer  /tmp/composer.bak
        
The `Org 3` config files we need, is the `fabric samples` repo from `github.com/mahoney1`, which you would have downloaded to your $HOME directory from the Multi-Org (pre-requisite) tutorial eg.   this is the repo you would have cloned previously - no need to execute. (Rob - this will be removed just FYI - merely verification that you would have done this in the 2-org tutorial)

        # git clone -b multi-org https://github.com/mahoney1/fabric-samples.git  


1. Change directory to the current `2-Org` fabric samples `$HOME/fabric-samples/first-network` directory:

         cd $HOME/fabric-samples/first-network ; git checkout multi-org

2. Finally, set up the 'PATH' environment variable, in preparation to run the Fabric samples 'Extend your Network' script - note you must still be located in `$HOME/fabric-samples/first-network` at this point.

        export PATH=$PATH:$HOME/fabric-samples:$HOME/fabric-samples/bin

3. Check you have 'cryptogen' set in your PATH :

        which cryptogen
        
This should reveal that the `cryptogen` executable has been located, in readiness for the next configuration steps.


<h2 class='everybody'>Step Two: Run the EYFN Fabric 'Add Organization' steps and build artifacts for Org 3</h2>

1. Run the `eyfn.sh` script (which was copied over from the downloaded Fabric Samples repo earlier) as follows:

         ./eyfn.sh up -t 60 -s couchdb  // - t 60 is TIMEOUT value FYI  

It should run through a sequence of adding more peers (two for Org3) and associated configuration / crypto artifacts to be able add Org 3 as an MSP and to join its peers to the existing `mychannel` already shared by Org1's and Org2's peers. This Fabric script will run some rudimentary sample chaincode deploys, and follow with some simple 'chaincode query' command line sequences to ensure that the peers are operating correctly at a Fabric level, on the channel / network and that they can query the ledger. You'll also see some peers were added as part of the on-screen messages. You may see a final message something like:

   ========= All GOOD, EYFN execution completed ===========


    _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/


PLEASE NOTE: the last `chaincode query` result may return a `Query result is invalid` message (and therefore you may not see this banner currently) - you can ignore this particular 'invalid' message for now (a 'check' in the script isn't producing the right return value, even though the query is actually successful). Any other result, will likely indicate some issues with running the script in your environment.

This 'EYFN' {{site.data.conrefs.hlf_full} script adds 2 peers for Organisation 3 ('Org 3') and performs the requisite steps to join the existing `mychannel` channel successfully. Next,  we need to add Org 3's own Fabric CA server, so that an Org 3 admin is able to issue identity certificates for Org 3 identities (and which are mapped to participants in Composer, which we'll see later).

2.Check that a `docker-compose-ca3.yaml ` has been created in the current directory and contains the 'keyfile' information - something like that shown below: (Rob - this will be removed for reasons already known just FYI - merely left for verification)

        version: '2'

        networks:
          byfn:
        services:
          ca2:
            image: hyperledger/fabric-ca
            environment:
              - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
              - FABRIC_CA_SERVER_CA_NAME=ca-org3
              - FABRIC_CA_SERVER_TLS_ENABLED=true
              - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org3.example.com-cert.pem
              - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/b5f973338e883bdd4cf8346ce40cd431d65fe83bccbbb2771349d9a38f674102_sk
        ports:
              - "9054:7054"
            command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org3.example.com-cert.pem --    ca.keyfile /etc/hyperledger/fabric-ca-server-config/b5f973338e883bdd4cf8346ce40cd431d65fe83bccbbb2771349d9a38f674102_sk -b admin:adminpw -d'
            volumes:
              - ./org3-artifacts/crypto-config/peerOrganizations/org3.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
            container_name: ca_peerOrg3
            networks:
              - byfn

3. // IGNORE - Run the following command manually to start up the 3rd Organization's CA server - ignore the messages about other nodes for now - you should see the CA server docker container is launched: (NOTE to Rob; this will eventually go into the EYFN.sh script - just want to see messages on-screen for now)

         docker-compose -f docker-compose-ca3.yaml up -d  2>&1

We've now completed the 'Fabric' elements of the 3-Org configuration. We'll now move on to the Composer tasks of onboarding a business network participant from Org 3, such that he can transact on the existing business network and it can be seen by the others.

The remainder of the tutorial describes Composer tasks, namely:

    - update the common connection profile to includes Org 3's Fabric config (eg. Org 3's CA server, two peers etc) and Organizational metadata
    - build an Org3 PeerAdmin business network card to install the `trade-network` code on Org 3's peers
    - the act of an existing Org admin binding an Org3 participant (and associated Org 3 identity) to the business network
    - issuing identities, as an admin of Org 3 and creating business network cards for the Org 3 participants
    - checking the ability to create transactions, as an Org 3 issued identity (and participant) on the same business network and verifying the ledger changes are seen, from someone in Org1. 
     - finally, in the Appendix: the steps for updating the endorsement policy on the running network (to require that 3 Organizations endorse transactions (rather than 2 previously) on the business network, before they are committed to the blockchain).
     
<h2 class='everybody'>Step Four: Create Composer artifacts in preparation for Org 3's business network interaction</h2>


1. Start to create Org 3's Composer configuration metadata, to be used for building creating business network cards etc

        mkdir /tmp/composer/org3

        # From $HOME/fabric-samples/first-network

        export ORG3=org3-artifacts/crypto-config/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp

        cp -p $ORG3/signcerts/A*.pem /tmp/composer/org3

        cp -p $ORG3/keystore/*_sk /tmp/composer/org3

        awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' org3-artifacts/crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt > /tmp/composer/org3/ca-org3.txt

2. Go to Org1's temporary directory in `/tmp/composer/org1`, and edit the existing `byfn-network-org1.json` file - we will add Org3's essential JSON-formatted config data to this connection profile: If you have issues inserting these stanzas - tip: you can use https://jsonformatter.curiousconcept.com/ to validate your edit session for the connection profile.

Under `channels` section / stanza - add the two peers as follows, after `peer1.org2` entry (don't forgot to add a comma):

                "peer0.org3.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                },
                "peer1.org3.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                }

3. In the same profile, under the `Organizations` section - add Org3's organization config info, after Org2's definition:

        "Org3": {
            "mspid": "Org3MSP",
            "peers": [
                "peer0.org3.example.com",
                "peer1.org3.example.com"
            ],
            "certificateAuthorities": [
                "ca.org3.example.com"
            ]
        }
        
 4. Scroll further down to the `peers` section and add Org 3's peer 0 and peer 1 entries as follows:
 
        "peer0.org3.example.com": {
            "url": "grpcs://localhost:11051",
            "eventUrl": "grpcs://localhost:11053",
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org3.example.com"
            },
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----INSERT_CA_ORG3_CERT\n-----END CERTIFICATE-----\n"
            }
        },
        "peer1.org3.example.com": {
            "url": "grpcs://localhost:12051",
            "eventUrl": "grpcs://localhost:12053",
            "grpcOptions": {
                "ssl-target-name-override": "peer1.org3.example.com"
            },
            "tlsCACerts": {
                "pem": "-----BEGIN CERTIFICATE-----INSERT_CA_ORG3_CERT\n-----END CERTIFICATE-----\n"
            }
        }

5. Under the `certificateAuthorities` section - add the CA definition for Org3's CA server as follows (ensuring you add the comma after Org2's entry):

        "ca.org3.example.com": {
            "url": "https://localhost:9054",
            "caName": "ca-org3",
            "httpOptions": {
                "verify": false
            }
        }

6. With Org1's connection profile now complete for all 3 Orgs, we can now make copies of this from `/tmp/composer/org1` to Org2 and Org 3's temporary directories, and change the Org Id accordingly:

        cp byfn-network*.json ../org2/byfn-network-org2.json
        cp byfn-network*.json ../org3/byfn-network-org3.json


7. Edit `org2/byfn-network-org2.json` and `org3/byfn-network-org3.json` , in turn and, under the `client` section, change the "organization" entry to be `"Org2"` for org 2's profile,  and `"Org3"` for org 3's profile ,respectively. Save both files in the respective directories.

In the next section, we will build the new 3-Org business network card, for participants aligned to the 3rd Organization.

<h2 class='everybody'>Step Five: Create Org3's Peer Admin card and install Trade Network onto Org3's peers</h2>


1. Perform the following sequence of commands to build cards then install the business network onto Org3's peers - ensure you're in the `first-network`:

        cd $HOME/fabric-samples/first-network

        composer card create -p /tmp/composer/org3/byfn-network-org3.json -u PeerAdmin -c /tmp/composer/org3/Admin@org3.example.com-cert.pem -k /tmp/composer/org3/*_sk -r PeerAdmin -r ChannelAdmin -f PeerAdmin@byfn-network-org3.card

2. Import and use the card:

        composer card import -f PeerAdmin@byfn-network-org3.card --card PeerAdmin@byfn-network-org3

        composer network install --card PeerAdmin@byfn-network-org3 --archiveFile $HOME/trade-network.bna
        
    
        
        
<h2 class='everybody'>Step Six: Bind Org3's admin using an existing Org Admin of the network</h2>
       
1. In order to bring Org3's admin into the network, we must bind the Org3 issued identity (eg using `composer identity bind`)  as an existing participant of the business network (eg `alice@trade-network` a Network Admin from Org 1). . This is effectively the act of the existing business network participants agreeing that the new Org3 business network participant can join and use their identity on the network. Let's issue that identity now:

        composer identity request -c PeerAdmin@byfn-network-org3 -u admin -s adminpw -d mike

2. Next, add a participant ("org3-admin") as an existing administrator from (say) Org 1.

        composer participant add -c alice@trade-network -d '{"$class":"org.hyperledger.composer.system.NetworkAdmin", "participantId":"org3-admin"}'

3. Bind the new participant, a task executed by an existing administrator - `alice@trade-network`:

        composer identity bind -c alice@trade-network -a "resource:org.hyperledger.composer.system.NetworkAdmin#org3-admin" -e mike/admin-pub.pem


<h2 class='everybody'>Step Seven: Issue new business network card and test it out</h2>

1. Build a card for the new Org3 admin, `mike`, import it and check that it can query/interrogate the business network:

        composer card create -p /tmp/composer/org3/byfn-network-org3.json -u mike -n trade-network -c mike/admin-pub.pem -k mike/admin-priv.pem
        composer card import -f mike@trade-network.card

2. Now `ping` the business network - please be patient, as this creates/deployes 2 'chaincode' containers (being the first contact, in Org 3, implicitly via instantiation) 

        composer network ping -c mike@trade-network

3. Verify you can see some business network artifacts 

        composer network list -c mike@trade-network

<h2 class='everybody'>Step Eight: Test that an Org 3 participants can issue transactions on the business network</h2>

The last phase of this tutorial, is to test that an Org 3 issued identity (linked to a participant, and using a business network card) can update an existing asset on the blockchain and change ownership of a Commodity to himself:

1. Issue a new participant - as `mike` the Org3 administrator, and issue identity / import the new business network card, as before:

        composer participant add -c mike@trade-network -d '{"$class":"org.example.trading.Trader","tradeId":"trader3-org3", "firstName":"Mo","lastName":"Salah"}'

        composer identity issue -c mike@trade-network -f mo.card -u mo  -a "resource:org.example.trading.Trader#trader3-org3"

        composer card import -f mo.card
        
        composer network ping -c mo@trade-network
        
2. Submit a `Trade` transaction as participant `trader3-org3` using the `mo@trade-network` card, to change ownership of the existing `EMA` Commodity asset on the `trade-network` to himself:

        composer transaction submit --card mo@trade-network -d '{"$class":"org.example.trading.Trade","commodity":"resource:org.example.trading.Commodity#EMA","newOwner":"resource:org.example.trading.Trader#trader3-org3"}'

3. Run a `composer network list`, as a participant from Org1 (in this case `jdoe`) - the ledger should now show the `EMA` Commodity asset is now owned by `trader3-org3`

        composer network list jdoe@trade-network 
       


<h2 class='everybody'>Conclusion</h2>

In this tutorial you have seen how to add an Organization to an existing blockchain network based on {{site.data.conrefs.composer_full}} . You've seen the important steps of an 'initiator' of the business network (the Org1 admin), admit an Org admin from Org3 to be able to perform business network operations, as an admin of the 3rd organization. Furthermore, you've seen how to add new Org3 aligned participants to the business network, and successfully submit a Trade transaction on the business network. 

We hope you found the tutorial useful :-) - thanks for completing it !


<h2 class='everybody'>Appendix - the next steps</h2>

The tutorial added a 3rd organization, and showed how, initially at a Fabric level - then at a Composer and business network level, the minimum tasks to allow Org3 participants to participate in that `trade-network` Commodity Trading business network. However, you may recall, that we are still using an endorsement policy, that is a 'legacy' policy from our 2-Org multi-org tutorial. The endorsement policy should ideally be 'upgraded' to reflect rules around 3 organizations must endorse transactions before they can be committed to the blockchain.  You can find more information on endorsement policies in the Hyperledger Fabric documentation, in [Endorsement policies](https://hyperledger-fabric.readthedocs.io/en/release/endorsement-policies.html) .

Complete the following steps to add the requirement for the 3rd organization to be required to endorse transactions.

1. Edit the file `endorsement-policy.json` in the existing `/tmp/composer` directory.

2. Replace the contents with the configuration data below (we add the metadata for `Org3MSP` and change the rules to require `3-of`):

        {
            "identities": [
                {
                    "role": {
                        "name": "member",
                        "mspId": "Org1MSP"
                    }
                },
                {
                    "role": {
                        "name": "member",
                        "mspId": "Org2MSP"
                    }
                },
                {
                    "role": {
                        "name": "member",
                        "mspId": "Org3MSP"
                    }
                }
            ],
            "policy": {
                "3-of": [
                    {
                        "signed-by": 0
                    },
                    {
                        "signed-by": 1
                    },
                    {
                        "signed-by": 2
                    }
                ]
            }
        }

3. Save the changes to disk. Next step is to change the business network version, as part of the upgrade process.

4. Still in the `$HOME/fabric-samples/first-network` directory, create a temporary directory `trade` and change directory to it and extract the `trade-network.bna` file (if its in your $HOME directory, copy it to the `trade` network) that you edited in the previous multi-org tutorial:

        mkdir trade ; cd trade
        
        cp ../trade-network.bna
        
        unzip trade-network.bna
        
5. Edit the `package.json` file and increment the minor version set in the `"version":` field by `1` eg. from `0.1.14` to `0.1.15` . Then save the file.

6. Re-create the archive file with the new version as follows:

       composer archive create --sourceType dir --sourceName . -a trade-network-upgrade.bna
       
7. Install the newly upgraded business network archive as `PeerAdmin`:

       composer network install --card PeerAdmin@byfn-network-org1 --archiveFile trade-network-upgrade.bna
       
8. Finally, upgrade the running business network using the `Org1 PeerAdmin` card to activate the new business network version deployed previously - ensuring you supply the new endorsement policy file `endorsement-policy.json` as a parameter to the upgrade command:

       composer network upgrade -n trade-network -V 0.1.15 -o /tmp/composer/endorsement-policy.json --card PeerAdmin@byfn-network-org1

9. The policy is now in effect and any future transactions will require the approval of endorsing peers from all 3 organizations.

FINALLY: 

10. The last remaining steps for this 3 Org tutorial, is to delete all the REMAINING Org 1 and 2 business network cards (eg `kcoe@trade-network`, `jdoe@trade-network` etc) from the credentials wallet and recreate them - to reflect the 3-Org setup. This is because they currently only contain 2-Org profiles,  and should now use the 3 Org profile, created earlier in this tutorial). You would use `composer card delete` to remove them initially (cleanup) and then re-create them using `composer card create` with updated 3-Org profile. This sequence would be followed by `composer card import` (to import the cards, with the same crypto/identity data but a new connection profile)  and then use `composer network ping` to test their connection to the upgraded business network. 
