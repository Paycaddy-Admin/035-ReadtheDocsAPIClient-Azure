There are 9 additional types of transactions that you will receive online through the enlisted URL. It should be noted that these types do not affect balance and are merely informative of the card usage and describe the cause for rejected transactions.

## Rejection Notifications

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

## MoneySend Transactions

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