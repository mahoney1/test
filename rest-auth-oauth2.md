---
layout: default
title: Configuring OAUTH2 Authentication with a persistent Composer REST server instance
category: tutorials
section: tutorials
index-order: 307
sidebar: sidebars/accordion-toc0.md
---

# Configuring OAUTH2 Authentication with a persistent Composer REST server instance

This tutorial provides an insight into configuring the OAUTH2 authentication strategy (eg. Google, Facebook, Twitter etc) for end users of a blockchain network that will consume or interact with a smart contract in the form of a deployed business network. You will run the REST server in multi user mode as well as store user authentication tokens in a persistent data store using MongoDB. Administrators of any REST server must select Passport strategies to authenticate clients in multi user mode (there are a wide range of strategies (300+ at the time of writing)). 

In a business organisational sense, enterprise  strategies such as SAML or LDAP are more appropriate obviously, such as authentication against an Active Directory server. We will use Google as the authentication provider for this tutorial, as its easy for anyone to setup an account to carry out the tutorial without any real effort or prereqs to be installed.

For a production environment, it is normal to deploy multiple instances of a REST server to increase resiliency of the system. When doing this, all instances would share a data source (eg. network storage) so that the user doesn’t have to be authenticated on each individual instance.

Once authenticated, application users obtain an access token and, once defined as a participant in the business network, can consume the REST APIs. As a Blockchain developer, you wouldn't ordinarily be required to do this, but gives the developer an insight into how application users can be authenticated and from there,how to transact on the blockchain using their blockchain identity.
