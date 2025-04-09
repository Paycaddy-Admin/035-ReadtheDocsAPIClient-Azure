PayCaddy's API has differentiated calls for the creation of debit, credit, and prepaid cards. Through these calls, it is possible to create a physical or virtual card for an existing **UserID** linked to the available balance in a specific **WalletID** of that user.

All cards are created using a unique parameterization code for each card product, which must be previously requested from the PayCaddy integration team.

The creation of a card starts with the post [Debit Card POST](card.en.md#debit-card-post), [Credit Card POST](card.en.md#credit-card-post) or [Prepaid Card POST](card.en.md#prepaid-card-post) call, depending on the acquired issuance service.

>It is important to consider the type of user for which the card has been parameterized. Cards parameterized for natural persons can only be created by associating them with UserIDs representing EndUsers or EndUserSRs, while those parameterized for legal entities can only be created by associating them with MerchantUsers.

Once you have been granted a card creation code, you can begin testing card creation in the test environment.

![cardflow](./assets/imgs/cardFlow.svg)



>Physical cards are created in a default status of `PendingAck`for security during the logistics of delivery. See [Ack Reception](card_ops.en.md#ack-reception-post) for details on physical card activation.