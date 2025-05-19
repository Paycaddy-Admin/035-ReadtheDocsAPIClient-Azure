## **Debit Card <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/debitCards](https://api.api-sandbox.paycaddy.dev/v1/debitCards)

‍Esta llamada permite la creación de una **Tarjeta de Débito** y sigue la siguiente estructura:

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

Cada solicitud incluye el **userId** al que debe asociarse la tarjeta y el **walletId** cuyo saldo se utilizará para las transacciones de la tarjeta.

El campo **physicalCard** indica si se trata de una tarjeta que necesita imprimirse físicamente **(true)** o, de forma alternativa, de una tarjeta exclusivamente digital **(false)**.

Los datos de impresión de la tarjeta se extraen de los campos almacenados en la creación del usuario, por lo que es importante señalar que las tarjetas se imprimen tomando en cuenta los campos **FirstName** y **LastName** en el caso de personas naturales, y el campo **RegisteredName** en el caso de personas jurídicas. Es fundamental garantizar la integridad de estos campos en el flujo de creación de usuarios, incluidas las limitaciones de caracteres, ya que impactan el flujo posterior de creación de tarjetas.

El **"code"** enviado en la llamada debe ser proporcionado por el equipo de PayCaddy para cada tipo y variante de tarjeta incluida en el proyecto de habilitación. Por ejemplo, para un proyecto que habilite la emisión de tarjetas de débito virtuales y físicas para personas naturales, se proporcionarían dos códigos distintos, uno para **endUser physical** y otro para **endUser virtual**. Es responsabilidad del cliente invocar correctamente las llamadas teniendo en cuenta el tipo de usuario y la condición de impresión asociada al código proporcionado.

La respuesta exitosa de la llamada de creación de tarjeta devuelve un mensaje **200** que incluye el identificador único de la tarjeta (**cardId**), que debe usarse en todas las llamadas de operación de tarjetas **(consulta [cardOperations](./card_ops.es.md))**. Además del **cardId**, la respuesta 200 también proporciona un booleano que indica si la tarjeta está operativa (**isActive**) y un campo **status** que describe el estado de la tarjeta.

Los posibles valores de **status** son:

- **PendingAck** – Para tarjetas físicas recién creadas que aún no han sido activadas (consulta [AckReception POST](./card_ops.es.md#ack-reception-post)).
    
- **Temporarilyblocked** – Para bloqueos autoadministrables (consulta [UnblockCard POST](./card_ops.es.md#unblock-card-post)).
    
- **Cancel** – Para tarjetas canceladas (consulta [CancelCard POST](./card_ops.es.md#block-card-post)).
    
- **Active** – Para tarjetas activas.
    

Los datos sensibles de la tarjeta (PAN y CVV) pueden consultarse mediante las llamadas **[checkPan POST](./card_ops.es.md#check-pan-post)** y **[checkCvv POST](./card_ops.es.md#check-CVV-POST)**. Sin embargo, es importante señalar que estos datos **NO** deben almacenarse en bases de datos, ya que implican requisitos de ciberseguridad asociados con el estándar PCI que se abstraen con el uso de **cardId** en la API de PayCaddy.

La fecha de vencimiento de la tarjeta se presenta en la respuesta exitosa de la creación en el campo **"dueDate"** con el formato **YYYYMM**.

En caso de error o inconsistencia con los datos proporcionados, la API responderá con un error descriptivo **400** por alguno de los siguientes motivos frecuentes:

1. El **userId** al que se desea asociar la tarjeta no existe o está inactivo.
    
2. El **walletId** al que se desea asociar la tarjeta no existe o no pertenece al usuario indicado.
    
3. El **code** proporcionado en la llamada no coincide con el tipo de usuario al que se desea asociar la tarjeta.
    
4. El **code** proporcionado no coincide con el tipo de tarjeta (virtual o física) que se intenta crear.
    

> Es importante que los reintentos ante errores se gestionen de forma responsable, es decir, creando límites de tiempo de al menos **5 segundos** antes del reintento y limitando el número de reintentos a **3**.

---

## **Debit Card <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/debitCards/](https://api.api-sandbox.paycaddy.dev/v1/debitCards/)

La llamada **GET** para tarjetas de débito permite consultar los datos de una tarjeta a partir de un **cardId**.

=== "Request"
	```
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

Esta llamada es especialmente necesaria para gestionar el estado transaccional de la tarjeta, dado que la respuesta exitosa también incluye el booleano **"isActive"** y el campo descriptivo **"status"**, además de los otros campos presentes en la respuesta de creación de tarjeta. Si se envía un **cardId** no válido, la API de PayCaddy responderá con un error HTTP **400**.

Si se encuentra un error HTTP **500**, informa la incidencia al equipo de soporte de PayCaddy.

---

## **Prepaid Card <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/prepaidCards](https://api.api-sandbox.paycaddy.dev/v1/prepaidCards)

Esta llamada se utiliza para crear **Tarjetas Prepago** asociadas a un **userId** y un **walletId** determinados, y tiene la siguiente estructura:

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

Cada solicitud incluye el **userId** y el **walletId** cuyos fondos se utilizarán en las transacciones de la tarjeta.

El campo **physicalCard** indica si la tarjeta necesita imprimirse físicamente **(true)** o si será exclusivamente digital **(false)**.

Los datos de impresión y el uso de **code** siguen las mismas reglas descritas para **Debit Card POST**.

Los posibles valores de **status** y las advertencias sobre almacenamiento de PAN/CVV, formato de **dueDate** y manejo de errores también son idénticos a los descritos en la sección de tarjetas de débito.

---

## **Prepaid Card <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/prepaidCards/](https://api.api-sandbox.paycaddy.dev/v1/prepaidCards/)

La llamada **GET** para tarjetas prepago permite consultar la información de una tarjeta mediante su **cardId**.

=== "Request"
	```
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

Al igual que en el caso de las tarjetas de débito, la respuesta exitosa incluye **"isActive"** y **"status"**. Si se envía un **cardId** no válido, la API responderá con un error HTTP **400**. Para errores HTTP **500**, contacta al equipo de soporte de PayCaddy.

---

## **Credit Card <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/CreditCards](https://api.api-sandbox.paycaddy.dev/v1/CreditCards)

Esta llamada permite la creación de una **Tarjeta de Crédito** asociada a un **userId** y a un **walletId** concretos. Debe seguirse la siguiente estructura:

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

En cada llamada se incluye el **userId** y el **walletId** cuyo saldo respaldará las transacciones de la tarjeta.

Para tarjetas de crédito, el **walletId** asociado debe ser una **wallet de crédito** (consulta [WalletCredits](./creditCore.es.md#wallet-credit-post)) o una wallet prepago con el atributo **"walletType"** igual a **1**.

Los requisitos sobre **physicalCard**, **code**, valores de **status**, protección de datos sensibles, formato de **dueDate** y manejo de errores son los mismos que los descritos para **Debit Card POST**.

---

## **Credit Card <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/CreditCards/](https://api.api-sandbox.paycaddy.dev/v1/CreditCards/)

La llamada **GET** para tarjetas de crédito permite recuperar la información relevante asociada a un **cardId**:

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

Esta llamada resulta esencial para la gestión del estado de la tarjeta, dado que la respuesta incluye **"isActive"** y **"status"** junto con los demás campos. Si se envía un **cardId** no válido, la API responderá con un error HTTP **400**. Ante un error HTTP **500**, informa al equipo de soporte de PayCaddy.

---