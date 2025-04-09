# Credit Operations

## **Wallet Credit <font color="green">POST</font>** 

**URL:** https://api.paycaddy.dev/v1/WalletCredits

La creación de una billetera de crédito se realiza a través de una llamada POST que lleva la siguiente estructura:


=== "Request"
    ```json
    {
        "userId": "string",
        "currency": "string",
        "description": "string",
        "time": 0,
        "limit": 0,
        "firstCutDate": "2024-08-06T09:48:23.648Z",
        "creditProductCode": "string"
    }
    ```
=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "currency": "USD",
        "description": "string",
        "limit": 0,
        "creationDate": "2024-07-30T07:26:52.9871618+00:00"
    }
    ```

Esta llamada debe realizarse asociándola a un **userId** activo creado previamente.

El campo **“currency”** debe considerar códigos ISO 4217. (Por ejemplo, Dólares de Estados Unidos se ingresarían con el código “USD”).

El campo **“description”** se utiliza para nombrar la billetera según su uso previsto, pero no es obligatorio para crear una nueva billetera.

El campo **“time”** define la cantidad de días que la línea de crédito debe permanecer activa. Este campo se utiliza para definir el plazo de un crédito revolving **(ver [ReportPayCredit](#report-pay-credit-post))**.

El campo **“limit”** establece el monto del límite de crédito en centavos que tendrá la billetera por mes.

El **“firstCutDate”** es un valor de marca de tiempo en formato ISO 8601, que define cuándo se regenerará la línea de crédito y según las especificaciones de **creditProductCode**, generará los cargos necesarios.

Finalmente, el **creditProductCode** es un código de 3 números generado previamente por el equipo de PayCaddy que especifica las reglas para la línea de crédito que se establecerá para esta nueva billetera.

---

## **Wallet Credit <font color="sky-blue">GET</font>** 

**URL:** https://api.paycaddy.dev/v1/WalletCredits/

La llamada GET para carteras de crédito permite recuperar la información asociada al walletId consultado. Esta llamada es particularmente importante para verificar el saldo disponible en una cartera, el saldo retenido debido a transacciones pendientes, los límites internos de la cartera y el monto gastado durante el mes.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/WalletCredits/{WALLET_ID}
    ``` 

=== "Response"
    ```json
    {
        "id": "string",
        "userId": "string",
        "currency": "string",
        "description": "string",
        "limit": 0,
        "amountOwed": 0,
        "interestOwed": 0,
        "amountAvailable": 0,
        "creationDate": "2024-11-12T12:48:39.573Z"
    }
    ``` 

El **saldo total** se refleja en el campo "saldo" de la respuesta 200 exitosa a esta llamada, mientras que el **saldo retenido** se refleja en el campo **"amountWithheld"**.

Todos los montos se reflejan en centavos, lo que significa que USD 1000,00 se representaría como 100 000.

Para una billetera creada recientemente, los campos **internalWalletLimit** y **amountSpentMonth** vienen de manera predeterminada con el valor 0. Para cambiar el **internalWalletLimit**, verifique **[Change Internal Wallet Limit POST](wallet.es.md#change-internal-card-limit-post).**

La llamada puede fallar si se proporciona un walletId incorrecto, en cuyo caso la API de NeoBank respondería con un error HTTP 400.

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

## **Report Pay Credit <font color="green">POST</font>** 

**URL:** https://api.paycaddy.dev/v1/reportPayCredits

El endpoint ReportPayCredit está dirigido a usuarios que tienen una línea de crédito activa, el valor del pago puede ser correspondiente o parcial, en este caso generando intereses al deudor, a la deuda total para restablecer el crédito a través de su walletId.

El valor reportado para amountToCapital se utilizará para pagar las deudas deudoras y nominales. El monto para cada uno se determina considerando el valor desdoblado de creditProduct relacionado con el walletId.

=== "Request"
    ```json
    {
        "walletId": "string",
        "amountToCapital": 0,
        "currency": "string",
        "statement": "string"
    }
    ```
=== "Response"
    ```json
    {
        "creationDate": "2024-08-07T15:52:55.407Z",
        "limit": "string",
        "amountAvailable": "string"
    }
    ```

El campo denominado **amountToCapital** determina el monto que el usuario está reportando, el cual tendrá un impacto en el monto disponible, por parte del usuario dentro de la línea de crédito.

El campo **currency** corresponde a la moneda de la billetera. El campo **statement** se utilizará para dejar una breve descripción de la operación realizada.

En caso de ser exitosa, se generará el reporte de pago con los datos especificados, y la llamada retornará un HTTP 200 detallando la fecha en la que se realizó el reporte, así como el monto del límite de la billetera y el saldo disponible. También se envía una notificación de webhook **(ver [notificationEnlist](prefundedTRX.es.md#notification-enlist-post))** para informar el reporte y sus efectos.

En caso contrario, la API de NeoBank responderá con un error HTTP 422. Los errores más comunes pueden incluir:

- **walletId** no válido o no crediticio.
- El valor de la moneda no coincide con la moneda original de la billetera.
- **amountToCapital** mayor que el monto adeudado.

---

## **Report Pay Credit Capital <font color="green">POST</font>** 

**URL:** https://api.paycaddy.dev/v1/ReportPayCreditsCapital

El endpoint **ReportPayCreditCapital** es utilizado por usuarios que tienen una línea de crédito activa. Espera un pago correspondiente o parcial, en casos parciales generando intereses deudores, para restaurar el crédito a través de su **walletId**.

Este endpoint no considera el valor **split** del creditProductCode **(ver [changeInterestCapital](#))**, ya que solo descuenta el monto reportado del **amountOwed**, no considera si la billetera está en deuda y tiene intereses deudores ejecutados no pagados.

=== "Request"
    ```json
    {
        "walletId": "string",
        "amountToCapital": 0,
        "currency": "string",
        "statement": "string"
    }
    ```
=== "Response"
    ```json
    {
        "creationDate": "2024-08-07T11:28:37.022Z",
        "limit": "string",
        "amountAvailable": "string"
    }
    ```

El campo denominado **amountToCapital** determina el monto que el usuario está reportando, el cual tendrá un impacto en el monto disponible, por parte del usuario dentro de la línea de crédito.

El campo **currency** corresponde a la moneda de la billetera. 

El campo **statement** se utilizará para dejar una breve descripción de la operación realizada.

En caso de ser exitosa, se generará el reporte del pago con los datos especificados, y la llamada retornará un HTTP 200 detallando la fecha en la que se realizó el reporte, así como los valores del límite de la tarjeta y el saldo disponible. También se envía una notificación de webhook **(ver [notificationEnlist](prefundedTRX.es.md#notification-enlist-post))** para informar el reporte y sus efectos.

En caso contrario, la API de NeoBank responderá con un error HTTP 422. Los errores más comunes pueden incluir:

- **walletId** no válido o no crediticio.
- El valor de la moneda no coincide con la moneda original de la billetera.
- **amountToCapital** mayor que el monto adeudado.   

---

## **Change Wallet Limit <font color="green">POST</font>** 

**URL:** https://api.paycaddy.dev/v1/WalletCredits/ChangeWalletLimit

Este método se utilizará para modificar los límites previamente establecidos en las billeteras de crédito de los usuarios de la API de PayCaddy. El límite solo se puede aumentar por cuestiones contables.


=== "Request"
    ```json
    {
        "walletId": "string",
        "newLimit": 0,
        "currency": "string",
        "statement": "string"
    }
    ```
=== "Response"
    ```json
    {
        "walletId": "string",
        "newLimit": 0
    }
    ```

La llamada se asociará al 'walletId' proporcionado, seguido del valor ingresado en el campo 'newLimit', que representará el monto que se debe actualizar en la propiedad 'limit' de la billetera.

El campo 'currency' debe corresponder a la propiedad 'currency' de la billetera indicada y, finalmente, el campo 'statement' proporciona una breve descripción de la operación realizada.

Todos los montos se reflejan en centavos, es decir, USD 1,000.00 se representaría como 100000.

La llamada puede fallar si se proporciona un walletId incorrecto o se envía un valor menor en **newLimit**, en cuyo caso la API de PayCaddy respondería con un error HTTP 400.

---


## **Check Credit Operation <font color="sky-blue">GET</font>** 

**URL:** https://api.paycaddy.dev/v1/CheckCredit/

La llamada GET de CheckCreditOperation permite recuperar información sobre un **'walletId'** consultado. Esta llamada es especialmente importante para verificar el estado de actividad del monedero.

Si se realiza correctamente, la llamada devolverá un HTTP 200 que detalla el 'walletId' asociado y el estado del monedero.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/CheckCredit/{WALLET_ID}
    ```
=== "Response"
    ```json
    {
        "walletId": "string",
        "status": "string"
    }
    ```

El valor del atributo 'status' puede ser:

- **True** si la billetera está activada y disponible para transacciones.
- **False** si está bloqueada, ya sea por el sistema o por falta de pago (ver [ReportPayCredit](#report-pay-credit-post)).

