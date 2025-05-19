El flujo **Prefunded** en PayCaddy te permite mantener y administrar fondos dentro de las **wallets** de los usuarios antes de que ocurran las transacciones. Cada **wallet** puede estar asociada a una o más tarjetas y funciona como contenedor de dinero electrónico que respalda los pagos. Este capítulo ofrece una visión general de cómo opera este flujo, cómo se autorizan y liquidan las transacciones, y cómo se depositan (y retiran) fondos en las **wallets**.

## Descripción general

1. **Wallets de usuario**  
    Cada usuario puede tener una o más **wallets** con saldo electrónico. Estas wallets son la fuente de fondos para las tarjetas de pago.
    
2. **Asociación con tarjetas**  
    Una **wallet** puede vincularse a una o más tarjetas. Cuando se produce una transacción _open-loop_, el motor de aprobación de PayCaddy revisa la wallet asociada para confirmar que exista saldo suficiente.
    
3. **Autorización de transacciones y notificaciones**
    
    - **Chequeo de autorización**: En una compra, el sistema verifica el saldo de la wallet. Si hay fondos suficientes, la transacción se autoriza y el monto se retiene o captura según corresponda.
        
    - **Notificación de transacción**: Una vez autorizada la transacción, se envía una notificación webhook a la callback URL registrada mediante el endpoint _Notification Enlist POST_.
        
4. **Liquidación y ajustes**
    
    - **Notificación de liquidación**: La liquidación se procesa de forma asíncrona en lotes. Al completarse, PayCaddy envía una notificación de liquidación a la misma callback URL.
        
    - **Ajustes**: El monto liquidado final puede diferir del autorizado inicialmente. Cualquier cambio (por ejemplo, propinas o variaciones de moneda) llega incluido en la notificación de liquidación.
        

![prefundedflow](./assets/imgs/prefundedflow.svg)

---

## Fondeo de una Wallet

Una wallet debe tener fondos suficientes para cubrir las transacciones. Existen dos formas principales de agregar fondos:

1. **PayIns directos**  
    Usa el endpoint _PayIn_ para añadir fondos directamente a una wallet activa. Los fondos quedan disponibles de inmediato para gastar. PayCaddy liquidará estos PayIns offline mediante un proceso diario de ACH o transferencia bancaria.  
    ![payindirect](./assets/imgs/payindirect.svg)
    
2. **Transferencia desde una Company Wallet**  
    Método recomendado para quienes prefieren centralizar los PayIns en una única **Company Wallet** y luego distribuir fondos a las wallets individuales. Esto simplifica la conciliación y mejora la trazabilidad del dinero.  
    ![payincentral](./assets/imgs/payincentral.svg)
    

Los detalles sobre cómo usar el método PayIn se encuentran en el capítulo [**Wallet Operations**](wallet_ops.es.md).

---

### Liquidación de PayIns

**PayIns directos**  
PayCaddy consolida los PayIns ejecutados y envía un reporte detallado para conciliación. Luego liquida vía ACH o transferencia bancaria, asegurando que los PayIns puestos a disposición del usuario queden correctamente liquidados.

**Transferencia desde una Company Wallet**  
Con la presentación del comprobante de la transferencia ACH o wire, puede solicitarse la aprobación para ejecutar un PayIn centralizado, garantizando que los fondos de cada wallet queden liquidados por adelantado.

> Dependiendo de la operación del Programa de Tarjeta, los parámetros de liquidación de PayIns se definirán con el equipo de Treasury Operations de PayCaddy.

## Eliminación de fondos (PayOut)

Para retirar fondos de una wallet, usa el método _PayOut_. Una transacción PayOut revierte el flujo, permitiendo extraer dinero y enviarlo a una cuenta bancaria externa u otro destino designado. El total de PayOuts ejecutados en un día se descuenta del proceso offline de liquidación de PayIns.