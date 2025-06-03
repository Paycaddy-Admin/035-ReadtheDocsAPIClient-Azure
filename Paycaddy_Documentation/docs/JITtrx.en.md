This document explains the Just In Time Funding (JIT) flow used by PayCaddy to authorize card transactions using an external authorization endpoint provided by the client. It describes the different types of online transaction notifications, how to handle authorization requests, and how to implement balance inquiry support within the same flow.



## Authorization Requests in JIT Flow

For transactions of type `PeticionAutorizacion`, a POST request will be made to the external authorization endpoint provided by the client. The request will include all relevant fields and follow specific HTTP requirements:

- Use of `X-API-KEY` header for authentication.
    
- HTTP header `Accept: application/json` must be present.
    
- HTTP header `Connection: keep-alive` must be included.
    

> The external endpoint must respond within **1 second**. If the timeout is exceeded, the transaction will be automatically denied with an internal code representing a timeout.


#### **Request Format**

```json
{
  "password": "password",
  "c1Tipo": "PeticionAutorizacionJIT",
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
  "IdCadena": "0102020963010037725"
}
```


---

#### **Expected Responses**

The endpoint should always return a boolean response in JSON format and in some cases include the `amount` attribute to display balance data:


=== "Authorize Transaction"
    ```json
    {
        "Funding": true
    }

    ```

=== "Deny Transaction"
    ```json
    {
        "Funding": false
    }

    ```

=== "Authorize Transaction with Balance Data Inquiry"
    ```json
    {
        "Funding": false,
        "amount": int64 //Balance in cents
    }

    ```

=== "Approved Balance Data Inquiry"
    ```json
    {
        "amount": int64 //Balance in cents
    }

    ```



---

#### **Webhook Notification on Rejection**

If a transaction is denied, a webhook will be sent to the client with one of the following types:


=== "Rejection with On-Time Response"
    ```json
    {
        "c1Tipo": "RechazadaPorAutorizador",
		  ...
    }

    ```

=== "Rejection Due to Timeout"
    ```json
    {
        "c1Tipo": "ExcedeTiempoAutorizacion",
		  ...
    }

    ```


---



## Balance Inquiry

JIT Funding supports balance inquiries under two scenarios:




1. **Pure Balance Inquiries (Not Associated with ATM Withdrawal)**
	
	These transactions are identified with: `"C1Tipo": "JITConsultaSaldo"`
	
	The request body matches the standard webhook structure. Valid `C3CodigoProceso` values include, but are not limited to: `300000, 300200, 300220, 300230, 301000, 302000, 303000`
	
	The expected responses format for such requests is as follows:

=== "Allow Balance Display"
    ```json
    {
	    "amount": int64 //Balance in cents
    }

    ```

=== "Deny Balance Display"
    ```json
    {
        "Funding": false,
    }

    ```


>If the response exceeds 1 second, the transaction is automatically denied and a webhook of type `ExcedeTiempoAutorizacion` is sent.




---
2. **ATM Withdrawals with Balance Display**
	
	These transactions are sent with: `"C1Tipo": "PeticionAutorizacionJIT"`
	
	They can be identified by their `C3CodigoProceso`, which follows the pattern: `01XXXXXX`
	
	To enable balance display on the ATM screen, the authorization response must include:

```
{
  "Funding": true,
  "amount": int64
}
```




---
## General Considerations

- All values in the `amount` field must be expressed in cents (USD).
    
- The response time limit is 1 second for all JIT-related requests.
    
- Rejections due to timeout or insufficient funds are notified via webhook.
    
- Some merchants may initiate balance inquiries as part of verification flows, such as subscription enrollment.




---
## Online Transaction Notification Types 

All online transactions are notified via webhook to the callback URL configured by the client using the **NotificationEnlist POST** endpoint. The webhook format follows the structure described in our Just In Time 

1. **Authorization Request**
	This transaction represents a pending authorization request. It reduces the available balance on the cardholder's wallet

```json
{
  "password": "password",
  "c1Tipo": "PeticionAutorizacion",
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
  "IdCadena": "0102020963010037725"
}
```

2. **Authorization Communication**
	This transaction represents an automatic authorization that does not require approval from the external Authorizer. It reduces the cardholder's available balance.

```json
{
  "c1Tipo": "ComunicacionAutorizacion",
  ...
}
```

3. **Annulment Communication**
	This transaction increases the available balance in the cardholder's wallet.

```json
{
  "c1Tipo": "ComunicacionAnulacion",
  ...
}
```

4. **Reversal Request**
	This represents a refund request that will be processed offline via the batch process. The transaction amount will be declared as "0". Once confirmed by the network, the processed amount will be declared in a related `TransaccionCorregidaPositiva` sent via the batch.

```json
{
  "c1Tipo": "PeticionDevolucion",
  ...
}
```

>Online Transaction Notifications don't expect a reply, they represent a notice of a card event that can affect balance.

>In Just In Time Funding operations, Transaction Notifications can be the result of an approval or denial in the Authorization Request in JIT Flow.




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