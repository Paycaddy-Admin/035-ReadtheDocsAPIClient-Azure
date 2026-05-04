!!!Important
    **Actualización de Esquema de Webhooks — 23 de junio de 2026:** Se han añadido nuevos campos a los payloads de webhooks de autorización y liquidación. Los campos existentes permanecen sin cambios. Consulta esta página para conocer los detalles de cada nuevo campo.

## Nuevos Tipos de Rechazo

Se han añadido dos nuevos valores para el campo `c1Tipo` en las notificaciones de rechazo. Al igual que los tipos de rechazo existentes, estos no afectan el saldo y son meramente informativos del uso de la tarjeta.

|**Valor de c1Tipo**|**Descripción**|
|---|---|
|`DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE`|La transacción fue rechazada porque el comercio ha sido bloqueado por el tarjetahabiente.|
|`DENEGACION POR COMERCIO BLOQUEADO (GENERAL)`|La transacción fue rechazada porque el comercio está bloqueado bajo una regla de bloqueo general.|

> Estos tipos de rechazo siguen el mismo esquema de webhook que las notificaciones de rechazo existentes descritas en [Otras Transacciones](./otherTransactions.es.md).

---

## Datos del Punto de Servicio

Todos los webhooks transaccionales ahora incluyen datos del terminal del punto de servicio a través de dos nuevos campos. Estos campos proporcionan información detallada sobre el entorno y método en que se capturó la transacción con tarjeta.

### c22DatosPuntoServicio

Una cadena de texto que contiene los datos codificados del punto de servicio tal como se reciben de la red.

**Ejemplo:** `"000580JA0001"`

### c22Descripcion

Un objeto que proporciona un desglose legible del valor de `c22DatosPuntoServicio`. Cada subcampo describe un aspecto específico del entorno del punto de servicio:

|**Subcampo**|**Descripción**|**Valores Posibles**|
|---|---|---|
|`CapturaDatosTarjeta`|Método de captura de datos de tarjeta|`Sin especificar`, `Datos manuales`, `Banda magnética`, `Código de barras`, `OCR`, `Tarjeta Chip`, `Entrada Clave`, `Banda magnética ICC`, `Banda magnética clave`, `Banda ICC Clave`, `Capt. Terminal EMV`, `Capt. Terminal S-set`, `Capt. Terminal S-no-set`, `Capt. Terminal SSL`, `Capt. Terminal Virtual`, `Chip Sin Contacto`, `Lectura Banda sin contacto`|
|`AutenticacionCliente`|Capacidad de autenticación del cliente|`Sin capacidad de lectura`, `Autentica Pin`, `Firma electrónica`, `No operativa`, `Otros idclien`|
|`RetencionTarjeta`|Capacidad de retención de tarjeta|`Sin capacidad de Captura`, `Con capacidad de captura`|
|`TipoTerminal`|Tipo de terminal|`No Terminal`, `Terminal atendido comercio`, `Terminal no atendido comercio`, `Terminal atendido fuera`, `Terminal no atendido fuera`, `Terminal no atendido casa`, `Terminal móvil mpos`, `Dispositivo Tarjeta`, `Mobile Net Operation`, `Dispositivo Token`, `Dispositivo Reloj`, `Dispositivo Telepeaje`, `Dispositivo Muñequera`, `Dispositivo Base Lectora`, `Móvil con SIM controlada`, `Móvil con SIM fija`, `Memoria extraíble móvil`, `Tablet con SIM`, `Tablet con SIM fija`, `Memoria extraíble tablet`, `Elemento fijo no controlado`|
|`PresenciaCliente`|Indicador de presencia del cliente|`Cliente presente`, `Cliente no presente`, `Cliente correo`, `Cliente teléfono`, `Cliente no presente Aut. Per.`, `Cliente electrónico seguro`, `Cliente electrónico no seguro`|
|`PresenciaTarjeta`|Indicador de presencia de la tarjeta|`Tarjeta no presente`, `Tarjeta presente`|
|`MetodoCapturaDatos`|Método de captura de datos utilizado|`Sin especificar`, `Manual sin terminal`, `Aut. banda magnética`, `Código de barras`, `OCR`, `Tarjeta Chip`, `Pista 1`, `Ing Err. Chip Manual`, `Ing. Err Chip Banda`, `Internet`, `Banda Magnética y Pistas`, `Tarjeta Chip No CVV`, `Clave`, `Voz`, `eBanking`|
|`MetodoAutenticacionCliente`|Detalle del método de autenticación del cliente|`Cliente no autenticado`, `Cliente autentica PIN`, `Cliente firma electrónica`, `Verificación manual firma`, `Verificación documento`, `Certificado SET`, `Autentica Cliente OT Certificado`, `Autentica Cliente PIN Offline`, `Datos 3D presentes`, `Datos 3D no presentes`|
|`EntidadAutenticadora`|Entidad autenticadora|`Dispositivo no autentica cliente`, `Autentica Tarjeta Chip`, `Autentica Terminal`, `Autentica Emisor`, `Autentica Establecimiento`, `Autentican otros`|
|`ActualizacionTarjeta`|Capacidad de actualización de tarjeta|`Capacidad de actualización desconocida`, `Sin capacidad de actualización`, `Actualización de la banda`, `Actualización de la Tarjeta Chip`|
|`ImpresionOMensaje`|Capacidad de impresión o mensaje|`Capacidad de impresión desconocida`, `Sin capacidad de impresión`, `Con capacidad de impresión`, `Con capacidad de mostrar displays`, `Con impresión y displays`|
|`LongitudMaximaPIN`|Longitud máxima de PIN soportada|`No trata PIN`, `Sin longitud máxima`, `Trata cuatro`, `Trata cinco`, `Trata seis`, `Trata siete`, `Trata ocho`, `Trata nueve`, `Trata diez`, `Trata once`, `Trata doce`|

**Ejemplo de fragmento del payload:**

```json
{
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
    }
}
```

> Los valores dentro de `c22Descripcion` son cadenas descriptivas en español proporcionadas por la red. Describen las capacidades del terminal y el contexto de la transacción en el punto de venta.

> Estos campos aparecen en todos los webhooks transaccionales a través de los flujos de [Transacciones Prefundidas](./prefundedTRX.es.md), [Fondeo Just-In-Time](./JITtrx.es.md), [Otras Transacciones](./otherTransactions.es.md) y [Liquidación](./settlement.es.md).

---

## Campos del Ciclo de Vida de Liquidación

Todos los webhooks transaccionales ahora incluyen tres campos booleanos que proporcionan visibilidad sobre el ciclo de vida de liquidación de una transacción. En los webhooks que no son de liquidación (`PeticionAutorizacion`, `ComunicacionAutorizacion`, `ComunicacionAnulacion`, `PeticionDevolucion`, notificaciones de rechazo, etc.) los tres siempre son `false`. En los webhooks de liquidación (`TransaccionCorregidaPositiva`, `TransaccionCorregidaNegativa`, `TransaccionConfirmada`) reflejan el estado real del ciclo de vida de la transacción.

|**Campo**|**Tipo**|**Descripción**|
|---|---|---|
|`isExpiration`|bool|`true` si la liquidación fue generada por expiración automática. Esto ocurre cuando transcurren 30 días sin que la transacción reciba mensajes de liquidación de la red.|
|`isMulticlearing`|bool|`true` si la transacción es de tipo Multiclearing, lo que significa que puede recibir más de un mensaje de liquidación antes de ser finalizada.|
|`multiclearingClose`|bool|`true` si este mensaje de liquidación cierra el ciclo de Multiclearing. Después de recibir un mensaje donde este campo es `true`, no se deben esperar más mensajes de liquidación para la misma autorización.|

### Expiración

Cuando una transacción autorizada no recibe confirmación de liquidación de la red dentro de 30 días, el sistema genera automáticamente una notificación de liquidación con `isExpiration: true`. Este mecanismo asegura que todas las transacciones autorizadas alcancen un estado terminal, incluso cuando la red no envía un archivo de liquidación.

### Multiclearing

Ciertas transacciones — como las relacionadas con viajes, servicios de suscripción o envíos fraccionados — pueden liquidarse a través de múltiples mensajes de compensación. El campo `isMulticlearing` identifica estas transacciones, y el campo `multiclearingClose` señala cuándo se ha recibido el mensaje de liquidación final.

**Ejemplo de fragmento del payload:**

```json
{
    "isExpiration": false,
    "isMulticlearing": true,
    "multiclearingClose": false
}
```

> Cuando `isMulticlearing` es `true` y `multiclearingClose` es `false`, se deben esperar webhooks de liquidación adicionales para la misma autorización.

> Para los esquemas completos de liquidación y contexto, consulta [Liquidación](./settlement.es.md).

---
