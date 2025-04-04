# Card Operations

## **ACK Reception <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/ackReception

Como medida de seguridad en el proceso logístico de entrega de plásticos físicos, por defecto todas las tarjetas físicas creadas en la API de NeoBank nacen inactivas. Esto se refleja en los campos “isActive” y “status” de las respuestas exitosas para la creación de estas tarjetas, así como al realizar una llamada GET para todo tipo de tarjetas, como se ve en el siguiente ejemplo:

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

Una vez al día, a una hora de corte específica, todas las tarjetas físicas creadas en el ciclo se envían a la instalación de impresión. La llamada **ackReception POST** no devolverá una respuesta exitosa hasta que se haya impreso la tarjeta solicitada, por lo que es una llamada que solo debe realizarse cuando la tarjeta impresa esté en la mano o 48 horas después de una respuesta exitosa para la creación de la tarjeta.

El propósito de esta llamada es permitir que la tarjeta se entregue al titular de la tarjeta en un estado inactivo, a la espera de la validación de recepción por parte del titular de la tarjeta una vez que tenga la tarjeta en la mano.

La validación de la tarjeta en la mano debe gestionarse comparando los datos ingresados ​​por el titular de la tarjeta (como un rango del PAN o el CVV) con los resultados de las llamadas realizadas para obtener dichos datos sensibles (consulte **[checkPan POST](#check-pan-post)** y **[checkCvv POST](#check-cvv-post)**). Si los datos ingresados ​​coinciden, se puede realizar la llamada ackReception POST para activar la tarjeta por primera vez, simplemente enviando el cardId de la tarjeta que se activará en la llamada.

La respuesta exitosa a la llamada simplemente replica el **cardId** ingresado. En caso de encontrar un error 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

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

**URL:** https://api.paycaddy.dev/v1/cardOperations/blockCard

Esta llamada permite el cambio autogestionado del estado de actividad de la tarjeta. Esta llamada afecta directamente al booleano **"isActive"** transformándolo en **"false"** en función de una llamada simple que solo lleva el **cardId**.

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

La API podría responder con un error HTTP 400 descriptivo si se trata de un intento de bloquear una tarjeta que ya se ha bloqueado anteriormente. En el caso de las tarjetas activas, a menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Unblock Card <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/unblockCard

Esta llamada permite el cambio autogestionado del estado de actividad de la tarjeta. Esta llamada afecta directamente al booleano "isActive" transformándolo en **"true"** en función de una llamada simple que solo lleva el cardId.

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

La API podría responder con un error HTTP 400 descriptivo si se trata de un intento de desbloquear una tarjeta activa. En el caso de las tarjetas bloqueadas previamente que actualmente están inactivas, a menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Cancel Card <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/cancelCard

!!!Danger
    Los resultados que se producen con la acción de este método son irreversibles. No se debe utilizar a la ligera si produce efectos contables.

Esta llamada permite cancelar de forma permanente una tarjeta creada previamente. Esta llamada afecta directamente al booleano **"isActive"**, cambiándolo a **"false"**, así como al campo **"status"**, cambiándolo a **"Cancel"** a partir de una llamada que simplemente carga el **cardId**.

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

La API de PayCaddy podría responder con un error HTTP 400 descriptivo si se trata de un intento de cancelar una tarjeta que ya ha sido cancelada.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Check PAN <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/checkPan

Esta llamada permite acceder a datos confidenciales como el número de tarjeta y la fecha de vencimiento. Requiere un cardId como entrada y tiene la siguiente estructura:

!!!Warning
    Las respuestas a esta convocatoria no deben almacenarse en bases de datos por razones de seguridad.

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

A menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Check CVV <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/checkCvv

Esta llamada permite acceder a datos confidenciales como el CVV (valor de verificación de la tarjeta) de la tarjeta. Requiere un ID de cardId como entrada y tiene la siguiente estructura:

!!!Warning
    Las respuestas a esta convocatoria no deben almacenarse en bases de datos por razones de seguridad.

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

A menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Check Exp. Date <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/checkDueDate

Esta llamada permite acceder a la fecha de caducidad de la tarjet. Requiere una cardId como entrada y lleva la siguiente estructura:

!!!Warning
        Las respuestas a esta convocatoria no deben almacenarse en bases de datos por razones de seguridad.

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

A menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Check PIN <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/checkPin

Esta llamada permite acceder a los datos sensibles del PIN de la tarjeta, necesario para todos los retiros en cajeros automáticos y para las transacciones en puntos de venta de algunas regiones geográficas.

!!!Warning
    Las respuestas de esta convocatoria no deben almacenarse en bases de datos por razones de seguridad.

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

A menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Unblock PIN <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/unblockPin

Esta llamada permite reactivar el PIN de una tarjeta luego de que se produzca un bloqueo de seguridad debido a tres intentos incorrectos. El número total de intentos incorrectos aceptables antes del bloqueo de seguridad es un parámetro que se puede discutir durante la fase de definición del alcance del proyecto, específicamente para perfiles de tarjetas personalizados.

!!!Warning
    Las respuestas de esta convocatoria no deben almacenarse en bases de datos por razones de seguridad.

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

A menos que se ingrese un ID de tarjeta incorrecto, la API responderá con un mensaje HTTP 200 exitoso.

En caso de encontrar un error HTTP 500, comuníquese con el equipo de soporte de PayCaddy con los detalles del caso.

---

## **Change PIN <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/cardOperations/changePin

Esta llamada permite gestionar los datos sensibles del PIN de la tarjeta, necesarios para todas las extracciones en cajeros automáticos y para algunas zonas geográficas en los dispositivos de punto de venta. La llamada está diseñada para asignar un PIN de 4 dígitos especificado por el usuario. Las respuestas a esta llamada no deben almacenarse en bases de datos por razones de seguridad.

!!!Warning
    Las respuestas de esta convocatoria no deben almacenarse en bases de datos por razones de seguridad.

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



