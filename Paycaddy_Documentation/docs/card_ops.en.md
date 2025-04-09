# Card Operations

## **ACK Reception <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/ackReception

As a security measure in the logistical process of delivering physical plastics, by default, all physical cards created in the NeoBank API are born inactive. This is reflected in the "isActive" and "status" fields of successful responses for the creation of these cards, as well as when making a GET call for all types of cards, such as seen in the example below:

```json
    {
        "id": "string",
        "userId": "string",
        "walletId": "string",
        "physicalCard": true,
        "code": "string",
        "isActive": false,
        "status": "pendingAck",
        "creationDate": "2022-08-01T19:37:50.392Z",
        "dueDate": 0
    }
```

Once a day, at a specific cut-off time, all physical cards created in the cycle are sent to the printing facility. The **ackReception POST** call will not return a successful response until the requested card has been printed, so it's a call that should only be made when the printed card is in hand or 48 hours after a successful response for card creation.

The purpose of this call is to allow the card to be delivered to the cardholder in an inactive state, waiting for validation of receipt from the cardholder once they have the card in hand.

The card-in-hand validation should be managed by comparing data entered by the cardholder (such as a range of the PAN or the CVV) with the results of calls made to obtain such sensitive data (see **[checkPan POST](#check-pan-post)** and **[checkCvv POST](#check-cvv-post)**). If the entered data matches, the ackReception POST call can be made to activate the card for the first time, simply by sending the cardId of the card to be activated in the call.

The successful response to the call simply replicates the entered **cardId**. In case of encountering a 500 error, contact PayCaddy support team with the details of the case.

=== "Request"
    ```json
        {
            "cardId": "string"
        }
    ```
=== "Response"
    ```json
        {
            "cardId": "string"
        }
    ```

---

## **Block Card <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/blockCard

‍This call allows for self-managed change of the card's activity status. This call directly affects the **"isActive"** boolean by transforming it to **"false"** based on a simple call that only carries the **cardId**.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "isActive": false
    }
    ```

The API could respond with a descriptive HTTP 400 error if it is an attempt to block a card that has already been previously blocked. For active cards, unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Unblock Card <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/unblockCard

‍This call allows for self-managed change of the card's activity status. This call directly affects the "isActive" boolean by transforming it to **"true"** based on a simple call that only carries the cardId.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "isActive": true
    }
    ```

The API could respond with a descriptive HTTP 400 error if it is an attempt to unblock an active card. For previously blocked cards that are currently inactive, unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Cancel Card <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/cancelCard

!!!Danger
    The results caused by the action of this method is unreversible. It shall not be use lightly once it results in countable effects

This call allows for permanently canceling a previously created card. This call directly affects the **"isActive"** boolean, changing it to **"false"**, as well as the **"status"** field changing it to **"Cancel"** from a call that simply loads the **cardId**.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "isActive": "false"
    }
    ```

The PayCaddy API could respond with a descriptive HTTP 400 error if it is an attempt to cancel a card that has already been canceled.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Check PAN <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkPan

‍This call allows access to sensitive data such as the card number and expiration date.
It requires a cardId as an input and carries the following structure:

!!!Warning
    The responses to this call should not be stored in databases for security reasons.


=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "pan": "string",
        "expDate": 0
    }
    ```

Unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Check CVV <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkCvv

This call allows access to sensitive data such as the card's CVV (Card Verification Value).
It requires a cardId as an input and carries the following structure:

!!!Warning
    The responses to this call should not be stored in databases for security reasons.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "cvv": "string"
    }
    ```

Unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Check Exp. Date <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkDueDate

This call allows access the expiration date from the card.
It requires a cardId as an input and carries the following structure:

!!!Warning
    The responses to this call should not be stored in databases for security reasons.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "dueDate": "string"
    }
    ```
Unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Check PIN <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkPin

This call allows access to the sensitive PIN data of the card, required for all ATM withdrawals and for some geographic regions' POS transactions.

!!!Warning
    Responses from this call should not be stored in databases for security reasons.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "pin": "string"
    }
    ```

Unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Unblock PIN <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/unblockPin

This call allows the reactivation of a card's PIN after a security block occurs due to three incorrect attempts. The total number of acceptable incorrect attempts before the security block is a parameter that can be discussed during the project's scope definition phase, specifically for customized card profiles.

!!!Warning
    Responses from this call should not be stored in databases for security reasons.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "pin": "string"
    }
    ```

Unless an incorrect cardId is inputted, then the API will reply with a successful HTTP 200 message.

In case of encountering an HTTP 500 error, contact the PayCaddy support team with the details of the case.

---

## **Change PIN <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/cardOperations/changePin

This call allows managing the sensitive PIN data of the card, which is required for all ATM withdrawals and for some geographic areas in point-of-sale devices. The call is designed to assign a 4-digit PIN specified by the user. The responses to this call should not be stored in databases for security reasons.
‍
!!!Warning
    Responses from this call should not be stored in databases for security reasons.

=== "Request"
    ```json
    {
        "cardId": "string"
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "pin": "string"
    }
    ```


## **Change Internal Card Limit <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/ChangeLimitIWL

Each card generated by PayCaddy's API has a series of properties that are used when conducting transactions. One of these is the limit assigned by clients to control the spending of their users. The value of the limit will determine the amount that can be transacted and may vary for each user. Only clients can make changes and updates to the limit value.

The limits are reset at the beginning of each month, and each transaction is approved according to the current limit at the time.

=== "Request"
    ```json
    {
        "cardId": "string",
        "limit": 0
    }
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "limit": 0
    }
    ```

The API will respond with a HTTP 422 Error "card not found" if the provided cardId has no match for the authenticated client.

---

## **Change Internal Card Limit <font color="sky-blue">GET</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/ChangeLimitIWL/

To retrieve the current limit of a particular card the GET method for the Internal Wallet Limit must be used.

The request must be made with the particular **cardId** that is to be consulted. In the example below, the cardId is highlighted in the request URL.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/ChangeLimitIWL/{CARD_ID}
    ```
=== "Response"
    ```json
    {
        "cardId": "string",
        "limit": 0
    }
    ```

In case the Limit is 0 or has not been previously set, the response will carry a message of **“No stablish internal limit”** as the limit attribute.

The API will respond with a HTTP 422 Error **"card not found"** if the provided cardId has no match for the authenticated client.


---