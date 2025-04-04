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
