In order to handle the completion of a transaction's lifecycle, we utilize a Batch Process to update and finalizes the lifecycle of each approved transaction, enabling fast visibility over actual settlement data of each individual transaction in you card issuing program.

PayCaddy leverages this batch processing to deliver notifications through the conventional transactional webhook route (see [**NotificationEnlist POST**](./notificationsEnlist.en.md)) as part of a daily process to confirm and-or adjust the amount previously approved for each transaction.

Our system utilizes three key types of webhook notifications to manage various transaction outcomes:

1. **TransaccionCorregidaPositiva:** Issued when the final settlement results in a positive adjustment to the initially authorized amount, thereby adding funds to the user’s wallet.
2. **TransaccionCorregidaNegativa:** Issued when the final settlement results in a negative adjustment, thereby deducting funds from the user's wallet.
3. **TransaccionConfirmada:** Introduced to confirm that the transaction has settled exactly at the amount authorized initially, providing clear and precise confirmation of the transaction outcome.


![settlement](./assets/imgs/settlement.svg)



## Amount Correction Use Cases

A couple of examples of transactions that could be corrected through a batch process are:

1. **Car rental:** When a customer rents a car, the rental company usually authorizes an initial amount on the customer's card to cover the cost of the rental and any possible damage. However, the final amount could vary depending on the actual use of the vehicle and other factors, such as the distance traveled or the cost of fuel. In these cases, the corrective message "TransaccionCorregidaPositiva" or "TransaccionCorregidaNegativa" could be used to adjust the final amount charged to the customer's card, once the vehicle has been returned and the final calculation has been made.
2. **Hotel bookings:** When a customer reserves a hotel room, the hotel usually blocks an amount on the customer's card to cover the cost of the stay and possible additional charges, such as the minibar or room service. At the end of the stay, the final amount may be different from the initially blocked amount, depending on the actual use of these additional services. In these cases, the corrective message "TransaccionCorregidaPositiva" or "TransaccionCorregidaNegativa" could be used to adjust the final amount charged to the customer's card, once the final charges have been calculated.

Batch processing streamlines the handling of deferred transactions and ensures proper settlement, reducing discrepancies and the need for manual reconciliation. These webhooks improve transaction handling, minimize errors, and optimize overall management.

>The majority of webhooks related to these corrective transactions will be sent during the same period overnight. This is because our webhook generation process is a batch process that runs overnight to consolidate and notify if there were changes to the day's transactions.

Whenever possible, it is recommended to match the corrected transaction with the original transaction for UI purposes. This way, cardholders will have a better understanding of how and why their balance has been modified.

## Webhook Schemas

Settlement of transactions will be notified through a webhook that carries one of the following schemas:

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
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
        "IdCadena": "0"
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
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
        "IdCadena": "0"
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
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
        "IdCadena": "0"
    }
    ```


## Transaction Matching and Funds Capture

- **Matching Transactions:** Each transaction includes a unique identifier `c11NumeroIdentificativoTransaccion`, which is crucial for matching the corrected or confirmed settlement with its initial authorization. This matching process is essential for maintaining transaction integrity and provides cardholders with a clear understanding of how and why their balances were adjusted.
- **Notifications Details:** Settlement webhook notifications contain information about the transaction, including the transaction amount, the identifier, and the type of adjustment or confirmation.



>The "IdCadena" field serves to associate 'ComunicacionAnulacion' notifications with their respective 'PeticionAutorizacion'. Within the context of Settlement notifications, this field will remain as "0"


> Once a transaction has been modified, it will not be modified again.


>Although it is less common, it can also occur that a charge arrives only in the batch process and not online. If this is the case, the webhook may arrive with less information. It should be noted that this information is according to the network's protocol for handles these confirmations and does not depend on PayCaddy.

---
