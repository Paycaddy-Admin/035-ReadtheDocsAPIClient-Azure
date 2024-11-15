## **Debit Card <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/debitCards

‍Este metodo permite la creación de una Tarjeta de Débito y sigue la siguiente estructura:

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

Cada solicitud incluye el **userId** al que debe asociarse la tarjeta y el **walletId** con cuyo saldo se realizará la transacción.

El campo **tarjeta física** indica si se trata de una tarjeta que debe imprimirse físicamente **(verdadero)** o, por el contrario, de una tarjeta exclusivamente digital **(falso)**.

Los datos de impresión de las tarjetas se extraen de los campos almacenados en la creación de usuarios, por lo que es importante tener en cuenta que las tarjetas se imprimen teniendo en cuenta los campos **Nombre** y **Apellido** en el caso de personas físicas y el campo **NombreRegistrado** en el caso de personas jurídicas. Es fundamental garantizar la integridad de estos campos en el flujo de creación de usuarios, incluyendo la limitación de caracteres, ya que afecta al posterior flujo de creación de tarjetas.
El **"código"** enviado en la llamada debe ser proporcionado por el equipo de PayCaddy para cada tipo y variación de tarjeta incluida en el proyecto de habilitación. Es decir, para un proyecto que habilite la emisión de una tarjeta de débito virtual o física para personas físicas, se proporcionarían dos códigos diferentes, uno para usuario final físico y otro para usuario final virtual. Es responsabilidad del cliente invocar correctamente las llamadas teniendo en cuenta el tipo de usuario y la condición de impresión asociada al código proporcionado.

La respuesta satisfactoria de la llamada de creación de tarjeta de débito devuelve un mensaje 200 que lleva el identificador único de la tarjeta que debe utilizarse en todas las llamadas de operación de tarjeta **(véase [cardOperations](card_operations.es.md))**. Además del **cardId**, la respuesta 200 también proporciona un booleano que indica si la tarjeta está operativa o no, y un campo de estado que describe el estado de la tarjeta.

A continuación se detallan los posibles estados:

- **PendingAck** - Para tarjetas físicas recién creadas que no han sido activadas. (ver [AckReception POST](card_operations.es.md#ack-reception-post))
- **Temporarilyblocked** - Para bloqueos autogestionables. (ver [UnblockCard POST](card_operations.es.md#unblock-card-post))
- **Cancel** - Para tarjetas canceladas (ver [CancelCard POST](card_operations.es.md#block-card-post))
- **Active** - Para tarjetas activas

Los datos sensibles de la tarjeta (PAN y CVV) pueden ser consultados utilizando las llamadas **[checkPan POST](card_operations.es.md#check-pan-post)** y **[checkCvv POST](card_operations.es.md#check-cvv-post)**. Sin embargo, es importante notar que dichos datos NO deben almacenarse en bases de datos ya que implican requisitos de ciberseguridad asociados con el estándar PCI que han sido abstraídos con el uso del cardId en la API de PayCaddy.

La fecha de expiración de la tarjeta se presenta en la respuesta exitosa de la llamada de creación de la tarjeta en el campo "dueDate" siguiendo el formato YYYYMM.

En caso de cualquier error o inconsistencia con los datos proporcionados, la API responderá con un error descriptivo 400 de una de las razones más comunes posibles:

1. El userId al cual se debe asociar la tarjeta no existe o está inactivo.
2. El walletId al cual se debe asociar la tarjeta no existe o no pertenece al usuario ingresado.
3. El código proporcionado en la llamada no coincide con el tipo de usuario al cual se debe asociar la tarjeta.
4. El código proporcionado en la llamada no coincide con el tipo de tarjeta (virtual o física) que se intenta crear.

> Es importante notar que los reintentos en caso de errores deben ser gestionados de forma responsable, es decir, creando límites de tiempo de al menos 5 segundos antes de un nuevo intento y limitando el número de reintentos a 3.

---

## **Debit Card <font color="sky-blue">GET</font>**

**URL:** https://api.paycaddy.dev/v1/debitCards/
 
El metodo GET para tarjetas de débito permite consultar los datos de una tarjeta a partir de un cardId.

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
 
Esta llamada es especialmente necesaria para gestionar el estado de la transacción de la tarjeta, ya que la respuesta correcta también incluye la actividad booleana **"isActive“** y el campo descriptivo **"status"**, además de los otros campos de la respuesta correcta de creación de tarjeta. En caso de que se envíe un cardId no válido, la API de PayCaddy responderá con un error HTTP 400.

Si se produce un error HTTP 500, comunique la incidencia al equipo de asistencia de PayCaddy.

---

## **Prepaid Card <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/prepaidCards

Esta es la llamada utilizada para crear tarjetas prepago asociadas a un **userId** y **walletId** concretos, y lleva la siguiente estructura:

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

Cada solicitud incluye el **userId** al que debe asociarse la tarjeta y el **walletId** con cuyo saldo se realizará la transacción.

El campo **tarjeta física** indica si se trata de una tarjeta que debe imprimirse físicamente **(verdadero)** o, por el contrario, de una tarjeta exclusivamente digital **(falso)**.

Los datos de impresión de las tarjetas se extraen de los campos almacenados en la creación de usuarios, por lo que es importante tener en cuenta que las tarjetas se imprimen teniendo en cuenta los campos **Nombre** y **Apellido** en el caso de personas físicas y el campo **NombreRegistrado** en el caso de personas jurídicas. Es fundamental garantizar la integridad de estos campos en el flujo de creación de usuarios, incluyendo la limitación de caracteres, ya que afecta al posterior flujo de creación de tarjetas.
El **"código"** enviado en la llamada debe ser proporcionado por el equipo de PayCaddy para cada tipo y variación de tarjeta incluida en el proyecto de habilitación. Es decir, para un proyecto que habilite la emisión de una tarjeta de débito virtual o física para personas físicas, se proporcionarían dos códigos diferentes, uno para usuario final físico y otro para usuario final virtual. Es responsabilidad del cliente invocar correctamente las llamadas teniendo en cuenta el tipo de usuario y la condición de impresión asociada al código proporcionado.

La respuesta satisfactoria de la llamada de creación de tarjeta de débito devuelve un mensaje 200 que lleva el identificador único de la tarjeta que debe utilizarse en todas las llamadas de operación de tarjeta **(véase [cardOperations](card_operations.es.md))**. Además del **cardId**, la respuesta 200 también proporciona un booleano que indica si la tarjeta está operativa o no, y un campo de estado que describe el estado de la tarjeta.

A continuación se detallan los posibles estados:

- **PendingAck** - Para tarjetas físicas recién creadas que no han sido activadas. (ver [AckReception POST](card_operations.es.md#ack-reception-post))
- **Temporarilyblocked** - Para bloqueos autogestionables. (ver [UnblockCard POST](card_operations.es.md#unblock-card-post))
- **Cancel** - Para tarjetas canceladas (ver [CancelCard POST](card_operations.es.md#block-card-post))
- **Active** - Para tarjetas activas

Los datos sensibles de la tarjeta (PAN y CVV) pueden ser consultados utilizando las llamadas **[checkPan POST](card_operations.es.md#check-pan-post)** y **[checkCvv POST](card_operations.es.md#check-cvv-post)**. Sin embargo, es importante notar que dichos datos NO deben almacenarse en bases de datos ya que implican requisitos de ciberseguridad asociados con el estándar PCI que han sido abstraídos con el uso del cardId en la API de PayCaddy.

La fecha de expiración de la tarjeta se presenta en la respuesta exitosa de la llamada de creación de la tarjeta en el campo "dueDate" siguiendo el formato YYYYMM.

En caso de cualquier error o inconsistencia con los datos proporcionados, la API responderá con un error descriptivo 400 de una de las razones más comunes posibles:

1. El userId al cual se debe asociar la tarjeta no existe o está inactivo.
2. El walletId al cual se debe asociar la tarjeta no existe o no pertenece al usuario ingresado.
3. El código proporcionado en la llamada no coincide con el tipo de usuario al cual se debe asociar la tarjeta.
4. El código proporcionado en la llamada no coincide con el tipo de tarjeta (virtual o física) que se intenta crear.

> Es importante notar que los reintentos en caso de errores deben ser gestionados de forma responsable, es decir, creando límites de tiempo de al menos 5 segundos antes de un nuevo intento y limitando el número de reintentos a 3.

## **Prepaid Card <font color="sky-blue">GET</font>**

**URL:** https://api.paycaddy.dev/v1/prepaidCards/

El metodo GET para tarjetas prepago permite consultar los datos de una tarjeta basándose en un cardId.

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


Esta llamada es especialmente necesaria para gestionar el estado de la transacción de la tarjeta, ya que la respuesta correcta también incluye la actividad booleana **"isActive“** y el campo descriptivo **"status"**, además de los otros campos de la respuesta correcta de creación de tarjeta. En caso de que se envíe un cardId no válido, la API de PayCaddy responderá con un error HTTP 400.

Si se produce un error HTTP 500, comunique la incidencia al equipo de asistencia de PayCaddy.

---

## **Credit Card <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/CreditCards

Esta llamada permite la creación de una Tarjeta de Crédito asociada a un **userId** y **walletId** en particular. 

Se debe mantener la siguiente estructura para enviar los datos pertinentes.

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

En cada llamada se incluye el **userId** al que se debe asociar la tarjeta y el walletId con cuyo saldo se transaccionará la tarjeta.

Para tarjetas de crédito, el **walletId** a asociar debe ser un wallet de crédito **(ver [WalletCredits](creditCore.es.md#wallet-credit-post))** o un wallet prepago con el atributo **"walletType"** igual a "1".

El campo **physicalCard** indica si se trata de una tarjeta que necesita ser impresa físicamente **(true)** o, alternativamente, una tarjeta exclusivamente digital **(false)**.

Los datos de impresión de la tarjeta se extraen de los campos almacenados en la creación del usuario, por lo que es importante tener en cuenta que las tarjetas se imprimen teniendo en cuenta los campos **FirstName** y **LastName** en el caso de personas físicas y el campo **RegisteredName** en el caso de personas jurídicas. Es fundamental asegurar la integridad de estos campos en el flujo de creación del usuario, incluyendo las limitaciones de caracteres, ya que afecta al flujo posterior de creación de la tarjeta.

El **"código"** enviado en la llamada debe ser provisto por el equipo de PayCaddy para cada tipo y variación de tarjeta incluida en el proyecto de habilitación. Es decir, para un proyecto que habilita la emisión de una tarjeta de débito virtual o física para personas físicas, se proporcionarían dos códigos diferentes, uno para endUser physical y otro para endUser virtual. Es responsabilidad del cliente invocar correctamente las llamadas teniendo en cuenta el tipo de usuario y la condición de impresión asociada al código provisto.

La respuesta exitosa de la llamada de creación de la tarjeta de débito devuelve un mensaje 200 que lleva el identificador único de la tarjeta que debe usarse en todas las llamadas de operación de la tarjeta **(ver [cardOperations](card_operations.es.md))**. Además del **cardId**, la respuesta 200 también proporciona un booleano que indica si la tarjeta está operativa o no, y un campo de estado que describe el estado de la tarjeta.

Los posibles estados se detallan a continuación:

- **PendingAck** - Para tarjetas físicas recién creadas que no han sido activadas. (ver [AckReception POST](card_operations.es.md#ack-reception-post))
- **Temporarilyblocked** - Para bloqueos autogestionables. (ver [UnblockCard POST](card_operations.es.md#unblock-card-post))
- **Cancel** - Para tarjetas canceladas (ver [CancelCard POST](card_operations.es.md#block-card-post))
- **Active** - Para tarjetas activas

Los datos sensibles de la tarjeta (PAN y CVV) pueden ser consultados utilizando las llamadas **[checkPan POST](card_operations.es.md#check-pan-post)** y **[checkCvv POST](card_operations.es.md#check-cvv-post)**. Sin embargo, es importante notar que dichos datos NO deben almacenarse en bases de datos ya que implican requisitos de ciberseguridad asociados con el estándar PCI que han sido abstraídos con el uso del cardId en la API de PayCaddy.

La fecha de vencimiento de la tarjeta se presenta en la respuesta exitosa de la llamada de creación de la tarjeta en el campo "dueDate" siguiendo el formato AAAAMM.

En caso de cualquier error o inconsistencia con los datos proporcionados, la API responderá con un error 400 descriptivo de una de las posibles razones más comunes:

1. El userId al que se desea asociar la tarjeta no existe o está inactivo.
2. El walletId al que se desea asociar la tarjeta no existe o no pertenece al usuario ingresado.
3. El código proporcionado en la llamada no coincide con el tipo de usuario al que se desea asociar la tarjeta.
4. El código proporcionado en la llamada no coincide con el tipo de tarjeta (virtual o física) que se intenta crear.

> Es importante tener en cuenta que los reintentos en caso de errores deben gestionarse de forma responsable, es decir, creando límites de tiempo de al menos 5 segundos antes del reintento y limitando el número de reintentos a 3.

---

## **Credit Card <font color="sky-blue">GET</font>**

**URL:** https://api.paycaddy.dev/v1/CreditCards/

El metodo GET para una tarjeta de crédito le permite recuperar información relevante asociada a un cardId particular proporcionado a través de la siguiente estructura:

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
Esta llamada es especialmente necesaria para gestionar el estado de la transacción de la tarjeta, ya que la respuesta correcta también incluye la actividad booleana **"isActive“** y el campo descriptivo **"status"**, además de los otros campos de la respuesta correcta de creación de tarjeta. En caso de que se envíe un cardId no válido, la API de PayCaddy responderá con un error HTTP 400.

Si se produce un error HTTP 500, comunique la incidencia al equipo de asistencia de PayCaddy.

