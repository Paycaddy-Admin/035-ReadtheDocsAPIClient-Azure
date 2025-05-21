En nuestro [Flujo Prefunded](./prefundedFlow.es.md), cuando una tarjeta emitida mediante la API de PayCaddy realiza una transacción en la red Mastercard, el motor de autorización de PayCaddy revisa la disponibilidad de fondos para esa transacción específica. Si, tras esta revisión, la operación procede, se envía un payload con uno de los cuatro tipos de transacción posibles a la URL de callback previamente registrada.

=== "PeticionAutorizacion"
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

=== "ComunicacionAutorizacion"
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

=== "ComunicacionAnulacion"
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

## Notificaciones de Transacciones Online

Existen cuatro tipos principales de transacciones que recibirás como eventos online:

1. **PeticionAutorizacion:** Esta transacción reducirá el saldo del cliente.
    
2. **ComunicacionAutorizacion:** Esta transacción también reducirá el saldo del cliente.
    
3. **ComunicacionAnulacion:** Esta transacción aumentará los fondos disponibles en la billetera del cliente.
    
4. **PeticionDevolucion:** Este tipo declara una solicitud de reverso que se procesará de forma offline mediante el proceso batch. La transacción se enviará con un importe declarado de “0”. Una vez que la red confirme el reembolso, el importe procesado se indicará en la **“TransaccionCorregidaPositiva”** correspondiente a través del proceso batch.
    

El tipo de transacción se indica en el campo **“c1Tipo”** del webhook. Por ejemplo, si **“c1Tipo”** indica **“PeticionAutorizacion”**, debe registrarse como una deducción en el saldo disponible.

> Es importante señalar que los webhooks de notificación de transacciones se envían con atributos y valores en español.  
> Esta documentación traduce algunos términos para facilitar su comprensión.

> El campo **“c7FechaHoraTransaccion”** representa la fecha y hora de la transacción en la zona donde se adquirió, con el formato “YYYYMMDDhhmmss”.

> El campo **“BDateUtcCreate”** representa la misma fecha y hora, pero en UTC, siguiendo el mismo formato que **c7FechaHoraTransaccion**.

---

## Conciliación y Correcciones del Proceso Batch

Una vez procesadas las transacciones online, se concilian con los archivos de liquidación recibidos de la red. Las discrepancias o ajustes a la transacción original pueden comunicarse mediante el [proceso batch](/settlement.es.md) usando los siguientes tipos de corrección:

1. **TransaccionCorregidaPositiva**  
    Indica un ajuste que confirma un importe mayor al autorizado inicialmente. El importe adicional se debitará de la cuenta del titular de la tarjeta.
    
2. **TransaccionCorregidaNegativa**  
    Indica una corrección que resulta en un importe final inferior al autorizado. El excedente se liberará nuevamente al saldo disponible de la billetera.
    
3. **TransaccionConfirmada**  
    Indica que la autorización original coincidió y se confirmó durante la liquidación sin necesidad de ajuste.
    

> Estas correcciones se procesan de forma asíncrona y se reportan vía batch, no mediante el mecanismo de webhooks JIT.
