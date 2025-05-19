Existen dos métodos para obtener una lista de transacciones asociadas a una entidad en nuestra API:

## **Lista de Detalle de Transacciones por Tarjeta <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/TransactionDetailList](https://api.api-sandbox.paycaddy.dev/v1/TransactionDetailList)

Esta llamada **POST** recupera todas las transacciones de la tarjeta cuyo **`cardId`** se indique.

=== "Request"
	```json
	{
	    "cardId": "904ee16a-db84-4120-9714-0180892341b3",
	    "startDate": "2023-01-23T18:21:02.157Z"
	}
	```

=== "Response"
	```json
	{
	    "transactionListJson": 
	    "[{
	        \"c1Tipo\":\"PeticionAutorizacion\",\"c2CardId\":\"904ee16a-db84-4120-9714-0180892341b3\",\"c3CodigoProceso\":\"000000\",\"c4ImporteTransaccion\":\"116\",\"c7FechaHoraTransaccion\":\"20230203194512\",\"c11NumeroIdentificativoTransaccion\":\"000088058\",\"c18CodigoActividadEstablecimiento\":\"4111\",\"c19CodigoPaisAdquiren
	        te\":\"724\",\"c38NumeroAutorizacion\":\"169\",\"c41TerminalId
	        \":\"00004001\",\"c42Comercio\":\"000000047219191\",\"c43Ident
	        ificadorComercio\":\"TRANVIA DE MURCIA        CHURRA-MURCIA  
	        \"},{\"c1Tipo\":\"PeticionAutorizacion\",\"c2CardId\":\"904ee16a-db84-4120-9714-0180892341b3\",\"c3CodigoProceso\":\"000000\",\"c4ImporteTrans
	        accion\":\"2266\",\"c7FechaHoraTransaccion\":\"20230317135513\
	        ",\"c11NumeroIdentificativoTransaccion\":\"000099870\",\"c18Co
	        digoActividadEstablecimiento\":\"5813\",\"c19CodigoPaisAdquire
	        nte\":\"724\",\"c38NumeroAutorizacion\":\"171\",\"c41TerminalI
	        d\":\"90315259\",\"c42Comercio\":\"000000175240563\",\"c43Iden
	        tificadorComercio\":\"BULEVAR CAFE EL RANERO   MURCIA
	        \"
	    }]"
	}
	```

La estructura de los datos de cada transacción sigue el mismo formato que el resto de notificaciones de transacciones. (Consulta [Webhooks de Notificación de Transacciones Online](./prefundedTRX.es.md))

---

## **Lista de Detalle de Transacciones por Billetera <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/TransactionDetailListByWallet](https://api.api-sandbox.paycaddy.dev/v1/TransactionDetailListByWallet)

El endpoint **TransactionDetailListByWallet** ofrece una vista detallada de todas las transacciones vinculadas a una billetera específica. Incluye transacciones propias de la billetera (pay-ins, pay-outs y transfers) y las transacciones de tarjeta asociadas al **walletId** indicado.

=== "Request"
	```json
	{
	    "walletId": "string",
	    "startDate": "Start Date in ISO Format",
	    "toDate": "End Date in ISO Format",
	    "maxTransactions": "Maximum Number of Transactions",
	    "offset": "Number of Transactions to Skip",
	    "c43IdentificadorComercio": "Optional Merchant Identifier Filter"
	}
	```

=== "Response"
	```json
	{
	    "transactionListJson": "[{transaction1},{transaction2},...,{transactionN}]",
	    "totalItems": "Total number of transactions within the requested date range",
	    "currentPage": "Current page number indicator",
	    "totalPages": "Total number of pages available"
	}
	```

Cada transacción de tarjeta en el payload de respuesta será la versión serializada del webhook de notificación de transacciones, con **tres** campos adicionales:

```json
{
    "c1Tipo": "string",
    "c2CardId": "string",
    "c3CodigoProceso": "string",
    "c4ImporteTransaccion": "string",
    "c7FechaHoraTransaccion": "string",
    "c11NumeroIdentificativoTransaccion": "string",
    "c18CodigoActividadEstablecimiento": "string",
    "c19CodigoPaisAdquirente": "string",
    "c38NumeroAutorizacion": "string",
    "c41TerminalId": "string",
    "c42Comercio": "string",
    "c43IdentificadorComercio": "string",
    "c51MonedaImporteTransaccion": "string",
    "c6ImporteMonedaTransaccion": "string",
    "isSettled": "true"
}
```

- **c51MonedaImporteTransaccion:** Moneda utilizada para la transacción en el POS.
    
- **c6ImporteMonedaTransaccion:** Monto de la transacción en la moneda del POS.
    
- **isSettled:** Estado de liquidación de la transacción (`false` = pendiente/en tránsito, `true` = liquidada).
    

Como PayCaddy abstrae el proceso de liquidación, este campo resulta útil en casos concretos como reversos y anulaciones offline.

La lista recuperada incluirá únicamente los tipos **`c1Tipo`** que impactan el saldo de la billetera. Transacciones denegadas no se mostrarán.  
Las operaciones de Batch con **`c1Tipo = "transaccionCorregidaNegativa"`** o **`"transaccionCorregidaPositiva"`** sí se incluirán, siguiendo la misma lógica de los webhooks.

Cada transacción de billetera se presentará con su payload estándar de respuesta **200**.

### Explicación de Parámetros

#### Request

|**Campo**|**Descripción**|
|---|---|
|**walletId**|ID único de la billetera cuyo historial se quiere consultar.|
|**startDate & toDate**|Rango de fechas para la consulta. Para paginar todas las transacciones, usa la fecha actual en ambos campos.|
|**maxTransactions**|Número máximo de transacciones a devolver. Si existen más dentro del rango, se omitirán.|
|**offset**|Punto inicial (en orden cronológico inverso) para la paginación.|
|**c43IdentificadorComercio**|Filtro opcional por comercio. Ej.: `"UBER"` devuelve las transacciones cuyo identificador de comercio contenga “UBER”.|

#### Response

|**Campo**|**Descripción**|
|---|---|
|**transactionListJson**|Cadena JSON con las transacciones solicitadas en orden cronológico inverso.|
|**totalItems**|Total de transacciones incluidas en **transactionListJson**.|
|**currentPage**|Página actual basada en **totalItems** y el **offset**.|
|**totalPages**|Total de páginas calculado según **totalItems** y **maxTransactions**.|

### Paginación

Controla la paginación ajustando **maxTransactions** y **offset** en tu solicitud. Esto permite una exploración controlada del historial manteniendo un rendimiento óptimo de la interfaz.

Los parámetros **currentPage** y **totalPages** se incluyen en cada respuesta para tu conveniencia, calculados a partir de **totalItems** y **offset**.

Por ejemplo, con **totalItems = "25"** y **offset = "125"**, **currentPage** será **"6"**, mostrando las transacciones 126-150 del rango solicitado en orden cronológico inverso.

---