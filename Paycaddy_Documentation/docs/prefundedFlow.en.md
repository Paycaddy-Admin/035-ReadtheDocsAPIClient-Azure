The Prefunded Flow in PayCaddy allows you to maintain and manage funds within user wallets before transactions occur. Each wallet can be associated with one or more cards, and serves as a container for electronic money that powers payment transactions. This chapter provides an overview of how this flow works, how transactions are authorized and settled, and how funds are placed into (and withdrawn from) wallets.

## Overview

1. **User Wallets**  
    Each user can have one or more wallets holding electronic money. These wallets are the source of funds for payment cards.
    
2. **Association with Cards**  
    A wallet may be associated with one or more cards. When a card transaction occurs in the open loop environment (e.g., at a merchant), PayCaddy’s approval engine checks the linked wallet to confirm sufficient balance.
    
3. **Transaction Authorization & Notifications**
    
    - **Authorization Check**: On a purchase, the system checks the wallet’s balance. If enough funds are available, the transaction is authorized; the amount is retained or captured, depending on the transaction type.
        
    - **Transaction Notification**: Once a transaction is authorized, a webhook notification is sent to the callback URL registered with PayCaddy via the _Notification Enlist POST_ endpoint.
        
4. **Settlement & Adjustments**
    
    - **Settlement Notification**: Settlement is processed asynchronously in a batch. When settlement completes for each transaction, PayCaddy sends a settlement notification to the same callback URL.
        
    - **Adjustments**: The final settled amount may differ from the initially authorized amount. Any changes in the settlement amount are also included in the settlement notification (e.g., tips or currency fluctuations).
        

## Funding a Wallet

A wallet must have sufficient funds to cover transactions. Funds can be placed into a wallet in two main ways:

1. **Direct PayIns**  
    Use the PayCaddy _PayIn_ endpoint to add funds directly to an active wallet. This method instantly makes the declared funds available for wallet transactions. Ultimately, PayCaddy will settle these PayIns offline through a daily ACH or wire transfer process.
    
2. **Transfer from a Company Wallet**  
    This is the recommended approach for clients who wish to maintain a single “Company Wallet” and distribute funds to individual user wallets as needed. Centralizing PayIns into the Company Wallet first allows for streamlined reconciliation and improved traceability of funds. You can then use the Transfer method to move the necessary amounts to each user’s wallet.
    

The details on how to utilize the PayIn method are found on our [Wallet Operations](wallet_ops.en) chapter.
### Daily Settlement of PayIns

On a daily basis, PayCaddy consolidates all PayIns executed the previous day and sends a detailed report to the client. Settlement is then completed via ACH or wire transfer. This ensures that all direct PayIns made available for user spending are properly settled. You can find a detailed explanation of this process in our [Settlement Chapter](settlement.en)

## Eliminating Funds (PayOut)

To remove funds from a wallet, use the _PayOut_ method. A PayOut transaction reverses the flow, allowing you to extract funds and send them to an external bank account or another designated destination. The totality of PayOuts executed in a day is subtracted from the offline PayIn settlement process.