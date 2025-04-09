## **Wallets <font color="green">POST</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/wallets

‍The creation of a wallet is done through a POST call that carries the following structure:

=== "Request"
    ```json
        {
            "userId": "string",
            "currency": "string",
            "description": "string",
            "walletType": 0
        }
    ```
=== "Response"
    ```json
        {
            "id": "string",
            "userId": "string",
            "currency": "string",
            "description": "string",
            "balance": 0,
            "amountWithheld": 0,
            "creationDate": "2022-07-19T20:08:29.970Z"
        }
    ```

This call must be made by associating it with a previously created active userId.

The currency field must consider the codes of ISO 4217. (e.g. US Dollars would be entered with the code "USD").

The description field is created to name the wallet created according to the intended use.

The "walletType" field defines whether the wallet can or cannot be associated with a credit card (see creditCard POST). This boolean must be kept as "0" for all wallets created for debit cards, prepaid cards, or for fund and transfer management. For wallets created for prefunded credit type cards, the "walletType" must be defined as "1".

> The **"Main Wallets"** created automatically in the user creation flow maintain a **"walletType"** of **"0"**.

---

## **Wallets <font color="sky-blue">GET</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/wallets

The GET call for wallets allows you to retrieve the information associated with the queried walletId. This call is particularly important for checking the available balance in a wallet and the balance withheld due to pending transactions.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/wallets/{WALLET_ID}
    
    ```
=== "Response"
    ```json
        {
            "id": "string",
            "userId": "string",
            "currency": "string",
            "description": "string",
            "balance": 0,
            "creationDate": "2024-11-12T12:17:29.710Z",
            "amountWithheld": 0
        } 
    ```

The total balance is reflected in the "balance" field of the successful 200 response to this call, while the balance withheld is reflected in the "amountWithheld" field.

All amounts are reflected in cents, meaning USD 1,000.00 would be represented as 100000.

The call may fail if an incorrect walletId is provided, in which case the NeoBank API would respond with a HTTP 400 error.

```json
    {
        "type": "",
        "title": "Wallet 'c4d165d1-xxxx-xxxx-xxxx-017f0c6facf8' not found",
        "status": 0,
        "detail": "",
        "instance": ""
    }
```

---

## **Wallet Per User ID <font color="green">POST</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/WalletsPerUserIds

‍This endpoint allows retrieving all the wallets associated with a user ID.By making a POST request with the user's ID, the system returns a list of the identifiers of the different wallets that the user has.

=== "Request"
    ```json
    {
        "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
    }
    ```
=== "Response"
    ```json
    {  
        "walletsJson": "[
        ‍{\"Id\":\"216fa217-6826-48b8-ab5b-0180891cef30\",\"BDateUtcCreate\":\"2022-05-03T08:50:16.4959966\",\"BTimestamp\":\"AAAAAAAb6Pk=\",
        \"ClientId\":\"1c5a29f5-b018-40c7-a6d4-017efe423d42\",\"UserId\":\"qa2c3b2a-23f6-49be-b882-0180891cef30\",\"Currency\":\"USD\",\"Description\":\"Main wallet\",\"AmountAvailable\":38574,\"AmountWithheld\":1110756},
        {
        \"Id\":\"d6928675-a4fe-4752-8e7e-018099d30fe1\",\"BDateUtcCreate\":\"2022-05-06T14:43:07.7804616\",\"BTimestamp\":\"AAAAAAAbv6Q=\",
        \"ClientId\":\"1c5a29f5-b018-40c7-a6d4-017efe423d42\",\"UserId\":\"ea2c3b2a-23f6-49be-b882-0180891cef30\",\"Currency\":\"USD\",\"Description\":\"SecondaryWallet\",\"AmountAvailable\":381599,\"AmountWithheld\":29253372}]"
    }```

The response to this call carries all the information from the multiple wallets associated to the same UserId in a string format that maintains the same structure as the response of the Wallet GET call.

