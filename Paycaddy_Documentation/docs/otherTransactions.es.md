!!!Important
    **Actualización de Esquema — 23 de junio de 2026:** Se han añadido dos nuevos tipos de rechazo. Consulta [Actualizaciones de Campos de Webhook](./webhookFieldUpdates.es.md) para más detalles.

Existen **12 tipos adicionales de transacciones** que recibirás en línea a través de la URL registrada. Estos tipos **no afectan el saldo**: sirven únicamente como información sobre el uso de la tarjeta y explican las causas de los rechazos.

## Notificaciones de Rechazo

Estos valores aparecerán en el campo **`"c1Tipo"`** del webhook de notificación:

1. **DENEGADA:** La transacción fue rechazada por un motivo no especificado.
    
2. **DENEGADA. PIN INCORRECTO:** La transacción fue rechazada porque el PIN ingresado es incorrecto.
    
3. **DENEGADA. INTENTOS PIN EXCEDID:** La transacción fue rechazada porque se superó el número máximo de intentos de PIN.
    
4. **DENEGADA. TARJETA NO EFECTIVA:** La transacción fue rechazada porque la tarjeta no es válida o no está activa.
    
5. **DENEGADA. INCORRECTO CVV2:** La transacción fue rechazada porque se ingresó un CVV2 incorrecto.
    
6. **IMPORTE SUPERA LIMITE:** La transacción fue rechazada porque el importe excede el límite permitido para la tarjeta.
    
7. **NO HAY FONDOS:** La transacción fue rechazada porque la cuenta asociada a la tarjeta no tiene fondos suficientes.
    
8. **EXCEDIDO NUMERO DE OPERACION DIARIO:** La transacción fue rechazada porque se superó el número máximo de operaciones diarias permitidas para la tarjeta.
    
9. **FECHA CADUCIDAD ERRONEO:** La transacción fue rechazada porque se ingresó una fecha de vencimiento incorrecta.
    
10. **NotificacionDenegacion:** Normalmente no se recibe vía webhook; aparece cuando se utiliza el endpoint **TransactionDetailList** y representa una transacción rechazada por fondos insuficientes.
11. **DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE:** La transacción fue rechazada porque el comercio ha sido bloqueado por el tarjetahabiente.
12. **DENEGACION POR COMERCIO BLOQUEADO (GENERAL):** La transacción fue rechazada porque el comercio está bloqueado bajo una regla de bloqueo general.
    

En el esquema JSON de la mayoría de estas notificaciones, los campos **`"c38NumeroAutorizacion"`** y **`"c11NumeroIdentificativoTransaccion"`** se incluyen para comodidad, pero pueden venir vacíos (`""`) si la red no provee esa información.

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
    "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
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

## Transacciones MoneySend

Para mantenerte informado sobre los detalles de una transacción **MoneySend** iniciada, PayCaddy envía notificaciones webhook que siguen la misma estructura y llegan a la misma URL que el resto de las notificaciones. Las transacciones MoneySend se identifican porque su **`c3CodigoProceso`** es **`"820000"`**.

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
    "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES",
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

Al manejar los webhooks de MoneySend, almacénalos igual que cualquier otra notificación de transacción; la diferencia radica en cómo procesas el movimiento:

> Las transacciones MoneySend con **`c1Tipo = "PeticionAutorizacion"`** deben tratarse como **positivas**, es decir, **añaden fondos** al saldo de la billetera del destinatario en lugar de deducirlos.
> 
> Las transacciones MoneySend con **`c1Tipo = "ComunicacionAnulacion"`** deben tratarse como **negativas**, es decir, **restan fondos** del saldo de la billetera del destinatario.

Cuando muestres las transacciones de la tarjeta o billetera en el front-end, refleja esta lógica visualizando estos importes como entradas (cargos positivos) en lugar de débitos.