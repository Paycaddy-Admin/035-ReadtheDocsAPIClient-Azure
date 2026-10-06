# **Endpoints de Actualización de Datos de Usuario**

!!!info
    **Propósito:**
    Estos endpoints permiten a los clientes de PayCaddy actualizar los datos de usuarios existentes, tanto de **personas naturales** (`EndUser`, `EndUserSR`) como de **personas jurídicas** (`MerchantUser`, `MerchantUserSR`).
    Se utilizan principalmente para gestionar **información relacionada con compliance**, **actualizaciones de contacto** o **correcciones menores** sobre registros existentes.
    Ciertos campos —como identificadores, userId y relaciones con wallets— **no pueden** modificarse mediante esta llamada.


## **V2/UserUpdateData <font color="green">POST</font>**

**URL de la solicitud:**
`https://api.api-sandbox.paycaddy.dev/v2/UserUpdateData`

Este endpoint actualiza la información de usuarios **EndUser** y **EndUserSR**.
El body acepta únicamente los campos editables listados más abajo; los identificadores, los datos de wallet, las marcas de tiempo de creación y el nombre legal del usuario no pueden modificarse mediante esta llamada.

!!!warning
    **El nombre legal no es editable:**
    `V2/UserUpdateData` **no** acepta `firstName` ni `lastName`. Ambos campos se devuelven en la respuesta solo como referencia. Si se requiere una corrección del nombre legal, contacta al equipo de soporte de PayCaddy.

=== "Request"
	```json
		{
		  "userId": "string",
		  "email": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": false,
		  "salary": 200000,
		  "telephone": "+50760001234",
		  "address": {
		    "addressLine1": "123 Main Street",
		    "addressLine2": "Tower B, Apt 12B",
		    "homeNumber": "12B",
		    "city": "Panama",
		    "region": "Panama Metropolitan Area",
		    "postalCode": "000000",
		    "country": "PA"
		  },
		  "countryOfOperations": "PA, US",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string"
		}
	```


=== "Response"
	```json
		{
		  "firstName": "string",
		  "lastName": "string",
		  "email": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": false,
		  "salary": 200000,
		  "telephone": "+50760001234",
		  "address": {
		    "addressLine1": "123 Main Street",
		    "addressLine2": "Tower B, Apt 12B",
		    "homeNumber": "12B",
		    "city": "Panama",
		    "region": "Panama Metropolitan Area",
		    "postalCode": "000000",
		    "country": "PA"
		  },
		  "countryOfOperations": "PA, US",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string"
		}
	```

> **Nota:**
>
> - La solicitud de actualización solo modifica los campos enviados.
>
> - Los campos omitidos permanecen sin cambios.
>
> - El sistema valida cada campo actualizado según las mismas reglas de formato y tipo descritas en el [capítulo de Creación de Usuarios](userv2.es.md).
>

---

### Campos Editables

|Campo|Descripción|Editable|Notas|
|---|---|---|---|
|`email`|Email de contacto|✅|Debe cumplir el formato RFC-5322|
|`occupation`, `placeOfWork`|Campos laborales|✅|Pueden requerirse para un KYC actualizado|
|`pep`|Indicador de persona expuesta políticamente|✅|Booleano|
|`salary`|Salario actualizado en centavos|✅|Entero, centavos de USD|
|`telephone`|Formato E.164|✅|Ejemplo: +50760001234|
|`address.*`|Todos los subcampos|✅|Ver el esquema de creación|
|`countryOfOperations`|ISO alpha-2, separados por coma|✅||

---

### Campos No Editables

|Campo|Motivo|
|---|---|
|`userId`|Clave primaria inmutable|
|`firstName`, `lastName`|Nombre legal; no aceptado por este endpoint (se devuelve en la respuesta como solo lectura)|
|`alias`, `nationality`|No aceptados por este endpoint|
|`walletId`|Generado por el sistema|
|`kycUrl`|Vinculado al proceso de KYC|
|`isActive`|Controlado por la lógica de compliance|
|`creationDate`|Inmutable|

---

### Validación y Manejo de Errores

Los errores siguen el mismo formato que en la creación de usuarios. Consulta “Requisitos de los Campos” en el capítulo de creación para el detalle de las validaciones.

|Código HTTP|Tipo|Descripción|
|---|---|---|
|`200`|OK|Actualización aceptada|
|`400`|ValidationError|Campo faltante o inválido|
|`422`|Business Rule Violation|Intento de modificar un campo restringido|
|`500`|Internal Error|Error inesperado del lado del servidor|

=== "Sample Error"

```json
{
  "type": "https://docs.paycaddy.com/errors/PC-422-READONLY",
  "title": "Attempted update of read-only field: walletId",
  "status": 422,
  "traceId": "00-1133a5e8f93b83b4b29d61d91cb-1a140dcbf259a24d-00"
}
```

---

## **V2/MerchantUserUpdateData <font color="green">POST</font>**

**URL de la solicitud:**
`https://api.api-sandbox.paycaddy.dev/v2/MerchantUserUpdateData`

Este endpoint actualiza la información de usuarios **MerchantUser** y **MerchantUserSR** (personas jurídicas).
Permite la modificación controlada de información comercial o de compliance, manteniendo inmutables los identificadores y los atributos vinculados al KYB. La razón social (`registeredName`) no puede modificarse mediante esta llamada; se devuelve en la respuesta solo como referencia.

=== "Request"
	```json
	{
	  "userId": "string",
	  "email": "string",
	  "taxId": "string",
	  "legalRepresentation": "string",
	  "kindOfBusiness": "string",
	  "telephone": "+50760001234",
	  "address": {
	    "addressLine1": "Avenue 5, Building 14",
	    "addressLine2": "Suite 200",
	    "city": "Panama City",
	    "region": "Panama",
	    "postalCode": "000000",
	    "country": "PA"
	  },
	  "firstName": "John",
	  "lastName": "Smith",
	  "countryOfOperations": "PA, US",
	  "certificateOfGoodStanding": "https://cdn.server.com/docs/certificateOfGoodStanding.pdf",
	  "businessLicense": "https://cdn.server.com/docs/businessLicense.pdf",
	  "registerShareholder": "https://cdn.server.com/docs/registerShareholder.pdf",
	  "idShareholders": "https://cdn.server.com/docs/idShareholders.pdf",
	  "addressVerificationShareholders": "https://cdn.server.com/docs/addressVerificationShareholders.pdf"
	}
	```

=== "Response"
	```json
	{
	  "registeredName": "string",
	  "email": "string",
	  "taxId": "string",
	  "legalRepresentation": "string",
	  "kindOfBusiness": "string",
	  "telephone": "+50760001234",
	  "address": {
	    "addressLine1": "Avenue 5, Building 14",
	    "addressLine2": "Suite 200",
	    "city": "Panama City",
	    "region": "Panama",
	    "postalCode": "000000",
	    "country": "PA"
	  },
	  "firstName": "John",
	  "lastName": "Smith",
	  "countryOfOperations": "PA, US",
	  "certificateOfGoodStanding": "https://cdn.server.com/docs/certificateOfGoodStanding.pdf",
	  "businessLicense": "https://cdn.server.com/docs/businessLicense.pdf",
	  "registerShareholder": "https://cdn.server.com/docs/registerShareholder.pdf",
	  "idShareholders": "https://cdn.server.com/docs/idShareholders.pdf",
	  "addressVerificationShareholders": "https://cdn.server.com/docs/addressVerificationShareholders.pdf"
	}
	```

---

### Campos Editables

|Campo|Descripción|Editable|Notas|
|---|---|---|---|
|`email`|Email de la empresa|✅|Formato RFC-5322|
|`legalRepresentation`|Representante legal|✅|Debe cumplir ITU-T.50|
|`kindOfBusiness`|Tipo o código de negocio|✅||
|`telephone`|Teléfono de contacto|✅|Formato E.164|
|`address.*`|Componentes de la dirección|✅||
|`firstName`, `lastName`|Representante (persona natural)|✅|Mismas reglas que en la creación|
|`countryOfOperations`|Datos de país|✅|ISO alpha-2, separados por coma|
|`businessLicense`, `registerShareholder`, `certificateOfGoodStanding`, `idShareholders`, `addressVerificationShareholders`|URLs de documentos|✅|URLs HTTPS (PDF/JPG/PNG) de entre 5kb y 10mb|

---

### Campos No Editables

|Campo|Motivo|
|---|---|
|`userId`|Clave primaria inmutable|
|`registeredName`|Razón social; no aceptada por este endpoint (se devuelve en la respuesta como solo lectura)|
|`nationality`|No aceptado por este endpoint|
|`walletId`|Generado por el sistema|
|`isActive`|Controlado por el compliance de KYB|
|`creationDate`|Marca de tiempo inmutable|

---

### Validación y Manejo de Errores

|Código HTTP|Tipo|Descripción|
|---|---|---|
|`200`|OK|Actualización aceptada|
|`400`|ValidationError|Campo faltante o inválido|
|`422`|Business Rule Violation|Intento de modificar un campo restringido|
|`500`|Internal Error|Error inesperado del lado del servidor|

=== "Sample Error"

```json
{
  "type": "https://docs.paycaddy.com/errors/PC-422-READONLY",
  "title": "Attempted update of read-only field: registeredName",
  "status": 422,
  "traceId": "00-77aa4e5b7f1e3b45c92f91f77aa4e5b-1a140dcbf259a24d-00"
}
```

---

### Notas y Recomendaciones

- Se permiten actualizaciones parciales: solo se modifican las claves enviadas.

- Los cambios en **campos sensibles** (por ejemplo, `legalRepresentation`) pueden activar una **revisión manual** por parte del equipo de compliance de PayCaddy.

- Todas las actualizaciones se versionan internamente y pueden auditarse a solicitud.

- Las URLs de documentos deben permanecer válidas durante al menos 24 horas después de la actualización.


---

> **Recordatorio de Compliance:**
>  El propósito de estos endpoints es mantener los datos de los usuarios actualizados para el compliance continuo de KYC/KYB.
> Cualquier uso indebido o intento de alterar datos de identidad o titularidad fuera de los flujos autorizados puede resultar en el rechazo de la solicitud o la suspensión de la cuenta.
