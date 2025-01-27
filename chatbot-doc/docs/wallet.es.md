## **Wallets <font color="green">POST</font>** 

**URL:** https://api.paycaddy.dev/v1/wallets

‍La creación de un monedero se realiza a través de una llamada POST que lleva la siguiente estructura:


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

Esta llamada debe realizarse asociándola a un userId activo previamente creado.

El campo de moneda debe considerar los códigos de la norma ISO 4217. (por ejemplo, los dólares estadounidenses se introducirían con el código «USD»).

El campo descripción se crea para dar nombre al monedero creado según el uso previsto.

El campo "walletType" define si el monedero puede o no asociarse a una tarjeta de crédito (ver POST creditCard). Este booleano debe mantenerse como «0» para todos los monederos creados para tarjetas de débito, tarjetas prepago o para la gestión de fondos y transferencias. Para los monederos creados para tarjetas de crédito de tipo prepago, el "walletType" debe definirse como "1".

> Las **"Carteras principales"** creadas automáticamente en el flujo de creación de usuarios mantienen un **"walletType"** de **"0"**.

---

## **Wallets <font color="sky-blue">GET</font>** 

**URL:** https://api.paycaddy.dev/v1/wallets

El metodo GET para monederos permite recuperar la información asociada al walletId consultado. Esta llamada es especialmente importante para comprobar el saldo disponible en un monedero y el saldo retenido debido a transacciones pendientes.

=== "Request"
    ```json
    https://api.paycaddy.dev/v1/wallets/{WALLET_ID}
    
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

El saldo total se refleja en el campo «saldo» de la respuesta 200 satisfactoria a esta llamada, mientras que el saldo retenido se refleja en el campo "importeRetenido".

Todas las cantidades se reflejan en céntimos, lo que significa que 1.000,00 USD se representarían como 100000.

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

## **Wallet Per User ID <font color="green">POST</font>** 

**URL:** https://api.paycaddy.dev/v1/WalletsPerUserIds

‍Este endpoint permite recuperar todos los monederos asociados a un ID de usuario.Realizando una petición POST con el ID del usuario, el sistema devuelve una lista con los identificadores de los diferentes monederos que tiene el usuario.

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
        ‍{\"Id\":\"216fa217-6826-48b8-ab5b-0180891cef30\",\"BDateUtcCreate\":\"2022-05-03T08:50:16.4959966\",\"BTimestamp\":\"AAAAAAAb6Pk=\",
        \"ClientId\":\"1c5a29f5-b018-40c7-a6d4-017efe423d42\",\"UserId\":\"qa2c3b2a-23f6-49be-b882-0180891cef30\",\"Currency\":\"USD\",\"Description\":\"Main wallet\",\"AmountAvailable\":38574,\"AmountWithheld\":1110756},

        {\"Id\":\"d6928675-a4fe-4752-8e7e-018099d30fe1\",\"BDateUtcCreate\":\"2022-05-06T14:43:07.7804616\",\"BTimestamp\":\"AAAAAAAbv6Q=\",
        \"ClientId\":\"1c5a29f5-b018-40c7-a6d4-017efe423d42\",\"UserId\":\"ea2c3b2a-23f6-49be-b882-0180891cef30\",\"Currency\":\"USD\",\"Description\":\"SecondaryWallet\",\"AmountAvailable\":381599,\"AmountWithheld\":29253372}]"
    }
    ```

La respuesta a esta llamada lleva toda la información de las múltiples carteras asociadas al mismo UserId en un formato de cadena que mantiene la misma estructura que la respuesta de la llamada Wallet GET.

---

## **Change Internal Card Limit <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/ChangeLimitIWL

Cada tarjeta generada por la API de PayCaddy tiene una serie de propiedades que se utilizan a la hora de realizar transacciones. Una de ellas es el límite asignado por los clientes para controlar el gasto de sus usuarios. El valor del límite determinará la cantidad que se puede transaccionar y puede variar para cada usuario. Sólo los clientes pueden realizar cambios y actualizaciones en el valor del límite.

Los límites se restablecen a principios de cada mes, y cada transacción se aprueba según el límite vigente en ese momento.

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

La API responderá con un error HTTP 422 "tarjeta no encontrada" si el cardId proporcionado no coincide con el del cliente autenticado.

---

## **Change Internal Card Limit <font color="sky-blue">GET</font>**

**URL:** https://api.paycaddy.dev/v1/ChangeLimitIWL/

Para recuperar el límite actual de una tarjeta en particular se debe utilizar el método GET para el Límite Interno de Cartera.

La solicitud debe realizarse con el **cardId** concreto que se desea consultar. En el ejemplo siguiente, el cardId aparece resaltado en la URL de la solicitud.

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

En caso de que el límite sea 0 o no se haya establecido previamente, la respuesta llevará un mensaje de **"No stablish internal limit** como atributo de límite.

La API responderá con un error HTTP 422 **"tarjeta no encontrada"** si el cardId proporcionado no coincide con el del cliente autenticado.

