# Transactional Controls

Transactional controls are mechanisms that let you shape how transactions are authorized on the cards in your program. The controls described in this chapter operate at the program level and apply across every card you have issued under your `clientId`.

The currently available control is the **merchant block list**, which lets you block specific merchants from approving transactions on the cards you've issued. Each blocked merchant is identified by its 15-digit alphanumeric `c42` code — the same `c42Comercio` value that appears on every transaction webhook. When a cardholder attempts a transaction at a merchant whose code is on your block list, the transaction is declined and a webhook of type `DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE` is sent to your callback URL. See [Other Transactions](./otherTransactions.en.md) for the rejection notification schema.

PayCaddy also maintains a global block list (the **general** list) and program-specific lists (**prefunded** and **jit**) that are applied automatically. Transactions blocked under those lists produce a rejection webhook of type `DENEGACION POR COMERCIO BLOQUEADO (GENERAL)`.

---

## **Update Block List <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/BlockListC42

This call adds or removes a single merchant from your block list. Each request operates on exactly one merchant — to update multiple merchants, send multiple requests. The `clientId` that owns the list is captured automatically from the API key authenticated for the session, so you only need to specify the merchant code and the operation.

=== "Request"
    ```json
    {
        "c42code": "string",
        "operation": "string"
    }
    ```

=== "Response"
    ```json
    {
        "values": "227759000156182",
        "operation": "add",
        "date": "2026-06-23T10:15:30Z"
    }
    ```

|Field|Description|
|---|---|
|`c42code`|The 15-digit alphanumeric merchant identifier (`c42Comercio` from the webhook payloads) to add or remove from your block list.|
|`operation`|Either `"add"` to include the merchant on your block list or `"remove"` to take it off.|

The successful response is **HTTP 201 Created**. The response echoes the merchant code that was modified, the `operation` that was applied (`"add"` or `"remove"`), and the timestamp at which the change was registered.

The API will respond with **HTTP 422** in the following cases:

=== "Invalid c42code"
    ```json
    {
        "status": 422,
        "detail": "c42code must be a 15-digit alphanumeric value"
    }
    ```

=== "Invalid operation"
    ```json
    {
        "status": 422,
        "detail": "operation must be 'add' or 'remove'"
    }
    ```

=== "Already on the list"
    ```json
    {
        "status": 422,
        "detail": "This c42code was already included in the list"
    }
    ```

=== "Not on the list"
    ```json
    {
        "status": 422,
        "detail": "This c42code does not exist in the list"
    }
    ```

> Every successful update is logged on PayCaddy's side. If you need an audit of historical changes to your block list, contact your integration team.

---

## **Get Block List <font color="sky-blue">GET</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/BlockListC42

Returns the merchant block list associated with your `clientId` along with any program-level lists that apply to the cards in your program (such as the **prefunded**, **jit**, and **general** lists maintained by PayCaddy).

=== "Response"
    ```json
    {
        "secret": "string",
        "date": "2026-06-23T10:15:30Z"
    }
    ```

The `secret` field contains the structured block list data: your client-specific list of `c42` codes, plus whichever PayCaddy-maintained lists (`prefunded`, `jit`, `general`) apply to your program.

> Transactions to merchants on your client-specific list produce a rejection webhook of type `DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE`. Transactions to merchants on the **general** list produce a rejection webhook of type `DENEGACION POR COMERCIO BLOQUEADO (GENERAL)`. See [Other Transactions](./otherTransactions.en.md) for both schemas.
