PayCaddy envía webhooks a tus URLs de callback para las siguientes notificaciones:

• **Notificaciones de Transacciones**, enlistadas mediante Notification Enlist POST  
• **Notificaciones de Movimientos de Crédito**, también enlistadas mediante Notification Enlist POST (beta)  
• [**Notificaciones de KYC Integrado**](./kyc.es.md), enlistadas manualmente por nuestro equipo de _onboarding_ durante la integración.

---

## Notification Enlist <font color="green">POST</font>

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/NotificationEnlist](https://api.api-sandbox.paycaddy.dev/v1/NotificationEnlist)

Las transacciones generadas por la red Mastercard a través de las tarjetas creadas con la API NeoBank se detallan en notificaciones vía webhook que deben almacenarse en una base de datos para ser consumidas por un módulo de gestión de transacciones.

Estas notificaciones se envían a una URL dedicada que el cliente comparte con el equipo de PayCaddy de forma segura consumiendo el endpoint **NotificationEnlist POST**. Para los clientes que usan nuestro _credit core_, se añadió un nuevo campo llamado **creditNotificationsURL**, donde se espera otra URL dedicada para recibir notificaciones relacionadas con el módulo de crédito, que contemplan información como intereses, valores ejecutados y monto adeudado.

!!!Tip  
	Recomendamos **encarecidamente** usar direcciones distintas para **URL** y **creditNotificationsURL**, aunque el endpoint acepte el mismo valor para ambas.

La estructura de la llamada empleada para enlistar una URL que reciba **Notificaciones de Transacciones y de Crédito** se detalla a continuación:

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

Los valores de **apiKey** y **Password** son opcionales; solo se necesitan si la URL indicada tiene un método de autenticación para recibir los webhooks.

- Si no se requiere **apiKey**, puede dejarse como valor vacío entre comillas o simplemente omitirse del cuerpo de la solicitud.
    
- El campo **Password** puede enviarse como cadena vacía o con un valor, pero **debe** incluirse en el cuerpo de la solicitud.
    

Durante la integración, una vez que la llamada NotificationEnlist POST se ejecute con éxito, se debe coordinar una prueba con el equipo de integración de PayCaddy para verificar la recepción correcta de los payloads de transacción.

---