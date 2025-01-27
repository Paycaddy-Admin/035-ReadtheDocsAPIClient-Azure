## **Notification Enlist <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/NotificationEnlist

Las transacciones generadas por la red Mastercard por tarjetas creadas en la API de NeoBank se detallan en notificaciones vía webhook que deben almacenarse en una base de datos para ser consumidas en un módulo de gestión de transacciones.

Estas notificaciones se envían a una URL dedicada compartida con el equipo de PayCaddy de forma segura mediante el consumo del endpoint **NotificationsEnlist POST**. Para los clientes que utilizan nuestro core de crédito, se agregó un nuevo campo llamado **creditNotificationsURL**, donde se espera otra URL dedicada para recibir notificaciones relacionadas con el core de crédito, que contemplan información como intereses, valores ejecutados y monto adeudado.

!!!Tip
    Recomendamos **encarecidamente** utilizar direcciones diferentes para **URL** y **creditNotificationURL**, aunque el punto final acepte el mismo valor para ambas

A continuación se detalla la estructura de la llamada empleada para registrar una URL para recibir **Notificaciones de transacciones y Notificaciones de crédito**:

=== "Request"
    ```json
    {
        "URL": "string",
        "Password": "string",
        "apiKey": "string",
        "creditNotificationsURL": "string"
    }
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "clientid": "string",
        "Message": "Enlist created",
        "URL": "string",
        "apiKey": "",
        "creditNotificationsURL": "string",
        "creationDate": "2024-08-06T09:24:57.4887257+00:00"
    }
    ```

Los valores de **apiKey** y **password** son opcionales, solo son necesarios si el valor de la URL tiene un método de autenticación para recibir los webhooks. Si no es necesario un apiKey, se puede dejar como un valor vacío, entre comillas, o no incluirlo en la solicitud. Aunque la **Password** se puede enviar como un valor vacío o una cadena, debe incluirse en el cuerpo de la solicitud.

Durante la integración, una vez que se realiza correctamente la llamada POST de NotificationEnlist, se debe gestionar una prueba con el equipo de integración de PayCaddy para verificar la recepción exitosa de las cargas útiles de la transacción.

---

## **Notificaciones de transacciones**

Cada vez que una tarjeta emitida en la API de NeoBank realiza una transacción en la red Mastercard, se enviará a la URL indicada una carga útil como uno de los cuatro tipos de transacciones ejemplificadas a continuación.

=== "Peticion Autorizacion"
    ```json
    {
        "password": "password",
        "c1Tipo": "PeticionAutorizacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```

=== "ComunAutorizacion"
    ```json
    {
        "password": "password",
        "c1Tipo": "ComunicacionAutorizacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }

    ```
=== "ComunAnulacion"
    ```json
    {
        "password": "password",
        "c1Tipo": "ComunicacionAnulacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```
=== "PeticionDevolucion"
    ```json
    {
        "password": "password",
        "c1Tipo": "PeticionDevolucion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "BDateUtcCreate": " YYYYMMDDHHMMSS ",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
    ```

## **Principales tipos de transacciones**

Existen tres tipos principales de transacciones que recibirás como tipo online:

1. **AuthorizationRequest:** Esta transacción disminuirá el saldo del cliente.
2. **AuthorizationCommunication:** Esta transacción también disminuirá el saldo del cliente.
3. **CancellationCommunication:** Esta transacción aumentará los fondos disponibles en la billetera del cliente.
4. **ReversalRequest:** Este tipo de transacción declara una solicitud de reversión de dinero que se procesará offline a través del Proceso por Lotes. Esta transacción tendrá un monto declarado de "0". Una vez que la red confirme el reembolso, el monto procesado se declarará en la "TransaccionCorregidaPositiva" relacionada a través del Proceso por Lotes.

El tipo de transacción se indica en el campo **"c1Tipo"** del webhook. Por ejemplo, si el campo **"c1Tipo"** indica que es una transacción de tipo **"AuthorizationRequest"**, debe presentarse como una deducción en el saldo disponible.

>Es importante tener en cuenta que los webhooks de Notificación de Transacción se envían con atributos y valores nombrados en español.
Esta documentación traduce algunos términos para facilitar su comprensión.

>El campo **"c7FechaHoraTransaccion"** representa la información de la hora de la transacción en la zona horaria donde se adquirió la transacción en el formato "AAAAMMDDhhmmss"

>El campo **"BDateUtcCreate"** representa la información de la hora de la transacción en la zona horaria UTC siguiendo el mismo formato que **c7FechaHoraTransaccion**

**Tipos adicionales de notificación de transacción**

Hay 9 tipos adicionales de transacciones que recibirá en línea a través de la URL indicada. Debe tenerse en cuenta que estos tipos no afectan el saldo y son simplemente informativos del uso de la tarjeta y describen la causa de las transacciones rechazadas.

Estos tipos se verán como valores en el campo **"c1Tipo"** de la notificación del webhook:

1. **DENEGADA:** La transacción fue rechazada por una razón no especificada.
2. **DENEGADA. PIN INCORRECTO:** La transacción fue rechazada porque el PIN ingresado es incorrecto.
3. **DENEGADA. INTENTOS PIN EXCEDID:** La transacción fue rechazada porque se ha excedido el número máximo de intentos de ingreso de PIN.
4. **DENEGADA. TARJETA NO EFECTIVA:** La transacción fue rechazada porque la tarjeta no es válida o no está activa.
5. **DENEGADA. INCORRECTO CVV2:** La transacción fue rechazada porque se ingresó un CVV2 incorrecto.
6. **IMPORTE SUPERA LIMITE:** La transacción fue rechazada porque el monto de la transacción excede el límite permitido para la tarjeta.
7. **NO HAY FONDOS:** La transacción fue rechazada porque la cuenta asociada con la tarjeta no tiene fondos suficientes.
8. **EXCEDIDO NÚMERO DE OPERACIÓN DIARIA:** La transacción fue rechazada porque se ha excedido el número máximo de operaciones diarias permitidas para la tarjeta.
9. **FECHA CADUCIDAD ERRÓNEA:** La transacción fue rechazada debido a que se ingresó una fecha de vencimiento incorrecta.
10. **NotificacionDenegacion:** Estas no se verán normalmente en los webhooks recibidos, sino que se encontrarán al usar el endpoint **TransactionDetailList**. Representan una transacción que ha sido rechazada debido a fondos insuficientes.

En el esquema del JSON enviado en la mayoría de estos tipos adicionales de notificaciones, los campos **"c38NumeroAutorización"** y **"c11NumeroIdentificativoTransaccion"** están presentes para conveniencia del cliente, sin embargo, es importante mencionar que estos campos no siempre contendrán información y, por lo tanto, se presentarán como cadenas vacías ("").

```json
    {
        "password": password,
        "c1Tipo": "DENEGADA. INCORRECTO CVV2",
        "c2CardId": cardId,
        "c3CodigoProceso": "000000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
```

---

## **Transacciones MoneySend**

Para mantenerse informado sobre los detalles de una transacción MoneySend iniciada, PayCaddy proporciona notificaciones webhook. Estos webhooks siguen la misma estructura y viajan a través de la misma URL que otras notificaciones de transacciones. Sin embargo, las transacciones MoneySend se pueden identificar por el valor **c3CodigoProceso** de **"820000"**.

```json
    {
        "password": "password",
        "c1Tipo": "PeticionAutorizacion",
        "c2CardId": "cardId",
        "c3CodigoProceso": "820000",
        "c4ImporteTransaccion": "000000001617",
        "c7FechaHoraTransaccion": "20220429052901",
        "c11NumeroIdentificativoTransaccion": "000004339",
        "c18CodigoActividadEstablecimiento": "5999",
        "c19CodigoPaisAdquirente": "442",
        "c38NumeroAutorizacion": "040031",
        "c41TerminalId": "00227759",
        "c42Comercio": "227759000156182",
        "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
    }
```

Al manejar las notificaciones de webhook de MoneySend, almacénelas como cualquier otra notificación de transacción. La principal diferencia está en cómo procesa la transacción.

>Las transacciones de MoneySend con **c1Tipo** igual a **“PeticionAutorizacion”** deben manejarse como transacciones **positivas** agregando fondos al saldo de la billetera del destinatario en lugar de deducirlos.

>Las transacciones de MoneySend con **c1Tipo** igual a **“ComunicacionAnulacion”** deben manejarse como transacciones **negativas** restando fondos del saldo de la billetera del destinatario en lugar de deducirlos.

Al registrar transacciones con tarjeta o billetera en el front-end, esto debe reflejarse mostrando el monto de dichas transacciones como entradas a los fondos de la billetera en lugar de cargos.

---

## **Proceso por lotes**

>**Notificación de actualización:** A partir del 11 de abril, la funcionalidad de nuestro proceso por lotes se ha ampliado para incluir la notificación de webhook "TransaccionConfirmada". Esta incorporación garantiza una supervisión integral de todas las liquidaciones de transacciones, ya sea que coincidan con los montos autorizados inicialmente o requieran ajustes.

Para gestionar la finalización del ciclo de vida de una transacción, utilizamos un proceso por lotes para actualizar y finalizar el ciclo de vida de cada transacción aprobada, lo que permite una rápida visibilidad de los datos de liquidación reales en su programa de emisión de tarjetas.

PayCaddy aprovecha este procesamiento por lotes para enviar notificaciones a través de la ruta de webhook transaccional convencional (consulte NotificationEnlist POST) como parte de un proceso diario para confirmar o ajustar el monto previamente aprobado para cada transacción.

Nuestro sistema utiliza tres tipos clave de notificaciones de webhook para gestionar varios resultados de transacciones:

1. **TransaccionCorregidaPositiva:** Se emite cuando la liquidación final da como resultado un ajuste positivo del monto autorizado inicialmente, lo que agrega fondos a la billetera del usuario.
2. **Transacción Corregida Negativa:** Se emite cuando la liquidación final da como resultado un ajuste negativo, deduciendo así fondos de la billetera del usuario.
3. **Transacción Confirmada (Nueva):** Se introduce para confirmar que la transacción se ha liquidado exactamente por el monto autorizado inicialmente, proporcionando una confirmación clara y precisa del resultado de la transacción.

Un par de ejemplos de transacciones que podrían corregirse mediante un proceso por lotes son:

1. **Alquiler de coches:** Cuando un cliente alquila un coche, la empresa de alquiler suele autorizar un importe inicial en la tarjeta del cliente para cubrir el coste del alquiler y los posibles desperfectos. No obstante, el importe final podría variar en función del uso real del vehículo y de otros factores, como la distancia recorrida o el coste del combustible. En estos casos, se podría utilizar el mensaje corrector "TransaccionCorregidaPositiva" o "TransaccionCorregidaNegativa" para ajustar el importe final cargado en la tarjeta del cliente, una vez devuelto el vehículo y realizado el cálculo final.
2. **Reservas de hotel:** Cuando un cliente reserva una habitación de hotel, el hotel suele bloquear un importe en la tarjeta del cliente para cubrir el coste de la estancia y los posibles cargos adicionales, como el minibar o el servicio de habitaciones. Al final de la estancia, el importe final puede ser diferente del importe bloqueado inicialmente, en función del uso real de estos servicios adicionales. En estos casos, se podría utilizar el mensaje correctivo "TransaccionCorregidaPositiva" o "TransaccionCorregidaNegativa" para ajustar el monto final cargado a la tarjeta del cliente, una vez calculados los cargos finales.

El procesamiento por lotes agiliza el manejo de transacciones diferidas y garantiza una liquidación adecuada, reduciendo las discrepancias y la necesidad de conciliación manual. Estos webhooks mejoran el manejo de las transacciones, minimizan los errores y optimizan la gestión general.

>La mayoría de los webhooks relacionados con estas transacciones correctivas se enviarán durante el mismo período durante la noche.
Esto se debe a que nuestro proceso de generación de webhooks es un proceso por lotes que se ejecuta durante la noche para consolidar y notificar si hubo cambios en las transacciones del día.

Siempre que sea posible, se recomienda hacer coincidir la transacción corregida con la transacción original. De esta manera, los tarjetahabientes comprenderán mejor cómo y por qué se modificó su saldo.

Esta práctica mejora la transparencia y fomenta la confianza del cliente al permitirle estar al tanto de todas las transacciones y cambios realizados en sus cuentas. Es esencial que los titulares de tarjetas se sientan informados y en control de su actividad financiera.


=== "TransaccionCorregidaPositiva"
    ```json
    {
        "password": "password",
        "c1Tipo": "TransaccionCorregidaPositiva",
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
    }
    ```
=== "TransaccionCorregidaNegativa"
    ```json
    {
        "password": "password",
        "c1Tipo": "TransaccionCorregidaNegativa",
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
    }
    ```
=== "TransaccionConfirmada"
    ```json
    {
        "password": "password",
        "c1Tipo": "TransaccionConfirmada",
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
    }
    ```

**Coincidencia de transacciones y generación de informes**

- **Transacciones coincidentes:** Cada transacción incluye un identificador único (**c11NumeroIdentificativoTransaccion**), que es crucial para hacer coincidir la liquidación corregida o confirmada con su autorización inicial. Este proceso de coincidencia es esencial para mantener la integridad de la transacción y proporciona a los titulares de tarjetas una comprensión clara de cómo y por qué se ajustaron sus saldos.
- **Detalles de notificaciones:** Las notificaciones de webhook brindan información detallada sobre la transacción, incluido el monto de la transacción, el identificador y el tipo de ajuste o confirmación. Esto garantiza que todas las partes estén informadas con precisión sobre el estado de la transacción.
Haga esto con la información que viaja a través del campo **"c11NumeroIdentificativoTransaccion"**.
> Una vez que se modificó una transacción, no se modificará nuevamente

Aunque es menos común, también puede ocurrir que un cargo llegue solo en el proceso por lotes y no en línea. Si este es el caso, el webhook puede llegar con menos información como se presenta a continuación. Cabe señalar que esta información es de acuerdo con el protocolo de la red para manejar estas confirmaciones y no depende de PayCaddy.

---

## **Notificaciones KYC integradas**

Como parte de nuestro flujo KYC integrado para personas físicas, después de que los usuarios completen los pasos de verificación de identidad en la URL compartida en el campo **kycURL** (consulte el POST de usuario final)**, el sistema de nuestro proveedor de verificación de identidad procesa los datos enviados y los compara con listas negras y realiza verificaciones AML. Nuestra API aprovecha estas verificaciones y proporciona notificaciones de estado a través de webhook.

Para recibir correctamente las notificaciones de las verificaciones KYC, es necesario mantener una URL para recibir mensajes de la API de PayCaddy a través de webhook. Debe comunicarse con el equipo de integración de PayCaddy para configurar el envío de notificaciones a esa URL.

### **Webhooks de verificación KYC**

El webhook de validación KYC (Know Your Customer) es una notificación que se envía a los clientes con información relevante sobre el estado del proceso de verificación KYC. La información se entrega en formato JSON y puede incluir diferentes estados y descripciones, según los casos de verificación.

| **Campo** | **Descripción** |
| ------------- | ------------- |
| **metadata** | Información crucial para identificar al usuario en forma de su **userId** |
| **status** | El estado de la verificación KYC, que puede ser "verificado", "rechazado" o "revisión necesaria" |
| **description** | Una descripción detallada del estado de la verificación. |
| **fullName** | El nombre completo del usuario |
| **age** | La edad del usuario |
| **Timestamp** | La marca de tiempo ISO 8601 de la respuesta de los webhooks |

```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "string",
        "description": "string",
        "fullName": "string",
        "age": "string",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
```

### **Estados de verificación y descripción.**

El flujo del webhook para una validación sigue el diagrama que se describe a continuación.

![entity_diagram](./assets/imgs/states_descriptions.png){class="img"}

Siguiendo este flujo de proceso, a continuación se muestran los webhooks correspondientes a cada estado.
Cada uno de estos webhooks proporciona una descripción del estado correspondiente, incluyendo los motivos de rechazo en caso de verificaciones fallidas. Las descripciones correspondientes se detallan a continuación para su referencia.

**‍Entradas de verificación completadas:** Este estado indica que un usuario ha completado con éxito la captura de datos KYC a través de **kycURL** y tendrá la estructura y la información que se muestran a continuación:

```json
    {
        "metadata": {
            "userId": "unique user identifier"
        },
        "status": "verification_inputs_completed",
        "description": "Ongoing verification",
        "fullName": "Document OCR capture ongoing",
        "age": "Document OCR capture ongoing",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
```

**Verified:** Este estado se refiere a cuando un usuario ha pasado con éxito el proceso de verificación KYC. Esto indica que el usuario ha sido activado con éxito en nuestra base de datos y puede continuar con otros flujos de creación y operaciones de tarjetas. La respuesta del webhook tendrá la siguiente estructura e información:

```json
    {
        "metadata": {
            "userId": "ID único del usuario"
        },
        "status": "verified",
        "description": "Verification signed",
        "fullName": "User's full name",
        "age": "User's age",
        "timeStamp": "ISO 8601 format timestamp"
    }
```

**Rejected:** Este estado se refiere a cuando un usuario no ha pasado el proceso de verificación KYC. A continuación se detallan los diferentes casos de rechazo.

=== "Input mismatch"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "Document doesn’t match input data",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
        }
    ```

=== "AML Checks Rejected"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "AML checks rejected",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "User underage"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "The user is underage",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "Document Expired"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "The document uploaded is expired",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "Negligence"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "The document uploaded has issues",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```
    
=== "Others"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "Validation Failed. User cannot be verified",
        "fullName": "User's complete name",
        "age": "null",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

**ReviewNeeded:** Este estado se refiere a cuando un usuario no ha pasado el proceso de verificación KYC. A continuación se detallan los diferentes casos de rechazo.

=== "Negligence"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "The document uploaded has issues",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "AML Checks Review"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "AML checks need to be reviewed",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```
=== "System Inoperative"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "Government validation system inoperative",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```
=== "Others"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "Validation failed. Pending compliance review",
        "fullName": "User's complete name",
        "age": "null",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```