
Todos los cambios notables en la documentación de nuestra API, como lanzamientos, deprecaciones y actualizaciones, se registrarán en este archivo.

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
