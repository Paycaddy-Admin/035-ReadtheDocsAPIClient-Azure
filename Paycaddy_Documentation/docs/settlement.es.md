Para manejar la finalización del ciclo de vida de una transacción, utilizamos un **Proceso Batch** para actualizar y cerrar cada transacción aprobada, lo que permite visibilidad rápida de los datos reales de liquidación de cada transacción en tu programa de emisión de tarjetas.

PayCaddy aprovecha este procesamiento por lotes para enviar notificaciones a través de la ruta convencional de webhooks transaccionales (consulta [**NotificationEnlist POST**](/.notificationsEnlist.es.md)) como parte de un proceso diario destinado a confirmar y/o ajustar el monto previamente aprobado para cada transacción.

Nuestro sistema emplea tres tipos clave de notificaciones webhook para gestionar los distintos resultados posibles:

1. **TransaccionCorregidaPositiva:** Se envía cuando la liquidación final produce un ajuste **positivo** al monto inicialmente autorizado, añadiendo fondos a la billetera del usuario.
    
2. **TransaccionCorregidaNegativa:** Se envía cuando la liquidación final produce un ajuste **negativo**, deduciendo fondos de la billetera del usuario.
    
3. **TransaccionConfirmada:** Se envía para confirmar que la transacción se liquidó exactamente por el monto autorizado inicialmente, brindando una confirmación clara y precisa del resultado.
    

![settlement](./assets/imgs/settlement.svg)

## Casos de Uso de Corrección de Monto

Algunos ejemplos de transacciones que pueden corregirse mediante un proceso batch son:

1. **Alquiler de autos:** Al rentar un auto, la agencia suele autorizar un monto inicial para cubrir el alquiler y posibles daños. El monto final puede variar según el uso real del vehículo (distancia recorrida, combustible, etc.). En estos casos, el mensaje correctivo **TransaccionCorregidaPositiva** o **TransaccionCorregidaNegativa** ajusta el cargo final una vez devuelto el vehículo y calculado el importe definitivo.
    
2. **Reservas de hotel:** Al reservar una habitación, el hotel bloquea un monto para cubrir la estancia y cargos adicionales (minibar, room service). Al finalizar la estancia, el monto final puede diferir del bloqueado inicialmente. El mensaje correctivo **TransaccionCorregidaPositiva** o **TransaccionCorregidaNegativa** ajusta el cargo final tras calcular los cargos adicionales reales.
    

El procesamiento por lotes agiliza el manejo de transacciones diferidas y garantiza una liquidación correcta, reduciendo discrepancias y la necesidad de conciliaciones manuales. Estos webhooks mejoran la gestión de transacciones, minimizan errores y optimizan la administración en general.

> La mayoría de los webhooks relacionados con estas transacciones correctivas se envían durante la misma ventana nocturna, ya que el proceso de generación de webhooks se ejecuta de noche para consolidar y notificar los cambios en las transacciones del día.

Siempre que sea posible, se recomienda emparejar la transacción corregida con la transacción original en la interfaz de usuario. Así, los tarjetahabientes comprenderán mejor cómo y por qué se modificó su saldo.

## Esquemas de Webhook

La liquidación de transacciones se notificará mediante un webhook con uno de los siguientes esquemas:

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

## Conciliación de Transacciones y Captura de Fondos

- **Emparejamiento de transacciones:** Cada transacción incluye un identificador único `c11NumeroIdentificativoTransaccion`, crucial para emparejar la liquidación corregida o confirmada con su autorización inicial. Este proceso es esencial para mantener la integridad de la transacción y permite a los tarjetahabientes entender cómo y por qué se ajustaron sus saldos.
    
- **Detalles de notificación:** Los webhooks de liquidación contienen información sobre la transacción, incluido el importe, el identificador y el tipo de ajuste o confirmación.
    

> Aunque es menos común, puede ocurrir que un cargo llegue únicamente a través del proceso batch y no online. En ese caso, el webhook puede incluir menos información. Esta limitación obedece al protocolo de la red que gestiona estas confirmaciones y no depende de PayCaddy.