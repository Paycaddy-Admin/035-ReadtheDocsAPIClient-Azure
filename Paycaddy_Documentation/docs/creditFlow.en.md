As an alternative to the debit-based system, the credit core gives you the flexibility to offer credit lines to your users based on your own criteria. This feature allows you to configure and manage credit lines to individual user profiles.

With the credit core, users are assigned credit wallets linked to a specific credit line settings. From there, they can begin making transactions using their available credit, just as they would with a balance in a debit wallet.

---
## Credit Products

It is the heart of our credit core as it is what governs how  each credit line will behave over time. Think of it as the "rule-book" that defines the terms and conditions of credit usage for your users.

Since each credit wallet issued to a user is tied to a credit product, which ensures consistent behavior across similar credit lines and allows you to manage risk, profitability and flexibility with precision and granularity.

They are provided by PayCaddy based on agreements/requests with the clients previously. You can have as much credit products as it considers necessary although every credit product request is reviewed internally to avoid conflicts or misunderstandings on how it scales over time.

It is build to be as granular as possible, you can control from basic settings as grace period, cut frequency and interest rate. Until the more specific ones such as base minimum payment rate, if it is revolving or not, the capital interest split and etc...

---
## Credit Notifications

Notification from our credit core, just as we do on debit-based system, are send via webhooks. We provide you a easy way to inform a URL on (see [NotificationsEnlist](NotificationsEnlist.en)). Based on this notifications you can be up to date with changes that may happen over the usage of the credit lines in each wallet. 

> It is crucial that the **URL** provided is able to receive it properly since there is valuable information on each one of them.

!!!Warning
	We highly recommend to use a different **URL** for credit notifications 

The most common notifications that are send is going to be the informs of interest generated/executed and cancelling of interest (which happens every-time a report of payment is made). But, there is some others that are send when an action happens, such as:

* Creation of a credit wallet;
* Change on wallet's limit;
* Change on specs of the credit product tied to a specific wallet;

---

## Wallet Creation

Once you already have a credit product and a URL set to receive credit notifications, we are set to start to create our first wallet.

Creating a credit wallet is very similar to create a debit wallet, it basically flows the some core principles, but with a few additional field to account for credit-specific logic.

Just like a debit wallet the fields: **userId, currency and description are required.**

However, because credit wallets are governed by time-based rules and borrowing limit, you'll also need to define: 

- The start date of the credit cycle (used to determine due dates and billing periods)
- The total credit limit available to the user
- The amount of time (in days) that the wallet will be active (meaning that once the time has passed, the wallet can no longer trade)
- The identifier for the credit product that regulates the wallet's behavior (e.g. interest, grace period, penalties, etc...)

P.S: The **code** is provided to you once the credit product requirements is approved by PayCaddy.

It's important to consider that a credit wallet won't be able to execute any pay ins, pay outs or transfers, since it will never have a **balance** if not, it will only have an limit and a counter of their expenses.

---
## Credit Usage

Once everything is set, the users will be able to consume their credit lines through their credit cards, previously created and related to a credit wallet. It works as any other credit program, they will be able to make purchases until they reach their wallet's **limit** on the current billing cycle. All purchases are deducted from the **available credit**, not from a stored balance.

Although, you can manage it in two ways:

- **Report a Payment:** It restores the amount reported to the **available credit**. It can either be partial or complete;
- **Increase the Wallet Limit:** The credit limit itself can be increased any time, allowing for a higher spending threshold if you thinks this is the best way to go.

When a cycle is completed, is expected for each credit wallet that had expenses, to receive a report pay. The report pays are the way to restore the limit from a wallet and pay it's interests that may be generated due its use.

When we don't find any report on the period between the end of the cycle and the end of the **grace period** (which is defined by the credit product specs), we consider that the wallet is currently debtor.

A debtor wallet is charged besides the regular interests, the debtor ones, which are incremented to the already existing interests. So, it means that it scales quickly as long as it takes for us to receive a report.

A credit wallet is no longer considered to be in debtor status once we receive a reported payment that meets or exceeds the minimum required amount (it's defined by the credit product) based on the interest owed + its expenses.

---

## Credit Operations

Our credit core allows you to manage yours users wallets as you like. Using a few endpoints provided you will be able to block/unblock temporary, change a wallet limit and dissolve a wallet.

Its important to understand the difference between block and dissolve a wallet, cause it may lead into misunderstandings. 

- A blocked wallet will stop your users to make transactions with cards related with that wallet, it doesn't stops the generation of new interests, neither ignores any debt that it may already have. Also, it can be undone any time you want;
- On the other hand, a dissolved wallet can no longer be used period. It won't generate any new interest although is still expected to receive a report pay for the debt that still may exist.

When changing the wallet limit, is important to taking in consideration that the amount can only goes **up**. We don't allow to decrease the limit already provided to a wallet.

### Change Interests

We also offer flexibility to tailor specific configurations at **wallet level**, without affecting the underlying product or other wallet tied to it. It means you can override certain settings from the credit product **specifically for an individual wallet**, allowing you to respond to unique user needs or exceptions.
>Please note: This customization is only possible if the associated credit product is explicitly configured to allow wallet-level overrides
