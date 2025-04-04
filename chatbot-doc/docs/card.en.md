## **Debit Card <font color="green">POST</font>**

**Request URL:** https://api.paycaddy.dev/v1/debitCards

‍This call enables the creation of a Debit Card and follows the following structure:

=== "Request"
	```json
    {
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string"
    }
    ```
=== "Response"
	```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": true,
        "status": "string",
        "creationDate": "2022-08-01T19:37:50.392Z",
        "dueDate": 0
    }
    ```

Each request includes the **userId** to whom the card should be associated and the **walletId** whose balance the card will be transacting with.



The **physicalCard** field indicates whether it is a card that needs to be printed physically **(true)** or, alternatively, an exclusively digital card **(false)**.

The card printing data is extracted from the fields stored in the user creation, so it is important to note that the cards are printed taking into account the **FirstName** and **LastName** fields in the case of natural persons and the **RegisteredName** field in the case of legal entities. It is essential to ensure the integrity of these fields in the user creation flow, including character limitations, since it affects the subsequent card creation flow.

The **"code"** sent in the call must be provided by the PayCaddy team for each type and variation of card included in the enablement project. That is, for a project that enables the issuance of a virtual or physical debit card for natural persons, two different codes would be provided, one for endUser physical and one for endUser virtual. It is the client's responsibility to correctly invoke the calls taking into consideration the type of user and the printing condition associated with the provided code.

The successful response of the debit card creation call returns a 200 message that carries the unique identifier of the card that must be used in all card operation calls **(see [cardOperations](card_operations.en.md))**. In addition to the **cardId**, the 200 response also provides a boolean indicating whether the card is operational or not, and a status field describing the card's status.

The possible statuses are detailed below:

- **PendingAck** - For newly created physical cards that have not been activated. (see [AckReception POST](card_operations.en.md#ack-reception-post))

- **Temporarilyblocked** - For self-manageable blocks. (see [UnblockCard POST](card_operations.en.md#unblock-card-post))

- **Cancel** - For canceled cards (see [CancelCard POST](card_operations.en.md#block-card-post))

- **Active** - For active cards

The sensitive card data (PAN and CVV) can be queried using the **[checkPan POST](card_operations.en.md#check-pan-post)** and **[checkCvv POST](card_operations.en.md#check-cvv-post)** calls. However, it is important to note that such data must NOT be stored in databases as they imply cybersecurity requirements associated with the PCI standard that have been abstracted with the use of the cardId in the PayCaddy API.

The card's expiration date is presented in the successful response of the card creation call in the "dueDate" field following the format YYYYMM.

In case of any error or inconsistency with the provided data, the API will respond with a descriptive 400 error of one of the most common possible reasons:

1. The userId to which the card is to be associated does not exist or is inactive.
2. The walletId to which the card is to be associated does not exist or does not belong to the entered user.
3. The code provided in the call does not match the type of user to which the card is to be associated.
4. The code provided in the call does not match the type of card (virtual or physical) being attempted to create.

> It is important to note that retries in case of errors must be managed responsibly, i.e., by creating time limits of at least 5 seconds before the retry and limiting the number of retries to 3.

---

## **Debit Card <font color="sky-blue">GET</font>**

**Request URL:** https://api.paycaddy.dev/v1/debitCards/

The GET call for debit cards allows you to query the data of a card based on a cardId.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/debitCards/{DEBIT_CARD_ID}
    ```

=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": true,
        "status": "string",
        "creationDate": "2024-11-12T17:09:56.722Z",
        "dueDate": "string"
    }
    ```
This call is especially necessary in managing the transaction status of the card, since the successful response also includes the **"isActive"** boolean activity and the descriptive **"status"** field, in addition to the other fields in the successful response of card creation.In case an invalid cardId is sent, the PayCaddy API will respond with an HTTP 400 error.

If there is a HTTP 500 error encountered, report the incident to the PayCaddy support team.

---

## **Prepaid Card <font color="green">POST</font>**

**Request URL:** https://api.paycaddy.dev/v1/prepaidCards

This is the call used to create prepaid cards associated with a particular **userId** and **walletId**, and it carries the following structure:

=== "Request"
    ```json
    {
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string"
    }
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": true,
        "status": "string",
        "creationDate": "2024-11-12T17:11:55.775Z",
        "dueDate": "string"
    }
    ```
Each request includes the **userId** to whom the card should be associated and the **walletId** whose balance the card will be transacting with.

The **physicalCard** field indicates whether it is a card that needs to be printed physically **(true)** or, alternatively, an exclusively digital card **(false)**.

The card printing data is extracted from the fields stored in the user creation, so it is important to note that the cards are printed taking into account the **FirstName** and **LastName** fields in the case of natural persons and the **RegisteredName** field in the case of legal entities. It is essential to ensure the integrity of these fields in the user creation flow, including character limitations, since it affects the subsequent card creation flow.

The **"code"** sent in the call must be provided by the PayCaddy team for each type and variation of card included in the enablement project. That is, for a project that enables the issuance of a virtual or physical debit card for natural persons, two different codes would be provided, one for endUser physical and one for endUser virtual. It is the client's responsibility to correctly invoke the calls taking into consideration the type of user and the printing condition associated with the provided code.

The possible statuses are detailed below:

- **PendingAck** - For newly created physical cards that have not been activated. (see [AckReception POST](card_operations.en.md#ack-reception-post))

- **Temporarilyblocked** - For self-manageable blocks. (see [UnblockCard POST](card_operations.en.md#unblock-card-post))

- **Cancel** - For canceled cards (see [CancelCard POST](card_operations.en.md#block-card-post))

- **Active** - For active cards

The sensitive card data (PAN and CVV) can be queried using the **[checkPan POST](card_operations.en.md#check-pan-post)** and **[checkCvv POST](card_operations.en.md#check-cvv-post)** calls. However, it is important to note that such data must NOT be stored in databases as they imply cybersecurity requirements associated with the PCI standard that have been abstracted with the use of the cardId in the PayCaddy API.

The card's expiration date is presented in the successful response of the card creation call in the "dueDate" field following the format YYYYMM.

In case of any error or inconsistency with the provided data, the API will respond with a descriptive 400 error of one of the most common possible reasons:

1. The userId to which the card is to be associated does not exist or is inactive.
2. The walletId to which the card is to be associated does not exist or does not belong to the entered user.
3. The code provided in the call does not match the type of user to which the card is to be associated.
4. The code provided in the call does not match the type of card (virtual or physical) being attempted to create.

> It is important to note that retries in case of errors must be managed responsibly, i.e., by creating time limits of at least 5 seconds before the retry and limiting the number of retries to 3.


---

## **Prepaid Card <font color="sky-blue">GET</font>**

**Request URL:** https://api.paycaddy.dev/v1/prepaidCards/

The GET call for prepaid cards allows you to query the data of a card based on a cardId.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/prepaidCards/{PREPAID_CARD_ID}
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": true,
        "status": "string",
        "creationDate": "2024-11-12T17:09:56.722Z",
        "dueDate": "string"
    }
    ```
This call is especially necessary in managing the transaction status of the card, since the successful response also includes the **"isActive"** boolean activity and the descriptive **"status"** field, in addition to the other fields in the successful response of card creation.In case an invalid cardId is sent, the PayCaddy API will respond with an HTTP 400 error.

If there is a HTTP 500 error encountered, report the incident to the PayCaddy support team.

---

## **Credit Card <font color="green">POST</font>**

**Request URL:** https://api.paycaddy.dev/v1/CreditCards

This call enables the creation of a Credit Card associated to a particular **userId** and **walletId**.
The following structure must be kept to send the relevant data.

=== "Request"
    ```json
    {
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string"
    }
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": true,
        "status": "string",
        "creationDate": "2024-11-12T17:17:42.133Z",
        "dueDate": "string"
    }
    ```

In each call, the **userId** to which the card should be associated and the walletId with whose balance the card will be transacting are included.

For credit cards, the **walletId** to be associated must be a credit wallet **(see [WalletCredits](creditCore.en.md#wallet-credit-post))** or a prepaid wallet with the **"walletType"** attribute equal to "1".

The **physicalCard** field indicates whether it is a card that needs to be printed physically **(true)** or, alternatively, an exclusively digital card **(false)**.

The card printing data is extracted from the fields stored in the user creation, so it is important to note that the cards are printed taking into account the **FirstName** and **LastName** fields in the case of natural persons and the **RegisteredName** field in the case of legal entities. It is essential to ensure the integrity of these fields in the user creation flow, including character limitations, since it affects the subsequent card creation flow.

The **"code"** sent in the call must be provided by the PayCaddy team for each type and variation of card included in the enablement project. That is, for a project that enables the issuance of a virtual or physical debit card for natural persons, two different codes would be provided, one for endUser physical and one for endUser virtual. It is the client's responsibility to correctly invoke the calls taking into consideration the type of user and the printing condition associated with the provided code.

The successful response of the debit card creation call returns a 200 message that carries the unique identifier of the card that must be used in all card operation calls **(see [cardOperations](card_operations.en.md))**. In addition to the **cardId**, the 200 response also provides a boolean indicating whether the card is operational or not, and a status field describing the card's status.

The possible statuses are detailed below:

- **PendingAck** - For newly created physical cards that have not been activated. (see [AckReception POST](card_operations.en.md#ack-reception-post))

- **Temporarilyblocked** - For self-manageable blocks. (see [UnblockCard POST](card_operations.en.md#unblock-card-post))

- **Cancel** - For canceled cards (see [CancelCard POST](card_operations.en.md#block-card-post))

- **Active** - For active cards

The sensitive card data (PAN and CVV) can be queried using the **[checkPan POST](card_operations.en.md#check-pan-post)** and **[checkCvv POST](card_operations.en.md#check-cvv-post)** calls. However, it is important to note that such data must NOT be stored in databases as they imply cybersecurity requirements associated with the PCI standard that have been abstracted with the use of the cardId in the PayCaddy API.

The card's expiration date is presented in the successful response of the card creation call in the "dueDate" field following the format YYYYMM.

In case of any error or inconsistency with the provided data, the API will respond with a descriptive 400 error of one of the most common possible reasons:

1. The userId to which the card is to be associated does not exist or is inactive.
2. The walletId to which the card is to be associated does not exist or does not belong to the entered user.
3. The code provided in the call does not match the type of user to which the card is to be associated.
4. The code provided in the call does not match the type of card (virtual or physical) being attempted to create.

> It is important to note that retries in case of errors must be managed responsibly, i.e., by creating time limits of at least 5 seconds before the retry and limiting the number of retries to 3.

---

## **Credit Card <font color="sky-blue">GET</font>**

**Request URL:** https://api.paycaddy.dev/v1/CreditCards/

The GET call for a Credit Card allows you to retrieve relevant information associated to a particular cardId provided through the following structure:

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/CreditCards/{CREDIT_CARD_ID}
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": true,
        "status": "string",
        "creationDate": "2024-11-12T17:21:18.643Z",
        "dueDate": "string"
    }
    ```

This call is especially necessary in managing the transaction status of the card, since the successful response also includes the **"isActive"** boolean activity and the descriptive **"status"** field, in addition to the other fields in the successful response of card creation.In case an invalid cardId is sent, the PayCaddy API will respond with an HTTP 400 error.

If there is a HTTP 500 error encountered, report the incident to the PayCaddy support team.