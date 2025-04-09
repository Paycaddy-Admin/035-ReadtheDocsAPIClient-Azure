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
