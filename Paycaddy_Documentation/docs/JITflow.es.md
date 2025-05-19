Este documento explica el flujo **Just-In-Time Funding (JIT)** que utiliza PayCaddy para autorizar transacciones con tarjeta mediante un endpoint de autorización externo proporcionado por el cliente. Ayuda a desarrolladores y clientes a entender cómo se procesa una transacción desde la solicitud inicial hasta la notificación final vía webhook.

En esta sección encontrarás una descripción de alto nivel del flujo de decisión para una transacción. Los esquemas de la API y los detalles de formato están disponibles en [JIT Funding Transactions](JITtrx.es.md).

### Flujo paso a paso

1. **Inicio de la transacción**  
    Se inicia una transacción en la red (por ejemplo, Mastercard) y PayCaddy recibe una solicitud de autorización. Esto es relevante para JIT funding cuando la transacción es de tipo `PeticionAutorizacion`, aunque el inicio podría ser cualquiera de los tipos de notificación en línea.
    
2. **Reenvío al endpoint del cliente**  
    PayCaddy crea una solicitud con los datos de la transacción y envía un `POST` al endpoint de autorización configurado por el cliente (siguiendo el formato JIT).
    
3. **Decisión del cliente**  
    El cliente recibe la solicitud y decide aprobar o denegar la transacción. Debe responder en máximo **1 segundo**.
    
    - Si la respuesta es `{ "Funding": true }`, la transacción se aprueba.
        
    - Si la respuesta es `{ "Funding": false }`, la transacción se rechaza.
        
4. **Manejo de tiempo de espera**  
    Si el cliente no responde dentro de 1 segundo, PayCaddy rechaza la transacción de forma automática y la registra como _timeout_.
    
5. **Notificación webhook**  
    Tras procesar la decisión, PayCaddy envía un webhook al endpoint registrado mediante `NotificationEnlist`.
    
    - Si se aprueba: la transacción continúa y se envía un webhook de confirmación con `c1Tipo: PeticionAutorizacion`.
        
    - Si se rechaza (por el cliente o por timeout): se envía un webhook de rechazo con `c1Tipo`:
        
        - `RechazadaPorAutorizador` (rechazo en tiempo)
            
        - `ExcedeTiempoAutorizacion` (timeout)
            
6. **Procesamiento por lotes y liquidación**  
    Más adelante, si corresponde, puede llegar un lote de liquidación con correcciones:
    
    - `TransaccionCorregidaPositiva`: se requiere más dinero que el inicialmente autorizado.
        
    - `TransaccionCorregidaNegativa`: el monto final fue menor; se libera el excedente.
        
    - `TransaccionConfirmada`: la autorización original se igualó y confirmó.
        

Una representación visual de este flujo se muestra a continuación:

![jitflow](./assets/imgs/JITflow.svg){class="img"}


---

