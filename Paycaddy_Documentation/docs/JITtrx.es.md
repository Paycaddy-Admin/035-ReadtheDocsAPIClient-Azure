!!!Important
    **Actualización de Esquema — 23 de junio de 2026:** Los webhooks de autorización ahora incluyen los campos `c22DatosPuntoServicio` y `c22Descripcion`. Consulta [Actualizaciones de Campos de Webhook](./webhookFieldUpdates.es.md) para más detalles.

Este documento explica el flujo de **Just In Time Funding (JIT)** que utiliza PayCaddy para autorizar transacciones de tarjeta mediante un endpoint de autorización externo provisto por el cliente. Describe los distintos tipos de notificaciones de transacciones online, cómo procesar las solicitudes de autorización y cómo implementar la consulta de saldo dentro del mismo flujo.

## Solicitudes de autorización en el flujo JIT

Para las transacciones de tipo `PeticionAutorizacion`, PayCaddy enviará una solicitud POST al endpoint de autorización externo del cliente. La solicitud incluye todos los campos relevantes y debe cumplir los siguientes requisitos HTTP:

- Usar el encabezado `X-API-KEY` para la autenticación.
    
- Incluir el encabezado `Accept: application/json`.
    
- Incluir el encabezado `Connection: keep-alive`.
    

> El endpoint externo debe responder en **1 segundo** como máximo. Si se excede este tiempo, la transacción se negará automáticamente con un código interno que representa un timeout.

#### **Formato de la solicitud**

```json
{
  "password": "password",
  "c1Tipo": "PeticionAutorizacionJIT",
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
  "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
  "IdCadena": "0102020963010037725",
  "c22DatosPuntoServicio": "000580JA0001",
  "c22Descripcion": {
      "CapturaDatosTarjeta": "Sin especificar",
      "AutenticacionCliente": "Sin capacidad de lectura",
      "RetencionTarjeta": "Sin capacidad de Captura",
      "TipoTerminal": "Terminal no atendido casa",
      "PresenciaCliente": "Cliente electrónico no seguro",
      "PresenciaTarjeta": "Tarjeta no presente",
      "MetodoCapturaDatos": "Internet",
      "MetodoAutenticacionCliente": "Datos 3D presentes",
      "EntidadAutenticadora": "Dispositivo no autentica cliente",
      "ActualizacionTarjeta": "Capacidad de actualización desconocida",
      "ImpresionOMensaje": "Capacidad de impresión desconocida",
      "LongitudMaximaPIN": "Sin longitud máxima"
  },
  "isExpiration": false,
  "isMulticlearing": false,
  "multiclearingClose": false
}
```

---

#### **Respuestas esperadas**

El endpoint **siempre** debe devolver una respuesta JSON booleana y, en algunos casos, incluir el atributo `amount` para mostrar saldo:

=== "Autorizar Transacción"
	```json
	{
	  "Funding": true
	}
	```

=== "Rechazar Transacción"
	```json
	{
	  "Funding": false
	}
	```

=== "Autorizar Transacción con Consulta de Saldo"
	```json
	{
	  "Funding": false,
	  "amount": int64 //Balance in cents
	}
	```

=== "Aprobar Consulta de Saldo"
	```json
	{
	  "amount": int64 //Balance in cents
	}
	```

---

#### **Webhook de notificación en caso de rechazo**

Si la transacción se niega, PayCaddy enviará un webhook al cliente con uno de los siguientes tipos:

=== "Rejection with On-Time Response"

	```json
	{
	  "c1Tipo": "RechazadaPorAutorizador",
	  ...
	}
	```

=== "Rejection Due to Timeout"

	```json
	{
	  "c1Tipo": "ExcedeTiempoAutorizacion",
	  ...
	}
	```

---

## Consulta de saldo

El JIT Funding admite consultas de saldo en dos escenarios:

1. **Consultas puras de saldo (sin retiro en cajero automático)**
    
    - Se identifican con `"C1Tipo": "JITConsultaSaldo"`.
        
    - El cuerpo de la solicitud es el mismo que el del webhook estándar.
        
    - Los valores válidos de `C3CodigoProceso` incluyen, entre otros: `300000, 300200, 300220, 300230, 301000, 302000, 303000`.
        
    
    === "Autorizar Consulta de Saldo"
	    ```json
	    {
	      "amount": int64 //Balance in cents
	    }
	    ```
    
    === "Rechazar Consulta de Saldo"
	    ```json
	    {
	      "Funding": false
	    }
	    ```
    
    > Si la respuesta excede 1 segundo, la transacción se niega automáticamente y se envía un webhook de tipo `ExcedeTiempoAutorizacion`.
    
2. **Retiros en cajero automático con visualización de saldo**
    
    - Se envían con `"C1Tipo": "PeticionAutorizacionJIT"`.
        
    - Se identifican porque su `C3CodigoProceso` sigue el patrón `01XXXXXX`.
        
    - Para mostrar el saldo en pantalla, la respuesta de autorización **debe** incluir:

```json
{
"Funding": true,
"amount": int64
}
```

---

## Consideraciones generales

- Todos los valores del campo `amount` se expresan en centavos (USD).
    
- El límite de tiempo de respuesta es de 1 segundo para todas las solicitudes JIT.
    
- Los rechazos por timeout o fondos insuficientes se notifican vía webhook.
    
- Algunos comercios pueden iniciar consultas de saldo como parte de flujos de verificación (por ejemplo, suscripciones).
    

---

## Tipos de notificación de transacciones online

Todas las transacciones online se notifican mediante webhook a la URL configurada por el cliente a través de **NotificationEnlist POST**. El formato del webhook respeta la estructura descrita en nuestro flujo JIT:

1. **Petición Autorización**  
    Disminuye el saldo disponible en la billetera del titular.
    
    ```json
{
  "password": "password",
  "c1Tipo": "PeticionAutorizacion",
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
  "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
  "IdCadena": "0102020963010037725",
  "c22DatosPuntoServicio": "000580JA0001",
  "c22Descripcion": {
      "CapturaDatosTarjeta": "Sin especificar",
      "AutenticacionCliente": "Sin capacidad de lectura",
      "RetencionTarjeta": "Sin capacidad de Captura",
      "TipoTerminal": "Terminal no atendido casa",
      "PresenciaCliente": "Cliente electrónico no seguro",
      "PresenciaTarjeta": "Tarjeta no presente",
      "MetodoCapturaDatos": "Internet",
      "MetodoAutenticacionCliente": "Datos 3D presentes",
      "EntidadAutenticadora": "Dispositivo no autentica cliente",
      "ActualizacionTarjeta": "Capacidad de actualización desconocida",
      "ImpresionOMensaje": "Capacidad de impresión desconocida",
      "LongitudMaximaPIN": "Sin longitud máxima"
  },
  "isExpiration": false,
  "isMulticlearing": false,
  "multiclearingClose": false
}
```

2. **Comunicación Autorización**  
    Autorización automática que no requiere aprobación externa; también reduce el saldo disponible.
    
    ```json
{
  "password": "password",
  "c1Tipo": "ComunicacionAutorizacion",
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
  "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
  "IdCadena": "0102020963010037725",
  "c22DatosPuntoServicio": "000580JA0001",
  "c22Descripcion": {
      "CapturaDatosTarjeta": "Sin especificar",
      "AutenticacionCliente": "Sin capacidad de lectura",
      "RetencionTarjeta": "Sin capacidad de Captura",
      "TipoTerminal": "Terminal no atendido casa",
      "PresenciaCliente": "Cliente electrónico no seguro",
      "PresenciaTarjeta": "Tarjeta no presente",
      "MetodoCapturaDatos": "Internet",
      "MetodoAutenticacionCliente": "Datos 3D presentes",
      "EntidadAutenticadora": "Dispositivo no autentica cliente",
      "ActualizacionTarjeta": "Capacidad de actualización desconocida",
      "ImpresionOMensaje": "Capacidad de impresión desconocida",
      "LongitudMaximaPIN": "Sin longitud máxima"
  },
  "isExpiration": false,
  "isMulticlearing": false,
  "multiclearingClose": false
}
    ```
    
3. **Comunicación Anulación**  
    Aumenta el saldo disponible en la billetera.
    
    ```json
{
  "password": "password",
  "c1Tipo": "ComunicacionAnulacion",
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
  "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
  "IdCadena": "0102020963010037725",
  "c22DatosPuntoServicio": "000580JA0001",
  "c22Descripcion": {
      "CapturaDatosTarjeta": "Sin especificar",
      "AutenticacionCliente": "Sin capacidad de lectura",
      "RetencionTarjeta": "Sin capacidad de Captura",
      "TipoTerminal": "Terminal no atendido casa",
      "PresenciaCliente": "Cliente electrónico no seguro",
      "PresenciaTarjeta": "Tarjeta no presente",
      "MetodoCapturaDatos": "Internet",
      "MetodoAutenticacionCliente": "Datos 3D presentes",
      "EntidadAutenticadora": "Dispositivo no autentica cliente",
      "ActualizacionTarjeta": "Capacidad de actualización desconocida",
      "ImpresionOMensaje": "Capacidad de impresión desconocida",
      "LongitudMaximaPIN": "Sin longitud máxima"
  },
  "isExpiration": false,
  "isMulticlearing": false,
  "multiclearingClose": false
}
    ```
    
4. **Petición Devolución**  
    Solicita un reembolso que se procesará offline mediante el proceso Batch. El importe declarado será “0”. Una vez confirmado por la red, el importe procesado se declarará en la `TransaccionCorregidaPositiva` correspondiente.
    
    ```json
{
  "password": "password",
  "c1Tipo": "PeticionDevolucion",
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
  "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
  "IdCadena": "0102020963010037725",
  "c22DatosPuntoServicio": "000580JA0001",
  "c22Descripcion": {
      "CapturaDatosTarjeta": "Sin especificar",
      "AutenticacionCliente": "Sin capacidad de lectura",
      "RetencionTarjeta": "Sin capacidad de Captura",
      "TipoTerminal": "Terminal no atendido casa",
      "PresenciaCliente": "Cliente electrónico no seguro",
      "PresenciaTarjeta": "Tarjeta no presente",
      "MetodoCapturaDatos": "Internet",
      "MetodoAutenticacionCliente": "Datos 3D presentes",
      "EntidadAutenticadora": "Dispositivo no autentica cliente",
      "ActualizacionTarjeta": "Capacidad de actualización desconocida",
      "ImpresionOMensaje": "Capacidad de impresión desconocida",
      "LongitudMaximaPIN": "Sin longitud máxima"
  },
  "isExpiration": false,
  "isMulticlearing": false,
  "multiclearingClose": false
}
    ```
    

> Las notificaciones de transacciones online no esperan respuesta; son avisos de eventos que pueden afectar el saldo.  
> En operaciones JIT, estas notificaciones pueden originarse tras la aprobación o el rechazo de la Autorización JIT.

>El campo **"IdCadena"** sirve para asociar las notificaciones de 'ComunicacionAnulacion' con su respectiva 'PeticionAutorizacion'. Una **PeticionAutorizacion** que reciba una **ComunicacionAnulacion** asociada se considerará una transacción confirmada o conciliada. Por lo tanto, no recibirá ninguna notificación posterior por conciliación via el Proceso Batch.


---

## Conciliación y correcciones del proceso Batch

Una vez procesadas las transacciones online, se concilian con los archivos de liquidación recibidos de la red. Las discrepancias o ajustes a la transacción original pueden reportarse mediante el [Proceso Batch](./settlement.es) con los siguientes tipos de corrección:

1. **TransaccionCorregidaPositiva**  
    Corrección en la que la liquidación final resulta **menor** al monto autorizado. La diferencia se **acredita** al saldo disponible de la billetera.
    
2. **TransaccionCorregidaNegativa**  
    Ajuste en el que la liquidación final resulta **mayor** al monto autorizado. El monto adicional se **debita** de la cuenta del titular.
    
3. **TransaccionConfirmada**  
    Indica que la autorización original coincidió y se confirmó durante la liquidación sin necesidad de ajuste.
    

> Estas correcciones se procesan de forma asíncrona y se reportan vía Batch, no mediante webhooks JIT.