There are two methods to retrieve a list of transactions that occurred associated to an entity in our API:

## **Transaction Detail List by Card <font color="green">POST</font>**

**Request URL:**  https://api.api-sandbox.paycaddy.dev/v1/TransactionDetailList

This POST call retrieves all transactions of a particular cardId provided.

=== "Request"
    ```json
    {
        "cardId": "904ee16a-db84-4120-9714-0180892341b3",
        "startDate": "2023-01-23T18:21:02.157Z"
    }

    ```

=== "Response"
    ```json
    {
        "transactionListJson": 
        "[{
            \"c1Tipo\":\"PeticionAutorizacion\",\"c2CardId\":\"904ee16a-db84-4120-9714-0180892341b3\",\"c3CodigoProceso\":\"000000\",\"c4ImporteTransaccion\":\"116\",\"c7FechaHoraTransaccion\":\"20230203194512\",\"c11NumeroIdentificativoTransaccion\":\"000088058\",\"c18CodigoActividadEstablecimiento\":\"4111\",\"c19CodigoPaisAdquiren
            te\":\"724\",\"c38NumeroAutorizacion\":\"169\",\"c41TerminalId
            \":\"00004001\",\"c42Comercio\":\"000000047219191\",\"c43Ident
            ificadorComercio\":\"TRANVIA DE MURCIA        CHURRA-MURCIA  
            \"},{\"c1Tipo\":\"PeticionAutorizacion\",\"c2CardId\":\"904ee16a-db84-4120-9714-0180892341b3\",\"c3CodigoProceso\":\"000000\",\"c4ImporteTrans
            accion\":\"2266\",\"c7FechaHoraTransaccion\":\"20230317135513\
            ",\"c11NumeroIdentificativoTransaccion\":\"000099870\",\"c18Co
            digoActividadEstablecimiento\":\"5813\",\"c19CodigoPaisAdquire
            nte\":\"724\",\"c38NumeroAutorizacion\":\"171\",\"c41TerminalI
            d\":\"90315259\",\"c42Comercio\":\"000000175240563\",\"c43Iden
            tificadorComercio\":\"BULEVAR CAFE EL RANERO   MURCIA
            \"
        }]"
    }

    ```

The structure of the data shared in each transaction follow the same format as all transaction notifications. (See [Online Transaction Notification Webhooks](prefundedTRX.es.md))

---

## **Transaction Detail List by Wallet <font color="green">POST</font>**

**Request URL:**  https://api.api-sandbox.paycaddy.dev/v1/TransactionDetailListByWallet

The **TransactionDetailListByWallet** endpoint offers a detailed view of all transactions linked to a specific wallet. This encompasses wallet-specific transactions like pay-ins, pay-outs, and transfers, as well as card transactions tied to the specified wallet ID.

=== "Request"
    ```json
    {
        "walletId": "string",
        "startDate": "Start Date in ISO Format",
        "toDate": "End Date in ISO Format",
        "maxTransactions": "Maximum Number of Transactions",
        "offset": "Number of Transactions to Skip",
        "c43IdentificadorComercio": "Optional Merchant Identifier Filter"
    }
    ```
=== "Response"
    ```json
    {
        "transactionListJson": "[{transaction1},{transaction2},...,{transactionN}]",
        "totalItems": "Total number of transactions within the requested date range",
        "currentPage": "Current page number indicator",
        "totalPages": "Total number of pages available"
    }
    ```

Each card transaction in the response payload will be a stringified version of the Transaction Notification Webhooks, augmented with three additional field:

```json
    {
        "c1Tipo": "string",
        "c2CardId": "string",
        "c3CodigoProceso": "string",
        "c4ImporteTransaccion": "string",
        "c7FechaHoraTransaccion": "string",
        "c11NumeroIdentificativoTransaccion": "string",
        "c18CodigoActividadEstablecimiento": "string",
        "c19CodigoPaisAdquirente": "string",
        "c38NumeroAutorizacion": "string",
        "c41TerminalId": "string",
        "c42Comercio": "string",
        "c43IdentificadorComercio": "string",
        "c51MonedaImporteTransaccion": "string",
        "c6ImporteMonedaTransaccion": "string",
        "isSettled": "true"
    }
```

- **c51MonedaImporteTransaccion:** Represents the currency used for the transaction at the POS.
- **c6ImporteMonedaTransaccion:** Denotes the transaction amount in the currency used at the POS.
- **isSettled:** Indicates the settlement status of a transaction. A value of "false" signifies transactions that are unsettled or in transit, while "true" indicates transactions that have been settled.

Since PayCaddy abstracts the settlement process for its clients, this field is particularly pertinent to specific use cases like reversals and offline cancellations.

The retrieved transaction list will encompass all transactions with "c1tipo" transaction types impacting the wallet's balance. Additional "c1tipo" such as denied transactions won't be included.

Transactions from the Batch Process with "c1tipo" values of "transaccionCorregidaNegativa" and "transaccionCorregidaPositiva" will be incorporated in the retrieved list, aligning with the expected logic in the webhooks.

Each wallet transaction will be presented in their standard 200 response payload:

**Parameter Explanation:**

**Request:**

| **Field**      | **Description**      |
| ------------- | ------------- |
| **walletId** | The unique ID for the wallet, essential for fetching associated transactions. |
| **startDate & toDate** | Set the date range for transaction retrieval. To fetch and paginate all transactions, use the current date for both fields. |
| **maxTransactions** | Determines the maximum transactions to be returned. If more transactions exist within the date range, they'll be excluded. |
| **offset** | Sets the starting point for transactions, fetched in reverse chronological order, facilitating pagination. |
| **c43IdentificadorComercio** | An optional filter based on a specific merchant. For instance, "UBER" fetches all transactions with "UBER" in their merchant identifier. |

**Response**

| **Field**      | **Description**      |
| ------------- | ------------- |
| **transactionListJson** | Contains a stringified JSON of all requested transactions in reverse chronological order. |
| **totalItems** | Specifies the total transactions in the **transactionListJson**. |
| **currentPage:** | Indicates the current page based on **totalItems** and the request's **offset**. |
| **totalPages:** | Shows the total pages based on **totalItems** and the request's **maxTransactions**. |

### Pagination

To manage pagination, adjust the **maxTransactions** and **offset** parameters in your request. This ensures a controlled exploration of historical transactions, maintaining a swift UX/UI performance.

The **currentPage** and **totalPages** parameters are included in each response for convenience, both calculated based on the **totalItems** and offset parameters.

For instance, with a **totalItems** of "25" and an **offset** of "125", the **currentPage** would be "6", displaying transactions numbered 126 to 150 from the date range in reverse chronological order.

---
