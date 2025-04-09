In our [Prefunded Flow](prefundedFlow.en), When a card issued in PayCaddy’s API makes a transaction on the Mastercard network, PayCaddy’s authorization engine reviews funds availability for the specific transaction. Based on this review if the transaction goes through, it sends a payload with one of four possible transaction types to an enlisted URL.

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

## Online Transaction Notifications

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


---

## Settlement and Batch Process Corrections

Once online transactions are processed, they are reconciled with the settlement files received from the network. Discrepancies or adjustments to the original transaction may be reported through the batch process using the following correction types:

1. **TransaccionCorregidaPositiva**
	Indicates an adjustment that confirms a transaction amount higher than initially authorized. The additional amount will be debited from the cardholder's account.

2. **TransaccionCorregidaNegativa**
	Indicates a correction that results in a lower final amount than originally authorized. The excess amount will be released back to the cardholder's available balance.

3. **TransaccionConfirmada**
	Indicates that the original authorization was matched and confirmed during settlement without requiring adjustment.

>These corrections are processed asynchronously and reported via batch, not through the JIT webhook mechanism.
---
