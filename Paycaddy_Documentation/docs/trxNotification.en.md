## **Transaction Notifications**

Each time a card issued in the NeoBank API makes a transaction on the Mastercard network, a payload like one of the four types of transactions exemplified below will be sent to the enlisted URL.

=== "AuthorizationRequest"
    ```json
    {
        "password": "password",
        "c1Tipo": "PeticionAutorizacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```

=== "AuthorizationComm"
    ```json
    {
        "password": "password",
        "c1Tipo": "ComunicacionAutorizacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```
=== "CancellationComm"
    ```json
    {
        "password": "password",
        "c1Tipo": "ComunicacionAnulacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```

=== "Reversal Request"
    ```json
    {
        "password": "password",
        "c1Tipo": "PeticionDevolucion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```

## **Main Type of Transactions**

There are three main types of transactions that you will receive as online type:

1. **AuthorizationRequest:** This transaction will decrease the customer's balance.
2. **AuthorizationCommunication:** This transaction will also decrease the customer's balance.
3. **CancellationCommunication:** This transaction will increase the available funds in the customer's wallet.
4. **ReversalRequest:** This transaction type declares a request for a money reversal that will be processed offline through the Batch Process. This transaction will have a declared amount of "0". Once the network confirms the refund, the processed amount will be declared in the related "TransaccionCorregidaPositiva" through the Batch Process.

The transaction type is indicated in the **"c1Tipo"** field of the webhook. For example, if the **"c1Tipo"** field indicates that it is a **"AuthorizationRequest"** type transaction, it should be presented as a deduction in the available balance.

>It's important to note that Transaction Notification webhooks are sent with attributes and values named in Spanish.
This documentation translates some terms for easy comprehension.

>The field **"c7FechaHoraTransaccion"** represents the transaction time information in the timezone where the transaction was acquired in the format "YYYYMMDDhhmmss"

>The field **"BDateUtcCreate"** represents the transaction time information in UTC timezone following the same format as the **c7FechaHoraTransaccion**

**Additional Types of Transaction Notification**

There are 9 additional types of transactions that you will receive online through the enlisted URL. It should be noted that these types do not affect balance and are merely informative of the card usage and describe the cause for rejected transactions.

These types will be seen as values in the **"c1Tipo"** field of the webhook notification:

1. **DENEGADA:** The transaction was declined for an unspecified reason.
2. **DENEGADA. PIN INCORRECTO:** The transaction was declined because the entered PIN is incorrect.
3. **DENEGADA. INTENTOS PIN EXCEDID:** The transaction was declined because the maximum number of PIN entry attempts has been exceeded.
4. **DENEGADA. TARJETA NO EFECTIVA:** The transaction was declined because the card is not valid or not active.
5. **DENEGADA. INCORRECTO CVV2:** The transaction was declined because an incorrect CVV2 was entered.
6. **IMPORTE SUPERA LIMITE:** The transaction was declined because the transaction amount exceeds the allowed limit for the card.
7. **NO HAY FONDOS:** The transaction was declined because the account associated with the card does not have sufficient funds.
8. **EXCEDIDO NUMERO DE OPERACION DIARIO:** The transaction was declined because the maximum number of daily operations allowed for the card has been exceeded.
9. **FECHA CADUCIDAD ERRONEO:** The transaction was declined because an incorrect expiration date was entered.
10. **NotificacionDenegacion:** These won't typically be seen in received webhooks but instead will be found when using the **TransactionDetailList** endpoint. They represent a transaction that has been declined due to insufficient funds.

In the schema of the JSON sent in these most of these additional types of notifications, the fields **"c38NumeroAutorizacion"** and **"c11NumeroIdentificativoTransaccion"** are present for customer convenience, however, it is important to mention that these fields will not always contain information and, therefore, will be presented as empty strings ("").

```json
    {
        "password": password,
        "c1Tipo": "DENEGADA. INCORRECTO CVV2",
        "c2CardId": cardId,
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
```

---

## **MoneySend Transactions**

To stay informed about the specifics of an initiated MoneySend transaction, PayCaddy provides webhook notifications. These webhooks follow the same structure and travel through the same URL as other transaction notifications. However, MoneySend transactions can be identified by the **c3CodigoProceso** value of **"820000"**.

```json
    {
        "password": "password",
        "c1Tipo": "PeticionAutorizacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "820000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
```

When handling MoneySend webhook notifications, store them like any other transaction notification. The main difference is in how you process the transaction.

>MoneySend transactions with **c1Tipo** equals to **“PeticionAutorizacion”** must be handled as **positive** transactions by adding funds to the recipient's wallet balance instead of deducting funds.

>MoneySend transactions with **c1Tipo** equals to **“ComunicacionAnulacion”** must be handled as **negative** transactions by subtracting funds from the recipient's wallet balance instead of deducting funds.

When enlisting card or wallet transactions on the front-end, this should be reflected by showcasing the amount of said transactions as inputs to the wallets funds instead of charges.

---

## **Batch Process**

>**Notifice of Update:** As of April 11, the functionality of our Batch Process has been expanded to include the "TransaccionConfirmada" webhook notification. This addition ensures comprehensive oversight over all transaction settlements, whether they match the initially authorized amounts or require adjustments.

In order to handle the completion of a transaction's lifecycle, we utilize a Batch Process to update and finalizes the lifecycle of each approved transaction, enabling fast visibility over actual settlement data in you card issuing program.

PayCaddy leverages this batch processing to deliver notifications through the conventional transactional webhook route (see NotificationEnlist POST) as part of a daily process to confirm and-or adjust the amount previously approved for each transaction.

Our system utilizes three key types of webhook notifications to manage various transaction outcomes:

1. **TransaccionCorregidaPositiva:** Issued when the final settlement results in a positive adjustment to the initially authorized amount, thereby adding funds to the user’s wallet.
2. **TransaccionCorregidaNegativa:** Issued when the final settlement results in a negative adjustment, thereby deducting funds from the user's wallet.
3. **TransaccionConfirmada (New):** Introduced to confirm that the transaction has settled exactly at the amount authorized initially, providing clear and precise confirmation of the transaction outcome.

A couple of examples of transactions that could be corrected through a batch process are:

1. **Car rental:** When a customer rents a car, the rental company usually authorizes an initial amount on the customer's card to cover the cost of the rental and any possible damage. However, the final amount could vary depending on the actual use of the vehicle and other factors, such as the distance traveled or the cost of fuel. In these cases, the corrective message "TransaccionCorregidaPositiva" or "TransaccionCorregidaNegativa" could be used to adjust the final amount charged to the customer's card, once the vehicle has been returned and the final calculation has been made.
2. **Hotel bookings:** When a customer reserves a hotel room, the hotel usually blocks an amount on the customer's card to cover the cost of the stay and possible additional charges, such as the minibar or room service. At the end of the stay, the final amount may be different from the initially blocked amount, depending on the actual use of these additional services. In these cases, the corrective message "TransaccionCorregidaPositiva" or "TransaccionCorregidaNegativa" could be used to adjust the final amount charged to the customer's card, once the final charges have been calculated.

Batch processing streamlines the handling of deferred transactions and ensures proper settlement, reducing discrepancies and the need for manual reconciliation. These webhooks improve transaction handling, minimize errors, and optimize overall management.

>The majority of webhooks related to these corrective transactions will be sent during the same period overnight.
This is because our webhook generation process is a batch process that runs overnight to consolidate and notify if there were changes to the day's transactions.

Whenever possible, it is recommended to match the corrected transaction with the original transaction. This way, cardholders will have a better understanding of how and why their balance has been modified.

This practice improves transparency and fosters customer trust by allowing them to be aware of all transactions and changes made to their accounts. It is essential that cardholders feel informed and in control of their financial activity.

=== "TransaccionCorregidaPositiva"
    ```json
    {
        "password": "password",
        "c1Tipo": "TransaccionCorregidaPositiva",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```
=== "TransaccionCorregidaNegativa"
    ```json
    {
        "password": "password",
        "c1Tipo": "TransaccionCorregidaNegativa",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```
=== "TransaccionConfirmada"
    ```json
    {
        "password": "password",
        "c1Tipo": "TransaccionConfirmada",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```

**Transaction Matching and Reporting**

- **Matching Transactions:** Each transaction includes a unique identifier (**c11NumeroIdentificativoTransaccion**), which is crucial for matching the corrected or confirmed settlement with its initial authorization. This matching process is essential for maintaining transaction integrity and provides cardholders with a clear understanding of how and why their balances were adjusted.
- **Notifications Details:** Webhook notifications furnish detailed information about the transaction, including the transaction amount, the identifier, and the type of adjustment or confirmation. This ensures all parties are accurately informed of the transaction status.
do this with the information that travels through the **"c11NumeroIdentificativoTransaccion"** field.
> Once a transaction has been modified, it will not be modified again

Although it is less common, it can also occur that a charge arrives only in the batch process and not online. If this is the case, the webhook may arrive with less information as presented below. It should be noted that this information is according to the network's protocol for handles these confirmations and does not depend on PayCaddy.

---
