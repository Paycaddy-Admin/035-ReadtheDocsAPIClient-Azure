# Card Operations

## **ACK Reception <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/ackReception](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/ackReception)

Como medida de seguridad en el proceso logístico de entrega de plásticos físicos, **todas las tarjetas físicas** creadas con la API de NeoBank nacen inactivas. Esto se refleja en los campos **"isActive"** y **"status"** de las respuestas exitosas de creación, así como en las respuestas de las llamadas **GET** para cualquier tipo de tarjeta:

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

Una vez al día, a una hora de corte específica, se envían a la imprenta todas las tarjetas físicas creadas durante el ciclo. La llamada **ackReception POST** **no** devolverá una respuesta exitosa hasta que la tarjeta solicitada haya sido impresa, por lo que solo debe invocarse cuando el plástico esté en mano o 48 horas después de la creación exitosa.

El objetivo es permitir que la tarjeta se entregue al titular en estado inactivo y esperar la validación de recepción una vez que la tenga en mano. Esa validación debe gestionarse comparando los datos que introduce el titular (por ejemplo, rango del PAN o el CVV) con los resultados de las llamadas que exponen dichos datos sensibles (**[checkPan POST](./card_ops.es.md#check-pan-post)** y **[checkCvv POST](./card_ops.es.md#check-cvv-post)**). Si los datos coinciden, se puede llamar a **ackReception POST** enviando únicamente el **cardId** para activar la tarjeta.

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

En caso de error **500**, contacta al equipo de soporte de PayCaddy con los detalles del caso.

---

## **Block Card <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/blockCard](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/blockCard)

Esta llamada permite **bloquear** la tarjeta de forma autogestionada, cambiando el booleano **"isActive"** a **false** mediante un llamado que únicamente incluye el **cardId**.

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

Si se intenta bloquear una tarjeta ya bloqueada, la API devolverá un error descriptivo **HTTP 400**. Para tarjetas activas, salvo que se envíe un **cardId** erróneo, la respuesta será **HTTP 200 OK**. Ante un error **500**, contacta a soporte.

---

## **Unblock Card <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/unblockCard](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/unblockCard)

Esta llamada permite **desbloquear** la tarjeta de forma autogestionada, cambiando **"isActive"** a **true** con solo enviar el **cardId**.

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

La API devolverá un **HTTP 400** si se intenta desbloquear una tarjeta ya activa. Para tarjetas bloqueadas válidas, la respuesta será **HTTP 200 OK**. Contacta a soporte si encuentras un **HTTP 500**.

---

## **Cancel Card <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/cancelCard](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/cancelCard)

!!!Caution
	Los resultados de esta acción son **irreversibles**. Úsala con precaución, ya que genera efectos contables permanentes.

Esta llamada **cancela permanentemente** una tarjeta, cambiando **"isActive"** a **false** y **"status"** a **"Cancel"** mediante un llamado que solo incluye el **cardId**.

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

Si se intenta cancelar una tarjeta ya cancelada, la API devolverá un **HTTP 400** descriptivo. Ante un **HTTP 500**, contacta a soporte.

---

## **Check PAN <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkPan](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkPan)

Esta llamada da acceso a datos sensibles como el número de la tarjeta (PAN) y la fecha de vencimiento. Necesita el **cardId** como entrada.

!!!Warning  
Las respuestas de esta llamada **no deben** almacenarse en bases de datos por motivos de seguridad.

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

Salvo que el **cardId** sea incorrecto, la respuesta será **HTTP 200 OK**. Para un **HTTP 500**, contacta a soporte.

---

## **Check CVV <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkCvv](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkCvv)

Permite acceder al **CVV** de la tarjeta. Requiere el **cardId**.

!!!Warning  
Las respuestas de esta llamada **no deben** almacenarse en bases de datos por motivos de seguridad.

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

Con un **cardId** válido, devuelve **HTTP 200 OK**; de lo contrario, error **400**. Ante un **HTTP 500**, contacta a soporte.

---

## **Check Exp. Date <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkDueDate](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkDueDate)

Permite obtener la fecha de expiración de la tarjeta. Requiere el **cardId**.

!!!Warning  
Las respuestas de esta llamada **no deben** almacenarse en bases de datos por motivos de seguridad.

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

Con un **cardId** válido, devuelve **HTTP 200 OK**; de lo contrario, error **400**. Para **HTTP 500**, contacta a soporte.

---

## **Check PIN <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkPin](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/checkPin)

Permite acceder al **PIN** (necesario para retiros en cajeros y algunas transacciones POS). Requiere el **cardId**.

!!!Warning  
Las respuestas de esta llamada **no deben** almacenarse en bases de datos por motivos de seguridad.

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

Con un **cardId** válido, devuelve **HTTP 200 OK**; de lo contrario, error **400**. Para **HTTP 500**, contacta a soporte.

---

## **Unblock PIN <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/unblockPin](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/unblockPin)

Reactiva el PIN después de un bloqueo de seguridad (tres intentos fallidos). El máximo de intentos puede definirse durante el alcance del proyecto. Requiere el **cardId**.

!!!Warning  
Las respuestas de esta llamada **no deben** almacenarse en bases de datos por motivos de seguridad.

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

Respuesta **HTTP 200 OK** con **cardId** válido; error **400** si el **cardId** es incorrecto. Para **HTTP 500**, contacta a soporte.

---

## **Change PIN <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/cardOperations/changePin](https://api.api-sandbox.paycaddy.dev/v1/cardOperations/changePin)

Permite asignar o cambiar el **PIN** (4 dígitos) de la tarjeta. Requiere el **cardId**.

!!!Warning  
Las respuestas de esta llamada **no deben** almacenarse en bases de datos por motivos de seguridad.

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

Respuesta **HTTP 200 OK** con **cardId** válido; error **400** si el **cardId** es incorrecto. Para **HTTP 500**, contacta a soporte.

---

## **Change Internal Card Limit <font color="green">POST</font>

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/ChangeLimitIWL](https://api.api-sandbox.paycaddy.dev/v1/ChangeLimitIWL)

Cada tarjeta tiene un **límite interno** configurado por el cliente para controlar el gasto del usuario. El límite se reinicia al inicio de cada mes y cada transacción se aprueba según el límite vigente. Solo los clientes pueden modificarlo.

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

La API responderá con **HTTP 422 "card not found"** si el **cardId** no corresponde al cliente autenticado.

---

## **Change Internal Card Limit <font color="sky-blue">GET</font>

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/ChangeLimitIWL/](https://api.api-sandbox.paycaddy.dev/v1/ChangeLimitIWL/)

Permite consultar el **límite interno** actual de una tarjeta específica. Envía el **cardId** en la URL.

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

Si el límite es **0** o no se ha establecido, el campo **limit** mostrará **"No stablish internal limit"**. 

Se devolverá **HTTP 422 "card not found"** si el **cardId** no pertenece al cliente autenticado.
