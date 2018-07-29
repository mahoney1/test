
# Access Control Tutorial (Advanced) {{site.data.conrefs.composer_full}}  - Dynamic ACLs

Understanding how ACLs are evaluated is a key part of implementing an access control rule strategy. In the Hyperledger Composer [Access Control tutorial](https://hyperledger.github.io/composer/latest/tutorials/acl-trading) we see how to incrementally implement smart contract rules that restrict controls based on ownership or responsibility for particular resources such as target assets and what they are authorized to do.

In this tutorial, we will explore, in Hyperledger Composer authorization rules,  how to restrict transaction execution to two parties to a transaction - and which is 'locked down' at the point the transaction is submitted.

In our example below, we have an Account Trading settlement business network/smart contract, which has Account Traders that are owners of bank accounts. These bank accounts can only be updated (eg. a withdrawal or deposit of a specific amount) by specific authorised transaction types - furthermore, it is 'locked down' to the two identities (mapped to the Account Traders) involved. As part of the audit trail, we will write a transaction record to the Bank Account transaction list.

Figure 1 ![Overview of Trade Settlement Network rules](../assets/img/tutorials/acl2/define_settle_acl.png)


The tutorial uses the online Playground to try out our sample access rules. In completing the tutorial, you will interact with the sample network as various identities - ultimately,  it is the users of the blockchain that we want to apply access control to. 

If you wish, you can also apply the rules in this tutorial against an existing v1 Fabric. You just need to grab and deploy the sample Account settlement business network shown below, create a Business Network archive and deploy the network - then you're ready to start working with that environment and add the rules / data as shown below.


## Prerequisites

None - just an internet connection, which you have right now :-) 

## Step One: Access the Online Playground and select your business network


1. Go to the [Online Playground](https://composer-playground.mybluemix.net/login) and if necessary clear local storage when prompted. Accept the Welcome logo, let's blockchain - you are ready to start.

2. Click on the `Deploy a new business network` modal / icon. Choose to deploy an 'empty business network' and enter a name of `account-trade` and description of `Account Settlement`.  

3. Enter a default network admin card name as `admin@account-trade`

4. Confirm that the network name is `account-trade` - click on **Deploy** to deploy the business network. It is deployed.

5. Next, on the 'My Business Networks' panel - click on 'Connect Now' to connect to the deployed business network (as id `admin@account-trade` - its shown top left).


## Step Two: Define your Business Network README

1. The 'README file should be active but blank - click on the '>' (right-chevron) character on thetop right to edit the README - delete any current contents and paste the following into the README

```

# Sample Account Trader Settlement Network

# Sample Trader Settlement Network

> This is a sample Trader Settlement network. Allows balances to be updated directly between Trader Bank Accounts via an agreed transaction mechanism, such that only Source and Target Trader accounts are updated on the ledger.  Source and target Trader Bank accounts are updated via a TransferAmount transaction. We know the current participant (invoking the transaction), and we specify the targetAccount as a relationship field in the transaction _TransferAmount_.  At all other times, the Bank Accounts are locked down to potentially spurious unauthorised transaction updates, as far as the ledger is concerned.

This business network defines:

**Participant**
`AccountTrader`
`Bank`

**Asset**
`BankAccount`

**Transaction**
`TransferAmount`


Banks are securers of BankAccounts. 
AcccountTrader is the owner of a BankAccount 
Balance on a BankAccount for an owner can be updated by a TransferAmount transaction - specifying an operation (D deposit, W withdrawal) to execute.  A transfer of type 'D' deposits the amount on the Target Account ; and deducting from the source Account. A transfer of type 'W' withdraws the specified amount from the Target, adding to the source Account balance. Transaction history is also recorded on each BankAccount.

To test this Business Network Definition in the **Test** tab:

Create an `AccountTrader` participant:


{
  "$class": "org.acme.account.AccountTrader",
  "userID": "nick1",
  "firstName": "Nick",
  "lastName": "Hunter"
}

{
  "$class": "org.acme.account.AccountTrader",
  "userID": "jon1",
  "firstName": "Jon",
  "lastName": "Miller"
}


Create a `Bank` participant:


{
  "$class": "org.acme.account.Bank",
  "bankID": "Hall_1",
  "description": "the world-friendly bank currently known as Hallelujah",
  "BankAccount": "[1,2]"
}




{
  "$class": "org.acme.account.BankAccount",
  "accountID": "1",
  "balance": 100.00,
  "transactions": [
    {
      "$class": "org.acme.account.TransferAmount",
      "amount": 100.00,
      "operation": "D",
      "sourceAccount": "resource:org.acme.account.BankAccount#100",
      "targetAccount": "resource:org.acme.account.BankAccount#1"
    }
  ],
  "owner": "resource:org.acme.account.AccountTrader#nick1"
}

"$class": "org.acme.account.BankAccount",
  "accountID": "2",
  "balance": 80.00,
  "transactions": [],
  "owner": "resource:org.acme.account.AccountTrader#jon1"
}


To submit an `TransferAmount` transaction:

{
  "$class": "org.acme.account.TransferAmount",
  "amount": 30.49,
  "operation": "W",
  "sourceAccount": "resource:org.acme.account.BankAccount#1",
  "targetAccount": "resource:org.acme.account.BankAccount#2"
}


After submitting this transaction, you should now see the transaction in the Transaction Registry (and in each Bank Account history).

Congratulations!

```

2. Click on the 'eye' icon (again, top right) to return to the main playground screen. Then click on UPDATE bottom left.


### Step Two: Add the Sample Account Trading Settlement model file

1. Click on the 'Add a file' option and choose to add a Model file when prompted. Clear any existing model file entry on the screen and paste the following into this page.

```
/**
 * Sample Account Trading business network definition.
 */
namespace org.acme.account

participant AccountTrader identified by userID {
  o String userID
  o String firstName
  o String lastName
}    

participant Bank identified by bankID {
  o String bankID
  o String description
  --> BankAccount[] accounts optional
}

asset BankAccount identified by accountID {
  o String accountID
  o Double balance
  o TransferAmount[] transactions optional
  --> AccountTrader owner
}

transaction TransferAmount {
  o Double amount
  o String operation
  --> BankAccount sourceAccount
  --> BankAccount targetAccount
}
```

### Step Three: Add a Transaction JS script file

We need to add the corresponding smart contract logic that specifies what assets need updating - our transaction logic will either initiate a 'Withdraw' (W) or 'Deposit'(D) type transaction, and the two BankAccounts that are involved with be debited and credited accordingly. A transaction history is also written as a transaction on the Account (as well as the fact that the history of the transaction and changes are written to the Historian out on the blockchain ledger).

1. Delete the current contents in `lib/script.js` file on screen and add the following transaction logic file pane:

```
/*
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * Perform a deposit or withdrawal from a bank account
 * @param {org.acme.account.TransferAmount} transaction
 * @transaction
 */

function execTxfr(transaction) {   
    // initialize array of transactions if none exist

   var curr = getCurrentParticipant();
  
    //console.log('source object is ' + transaction.sourceAccount) ;
    //console.log('target object is ' + transaction.targetAccount) ;
    //console.log(' identifier is ' + curr.getIdentifier() );
    //console.log(' bank id is ' + transaction.targetAccount.getIdentifier() ) ;
  
    if(transaction.sourceAccount.transactions == null) {
        transaction.sourceAccount.transactions = [];
    }

  if(transaction.targetAccount.transactions == null) {
        transaction.targetAccount.transactions = [];
    }
  
    // determine whether this is a deposit or withdrawal transaction and execute balance changes accordingly

    if(transaction.operation == 'W') {
        transaction.sourceAccount.balance -= transaction.amount;
        transaction.targetAccount.balance += transaction.amount;
      
    } else if(transaction.operation == 'D') {
        transaction.sourceAccount.balance += transaction.amount;
        transaction.targetAccount.balance -= transaction.amount;
    }

    // add the current transaction to the bank account's transaction history

    transaction.sourceAccount.transactions.push(transaction);
    transaction.targetAccount.transactions.push(transaction);
  
    
    // update the BankAccount registry with changed asset values (amounts) simultaneously

    return getAssetRegistry('org.acme.account.BankAccount')
    .then(function(regBankAccount) {
    return regBankAccount.updateAll([transaction.sourceAccount, transaction.targetAccount]);
    //  return regBankAccount.update(transaction.sourceAccount);
    });
}

```
 
Note: Included in the script file are some `console.log()` statements which you can review by doing CTRL + SHIFT + J to see in the Developer console (javascript JS pane) and seeing the values being evaluated.

### Step Four: Verify Default ACL rules for Trade Settlement network

1. Click on the `permissions.acl` file to verify that we have just 1 'system' and 2 'network' (admin, user) rules - remove any other rules relating to `org.acme.account` namespace at this time. The rules in `permissions.acl` should be exactly as follows:

```
rule SystemACL {
  description:  "System ACL to permit all access"
  participant: "org.hyperledger.composer.system.Participant"
  operation: ALL
  resource: "org.hyperledger.composer.system.**"
  action: ALLOW
}

rule NetworkAdminUser {
    description: "Grant business network administrators full access to user resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "**"
    action: ALLOW
}

rule NetworkAdminSystem {
    description: "Grant business network administrators full access to system resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}

```

Finally, we should be good to go and be able to click on the 'Update' button to deploy our new business network. Verify that the deployment was successful. All of the testing will be done in the online Playground in memory - as mentioned before, you could optionally deploy this network to a local Playground and Fabric and try it out there.



### Step Five: Create Account Trader Participant data to test ACL rules against

1. Click on the 'Test' tab near the top of the screen. This is where we create sample Trader participants. 

2. Click on `AccountTrader` on the left - Create New Participants (top right) as follows:

    {
      "$class": "org.acme.account.AccountTrader",
      "userID": "1",
      "firstName": "Jenny",
      "lastName": "Jones"
    }
   

   
    
3. Repeat step 2 and create 3 additional `AccountTrader` participants ('2' through '4') using the sample data below.


  {
      "$class": "org.acme.account.AccountTrader",
      "userID": "2",
      "firstName": "Jack",
      "lastName": "Sock"
   }
   
   {
      "$class": "org.acme.account.AccountTrader",
      "userID": "3",
      "firstName": "Rainer",
      "lastName": "Valens"
   }
   
      
   {
      "$class": "org.acme.account.AccountTrader",
      "userID": "4",
      "firstName": "Lars",
      "lastName": "Graf"
   }
   
    

Figure 2 ![Account Trader records added](../assets/img/tutorials/acl2/traders.png)

### Step Six: Create Bank and BankAccount Assets

1. Still in the 'Test' panel, create one Bank Record
   
   
   {
    "$class": "org.acme.account.Bank",
    "bankID": "1",
    "description": "Bank of Hallelujah"
   }
   
2. Create 4 BankAccount Asset records (resources) as follows:


  {
     "$class": "org.acme.account.BankAccount",
     "accountID": "1",
     "balance": 100,
     "owner": "resource:org.acme.account.AccountTrader#1"
  }
  
  
  {
     "$class": "org.acme.account.BankAccount",
     "accountID": "2",
     "balance": 100,
     "owner": "resource:org.acme.account.AccountTrader#2"
  }
  
  
  {
     "$class": "org.acme.account.BankAccount",
     "accountID": "3",
     "balance": 100,
     "owner": "resource:org.acme.account.AccountTrader#3"
  }
  
  {
    "$class": "org.acme.account.BankAccount",
    "accountID": "4",
    "balance": 100,
    "owner": "resource:org.acme.account.AccountTrader#4"
  }
  
  
Note that the `owner`(above) relates back to the 'AccountTrader' participant for the purposes of this tutorial and it is a relationship field.
  
     

### Step Seven: Create Identities to test ACLs

Next, lets create some trader identities - we need to issue identities for the Trader participants (numbers 1 - 4 created earlier) so that we can test those identities' access (each being mapped to their respective Trader participant record)

1. Click on 'admin` (top right) and select 'ID Registry' from the drop-down
2. Click 'Issue new ID' top right and it will present an 'Issue New Identity' dialog
3. In the ID Name field - enter `tid1` as the identity we'll use for the first AccountTrader with id '1'.
4. In the Participant field simply enter  `1` for typeahead search - and you'll see that AccountTrader (participant '1' shows - and then select that as the fully-qualified participant name to map to this identity.
5. Click on 'Create New'  to continue.

Repeat the 'Issue new ID' sequence (step 2 through 5 above) for identities `tid2`, `tid3`, `and tid4` respectively, mapping these to their respective AccountTrader participants 2, 3 and 4.
  
 Now we're ready to start creating our access control rules.

**Important**: if you are issuing new identities (in Playground) for a Fabric-based environment (as opposed to the online environment above), make sure (ie in 'ID Registry') to add each issued identity to your wallet using the'Add to Wallet' option.

### Step Eight: Add Account Trading settlement network access control rules

Our standard business network defines some System and Network ACL rules, that govern the administrators of the business network.

Next we want to add some Account Trading settlement network access control rules - let's start by defining what we want to achieve first ! The golden rule with ACLs is that access to resources inside a business network are by default implicitly 'DENIED' to Participants, unless explicitly ALLOWED.

You will note from reviewing the current ACLs in `permissions.acl` that certain 'system' or 'administrator' type rules are defined in the ACLs file - this is to allow participants to be able to use Composer system operations such as being able to write to the Composer system Historian registry. Please ensure you do not remove these two rules.


In terms of our rule objectives - we want the Account Traders to have nimimal privileges defined below.

1a. Account Traders can see and update their own profile only (participant record)
1b. Allow Traders access to their own BankAccount only

2. Restrict Participants of type 'Trader' such that only these Participant types can submit `Trade` transactions 

3. Allow a transaction `TransferBal` to be able to update a bank account balance on the source and target accounts in the asset registry and therefore being able to lockdown the BankAccount asset records dynamically..


#### Step Nine: Define Rule 1a - Trader 'self' restriction rule

First up - rule to restrict Traders to only see and update their own record.

1. Switch identity to `tid1` (click the current identity top right and choose ID Registry, select to 'use now' for `tid1`) - and click on the 'Test' tab
2. Confirm that you do not see any Trader records.
3. Switch identity to the 'admin' user (top right, 'ID Registry'), then go to the 'Define'  tab and click on 'Access Control' (`permissions.acl`) on the left.
4. Paste the following rule in line 1 in your edit session, pasted above the existing 3 'System' and 'Network' system rules:

```
rule TradersCanUpdateSelf {
    description: "Only Allow Account Traders to update self"
    participant(p): "org.acme.account.AccountTrader"
    operation: ALL
    resource(v): "org.acme.account.AccountTrader"
    condition: (p.getIdentifier() === v.getIdentifier())
    action: ALLOW
}
```
Then click on the **UPDATE** button on the bottom left to update the business network.  

This rule will allow the `current` Participant(p) (mapped to the `current` identity whether in playground (here) or indeed in your application) to READ and UPDATE their own target Trader record(v). 

5. **TEST THE ACL**: Switch user to identity `tid1` (top right, 'ID Registry') and click on the 'Test' tab - check that TRADER1 record only, is visible to this identity.


#### Step Ten: Define Rule 1b - Traders can update their own BankAccounts only

We need a rule that restricts Account Traders from having access to other Trader's accounts but their own on the ledger.

1. Switch identity to `admin` (click the current identity top right and choose ID Registry, then go to the 'Define'  tab and click on 'Access Control' (`permissions.acl`) on the left.

2. Paste the following rule in line 1 in your edit session, pasted above the existing rules:

```
rule OnlyAllowOwnersAccesstoAsset {
    description: "Traders see their own BankAccount only"
    participant(p): "org.acme.account.AccountTrader"
    operation: ALL
    resource(v): "org.acme.account.BankAccount"
    condition: (p.getIdentifier() === v.owner.getIdentifier())
    action: ALLOW
}
```
3. **TEST THE ACL**: Switch user to identity `tid1` (top right, 'ID Registry') and click on the 'Test' tab - check that BankAccount #1 record only is visible to this participant (ccountTrader 1, mapped to `tid1`.


#### Step Ten: Define Rule 2 - Trader TransferAmount (transaction) ACL rule

We need a rule that allows AccountTrader participant types to submit transactions on the business network. This is defined as a resource in the network.

1. Switch identity to `admin` (click the current identity top right and choose ID Registry, then go to the 'Define'  tab and click on 'Access Control' (`permissions.acl`) on the left.

2. Paste the following rule in line 1 in your edit session, pasted above the existing rules:

```
rule TradersCanSubmit {
    description: "Allow AccountTrader participants to submit TransferAmount txn type"
    participant: "org.acme.account.AccountTrader"
    operation: ALL
    resource: "org.acme.account.TransferAmount"
    action: ALLOW
}

```

Then click on the **UPDATE** button on the bottom left to update the business network. 

We will test this ACL as part of rule 3 below.

#### Step Eleven: Define Rule 3 - Dynamic ACL to restrict transaction to update BankAccounts to the two transacting parties.

We need a rule will allow only the participant identities to a transaction to update each others BankAccounts  : the transaction will know the originating participant and bank account  ; and it will know the destination bank account. This is all that is needed to allow a transfer of money to occur, but only via the transaction class `TransferAmount`

Paste the following rule into the rules file `permissions.acl`

```
rule TradersTransact {
    description: "Only Allow Update between Transferor/Transferee"
    participant(p): "org.acme.account.AccountTrader"
    operation: ALL
    resource(v): "org.acme.account.BankAccount"
    transaction(tx): "org.acme.account.TransferAmount"
   condition: ( p.getIdentifier() === v.owner.getIdentifier()  && v.getIdentifier() === tx.targetAccount.getIdentifier() )
 // condition: ( p.getIdentifier() === '3' && v.getIdentifier() === '2' ) 
    action: ALLOW
}

```
Note the // comment line in the middle - this is a literal rule, purely for test purposes (later on) and represents a commented (inactive) line at the present time.

You should see the message `Everything looks good` at the bottom.

Lastly, click on 'UPDATE' to update the business network with the new ACL rules.


#### Step Twelve: Lets TEST the ACL

5. Switch user to identity `tid1` (top right, 'ID Registry') - the owner of BankAccount `1`

a. Click on the 'Test' tab. Submit a _TransferAmount_ transaction (Submit Transaction button) copying and pasting this transaction, replacing current contents with the transaction provided below:

```
{
  "$class": "org.acme.account.TransferAmount",
  "amount": 1.01,
  "operation": "W",
  "sourceAccount": "resource:org.acme.account.BankAccount#1",
  "targetAccount": "resource:org.acme.account.BankAccount#2"
}
```
Figure 3 ![Six Trader records added](../assets/img/tutorials/acl2/submit_txn.png)


b. Click on the BankAccount asset on the left and confirm that BankAccount 1 is showing a balance of 98.99. Expand the record to see the transaction list (confirming the withdrawal). Of course, due to ACL restrictions, we can only access `tid's` account at this point in time.

Figure 4  ![BankAccount 1 after Rule 3](../assets/img/tutorials/acl2/after_rule3.png)

c. Switch identity to `tid2` (top right, 'ID Registry') - the owner of 'BankAccount#2' and check the balance shows 101.01. Expand the record to see the transaction list (confirming the deposit))

We can see the originating and target accounts in each listing. 

#### Step Thirteen: Modify the ACL Rule to check non-compliant transaction is rejected

OK - so how can we quickly test the ACL restriction we implemented? The answer is:  to use our comment line (denoted by `//`) in our ACL rule (see below). We can comment out the current `condition:` under rule 'TradersTransact' (locate this in the `permissions.acl` file - then do the following):

1. Switch identity to 'admin' in the ID registry, top right.

2. Comment (using '//') the current condition and uncomment the condition line below it such that the rule now looks like this:

```
rule TradersTransact {
    description: "Only Allow Updatee between Transferor/Transferee"
    participant(p): "org.acme.account.AccountTrader"
    operation: ALL
    resource(v): "org.acme.account.BankAccount"
    transaction(tx): "org.acme.account.TransferAmount"
//   condition: ( p.getIdentifier() === v.owner.getIdentifier()  && v.getIdentifier() === tx.targetAccount.getIdentifier() )
    condition: ( p.getIdentifier() === '3'  && v.getIdentifier() === '2' )
    action: ALLOW
}
```
We will now be able to test the rule by changing the condition to a literal value

3. Click on the `Update` button bottom left. We have now updated the ACL rules.

#### Step Fourteen: Test the newly modified ACL Rule

Next,  submit an _AccountTransfer_ transaction as untested identity `tid3` - this identity will become the current participant to the transaction (ie in terms of account ownership in our transaction JSON)' plus a targetAccount id #4 to be submitted in the transaction JSON (ie won't match the `targetAccount` portion of our literal rule condition).

1. Switch identity to 'tid3' in the ID registry, top right in Playground.

2. Submit a _TransferAmount_ transaction (copy the transaction JSON below) -note that the rule states that BOTH conditions must be met, for the update of the resources in the BankAccount registry to take place - all conditions must be met for the ACL will trigger (ie ALLOW a transaction of _TransferAmount_ to occur). Click `Submit Transaction`

```
{
  "$class": "org.acme.account.TransferAmount",
  "amount": 1.01,
  "operation": "W",
  "sourceAccount": "resource:org.acme.account.BankAccount#3",
  "targetAccount": "resource:org.acme.account.BankAccount#4"
}
```

You will get an Error saying "Error: Object with ID '4' in collection with ID 'Asset:org.acme.account.BankAccount'. While we are 'legitimately' submitting the transaction as the source BankAccount owner (the current participant AccountTrader #3) , we cannot fake the rule to be able submit a _TransferAmount_ transaction to a target Account that doesn't match the true owner and hence the ACL rule conditions are not met and therefore, is DENIED by default (and skips onto the next rule to evaluate). Furthermore, it also fails the subsequent rule  _OnlyAllowOwnersAccesstoAsset_ - (target participant is not the owner) - and a BankAccount ID '4'transfer is also DENIED (and hence the error is shown for #4 only - because #3 is allowed by this rule as the source BankAccount owner matches the current participant). The error means that the transaction as a whole fails - both Source and Target account remain as they were, before the invocation of this  _TransferAmount_ transaction attempt.

Figure 5 ![BankAccount 1 after Rule 3](../assets/img/tutorials/acl2/unauthorized_submit.png)

3. Switch identity to 'admin' and then change the `TradersTransact` rule back to the original condition state

```
rule TradersTransact {
    description: "Only Allow Updatee between Transferor/Transferee"
    participant(p): "org.acme.account.AccountTrader"
    operation: ALL
    resource(v): "org.acme.account.BankAccount"
    transaction(tx): "org.acme.account.TransferAmount"
  condition: ( p.getIdentifier() === v.owner.getIdentifier()  && v.getIdentifier() === tx.targetAccount.getIdentifier() )
 //   condition: ( p.getIdentifier() === '3'  && v.getIdentifier() === '2' )
    action: ALLOW
}

```

and click 'Update` to update the rules once again.

4. Switch identity back to `tid3`, and copy the same transaction above once again and submit the transaction 

5. Verify you are able to submit without any errors - and verify that the appopriate amounts are updated in the BankAccount asset on the left (switching identities between `tid3` and `tid4` to verify either asset values).

This is the end of the dynamic ACL tutorial.
