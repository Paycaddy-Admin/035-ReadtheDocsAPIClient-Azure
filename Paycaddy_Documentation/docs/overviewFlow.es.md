Existen varios flujos funcionales posibles en nuestra API. Pueden clasificarse según la entidad afectada o el resultado funcional. Prácticamente todos los Programas de Tarjeta requieren una combinación de los flujos que se describen a continuación:

[**Flujo de Creación de Usuarios**](userFlow.es.md)  
Habilita el registro y la verificación KYC de nuevos usuarios asociados a un Programa de Tarjeta en particular. Los usuarios pueden ser Personas Naturales o Empresas y la verificación KYC puede ser _Integrado_ o _Delegado_.

> _KYC Integrado_ utiliza la solución de PayCaddy para la captura de datos KYC y la activación del usuario depende de la verificación programática a través de un proveedor externo de verificación de identidad.  
> _KYC Delegado_ permite que las entidades financieras reguladas lideren la captura de datos KYC y registren usuarios que se crean activos por defecto.

[**Flujo de Creación de Tarjetas**](cardFlow.es.md)  
Permite la creación de tarjetas Prepaid, Debit o Credit asociadas a un usuario, wallet y especificación de producto de tarjeta determinados. Este flujo también abarca la activación inicial de tarjetas físicas.

[**Flujo de Transacciones Prefunded**](prefundedFlow.es.md)  
Incluye el fondeo de `wallet` mediante [wallet operations](wallet_ops.es.md), la aprobación de transacciones _open-loop_, el registro de una Callback URL y las notificaciones enviadas vía webhook a dicha dirección.

[**Flujo de Transacciones Just-In-Time Funding**](JITflow.es.md)  
Habilita la autorización externa de transacciones con tarjeta. Describe los requisitos y especificaciones para el _webservice_ de autorización que debe estar disponible, así como los esquemas de solicitud de autorización y las notificaciones de eventos de transacción resultantes.

---

A alto nivel, el flujo desde la creación de un usuario hasta la recepción de notificaciones de transacción puede verse en el siguiente diagrama (lógica Prefunded):

![generalFlow](./assets/imgs/generalFlow.svg){class="img"}

> Para programas de Just-In-Time Funding, no se utilizarán PayIns y el flujo de aprobación de transacciones seguirá una ruta distinta. Para más detalles consulta el [**JIT Flow**](JITflow.es.md).


---
