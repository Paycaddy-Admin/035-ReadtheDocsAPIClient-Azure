# Wallet Operations

## **Pay In <font color="green">POST</font>**

**Request URL:**  https://api.paycaddy.dev/v1/payIns

‍PayIn is PayCaddy API's method for adding funds to a specific wallet. It is a simple call that loads the information of the **'walletId'** to which funds are to be loaded, the amount to be credited to the wallet in cents, the code of the currency according to ISO 4217, and a description of the transaction.

=== "Request"
    ```json
    {
        "walletIdTo": "string",
        "amount": 0,
        "currency": "string",
        "statement": "string"
    }
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "walletIdTo": "string",
        "amount": 0,
        "currency": "string",
        "statement": "string",
        "creationDate": "2022-07-20T20:47:04.060Z"
    }
    ```

The successful response of this call, in addition to the provided information, includes a timestamp with the execution date of the transaction and a unique transaction identifier that allows retrieving the associated information with the GET call.

If there is a problem with the data provided the PayCaddy API will respond with a HTTP 400 error.

=== "Wallet not found"
    ```json
    {
        "type": "Decoding body",
        "title": "Error ",
        "status": 422,
        "detail": "Wallet not found",
        "instance": ""
    }
    ```
=== "Incorrect currency"
    ```json
    {
        "type": "",
        "title": "currency code does not match target walletId",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```

---

## **Pay In <font color="sky-blue">GET</font>**

**Request URL:**  https://api.paycaddy.dev/v1/payIns/

The GET call for a PayIn allows accessing information related to a specific transactionId. The successful response of this call loads the following structure:

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/payIns/{PAYIN_ID}
    ```
=== "Response"
    ```json
    {
        "data": [{
            "id": "string",
            "walletIdTo": "string",
            "amount": 0,
            "currency": "string",
            "statement": "string",
            "creationDate": "2022-07-20T20:47:04.056Z"
        }]
    }
    ```

The call may fail if an incorrect walletId is provided, in which case the PayCaddy API would respond with a HTTP 400 error.

---

## **Pay Out <font color="green">POST</font>**

**Request URL:**  https://api.paycaddy.dev/v1/payOuts

PayOut is a method to debit funds available in a specific wallet. Unlike a transfer, using this method eliminates the existence of the funds from the NeoBank API. This method should be associated with an accounting operation that requires such a call.

Similar to PayIn, the successful response will return the entered data accompanied by a unique transaction identifier and its timestamp.

=== "Request"
    ```json
    {
        "walletIdFrom": "string",
        "amount": 0,
        "currency": "string",
        "statement": "string"
    }
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "walletIdFrom": "string",
        "amount": 0,
        "currency": "string",
        "statement": "string",
        "creationDate": "2022-07-20T20:47:04.060Z"
    }
    ```

If there is a problem with the data provided the PayCaddy API will respond with a HTTP 400 error.

=== "Wallet not found"
    ```json
    {
        "type": "Decoding body",
        "title": "Error ",
        "status": 422,
        "detail": "Wallet not found",
        "instance": ""
    }
    ```
=== "Incorrect currency"
    ```json
    {
        "type": "",
        "title": "currency code does not match target walletId",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```
=== "Insufficient funds"
    ```json
        {
            "type": "",
            "title": "Wallet does not have sufficient funds",
            "status": 0,
            "detail": "",
            "instance": ""
        }
    ```

---

## **Pay Out <font color="sky-blue">GET</font>**

**Request URL:**  https://api.paycaddy.dev/v1/payOuts/

The GET call for a PayOut allows access to information related to a specific **transactionId**. The successful response of this call loads the following structure:

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/payOuts/{PAYOUT_ID}
    ```
=== "Response"
    ```json
    {
        "data": [{
            "id": "string",
            "walletIdTo": "string",
            "amount": 0,
            "currency": "string",
            "statement": "string",
            "creationDate": "2022-07-20T20:47:04.056Z"
        }]
    }
    ```

The API will respond with an HTTP 400 error if it is an incorrect transactionId or a transactionId that is not related to a PayOut.

---

## **Transfer <font color="green">POST</font>**

**Request URL:**  https://api.paycaddy.dev/v1/transfers

To carry out transactions within the NeoBank API environment between two previously created wallets, the **Transfer POST** call must be used, which has the following structure:

=== "Request"
    ```json
    {
        "walletIdFrom": "string",
        "walletIdTo": "string",
        "amount": 0,
        "currency": "string",
        "statement": "string"
    }
=== "Response"
    ```json
    {
        "id": "string",
        "walletIdFrom": "string",
        "walletIdTo": "string",
        "amount": 0,
        "currency": "string",
        "statement": "string",
        "creationDate": "2022-07-22T15:54:08.059Z"
    }
    ```

In this call, you must specify the **walletIdFrom** which the funds are being sent and the **walletIdTo** which the funds are being sent. The **"currency"** field must load the code of the currency associated with both wallets following the ISO 4217 standard.

The PayCaddy API does not manage currency conversions, so it is necessary to consider that both wallets must be configured under the same currency.

The amount entered in the **"amount"** field must be in cents following the standard of the other PayCaddy API calls.

The **"statement"** field allows you to enter a description of the transaction for future reference and display on the front-end.

The successful response to this call loads a unique transaction identifier and the timestamp of its acceptance.

In case of an error in the call, the NeoBank API will respond with a descriptive HTTP 400 error.

Commonly, the call will fail if the walletIds involved in the transaction belong to an inactive userId, that is, maintaining its "isActive" attribute as "false".

Similarly, the call will always fail if the **walletIdFrom** does not maintain the adequate balance to cover the transaction or when the currency code of both wallets is not the same as the transfer POST call.

---

## **Transfer <font color="sky-blue">GET</font>**

**Request URL:**  https://api.paycaddy.dev/v1/transfers/

The GET call for a transfer between wallets allows access to information related to a specific transactionId and carries the following structure:

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/transfers/{TRANSFER_ID}
    ```
=== "Response"
    ```json
    {
        "data": [{
            "id": "string",
            "walletIdFrom": "string",
            "walletIdTo": "string",
            "amount": 0,
            "currency": "string",
            "statement": "string",
            "creationDate": "2022-07-22T16:07:33.092Z"
        }]
    }
    ```

The API will respond with a descriptive HTTP 400 error if an incorrect transaction ID is provided or if the transaction ID is not related to a transfer.
