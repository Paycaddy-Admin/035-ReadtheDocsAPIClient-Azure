Existen **10 tipos adicionales de transacciones** que recibirás en línea a través de la URL registrada. Estos tipos **no afectan el saldo**: sirven únicamente como información sobre el uso de la tarjeta y explican las causas de los rechazos.

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
    "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
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
    "c43IdentificadorComercio": "AMZN Mktp ES             Amazon.ES"
}
```

Al manejar los webhooks de MoneySend, almacénalos igual que cualquier otra notificación de transacción; la diferencia radica en cómo procesas el movimiento:

> Las transacciones MoneySend con **`c1Tipo = "PeticionAutorizacion"`** deben tratarse como **positivas**, es decir, **añaden fondos** al saldo de la billetera del destinatario en lugar de deducirlos.
> 
> Las transacciones MoneySend con **`c1Tipo = "ComunicacionAnulacion"`** deben tratarse como **negativas**, es decir, **restan fondos** del saldo de la billetera del destinatario.

Cuando muestres las transacciones de la tarjeta o billetera en el front-end, refleja esta lógica visualizando estos importes como entradas (cargos positivos) en lugar de débitos.