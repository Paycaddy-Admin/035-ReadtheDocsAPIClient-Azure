# Controles Transaccionales

Los controles transaccionales son mecanismos que te permiten determinar cómo se autorizan las transacciones sobre las tarjetas en tu programa. Los controles descritos en este capítulo operan a nivel de programa y aplican a todas las tarjetas emitidas bajo tu `clientId`.

El control disponible actualmente es la **lista de bloqueo de comercios**, que te permite bloquear comercios específicos para que no puedan aprobar transacciones sobre las tarjetas que has emitido. Cada comercio bloqueado se identifica con su código `c42` alfanumérico de 15 dígitos — el mismo valor `c42Comercio` que aparece en cada webhook transaccional. Cuando un titular intenta una transacción en un comercio cuyo código está en tu lista de bloqueo, la transacción se rechaza y se envía un webhook de tipo `DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE` a tu URL de callback. Consulta [Otras Transacciones](./otherTransactions.es.md) para el esquema de la notificación de rechazo.

PayCaddy también mantiene una lista de bloqueo global (la lista **general**) y listas específicas por programa (**prefunded** y **jit**) que se aplican de forma automática. Las transacciones rechazadas bajo estas listas producen un webhook de rechazo de tipo `DENEGACION POR COMERCIO BLOQUEADO (GENERAL)`.

---

## **Update Block List <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/BlockListC42](https://api.api-sandbox.paycaddy.dev/v1/BlockListC42)

Esta llamada agrega o elimina un único comercio de tu lista de bloqueo. Cada petición opera sobre un único comercio — para modificar varios comercios, envía varias peticiones. El `clientId` propietario de la lista se captura automáticamente desde el API key autenticado en la sesión, por lo que solo necesitas especificar el código del comercio y la operación.

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

|Campo|Descripción|
|---|---|
|`c42code`|Identificador alfanumérico de 15 dígitos del comercio (`c42Comercio` en los webhooks) que se desea agregar o eliminar de la lista de bloqueo.|
|`operation`|`"add"` para incluir el comercio en la lista de bloqueo o `"remove"` para retirarlo.|

La respuesta exitosa es **HTTP 201 Created**. La respuesta replica el código del comercio modificado, la `operation` aplicada (`"add"` o `"remove"`) y el timestamp en el que se registró el cambio.

La API responderá con **HTTP 422** en los siguientes casos:

=== "c42code inválido"
	```json
	{
	    "status": 422,
	    "detail": "c42code must be a 15-digit alphanumeric value"
	}
	```

=== "Valor en campo `operation` inválido"
	```json
	{
	    "status": 422,
	    "detail": "operation must be 'add' or 'remove'"
	}
	```

=== "Ya existe en la lista"
	```json
	{
	    "status": 422,
	    "detail": "This c42code was already included in the list"
	}
	```

=== "No existe en la lista"
	```json
	{
	    "status": 422,
	    "detail": "This c42code does not exist in the list"
	}
	```

> Cada actualización exitosa se registra del lado de PayCaddy. Si necesitas una auditoría de los cambios históricos sobre tu lista de bloqueo, contacta al equipo de integración.

---

## **Get Block List <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/BlockListC42](https://api.api-sandbox.paycaddy.dev/v1/BlockListC42)

Devuelve la lista de bloqueo de comercios asociada a tu `clientId` junto con cualquier lista de nivel de programa que aplique a las tarjetas de tu programa (como las listas **prefunded**, **jit** y **general** mantenidas por PayCaddy).

=== "Response"
	```json
	{
	    "secret": "string",
	    "date": "2026-06-23T10:15:30Z"
	}
	```

El campo `secret` contiene la información estructurada de la lista de bloqueo: tu lista específica de códigos `c42`, más las listas mantenidas por PayCaddy (`prefunded`, `jit`, `general`) que apliquen a tu programa.

> Las transacciones hacia comercios en tu lista específica producen un webhook de rechazo de tipo `DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE`. Las transacciones hacia comercios en la lista **general** producen un webhook de rechazo de tipo `DENEGACION POR COMERCIO BLOQUEADO (GENERAL)`. Consulta [Otras Transacciones](./otherTransactions.es.md) para ambos esquemas.
