The goal of this ARC is to allow asset transfers without the receiver having to explicitly opt-in first.

# Warning 
This code is not fully tested nor has it been audited. Right now it simply serves as a proof of concept.

# Processes

## Definitions

**OPEN_OPT_IN_PROGRAM** - The program for the logic signature that can be used by anyone to opt the signing account into assets ([source](./contracts/open_optin_lsig.teal))

**OPEN_OPT_IN_SIGNATURE** - The signature of **OPEN_OPT_IN_PROGRAM**

**ADDRESS_OPT_IN_PROGRAM** - The program for the logic signature that can be used by one specific account to opt the signing account into assets ([source](./contracts/address_optin_lsig.teal))

**ADDRESS_OPT_IN_SIGNATURE** - The signature of **ADRESS_OPT_IN_PROGRAM** for a specific account

**VERIFIER_LSIG** - An lsig whos sole purpose is to verify a given **OPEN_OPT_IN_SIGNATURE** or **ADDRESS_OPT_IN_SIGNATURE** against the authorization address of a given account ([source](./contracts/verifier_lsig.teal))

**DELEGATED_OPTIN_APP** - A single app deployed on an Algorand network used for storing signatures, verifying delegated opt-ins, and setting expiration times ([source](./contracts/delegated_optin_app.algo.ts))

**DYANMO_DB** - A centralized server containing a mapping of addresses to signatures


## Wallet Onboarding
This is the process for onboarding new users in a wallet (ie. Defly, Pera).

### Steps

1. Wallet generates keypair as per usual
2. Wallet prompts user if they want to enable deletgated opt-ins
3. If yes, the wallet will sign **OPEN_OPT_IN_PROGRAM** with the generated sk
   1. This can be a single click
4. Wallet sends address and lsig signature to **DYANMO_DB**

## On-Chain Signature Storage
This is the process for a user adding their **OPEN_OPT_IN_SIGNATURE** to box strorage of the master contract. This can be done at any time by any account.

### Transaction Group

1. MBR Payment: Payent to **DELEGATED_OPTIN_APP** to cover cost of storing **OPEN_OPT_IN_SIGNATURE** in a box 
2. **VERIFIER_LSIG**: Any type of transaction from **VERIFIER_LSIG**
3. **DELEGATED_OPTIN_APP**: `setOpenOptInSignature`
   1. *sig* - **OPEN_OPT_IN_SIGNATURE**
   2. *acct* - The account for which we are adding the **OPEN_OPT_IN_SIGNATURE**
   3. *authAddr* - The auth address of the aforementioned account

## Open Opt-In

This is the process for initiating a delegated opt in using the **OPEN_OPT_IN_PROGRAM**.

### Steps
1. Attempt to read signature from box in **DELEGATED_OPTIN_APP**
   1. If missing or invalid, attempt to read signature from **DynamoDB**
2. Send transaction group below

### Transaction Group
1. MBR Payment - A payment transaction from the sender to the account opting in that covers the ASA MBR (0.1 ALGO)
2. Opt In - A opt-in transaction signed by **OPEN_OPT_IN_PROGRAM** with **OPEN_OPT_IN_SIGNATURE**
3. **DELEGATED_OPTIN_APP**: `verify`

## Setting End Time
This is the process for an end user setting an end time for their delegated opt ins. Any opt ins after this time will get rejected.

### Transaction Group

1. **DELEGATED_OPTIN_APP**: `setOpenOptInEndTime`
   1. *timestamp* - uint64 timestamp

# Open Questions

* Should the first sender be required to put the signature in the box?