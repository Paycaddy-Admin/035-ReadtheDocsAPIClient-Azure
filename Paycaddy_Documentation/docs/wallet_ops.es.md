# Operaciones de Wallet

## **Pay In <font color="green">POST</font>**

**URL de la solicitud:** `https://api.api-sandbox.paycaddy.dev/v1/payIns`

`Pay In` es el método de la API de PayCaddy para **añadir fondos** a una `wallet` específica. La llamada requiere el **walletId** de destino, el monto a acreditar en centavos, el código de la divisa según ISO 4217 y una descripción de la transacción.

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

La respuesta exitosa incluye la marca de tiempo de la operación y un identificador único que permite recuperar la información con la llamada **GET** correspondiente.

Si los datos son incorrectos, la API devuelve un error HTTP 400.

=== "Wallet not found"
	```json
	{
	    "type": "Decoding body",
	    "title": "Error",
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

**URL de la solicitud:** `https://api.api-sandbox.paycaddy.dev/v1/payIns/{PAYIN_ID}`

La llamada **GET** devuelve los datos de un `Pay In` específico:

=== "Request"

```json
https://api.api-sandbox.paycaddy.dev/v1/payIns/{PAYIN_ID}
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

Un **PAYIN_ID** inválido genera un error HTTP 400.

---

## **Pay Out <font color="green">POST</font>**

**URL de la solicitud:** `https://api.api-sandbox.paycaddy.dev/v1/payOuts`

`Pay Out` permite **debitAR fondos** disponibles en una `wallet`. A diferencia de una transferencia, los fondos **salen** del ecosistema PayCaddy y deben corresponderse con una operación contable externa.

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

Errores habituales (HTTP 400):

=== "Wallet not found"
	```json
	{
	    "type": "Decoding body",
	    "title": "Error",
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

**URL de la solicitud:** `https://api.api-sandbox.paycaddy.dev/v1/payOuts/{PAYOUT_ID}`

Devuelve los datos de un `Pay Out` específico:

=== "Request"

```json
https://api.api-sandbox.paycaddy.dev/v1/payOuts/{PAYOUT_ID}
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

Un **PAYOUT_ID** incorrecto genera un error HTTP 400.

---

## **Transfer <font color="green">POST</font>**

**URL de la solicitud:** `https://api.api-sandbox.paycaddy.dev/v1/transfers`

Para mover fondos **entre dos wallets** dentro de PayCaddy se usa `Transfer POST`:

=== "Request"
	```json
	{
	    "walletIdFrom": "string",
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
	    "walletIdFrom": "string",
	    "walletIdTo": "string",
	    "amount": 0,
	    "currency": "string",
	    "statement": "string",
	    "creationDate": "2022-07-22T15:54:08.059Z"
	}
	```

Consideraciones:

- Ambas wallets deben usar la **misma divisa**; PayCaddy no realiza conversiones.
    
- El monto se expresa en **centavos**.
    
- La transacción falla si alguna wallet pertenece a un usuario inactivo o si el saldo es insuficiente.
    

---

## **Transfer <font color="sky-blue">GET</font>**

**URL de la solicitud:** `https://api.api-sandbox.paycaddy.dev/v1/transfers/{TRANSFER_ID}`

Recupera la información de una `Transfer` específica:

=== "Request"

```json
https://api.api-sandbox.paycaddy.dev/v1/transfers/{TRANSFER_ID}
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

Un **TRANSFER_ID** incorrecto produce un error HTTP 400.

---
