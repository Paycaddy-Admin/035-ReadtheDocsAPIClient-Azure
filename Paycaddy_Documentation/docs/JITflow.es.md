This document explains the Just In Time Funding (JIT) flow used by PayCaddy to authorize card transactions using an external authorization endpoint provided by the client, this helps developers and clients understand how a single transaction is processed from request to final webhook notification. 

Find in this section a high-level explanation of the decision flow for a transaction. The API schemas and format details can be found in [JIT Funding Transactions](JITtrx.es.md)

### Step-by-Step Flow

1. **Transaction Initiation**  
    A transaction is initiated on the card network (e.g., Mastercard), and PayCaddy receives a request for authorization. This is most relevant for JIT funding when the transaction is of type `PeticionAutorizacion` but the transaction initiation could be any of the Online Transaction Notification types.
    
    
    
2. **Forwarding to the Client's Endpoint**  
    PayCaddy creates a request with all relevant transaction data and sends a `POST` request to the client’s configured authorization endpoint (following the JIT format).
    
    
    
3. **Client Decision**  
    The client receives the request and decides whether to approve or deny the transaction. The client must respond within 1 second.
    
    - If the response is `{ "Funding": true }`, the transaction is approved.
        
    - If the response is `{ "Funding": false }`, the transaction is denied.
        
        
        
4. **Timeout Handling**  
    If the client does not respond within 1 second, PayCaddy automatically denies the transaction and records it as a timeout.
    
    
    
5. **Webhook Notification**  
    After processing the decision, PayCaddy sends a webhook to the client’s `NotificationEnlist` endpoint.
    
    - If approved: transaction proceeds, a confirmation webhook is sent with `c1Tipo: PeticionAutorizacion`.
        
    - If denied (either by client or due to timeout): a rejection webhook is sent with `c1Tipo`:
        
        - `RechazadaPorAutorizador` (rejected in time)
            
        - `ExcedeTiempoAutorizacion` (timeout)
            
            
            
6. **Batch Processing & Settlement**  
    Later, if applicable, a settlement batch may arrive with corrections:
    
    - `TransaccionCorregidaPositiva`: more funds are needed than initially authorized.
        
    - `TransaccionCorregidaNegativa`: the final amount was lower; excess is released.
        
    - `TransaccionConfirmada`: original authorization was matched and confirmed.



A visualization of this flow can be found below:

![jitflow](./assets/imgs/JITflow.svg){class="img"}


---

