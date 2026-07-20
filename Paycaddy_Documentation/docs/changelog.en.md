
All notable changes in our API documentation, such as releases, deprecations and updates will be documented in this file.

## 1.0.4 - 2026-07-20

#### Fixed
- Corrected an inverted credit/debit direction for `TransaccionCorregidaPositiva` and `TransaccionCorregidaNegativa` in the JIT and prefunded transaction chapters. **Positiva** credits the wallet (final settlement lower than authorized); **Negativa** debits it (final higher). The previous text stated the opposite.

## 1.0.3 - 2026-06-23

#### Added
- Webhook schema update: new fields `c22DatosPuntoServicio`, `c22Descripcion`, `isExpiration`, `isMulticlearing`, and `multiclearingClose` are now included in transactional webhook payloads. See [June 23, 2026 - Webhook Schema Update](./webhookFieldUpdates.en.md) for full details.

## 1.0.2 - 2025-01-29

#### Added
- Implemented changelog to improve control of releases, deprecations and updates over our API documentation
#### Removed
* 
#### Deprecated
* 
