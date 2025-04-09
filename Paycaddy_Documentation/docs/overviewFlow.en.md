There are several achievable functional flows in our PayCaddy API. They can be categorized by the affected entity or the functional outcome. Essentially all Card Programs require a combination of these flows described below:

[**User Creation Flow**](userFlow.en.md)
Enables the registration and KYC verification of new users associated to a particular Card Program. Users can be Natural Persons or Businesses and the KYC verification can be either *Integrated*, or *Delegated*.

>*Integrated KYC* flows utilize PayCaddy's solution for KYC data capture and user activation is conditional to programatic verification through a 3rd party identity verification provider.

>*Delegated KYC* flows enable regulated financial entities to lead KYC data capture and register users that are created active by default


[**Card Creation Flow**](card.en.md)
Enables the creation of Prepaid, Debit or Credit cards associated to a particular user, wallet and card product specification. This flow also encompasses the initial activation of physical cards.


[**Prefunded Transactions Flow**](prefundedFlow.en.md)
This flow encompasses the funding of wallets through [wallet operations](wallet_ops.en.md), the approval of open-loop transactions, the enlisting of a Callback URL and the notifications sent via webhook to said address.

[**Just-In-Time Funding Transactions Flow**](JITllow.en.md)
Enables external authorization of card transactions. This flow describes the requirements and specifications for the authorization webservice that needs to be made available, as well as the authorization request schemas and consequential transaction event notifications.

---

At a high level, the beginning-to-end from the creation of a user to the reception of transaction notifications can be seen in the diagram below (Prefunded Logic Depicted):

![generalFlow](./assets/imgs/generalFlow.svg){class="img"}

>For Just-In-Time Funding programs, PayIns won't be utilized and the transaction approval flow will follow a distinct path. For more details see [**JIT Flow**](JITflow.en.md)



---
