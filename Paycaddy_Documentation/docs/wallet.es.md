## **Wallets <font color="green">POST</font>** 

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v1/wallets

‍La creación de una wallet se realiza mediante una llamada **POST** que lleva la siguiente estructura:

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

Esta llamada debe realizarse asociándola a un **userId** activo previamente creado.

El campo **currency** debe utilizar los códigos de ISO 4217 (p. ej., los dólares estadounidenses se ingresan con el código `"USD"`).

El campo **description** se utiliza para nombrar la wallet de acuerdo con el uso previsto.

El campo **"walletType"** define si la wallet puede o no asociarse a una tarjeta de crédito (consulta **creditCard POST**). Este booleano debe mantenerse en **`0`** para todas las wallets creadas para tarjetas de débito, tarjetas prepago o para la gestión de fondos y transferencias. Para las wallets creadas para tarjetas de crédito **prefunded**, el **"walletType"** debe establecerse en **`1`**.

> Las **"Main Wallets"** creadas automáticamente en el flujo de creación de usuario mantienen un **"walletType"** de **`0`**.

---

## **Wallets <font color="sky-blue">GET</font>** 

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v1/wallets

La llamada **GET** para wallets permite recuperar la información asociada al **walletId** consultado. Esta llamada es especialmente importante para comprobar el saldo disponible en una wallet y el saldo retenido debido a transacciones pendientes.

=== "Request"
	```
	https://api.api-sandbox.paycaddy.dev/v1/wallets/{WALLET_ID}
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

El saldo total se refleja en el campo **"balance"** de la respuesta 200 exitosa, mientras que el saldo retenido se muestra en el campo **"amountWithheld"**.

Todos los montos se expresan en centavos; por ejemplo, USD 1 000,00 se representaría como **100000**.

La llamada puede fallar si se proporciona un **walletId** incorrecto, en cuyo caso la API de NeoBank responderá con un error HTTP 400:

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

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/WalletsPerUserIds](https://api.api-sandbox.paycaddy.dev/v1/WalletsPerUserIds)

‍Este endpoint permite recuperar todas las wallets asociadas a un **userId**. Al realizar una solicitud **POST** con el ID del usuario, el sistema devuelve una lista con los identificadores de las distintas wallets que posee.

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
	        {\"Id\":\"216fa217-6826-48b8-ab5b-0180891cef30\",\"BDateUtcCreate\":\"2022-05-03T08:50:16.4959966\",\"BTimestamp\":\"AAAAAAAb6Pk=\",
	        \"ClientId\":\"1c5a29f5-b018-40c7-a6d4-017efe423d42\",\"UserId\":\"qa2c3b2a-23f6-49be-b882-0180891cef30\",\"Currency\":\"USD\",\"Description\":\"Main wallet\",\"AmountAvailable\":38574,\"AmountWithheld\":1110756},
	        {\"Id\":\"d6928675-a4fe-4752-8e7e-018099d30fe1\",\"BDateUtcCreate\":\"2022-05-06T14:43:07.7804616\",\"BTimestamp\":\"AAAAAAAbv6Q=\",
	        \"ClientId\":\"1c5a29f5-b018-40c7-a6d4-017efe423d42\",\"UserId\":\"ea2c3b2a-23f6-49be-b882-0180891cef30\",\"Currency\":\"USD\",\"Description\":\"SecondaryWallet\",\"AmountAvailable\":381599,\"AmountWithheld\":29253372}]"
	}
	```

La respuesta a esta llamada devuelve la información de todas las wallets asociadas al mismo **userId** en formato de cadena, manteniendo la misma estructura que la respuesta de la llamada **Wallet GET**.

---