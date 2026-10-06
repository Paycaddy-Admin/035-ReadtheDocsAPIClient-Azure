
All notable changes in our API documentation, such as releases, deprecations and updates will be documented in this file.

## 1.0.5 - 2026-10-06

#### Fixed
- `V2/UserUpdateData`: the [Edit or Update User Information](./editUser.en.md) chapter stated that `firstName` and `lastName` could be edited for compliance purposes. The endpoint does not accept either field — they are returned in the response for reference only. The request and response examples and the editable / non-editable field tables were aligned with the published OpenAPI specification (sandbox and production), which also removes `alias` and `nationality` from the editable fields.
- `V2/MerchantUserUpdateData`: `registeredName` and `nationality` were documented as editable, but the endpoint does not accept them — `registeredName` is returned in the response for reference only. The request and response examples (including the `certificateOfGoodStanding`, `idShareholders` and `addressVerificationShareholders` document fields) and the field tables were aligned with the same specification, and the response example no longer shows a `merchantUserId` field the endpoint does not return.

#### Added
- Spanish version of the [Edit or Update User Information](./editUser.en.md) chapter.

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
