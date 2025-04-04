# Transactions

## **Transaction Detail List <font color="green">POST</font>**

**URL:**  https://api.paycaddy.dev/v1/TransactionDetailList

Esta llamada POST recupera todas las transacciones de un cardId particular proporcionado.


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

La estructura de los datos compartidos en cada transacción sigue el mismo formato que todas las notificaciones de transacciones (consulte [Webhooks de transacciones](webhooks.es.md)).

---

## **TDL By Wallet <font color="green">POST</font>**

**URL:**  https://api.paycaddy.dev/v1/TransactionDetailListByWallet

El punto de conexión **TransactionDetailListByWallet** ofrece una vista detallada de todas las transacciones vinculadas a una billetera específica. Esto incluye transacciones específicas de la billetera, como ingresos, pagos y transferencias, así como transacciones con tarjeta vinculadas a la ID de billetera especificada.

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

Cada transacción de tarjeta en la carga de respuesta será una versión en cadena de los webhooks de notificación de transacciones, aumentada con tres campos adicionales:

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

- **c51MonedaImporteTransaccion:** Representa la moneda utilizada para la transacción en el POS.
- **c6ImporteMonedaTransaccion:** Indica el monto de la transacción en la moneda utilizada en el POS.
- **isSettled:** Indica el estado de liquidación de una transacción. Un valor de "false" significa transacciones que no se han liquidado o que están en tránsito, mientras que "true" indica transacciones que se han liquidado.

Dado que PayCaddy abstrae el proceso de liquidación para sus clientes, este campo es particularmente pertinente para casos de uso específicos como reversiones y cancelaciones fuera de línea.

La lista de transacciones recuperada incluirá todas las transacciones con tipos de transacción "c1tipo" que afecten el saldo de la billetera. No se incluirán "c1tipo" adicionales, como las transacciones rechazadas.

Las transacciones del proceso por lotes con valores "c1tipo" de "transaccionCorregidaNegativa" y "transaccionCorregidaPositiva" se incorporarán a la lista recuperada, alineándose con la lógica esperada en los webhooks.

Cada transacción de billetera se presentará en su carga útil de respuesta estándar 200:

**Explicación de parámetros:**

| **Campo** | **Descripción** |
| ------------- | ------------- |
| **walletId** | El ID único de la billetera, esencial para obtener las transacciones asociadas. |
| **startDate & toDate** | Establece el rango de fechas para la recuperación de transacciones. Para obtener y paginar todas las transacciones, utiliza la fecha actual para ambos campos. |
| **maxTransactions** | Determina la cantidad máxima de transacciones que se devolverán. Si existen más transacciones dentro del rango de fechas, se excluirán. |
| **offset** | Establece el punto de inicio de las transacciones, que se obtienen en orden cronológico inverso, lo que facilita la paginación. |
| **c43IdentificadorComercio** | Un filtro opcional basado en un comerciante específico. Por ejemplo, "UBER" obtiene todas las transacciones con "UBER" en su identificador de comerciante. |

**Response**

| **Campo** | **Descripción** |
| ------------- | ------------- |
| **transactionListJson** | Contiene un JSON en formato de cadena de todas las transacciones solicitadas en orden cronológico inverso. |
| **totalItems** | Especifica el total de transacciones en **transactionListJson**. |
| **currentPage:** | Indica la página actual en función de **totalItems** y el **offset** de la solicitud. |
| **totalPages:** | Muestra el total de páginas en función de **totalItems** y el **maxTransactions** de la solicitud. |

### **Pagination**

Para administrar la paginación, ajuste los parámetros **maxTransactions** y **offset** en su solicitud. Esto garantiza una exploración controlada de las transacciones históricas, manteniendo un rendimiento rápido de la experiencia de usuario y la interfaz de usuario.

Los parámetros **currentPage** y **totalPages** se incluyen en cada respuesta para mayor comodidad, ambos calculados en función de los parámetros **totalItems** y offset.

Por ejemplo, con un **totalItems** de "25" y un **offset** de "125", el **currentPage** sería "6", mostrando las transacciones numeradas del 126 al 150 del rango de fechas en orden cronológico inverso.

---

## **Chargeback Operation <font color="green">POST</font>**

**URL:**  https://api.paycaddy.dev/v1/ChargeBackOperation

Este endpoint permite marcar una transacción específica asociada a una tarjeta para que sea estudiada por un equipo de especialistas en contracargos. Al solicitar el análisis de un contracargo, el equipo evaluará si la transacción puede revertirse y los fondos pueden ser devueltos al titular de la tarjeta.

=== "Request"
    ```json
    {
        "numeroidentificativoTransaction": "string",
        "cardId": "string"
    }
    
    ```
=== "Response"
    ```json
    {
        "TransactionId": "3fa85f64-xxxx-xxxx-xxx-2c963f66afa6",
        "IsChargeBack": true,
        "ChargeBackStatus": "string"
    }
    ```

Una vez marcada la transacción, el estado y las actualizaciones del contracargo se notificarán a los clientes a través del webhook de contracargo. (Ver [Webhooks](webhooks.es.md))   