# Wallet Operations

## **Pay In <font color="green">POST</font>**

**URL:**  https://api.paycaddy.dev/v1/payIns

PayIn es el método de la API de PayCaddy para agregar fondos a una billetera específica. Es una llamada simple que carga la información del **'walletId'** al que se cargarán los fondos, el monto que se acreditará a la billetera en centavos, el código de la moneda según ISO 4217 y una descripción de la transacción.

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

La respuesta exitosa de esta llamada, además de la información proporcionada, incluye un timestamp con la fecha de ejecución de la transacción y un identificador único de transacción que permite recuperar la información asociada a la llamada GET.

Si hay un problema con los datos proporcionados, la API de PayCaddy responderá con un error HTTP 400.

=== "Wallet not found"
    ```json
    {
        "type": "Decoding body",
        "title": "Error ",
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

**URL:**  https://api.paycaddy.dev/v1/payIns/

El metodo GET para un PayIn permite acceder a información relacionada con un ID de transacción específico. La respuesta correcta de esta llamada carga la siguiente estructura:

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/payIns/{PAYIN_ID}
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

La llamada puede fallar si se proporciona un walletId incorrecto, en cuyo caso la API de PayCaddy respondería con un error HTTP 400.

---

## **Pay Out <font color="green">POST</font>**

**URL:**  https://api.paycaddy.dev/v1/payOuts

PayOut es un método para debitar fondos disponibles en una billetera específica. A diferencia de una transferencia, el uso de este método elimina la existencia de los fondos de la API de NeoBank. Este método debe estar asociado con una operación contable que requiera dicha llamada.

Al igual que PayIn, la respuesta exitosa devolverá los datos ingresados ​​acompañados de un identificador único de transacción y su marca de tiempo.

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

Si hay un problema con los datos proporcionados, la API de PayCaddy responderá con un error HTTP 400.

=== "Billetera no encontrada"
    ```json
    {
        "type": "Decoding body",
        "title": "Error ",
        "status": 422,
        "detail": "Wallet not found",
        "instance": ""
    }
    ```
=== "Moneda Incorrecta"
    ```json
    {
        "type": "",
        "title": "currency code does not match target walletId",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```
=== "Fondos Insuficientes"
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

**URL:**  https://api.paycaddy.dev/v1/payOuts/

El metodo GET para un pago permite acceder a información relacionada con un **transactionId** específico. La respuesta correcta de esta llamada carga la siguiente estructura:


=== "Request"
    ```json
    https://api.paycaddy.dev/v1/payOuts/{PAYOUT_ID}
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

La API responderá con un error HTTP 400 si se trata de un transactionId incorrecto o de un transactionId que no está relacionado con un PayOut.

---

## **Transfer <font color="green">POST</font>**

**URL:**  https://api.paycaddy.dev/v1/transfers

Para realizar transacciones dentro del entorno API de NeoBank entre dos wallets previamente creadas, se debe utilizar la llamada **Transfer POST**, la cual tiene la siguiente estructura:

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

En esta llamada se debe especificar el **walletIdFrom** al cual se están enviando los fondos y el **walletIdTo** al cual se están enviando los fondos. El campo **"currency"** debe cargar el código de la moneda asociada a ambas billeteras siguiendo el estándar ISO 4217.

La API de PayCaddy no gestiona conversiones de moneda, por lo que es necesario considerar que ambas billeteras deben estar configuradas bajo la misma moneda.

El monto ingresado en el campo **"amount"** debe estar en centavos siguiendo el estándar de las otras llamadas a la API de PayCaddy.

El campo **"statement"** permite ingresar una descripción de la transacción para referencia futura y visualización en el front-end.

La respuesta exitosa a esta llamada carga un identificador único de la transacción y el timestamp de su aceptación.

En caso de un error en la llamada, la API de NeoBank responderá con un error HTTP 400 descriptivo.

Comúnmente, la llamada fallará si los walletIds involucrados en la transacción pertenecen a un userId inactivo, es decir, manteniendo su atributo "isActive" como "false".

De manera similar, la llamada siempre fallará si el **walletIdFrom** no mantiene el saldo adecuado para cubrir la transacción o cuando el código de moneda de ambos wallets no es el mismo que el de la llamada POST de transferencia.

---

## **Transfer <font color="sky-blue">GET</font>**

**URL:**  https://api.paycaddy.dev/v1/transfers/

El metodo GET para una transferencia entre billeteras permite acceder a la información relacionada con un transactionId específico y tiene la siguiente estructura:

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/transfers/{TRANSFER_ID}
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

La API responderá con un error HTTP 400 descriptivo si se proporciona un ID de transacción incorrecto o si el ID de transacción no está relacionado con una transferencia.