!!!Important
    **Webhook Schema Update — June 23, 2026:** New fields have been added to authorization and settlement webhook payloads. Existing fields remain unchanged. Review this page for full details on each new field.

## New Rejection Types

Two new values have been added for the `c1Tipo` field in rejection notifications. Like existing rejection types, these do not affect balance and are purely informative of the card usage.

|**c1Tipo Value**|**Description**|
|---|---|
|`DENEGACION POR COMERCIO BLOQUEADO POR CLIENTE`|The transaction was declined because the merchant has been blocked by the Card Program Manager.|
|`DENEGACION POR COMERCIO BLOQUEADO (GENERAL)`|The transaction was declined because the merchant is blocked under a general blocking rule.|

> These rejection types follow the same webhook schema as existing rejection notifications described in [Other Transactions](./otherTransactions.en.md).

---

## Point-of-Service Data

All transactional webhooks now include point-of-service terminal data through two new fields. These fields provide detailed information about the environment and method in which a card transaction was captured.

### c22DatosPuntoServicio

A raw string containing the encoded point-of-service data as received from the network.

**Example:** `"000580JA0001"`

### c22Descripcion

An object that provides a human-readable breakdown of the `c22DatosPuntoServicio` value. Each subfield maps a specific aspect of the point-of-service environment:

|**Subfield**|**Description**|**Possible Values**|
|---|---|---|
|`CapturaDatosTarjeta`|Card data capture method|`Sin especificar`, `Datos manuales`, `Banda magnética`, `Código de barras`, `OCR`, `Tarjeta Chip`, `Entrada Clave`, `Banda magnética ICC`, `Banda magnética clave`, `Banda ICC Clave`, `Capt. Terminal EMV`, `Capt. Terminal S-set`, `Capt. Terminal S-no-set`, `Capt. Terminal SSL`, `Capt. Terminal Virtual`, `Chip Sin Contacto`, `Lectura Banda sin contacto`|
|`AutenticacionCliente`|Client authentication capability|`Sin capacidad de lectura`, `Autentica Pin`, `Firma electrónica`, `No operativa`, `Otros idclien`|
|`RetencionTarjeta`|Card retention capability|`Sin capacidad de Captura`, `Con capacidad de captura`|
|`TipoTerminal`|Terminal type|`No Terminal`, `Terminal atendido comercio`, `Terminal no atendido comercio`, `Terminal atendido fuera`, `Terminal no atendido fuera`, `Terminal no atendido casa`, `Terminal móvil mpos`, `Dispositivo Tarjeta`, `Mobile Net Operation`, `Dispositivo Token`, `Dispositivo Reloj`, `Dispositivo Telepeaje`, `Dispositivo Muñequera`, `Dispositivo Base Lectora`, `Móvil con SIM controlada`, `Móvil con SIM fija`, `Memoria extraíble móvil`, `Tablet con SIM`, `Tablet con SIM fija`, `Memoria extraíble tablet`, `Elemento fijo no controlado`|
|`PresenciaCliente`|Client presence indicator|`Cliente presente`, `Cliente no presente`, `Cliente correo`, `Cliente teléfono`, `Cliente no presente Aut. Per.`, `Cliente electrónico seguro`, `Cliente electrónico no seguro`|
|`PresenciaTarjeta`|Card presence indicator|`Tarjeta no presente`, `Tarjeta presente`|
|`MetodoCapturaDatos`|Data capture method used|`Sin especificar`, `Manual sin terminal`, `Aut. banda magnética`, `Código de barras`, `OCR`, `Tarjeta Chip`, `Pista 1`, `Ing Err. Chip Manual`, `Ing. Err Chip Banda`, `Internet`, `Banda Magnética y Pistas`, `Tarjeta Chip No CVV`, `Clave`, `Voz`, `eBanking`|
|`MetodoAutenticacionCliente`|Client authentication method detail|`Cliente no autenticado`, `Cliente autentica PIN`, `Cliente firma electrónica`, `Verificación manual firma`, `Verificación documento`, `Certificado SET`, `Autentica Cliente OT Certificado`, `Autentica Cliente PIN Offline`, `Datos 3D presentes`, `Datos 3D no presentes`|
|`EntidadAutenticadora`|Authenticating entity|`Dispositivo no autentica cliente`, `Autentica Tarjeta Chip`, `Autentica Terminal`, `Autentica Emisor`, `Autentica Establecimiento`, `Autentican otros`|
|`ActualizacionTarjeta`|Card update capability|`Capacidad de actualización desconocida`, `Sin capacidad de actualización`, `Actualización de la banda`, `Actualización de la Tarjeta Chip`|
|`ImpresionOMensaje`|Print or message capability|`Capacidad de impresión desconocida`, `Sin capacidad de impresión`, `Con capacidad de impresión`, `Con capacidad de mostrar displays`, `Con impresión y displays`|
|`LongitudMaximaPIN`|Maximum PIN length supported|`No trata PIN`, `Sin longitud máxima`, `Trata cuatro`, `Trata cinco`, `Trata seis`, `Trata siete`, `Trata ocho`, `Trata nueve`, `Trata diez`, `Trata once`, `Trata doce`|

**Example payload fragment:**

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

> The values within `c22Descripcion` are descriptive strings in Spanish provided by the network. They describe the terminal capabilities and transaction context at the point of sale.

> These fields appear on all transactional webhooks across [Prefunded Transactions](./prefundedTRX.en.md), [Just-In-Time Funding](./JITtrx.en.md), [Other Transactions](./otherTransactions.en.md), and [Settlement](./settlement.en.md) flows.

---

## Settlement Lifecycle Fields

All transactional webhooks now include three boolean fields that provide visibility into the settlement lifecycle of a transaction. On non-settlement webhooks (`PeticionAutorizacion`, `ComunicacionAutorizacion`, `ComunicacionAnulacion`, `PeticionDevolucion`, rejection notifications, etc.) all three are always `false`. On settlement webhooks (`TransaccionCorregidaPositiva`, `TransaccionCorregidaNegativa`, `TransaccionConfirmada`) they reflect the actual lifecycle state of the transaction.

|**Field**|**Type**|**Description**|
|---|---|---|
|`isExpiration`|bool|`true` if the settlement was triggered by automatic expiration. This occurs when 30 days pass without the transaction receiving any settlement messages from the network.|
|`isMulticlearing`|bool|`true` if the transaction is a Multiclearing type, meaning it can receive more than one settlement message before being finalized.|
|`multiclearingClose`|bool|`true` if this settlement message closes the Multiclearing cycle. After receiving a message where this field is `true`, no further settlement messages should be expected for the same authorization.|

### Expiration

When an authorized transaction does not receive any settlement confirmation from the network within 30 days, the system automatically generates a settlement notification with `isExpiration: true`. This mechanism ensures that all authorized transactions eventually reach a terminal state, even when the network does not send a settlement file.

### Multiclearing

Certain transactions — such as those involving travel, subscription services, or split shipments — may be settled across multiple clearing messages. The `isMulticlearing` field identifies these transactions, and the `multiclearingClose` field signals when the final settlement message has been received.

**Example payload fragment:**

```json
{
    "isExpiration": false,
    "isMulticlearing": true,
    "multiclearingClose": false
}
```

> When `isMulticlearing` is `true` and `multiclearingClose` is `false`, expect additional settlement webhooks for the same authorization.

> For full settlement schemas and context, see [Settlement](./settlement.en.md).

---
