
Todos los cambios notables en la documentación de nuestra API, como lanzamientos, deprecaciones y actualizaciones, se registrarán en este archivo.

## 1.0.5 - 2026-10-06

#### Corregido
- `V2/UserUpdateData`: el capítulo [Edición y Actualización de Usuarios](./editUser.es.md) indicaba que `firstName` y `lastName` podían editarse con fines de compliance. El endpoint no acepta ninguno de los dos campos — se devuelven en la respuesta solo como referencia. Los ejemplos de request y response y las tablas de campos editables / no editables se alinearon con la especificación OpenAPI publicada (sandbox y producción), lo que también retira `alias` y `nationality` de los campos editables.
- `V2/MerchantUserUpdateData`: `registeredName` y `nationality` figuraban como editables, pero el endpoint no los acepta — `registeredName` se devuelve en la respuesta solo como referencia. Los ejemplos de request y response (incluidos los campos de documentos `certificateOfGoodStanding`, `idShareholders` y `addressVerificationShareholders`) y las tablas de campos se alinearon con la misma especificación, y el ejemplo de response ya no muestra un campo `merchantUserId` que el endpoint no devuelve.

#### Añadido
- Versión en español del capítulo [Edición y Actualización de Usuarios](./editUser.es.md).

## 1.0.4 - 2026-07-20

#### Corregido
- Se corrigió una dirección crédito/débito invertida de `TransaccionCorregidaPositiva` y `TransaccionCorregidaNegativa` en los capítulos de transacciones JIT y prefondeadas. **Positiva** acredita la billetera (liquidación final menor a la autorizada); **Negativa** la debita (final mayor). El texto anterior indicaba lo contrario.

## 1.0.3 - 2026-06-23

#### Añadido
- Actualización del esquema de webhooks: los payloads de webhooks transaccionales ahora incluyen los nuevos campos `c22DatosPuntoServicio`, `c22Descripcion`, `isExpiration`, `isMulticlearing` y `multiclearingClose`. Consulta [23 de junio de 2026 - Actualización del Esquema de Webhooks](./webhookFieldUpdates.es.md) para conocer todos los detalles.

## 1.0.2 - 2025-01-29

#### Añadido
- Se implementó el changelog para mejorar el control de lanzamientos, deprecaciones y actualizaciones sobre la documentación de nuestra API.
#### Eliminado
* 
#### Obsoleto
* 
