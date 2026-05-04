!!!Important
    Desde el 15 de octubre de 2025, estos endpoints han sido habilitados como parte del esfuerzo por migrar toda la creación de usuarios de v1 a los endpoints documentados a continuación. Las comunicaciones oficiales definirán los tiempos exactos para la adopción. Si estás integrando el sistema de PayCaddy por primera vez, estos son los endpoints actualmente aceptados para el flujo de creación de usuarios.



## **V2/End User  <font color="green">POST</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/endUsers

‍La creación de un nuevo usuario para una persona natural comienza con una llamada POST en la cual se consume un endpoint para enviar la información básica del usuario:

=== "Request"
    ```json
        {
		  "email": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string"
		}
    ```
=== "Response"
    ```json
        {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "isActive": false,
		  "walletId": "string",
		  "kycUrl": "string",
		  "creationDate": "2025-01-01T00:00:00.000Z"
		}
    ```

#### Validación y Manejo de Errores

PayCaddy valida el **formato y la presencia** de los campos clave, mientras que la exactitud comercial (por ejemplo, "¿es este el _verdadero_ empleador de la persona?") y la **deduplicación entre tus propios usuarios** siguen siendo responsabilidad del cliente.

- La API **no** rechazará por exactitud comercial ni por duplicados.

- La API **sí** rechazará cuando no se cumplan las reglas de formato/validación (consulta la sección "Requisitos de los Campos" más abajo).

Si la solicitud pasa la validación y se procesa, la API responde con **HTTP 200 OK** y devuelve el `userId` creado, su `walletId` inicial y campos de control como `isActive=false` hasta que se complete el KYC.

>**PayCaddy no realiza deduplicación.**
>Múltiples POSTs con datos idénticos crearán `userId` **distintos**. Si necesitas control de duplicados, impleméntalo de tu lado antes de llamar a la API.

>**Control de Spam**
>Los payloads idénticos (o la creación de usuarios con emails idénticos) enviados con menos de 5 minutos de diferencia generarán un error 422 como medida contra spam erróneo; el payload volverá a ser aceptable una vez transcurrido ese tiempo.

---
#### Reglas de Nombre y Embossing

1. **Conjunto de caracteres saneado**: `FirstName` y `LastName` **deben** enviarse ya saneados a [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en).

2. **Límite de 22 caracteres para embossing**: `(FirstName + LastName)` **sin espacios** debe ser **≤ 22** **después** del saneamiento.

3. **Alias requerido cuando > 22**:
    Si `(FirstName + LastName)` excede 22 después del saneamiento, **debes** proporcionar un `alias` (máx. 22, ITU-T.50).

    - Cuando se proporciona un `alias` válido, se utiliza para la línea del nombre en la tarjeta.

    - Cuando el nombre completo cabe (≤ 22), el `alias` **no es obligatorio** y puede ignorarse para el embossing.

4. **Sin apodos**: el `alias` es estrictamente para casos de excedente de embossing; **no** uses apodos no relacionados, ya que puede afectar la aceptación presencial.

5. **Uso del Nombre Real:** los campos `FirstName` y `LastName` deben coincidir o asemejarse al nombre real que aparece en los documentos de identidad; de lo contrario, la verificación KYC fallará repetidamente.


---

#### Requisitos de los Campos

| Campo                  | Tipo    | Valor de ejemplo                                  | Requerido | Reglas                                                                                                                                                                                                       | Ejemplo de respuesta de error                                                                                                                                                                                                |
| ---------------------- | ------- | -------------------------------------------------- | -------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `firstName`            | string  | "John"                                             |      Sí | No nulo, no vacío, solo ITU-T.50; contribuye al límite de 22 caracteres (junto con `LastName`)                                                                                                                          | {<br>        "type": "",<br>        "title": "Error: special characters present in first name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                   |
| `lastName`             | string  | "Smith"                                            |      Sí | No nulo, no vacío, solo ITU-T.50; contribuye al límite de 22 caracteres (junto con `FirstName`)                                                                                                                         | {<br>        "type": "",<br>        "title": "Error: special characters present in last name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                    |
| `alias`                | string  | "J Smith"                                          |    Cond. | **Requerido únicamente** si `(FirstName+LastName)` > 22 después del saneamiento; ITU-T.50; máx. 22                                                                                                                       | {<br>        "type": "",<br>        "title": "Error: special characters present in alias",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                        |
| `occupation`           | string  | `"21313"` representando "Computer Systems Designer" |      Sí | **Código** de la lista de ocupaciones proporcionada (coincidencia exacta)<br><br>Encuentra la lista completa de valores en CSV o JSON [AQUÍ](https://github.com/Paycaddy-Admin/PayCaddy-Integration)                                      | {<br>        "type": "",<br>        "title": "Error: value not in allowed occupation list",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                       |
| `email`                | string  | "jsmith@example.com"                               |      Sí | No nulo, formato estándar de email (compatible con RFC-5322)                                                                                                                                                       | {<br>        "type": "",<br>        "title": "Invalid format for email",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                          |
| `telephone`            | string  | "+50760001234"                                     |      Sí | Debe ser **E.164** (`+` + código de país + número)                                                                                                                                                             | {<br>        "type": "",<br>        "title": "Telephone value is incorrect, allowed format: E.164 (`+` + country code + number)",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    } |
| `placeOfWork`          | string  | "ACME, Inc."                                       |      Sí |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `pep`                  | boolean | `false`                                            |      Sí | `true` si la persona natural está políticamente expuesta; de lo contrario `false`                                                                                                                                           |                                                                                                                                                                                                                       |
| `salary`               | number  | `200000` representando un salario mensual de USD 2,000.00  |      Sí | Salario mensual en **centavos de USD**;                                                                                                                                                                                         | {<br>        "type": "",<br>        "title": "Error: salary must be an integer value in cents USD",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                               |
| `address.addressLine1` | string  | "123 Main Street"                                  |      Sí |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `address.addressLine2` | string  | "PH Residential Tower"                             |       No | Opcional                                                                                                                                                                                                    |                                                                                                                                                                                                                       |
| `address.homeNumber`   | string  | "12B"                                              |       No | Si está presente, debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`)                                                                                                                                         | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |
| `address.city`         | string  | "Panama"                                           |      Sí |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `address.region`       | string  | "Panama Metropolitan Area"                         |      Sí |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `address.postalCode`   | string  | "000000"                                           |      Sí | Usa el formato de código postal correcto para los países que lo apliquen. Usa "000000" como bypass para los países que no usan código postal. La lógica por país puede evolucionar y se comunicará oportunamente. |                                                                                                                                                                                                                       |
| `address.country`      | string  | "PA"                                               |      Sí | Debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`)                                                                                                                                                     | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |
| `nationality`          | string  | "PA"                                               |      Sí | Debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`)                                                                                                                                                     |                                                                                                                                                                                                                       |
| `countryOfOperations`  | string  | "PA, US"                                           |      Sí | Debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`); si hay múltiples valores, sepáralos por comas en una sola cadena.                                                                                    |                                                                                                                                                                                                                       |


> **Definición de PEP:** persona natural a la que se le han confiado responsabilidades públicas prominentes y que, por su posición e influencia, puede presentar mayor riesgo de exposición a soborno o corrupción.

> **Recordatorio KYC:** Después de que se asignen los IDs, el usuario final debe completar el KYC para activar el perfil. Hasta entonces, `isActive` permanece como `false` y todos los procedimientos están deshabilitados. Consulta Verificación KYC.

---
#### Errores Comunes


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "FirstName": [
	            "The FirstName field is required."
	        ]
	    }
	}
	```

| Campo de entrada | Código/Respuesta        | Mensaje                                                                                                                 |
| -------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| email          | invalid_format (422)    | Invalid format for email                                                                                                |
| firstName      | special_chars (422)     | Error: special characters present in first name                                                                         |
| lastName       | special_chars (422)     | Error: special characters present in last name                                                                          |
| alias          | special_chars (422)     | Error: special characters present in alias<br><br>The field Alias is required when First and LastName are above 22 char |
| telephone      | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number                                                     |
| country        | invalid_value (422)     | Country value is incorrect                                                                                              |
| occupation     | not_allowed_value (422) | Error: value not in allowed occupation list                                                                             |
| salary         | invalid_format (422)    | Error: salary must be an integer value in cents USD                                                                     |
| addressLine1   | min_length (422)        | Error: AddressLine1 must be at least 5 characters long                                                                  |
| region         | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                                                                     |



> **500 Internal Server Error** indica un problema interno de nuestro lado. Por favor, notifica al equipo de PayCaddy con la evidencia del payload para que podamos investigar prontamente.

---
#### Verificación KYC
En la respuesta de creación del usuario (EndUser POST) se presenta, en el campo **kycUrl**, un enlace a la validación de identidad asociada al userId en Metamap, donde se mostrarán al usuario final las instrucciones y los pasos a seguir para completar la verificación.
```json
    {
        "kycUrl": "https%3a%2f%2fsignup.getmati.com%2f%3fmerchantToken%
                  3d6046cc2a54816f001bedd641%26flowId%3d6046cc2a54816f0
                01bedd640%26metadata%3d%7b%22userid%22%3a%226955ea0f4
                f3-4254-b10a-0181f307298c%22%7d"
    }
```

Es importante considerar que la kycURL se comparte aplicando URL encoding, lo que permite el envío de metadata (userId) que vincula la validación con el usuario creado. Para asegurar la activación del usuario al completar exitosamente la validación, es necesario garantizar que la URL a la que se redirige al usuario desde la interfaz front-end cargue la siguiente estructura:

>https://signup.getmati.com/?merchantToken=6046cc2a54816f001dedd641&flowId=6046cc2a54816f001bedd640&metadata={"userid":"a955ea0f-34f3-4254-b10a-0181f30729kd"}

Para información completa sobre KYC, consulta el capítulo de KYC de esta documentación.

---

## **End User <font color="sky-blue">GET</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/endUsers/

‍La llamada GET de un endUser te permite conocer los datos almacenados de un userId en particular, especialmente el walletId de su billetera inicial y el estado de actividad del usuario en el campo "isActive". Ambos datos son cruciales para las demás llamadas a la API de NeoBank.

Esta llamada puede usarse para verificar el estado del usuario en cualquier punto del flujo.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v2/endUsers/${USER_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "isActive": false,
		  "walletId": "string",
		  "kycUrl": "string",
		  "creationDate": "2025-01-01T00:00:00.000Z"
		}
    ```

---

## **V2/Merchant User <font color="green">POST</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/merchantUsers

La creación de un nuevo usuario para una persona jurídica comienza con una llamada POST en la cual se consume un endpoint para enviar los datos básicos de la persona jurídica.

=== "Request"
    ```json
    {
		"email": "string",
		"registeredName": "string",
		"taxId": "string",
		"legalRepresentation": "string",
		"kindOfBusiness": "string",
		"telephone": "string",
		"address": {
			"addressLine1": "string",
			"addressLine2": "string",
			"homeNumber": "string",
			"city": "string",
			"region": "string",
			"postalCode": "string",
			"country": "string"
		},
		 "firstName": "string",
		 "lastName": "string",
		 "nationality": "string",
		"countryOfOperations": "string",
		"certificateOfGoodStanding": "string",
		"businessLicense": "string",
		"registerShareholder": "string",
		"idShareholders": "string",
		"addressVerificationShareholders": "string"
		}
	```


=== "Response"
    ```json
	{
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}
	```



>Debe tenerse en cuenta que la responsabilidad de validar la **exactitud y el formato** de los datos ingresados recae en el **cliente de PayCaddy**, lo que significa que nuestra API devolverá una respuesta exitosa **siempre que se cumplan los siguientes parámetros**, independientemente de la veracidad de la información o de la duplicación de los datos compartidos.

>**Control de Spam**
>Los payloads idénticos (o la creación de usuarios con emails idénticos) enviados con menos de 5 minutos de diferencia generarán un error 422 como medida contra spam erróneo; el payload volverá a ser aceptable una vez transcurrido ese tiempo.

#### Requisitos de los Campos

| Campo                             | Tipo         | Requerido | Reglas                                                                                                                                                                            | Ejemplo de respuesta de error                                                                                                                                                  |
| --------------------------------- | ------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `email`                           | string       | Sí      | Debe seguir el formato RFC-5322. No nulo, no vacío.                                                                                                                                | {  <br>"type": "",  <br>"title": "Invalid format for email",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                            |
| `registeredName`                  | string       | Sí      | Nombre registrado de la persona jurídica. Debe estar saneado ([ITU-T50](https://www.itu.int/rec/T-REC-T.50/en)), ≤ 22 caracteres, sin emojis ni símbolos.                                          | {  <br>"type": "",  <br>"title": "Error: special characters present in registeredName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `taxId`                           | string       | Sí      | Identificador único de contribuyente (RUC, VATIN, CUIT, CNPJ, NIT, etc.). Solo alfanumérico y guiones.                                                                                     | {  <br>"type": "",  <br>"title": "Error: Tax ID must contain alphanumeric or hyphen characters only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}   |
| `legalRepresentation`             | string       | Sí      | Nombre completo del representante legal. Solo caracteres ITU-T.50, ≤ 60 caracteres.                                                                                                         | {  <br>"type": "",  <br>"title": "Error: special characters present in legalRepresentation",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}            |
| `kindOfBusiness`                  | string       | Sí      | Descripción o código de la actividad económica. Debe coincidir con la lista permitida.<br><br>Encuentra la lista completa de valores en CSV o JSON [AQUÍ](https://github.com/Paycaddy-Admin/PayCaddy-Integration) | {  <br>"type": "",  <br>"title": "Error: value not in allowed kindOfBusiness list",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                     |
| `telephone`                       | string       | Sí      | Debe seguir el formato E.164 (`+` + código de país + número).                                                                                                                          | {  <br>"type": "",  <br>"title": "Telephone value is incorrect, allowed format: +(country code)number",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>} |
| `address.addressLine1`            | string       | Sí      | Mínimo 5 caracteres. Dirección física descriptiva.                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: AddressLine1 must be at least 5 characters long",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}              |
| `address.region`                  | string       | Sí      | Debe ser solo alfabético, no vacío.                                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: Region must be non-empty and alphabetic-only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `address.country`                 | string       | Sí      | Código ISO-3166-1 alpha-2 (p. ej., `PA`, `US`, `BR`).                                                                                                                                | {  <br>"type": "",  <br>"title": "Country value is incorrect",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                          |
| `firstName`                       | string       | Sí      | Nombre del propietario/representante. Solo [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en); forma parte del límite de 22 caracteres combinado con `lastName`.                                       | {  <br>"type": "",  <br>"title": "Error: special characters present in firstName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `lastName`                        | string       | Sí      | Apellido del propietario/representante. Solo [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en); forma parte del límite de 22 caracteres combinado con `firstName`.                                       | {  <br>"type": "",  <br>"title": "Error: special characters present in lastName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                       |
| `nationality`                     | string       | Sí      | Debe ser un código de país ISO-3166-1 alpha-2.                                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: Invalid nationality code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                     |
| `countryOfOperations`             | string       | Sí      | País donde opera principalmente la entidad; código ISO-3166-1 alpha-2. <br><br>Múltiples códigos pueden introducirse como valores separados por comas, p. ej.: "BR, PA"                             | {  <br>"type": "",  <br>"title": "Error: Invalid countryOfOperations code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                             |
| `certificateOfGoodStanding`       | string (URL) | Sí      | Debe ser una URL válida al documento oficial subido para KYB.                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: certificateOfGoodStanding must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                |
| `businessLicense`                 | string (URL) | Sí      | Debe ser una URL válida al documento de licencia comercial activa.                                                                                                                           | {  <br>"type": "",  <br>"title": "Error: businessLicense must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                          |
| `registerShareholder`             | string (URL) | Sí      | Debe ser una URL válida al documento oficial del registro de accionistas.                                                                                                                     | {  <br>"type": "",  <br>"title": "Error: registerShareholder must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `idShareholders`                  | string (URL) | Sí      | Debe ser una URL válida a las identificaciones escaneadas de los accionistas.                                                                                                                                | {  <br>"type": "",  <br>"title": "Error: idShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                           |
| `addressVerificationShareholders` | string (URL) | Sí      | Debe ser una URL válida a los documentos de comprobante de domicilio de los accionistas.                                                                                                                | {  <br>"type": "",  <br>"title": "Error: addressVerificationShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}          |

#### Activación KYB y Responsabilidades Adicionales sobre los Campos

Además de las verificaciones de formato, es importante destacar la responsabilidad del cliente de **enviar de forma consistente información comercial precisa y verificable** para la revisión de cumplimiento KYB:

1. El campo **`taxId`** debe incluir el **número oficial de identificación de contribuyente** de la entidad (p. ej., RUC, VATIN, CUIT, CNPJ, NIT o RUT), según la jurisdicción.

2. El campo **`legalRepresentation`** debe contener el nombre legal completo de la persona autorizada para representar a la empresa, tal como esté registrado en su acta de constitución o registro gubernamental.

3. Los campos **`firstName`** y **`lastName`** deben corresponder a la persona natural asociada con la **representación legal** o al individuo que se pretende sea el titular de la tarjeta. Estos deben cumplir con el mismo saneamiento ITU-T.50 y restricciones de caracteres usados para personas naturales. Estos serán los nombres impresos en la tarjeta, por lo que deben buscar identificar al titular.

4. Los campos **`certificateOfGoodStanding`**, **`businessLicense`**, **`registerShareholder`**, **`idShareholders`** y **`addressVerificationShareholders`** deben contener URLs válidas o referencias a documentos para la documentación de cumplimiento de la empresa.

    - Cada documento subido debe ser accesible al instante para captura programática; los archivos esperados deben tener al menos 5kb y no más de 10mb de tamaño, y deben ser de tipo PDF, JPG o PNG.

    - Los enlaces faltantes o inválidos generarán un error de validación y la creación del usuario fallará.

5. El equipo de cumplimiento de PayCaddy revisará la información compartida para su aprobación y la activación del usuario. Asegurar que la información se comparta correctamente disminuye las tasas de rechazo.


#### Errores Comunes


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "RegisteredName": [
	            "The RegisteredName field is required."
	        ]
	    }
	}
	```

| Campo de entrada                | Código/Respuesta        | Mensaje                                                             |
| ------------------------------- | ----------------------- | ------------------------------------------------------------------- |
| email                           | invalid_format (422)    | Invalid format for email                                            |
| registeredName                  | special_chars (422)     | Error: special characters present in registeredName                 |
| taxId                           | invalid_format (422)    | Error: Tax ID must contain alphanumeric or hyphen characters only   |
| legalRepresentation             | special_chars (422)     | Error: special characters present in legalRepresentation            |
| kindOfBusiness                  | not_allowed_value (422) | Error: value not in allowed kindOfBusiness list                     |
| telephone                       | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number |
| address.addressLine1            | min_length (422)        | Error: AddressLine1 must be at least 5 characters long              |
| address.region                  | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                 |
| address.country                 | invalid_value (422)     | Country value is incorrect                                          |
| firstName                       | special_chars (422)     | Error: special characters present in firstName                      |
| lastName                        | special_chars (422)     | Error: special characters present in lastName                       |
| nationality                     | invalid_value (422)     | Error: nationality must be a valid ISO alpha-2 code                 |
| countryOfOperations             | invalid_value (422)     | Error: countryOfOperations must be a valid ISO alpha-2 code         |
| certificateOfGoodStanding       | invalid_format (422)    | Error: certificateOfGoodStanding must be a valid URL                |
| businessLicense                 | invalid_format (422)    | Error: businessLicense must be a valid URL                          |
| registerShareholder             | invalid_format (422)    | Error: registerShareholder must be a valid URL                      |
| idShareholders                  | invalid_format (422)    | Error: idShareholders must be a valid URL                           |
| addressVerificationShareholders | invalid_format (422)    | Error: addressVerificationShareholders must be a valid URL          |

> **500 Internal Server Error** indica un problema interno de nuestro lado. Por favor, notifica al equipo de PayCaddy con la evidencia del payload para que podamos investigar prontamente.


---

## **Merchant User <font color="sky-blue">GET</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/merchantUsers/

La llamada GET de un merchantUser te permite conocer los datos almacenados de un userId en particular, especialmente el walletId de su billetera inicial y el estado de actividad del usuario en el campo "isActive". Ambos datos son cruciales para las demás llamadas a la API de NeoBank.

Esta llamada puede usarse para verificar el estado del usuario en cualquier punto del flujo.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v2/merchantUsers/{MERCHANT_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}
    ```

---

## **V2/End User SR  <font color="green">POST</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/SR/EndUserSRs

‍La creación de un nuevo usuario con tipo KYC Delegado para una persona natural comienza con una llamada POST en la cual se consume un endpoint para enviar la información básica del usuario. En el caso de End User SR (Subject to Regulation), los usuarios se crean activos por defecto.
Este endpoint no está disponible abiertamente; debe haber sido habilitado por el equipo de cumplimiento de PayCaddy durante el proceso de integración.

=== "Request"
    ```json
        {
		  "email": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string"
		}
    ```
=== "Response"
    ```json
        {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string",
		  "alias": "string",
		  "occupation": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "creationDate": "2025-10-14T19:24:26.919Z"
		}
    ```

#### Validación y Manejo de Errores

PayCaddy valida el **formato y la presencia** de los campos clave, mientras que la exactitud comercial (por ejemplo, "¿es este el _verdadero_ empleador de la persona?") y la **deduplicación entre tus propios usuarios** siguen siendo responsabilidad del cliente.

- La API **no** rechazará por exactitud comercial ni por duplicados.

- La API **sí** rechazará cuando no se cumplan las reglas de formato/validación (consulta la sección "Requisitos de los Campos" más abajo).

Si la solicitud pasa la validación y se procesa, la API responde con **HTTP 200 OK** y devuelve el `userId` creado, su `walletId` inicial y campos de control como `isActive=true` para este tipo de usuario con flujo KYC Delegado.

>**PayCaddy no realiza deduplicación.**
>Múltiples POSTs con datos idénticos crearán `userId` **distintos**. Si necesitas control de duplicados, impleméntalo de tu lado antes de llamar a la API.

>**Control de Spam**
>Los payloads idénticos (o la creación de usuarios con emails idénticos) enviados con menos de 5 minutos de diferencia generarán un error 422 como medida contra spam erróneo; el payload volverá a ser aceptable una vez transcurrido ese tiempo.

---
#### Reglas de Nombre y Embossing

1. **Conjunto de caracteres saneado**: `FirstName` y `LastName` **deben** enviarse ya saneados a ITU-T.50.

2. **Límite de 22 caracteres para embossing**: `(FirstName + LastName)` **sin espacios** debe ser **≤ 22** **después** del saneamiento.

3. **Alias requerido cuando > 22**:
    Si `(FirstName + LastName)` excede 22 después del saneamiento, **debes** proporcionar un `alias` (máx. 22, ITU-T.50).

    - Cuando se proporciona un `alias` válido, se utiliza para la línea del nombre en la tarjeta.

    - Cuando el nombre completo cabe (≤ 22), el `alias` **no es obligatorio** y puede ignorarse para el embossing.

4. **Sin apodos**: el `alias` es estrictamente para casos de excedente de embossing; **no** uses apodos no relacionados, ya que puede afectar la aceptación presencial.

5. **Uso del Nombre Real:** los campos `FirstName` y `LastName` deben coincidir o asemejarse al nombre real que aparece en los documentos de identidad para asegurar que los chequeos de cumplimiento futuros no marquen a los usuarios por mal uso.


---

#### Requisitos de los Campos

| Campo                  | Tipo    | Valor de ejemplo                                                  | Requerido | Reglas                                                                                                                                                                                                                                                                                       | Ejemplo de respuesta de error                                                                                                                                                                                                |     |
| ---------------------- | ------- | ----------------------------------------------------------------- | -------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `firstName`            | string  | "John"                                                            |      Sí | No nulo, no vacío, solo ITU-T.50; contribuye al límite de 22 caracteres (junto con `LastName`)                                                                                                                                                                                                           | {<br>        "type": "",<br>        "title": "Error: special characters present in first name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                   |     |
| `lastName`             | string  | "Smith"                                                           |      Sí | No nulo, no vacío, solo ITU-T.50; contribuye al límite de 22 caracteres (junto con `FirstName`)                                                                                                                                                                                                          | {<br>        "type": "",<br>        "title": "Error: special characters present in last name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                    |     |
| `alias`                | string  | "J Smith"                                                         |    Cond. | **Requerido únicamente** si `(FirstName+LastName)` > 22 después del saneamiento; ITU-T.50; máx. 22                                                                                                                                                                                                        | {<br>        "type": "",<br>        "title": "Error: special characters present in alias",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                        |     |
| `occupation`           | string  | `"21313"` representando "Computer Systems Designer"                |      Sí | **Código** de la lista de ocupaciones proporcionada (coincidencia exacta)<br><br>Encuentra la lista completa de valores en CSV o JSON [AQUÍ](https://github.com/Paycaddy-Admin/PayCaddy-Integration)                                                                                                                       | {<br>        "type": "",<br>        "title": "Error: value not in allowed occupation list",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                       |     |
| `email`                | string  | "jsmith@example.com"                                              |      Sí | No nulo, formato estándar de email (compatible con RFC-5322)                                                                                                                                                                                                                                       | {<br>        "type": "",<br>        "title": "Invalid format for email",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                          |     |
| `telephone`            | string  | "+50760001234"                                                    |      Sí | Debe ser **E.164** (`+` + código de país + número)                                                                                                                                                                                                                                             | {<br>        "type": "",<br>        "title": "Telephone value is incorrect, allowed format: E.164 (`+` + country code + number)",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    } |     |
| `placeOfWork`          | string  | "ACME, Inc."                                                      |      Sí |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `pep`                  | boolean | `false`                                                           |      Sí | `true` si la persona natural está políticamente expuesta; de lo contrario `false`                                                                                                                                                                                                                           |                                                                                                                                                                                                                       |     |
| `salary`               | number  | `200000` representando un salario mensual de USD 2,000.00                 |      Sí | Salario mensual en **centavos de USD**;                                                                                                                                                                                                                                                                       | {<br>        "type": "",<br>        "title": "Error: salary must be an integer value in cents USD",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                               |     |
| `address.addressLine1` | string  | "123 Main Street"                                                 |      Sí |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `address.addressLine2` | string  | "PH Residential Tower"                                            |       No | Opcional                                                                                                                                                                                                                                                                                     |                                                                                                                                                                                                                       |     |
| `address.homeNumber`   | string  | "12B"                                                             |       No | Si está presente, debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`)                                                                                                                                                                                                                          | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |     |
| `address.city`         | string  | "Panama"                                                          |      Sí |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `address.region`       | string  | "Panama Metropolitan Area"                                        |      Sí |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `address.postalCode`   | string  | "000000"                                                          |      Sí | Usa el formato de código postal correcto para los países que lo apliquen. Usa "000000" como bypass para los países que no usan código postal. La lógica por país puede evolucionar y se comunicará oportunamente.                                                                                  |                                                                                                                                                                                                                       |     |
| `address.country`      | string  | "PA"                                                              |      Sí | Debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`)                                                                                                                                                                                                                                      | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |     |
| `nationality`          | string  | "PA"                                                              |      Sí | Debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`)                                                                                                                                                                                                                                      |                                                                                                                                                                                                                       |     |
| `countryOfOperations`  | string  | "PA, US"                                                          |      Sí | Debe ser ISO-3166-1 **alpha-2** (p. ej., `PA`, `US`, `BR`); si hay múltiples valores, sepáralos por comas en una sola cadena.                                                                                                                                                                     | }                                                                                                                                                                                                                     |     |
| `idUrlFront`           | string  | "https://cdn.prod.server-files.com/directory/user_idFile.jpg"     |      Sí | La URL debe dirigir a un archivo real y accesible, ya sea `PDF`, `JPG` o `PNG`, con un tamaño entre 5kb y 10mb. Se espera que este archivo contenga los tipos de documento correctamente aceptados.<br><br>El equipo de PayCaddy se reserva el derecho de bloquear regularmente a los usuarios que presenten un uso indebido de estos campos. | {<br>		  "type": "",<br>		  "title": "The value provided for the IdUrlFront presents some issues in the process of validation",<br>		  "status": 0,<br>		  "detail": "",<br>		  "instance": ""<br>		}```              |     |
| `idUrlBack`            | string  | "https://cdn.prod.server-files.com/directory/user_idFileBack.jpg" |      Sí | La URL debe dirigir a un archivo real y accesible, ya sea `PDF`, `JPG` o `PNG`, con un tamaño entre 5kb y 10mb. Se espera que este archivo contenga los tipos de documento correctamente aceptados.<br><br>El equipo de PayCaddy se reserva el derecho de bloquear regularmente a los usuarios que presenten un uso indebido de estos campos. | {<br>		  "type": "",<br>		  "title": "The value provided for the IdUrlBack presents some issues in the process of validation",<br>		  "status": 0,<br>		  "detail": "",<br>		  "instance": ""<br>		}```               |     |
| `residenceProof`       | string  | "https://cdn.prod.server-files.com/directory/residenceProof.jpg"  |      Sí | La URL debe dirigir a un archivo real y accesible, ya sea `PDF`, `JPG` o `PNG`, con un tamaño entre 5kb y 10mb. Se espera que este archivo contenga los tipos de documento correctamente aceptados.<br><br>El equipo de PayCaddy se reserva el derecho de bloquear regularmente a los usuarios que presenten un uso indebido de estos campos. | {<br>		  "type": "",<br>		  "title": "The value provided for the residenceProof presents some issues in the process of validation",<br>		  "status": 0,<br>		  "detail": "",<br>		  "instance": ""<br>		}```          |     |


> **Definición de PEP:** persona natural a la que se le han confiado responsabilidades públicas prominentes y que, por su posición e influencia, puede presentar mayor riesgo de exposición a soborno o corrupción.

> **KYC Delegado:** Los usuarios de tipo EndUserSR son utilizados exclusivamente por clientes a los que se les ha asignado dicho privilegio. Estos usuarios se crean con su estado de actividad como "Activo" y no requieren un enlace de verificación KYC.

---
#### Errores Comunes


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "FirstName": [
	            "The FirstName field is required."
	        ]
	    }
	}
	```

=== "Document Access Issues"
	```json
		{
		  "type": "",
		  "title": "The value provided for the IdUrlBack presents some issues in the process of validation",
		  "status": 0,
		  "detail": "",
		  "instance": ""
		}```



| Campo de entrada | Código/Respuesta        | Mensaje                                                                                                                 |
| -------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| email          | invalid_format (422)    | Invalid format for email                                                                                                |
| firstName      | special_chars (422)     | Error: special characters present in first name                                                                         |
| lastName       | special_chars (422)     | Error: special characters present in last name                                                                          |
| alias          | special_chars (422)     | Error: special characters present in alias<br><br>The field Alias is required when First and LastName are above 22 char |
| telephone      | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number                                                     |
| country        | invalid_value (422)     | Country value is incorrect                                                                                              |
| occupation     | not_allowed_value (422) | Error: value not in allowed occupation list                                                                             |
| kindOfBusiness | not_allowed_value (422) | Error: value not in allowed kindOfBusiness list                                                                         |
| salary         | invalid_format (422)    | Error: salary must be an integer value in cents USD                                                                     |
| addressLine1   | min_length (422)        | Error: AddressLine1 must be at least 5 characters long                                                                  |
| region         | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                                                                     |



> **500 Internal Server Error** indica un problema interno de nuestro lado. Por favor, notifica al equipo de PayCaddy con la evidencia del payload para que podamos investigar prontamente.

---
#### KYC Delegado (SR) – URLs de Documentos

En SR, **no existe `kycUrl`** para la verificación KYC posterior a la creación. En su lugar, **debes** proporcionar **dentro de la solicitud de creación** los siguientes documentos:

```json
{
  "idUrlFront": "https://... (front of ID)",
  "idUrlBack": "https://... (back of ID)",
  "residenceProofUrl": "https://... (proof of residence)"
}
```

- Las URLs **deben ser HTTPS** y **públicamente accesibles** por el backend de PayCaddy (accesibles por al menos 24 horas, sin enlaces bloqueados por IP).

- El contenido **debe ser** una imagen (`image/*`) o `application/pdf`.

- Asegúrate de que los enlaces se mantengan válidos por al menos 24 horas.

- Si alguna URL es inválida, inaccesible o el tipo de archivo no es soportado, la solicitud será rechazada con **422**.



> **Activación:** los usuarios SR se crean con **`isActive=true`** inmediatamente al ser exitosos. Las acciones posteriores siguen sujetas a las verificaciones de cumplimiento estándar y al monitoreo continuo.


---

## **End User SR <font color="sky-blue">GET</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/SR/EndUserSRs/

La llamada GET de un EndUserSR te permite conocer los datos almacenados de un userId en particular, especialmente el walletId de su billetera inicial y el estado de actividad del usuario en el campo "isActive". Ambos datos son cruciales para las demás llamadas a la API de NeoBank.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs/{USER_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string",
		  "alias": "string",
		  "occupation": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "creationDate": "2025-10-14T19:24:26.919Z"
	}
    ```


---

## **V2/Merchant User SR <font color="green">POST</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/merchantUserSRs

La creación de un nuevo usuario para una persona jurídica con KYB Delegado comienza con una llamada POST en la cual se consume un endpoint para enviar los datos básicos de la persona jurídica. En el caso de Merchant User SR (Subject to Regulation), los usuarios se crean activos por defecto.
Este endpoint no está disponible abiertamente; debe haber sido habilitado por el equipo de cumplimiento de PayCaddy durante el proceso de integración.

>Previo al lanzamiento de nuestra rama V2 para la creación de usuarios, Merchant User SR no estaba disponible y todos los Merchant Users creados seguían un flujo KYB Delegado. Esta diferenciación se aplica desde octubre de 2025 con el lanzamiento de los endpoints V2.

=== "Request"
    ```json
    {
		"email": "string",
		"registeredName": "string",
		"taxId": "string",
		"legalRepresentation": "string",
		"kindOfBusiness": "string",
		"telephone": "string",
		"address": {
			"addressLine1": "string",
			"addressLine2": "string",
			"homeNumber": "string",
			"city": "string",
			"region": "string",
			"postalCode": "string",
			"country": "string"
		},
		 "firstName": "string",
		 "lastName": "string",
		 "nationality": "string",
		"countryOfOperations": "string",
		"certificateOfGoodStanding": "string",
		"businessLicense": "string",
		"registerShareholder": "string",
		"idShareholders": "string",
		"addressVerificationShareholders": "string"
		}
	```


=== "Response"
    ```json
	{
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}
	```

>Debe tenerse en cuenta que la responsabilidad de validar la **exactitud y el formato** de los datos ingresados recae en el **cliente de PayCaddy**, lo que significa que nuestra API devolverá una respuesta exitosa **siempre que se cumplan los siguientes parámetros**, independientemente de la veracidad de la información o de la duplicación de los datos compartidos.

>**Control de Spam**
>Los payloads idénticos (o la creación de usuarios con emails idénticos) enviados con menos de 5 minutos de diferencia generarán un error 422 como medida contra spam erróneo; el payload volverá a ser aceptable una vez transcurrido ese tiempo.

#### Requisitos de los Campos

| Campo                             | Tipo         | Requerido | Reglas                                                                                                                                                                            | Ejemplo de respuesta de error                                                                                                                                                  |
| --------------------------------- | ------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `email`                           | string       | Sí      | Debe seguir el formato RFC-5322. No nulo, no vacío.                                                                                                                                | {  <br>"type": "",  <br>"title": "Invalid format for email",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                            |
| `registeredName`                  | string       | Sí      | Nombre registrado de la persona jurídica. Debe estar saneado (ITU-T.50), ≤ 22 caracteres, sin emojis ni símbolos.                                                                                  | {  <br>"type": "",  <br>"title": "Error: special characters present in registeredName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `taxId`                           | string       | Sí      | Identificador único de contribuyente (RUC, VATIN, CUIT, CNPJ, NIT, etc.). Solo alfanumérico y guiones.                                                                                     | {  <br>"type": "",  <br>"title": "Error: Tax ID must contain alphanumeric or hyphen characters only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}   |
| `legalRepresentation`             | string       | Sí      | Nombre completo del representante legal. Solo caracteres ITU-T.50, ≤ 60 caracteres.                                                                                                         | {  <br>"type": "",  <br>"title": "Error: special characters present in legalRepresentation",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}            |
| `kindOfBusiness`                  | string       | Sí      | Descripción o código de la actividad económica. Debe coincidir con la lista permitida.<br><br>Encuentra la lista completa de valores en CSV o JSON [AQUÍ](https://github.com/Paycaddy-Admin/PayCaddy-Integration) | {  <br>"type": "",  <br>"title": "Error: value not in allowed kindOfBusiness list",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                     |
| `telephone`                       | string       | Sí      | Debe seguir el formato E.164 (`+` + código de país + número).                                                                                                                          | {  <br>"type": "",  <br>"title": "Telephone value is incorrect, allowed format: +(country code)number",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>} |
| `address.addressLine1`            | string       | Sí      | Mínimo 5 caracteres. Dirección física descriptiva.                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: AddressLine1 must be at least 5 characters long",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}              |
| `address.region`                  | string       | Sí      | Debe ser solo alfabético, no vacío.                                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: Region must be non-empty and alphabetic-only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `address.country`                 | string       | Sí      | Código ISO-3166-1 alpha-2 (p. ej., `PA`, `US`, `BR`).                                                                                                                                | {  <br>"type": "",  <br>"title": "Country value is incorrect",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                          |
| `firstName`                       | string       | Sí      | Nombre del propietario/representante. Solo ITU-T.50; forma parte del límite de 22 caracteres combinado con `lastName`.                                                                               | {  <br>"type": "",  <br>"title": "Error: special characters present in firstName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `lastName`                        | string       | Sí      | Apellido del propietario/representante. Solo ITU-T.50; forma parte del límite de 22 caracteres combinado con `firstName`.                                                                               | {  <br>"type": "",  <br>"title": "Error: special characters present in lastName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                       |
| `nationality`                     | string       | Sí      | Debe ser un código de país ISO-3166-1 alpha-2.                                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: Invalid nationality code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                     |
| `countryOfOperations`             | string       | Sí      | País donde opera principalmente la entidad; código ISO-3166-1 alpha-2. <br><br>Múltiples códigos pueden introducirse como valores separados por comas, p. ej.: "BR, PA"                             | {  <br>"type": "",  <br>"title": "Error: Invalid countryOfOperations code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                             |
| `certificateOfGoodStanding`       | string (URL) | Sí      | Debe ser una URL válida al documento oficial subido para KYB.                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: certificateOfGoodStanding must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                |
| `businessLicense`                 | string (URL) | Sí      | Debe ser una URL válida al documento de licencia comercial activa.                                                                                                                           | {  <br>"type": "",  <br>"title": "Error: businessLicense must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                          |
| `registerShareholder`             | string (URL) | Sí      | Debe ser una URL válida al documento oficial del registro de accionistas.                                                                                                                     | {  <br>"type": "",  <br>"title": "Error: registerShareholder must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `idShareholders`                  | string (URL) | Sí      | Debe ser una URL válida a las identificaciones escaneadas de los accionistas.                                                                                                                                | {  <br>"type": "",  <br>"title": "Error: idShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                           |
| `addressVerificationShareholders` | string (URL) | Sí      | Debe ser una URL válida a los documentos de comprobante de domicilio de los accionistas.                                                                                                                | {  <br>"type": "",  <br>"title": "Error: addressVerificationShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}          |

#### Responsabilidades Adicionales sobre los Campos

Además de las verificaciones de formato, es importante destacar la responsabilidad del cliente de **enviar de forma consistente información comercial precisa y verificable** para la revisión de cumplimiento KYB:

1. El campo **`taxId`** debe incluir el **número oficial de identificación de contribuyente** de la entidad (p. ej., RUC, VATIN, CUIT, CNPJ, NIT o RUT), según la jurisdicción.

2. El campo **`legalRepresentation`** debe contener el nombre legal completo de la persona autorizada para representar a la empresa, tal como esté registrado en su acta de constitución o registro gubernamental.

3. Los campos **`firstName`** y **`lastName`** deben corresponder a la persona natural asociada con la **representación legal** o al individuo que se pretende sea el titular de la tarjeta. Estos deben cumplir con el mismo saneamiento ITU-T.50 y restricciones de caracteres usados para personas naturales. Estos serán los nombres impresos en la tarjeta, por lo que deben buscar identificar al titular.

4. Los campos **`certificateOfGoodStanding`**, **`businessLicense`**, **`registerShareholder`**, **`idShareholders`** y **`addressVerificationShareholders`** deben contener URLs válidas o referencias a documentos para la documentación de cumplimiento de la empresa.

    - Cada documento subido debe ser accesible al instante para captura programática; los archivos esperados deben tener al menos 5kb y no más de 10mb de tamaño, y deben ser de tipo PDF, JPG o PNG.

    - Los enlaces faltantes o inválidos generarán un error de validación y la creación del usuario fallará.

5. El equipo de cumplimiento de PayCaddy revisará la información compartida para su aprobación y la activación del usuario. Asegurar que la información se comparta correctamente disminuye las tasas de rechazo.


#### Errores Comunes


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "RegisteredName": [
	            "The RegisteredName field is required."
	        ]
	    }
	}
	```

| Campo de entrada                | Código/Respuesta        | Mensaje                                                             |
| ------------------------------- | ----------------------- | ------------------------------------------------------------------- |
| email                           | invalid_format (422)    | Invalid format for email                                            |
| registeredName                  | special_chars (422)     | Error: special characters present in registeredName                 |
| taxId                           | invalid_format (422)    | Error: Tax ID must contain alphanumeric or hyphen characters only   |
| legalRepresentation             | special_chars (422)     | Error: special characters present in legalRepresentation            |
| kindOfBusiness                  | not_allowed_value (422) | Error: value not in allowed kindOfBusiness list                     |
| telephone                       | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number |
| address.addressLine1            | min_length (422)        | Error: AddressLine1 must be at least 5 characters long              |
| address.region                  | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                 |
| address.country                 | invalid_value (422)     | Country value is incorrect                                          |
| firstName                       | special_chars (422)     | Error: special characters present in firstName                      |
| lastName                        | special_chars (422)     | Error: special characters present in lastName                       |
| nationality                     | invalid_value (422)     | Error: nationality must be a valid ISO alpha-2 code                 |
| countryOfOperations             | invalid_value (422)     | Error: countryOfOperations must be a valid ISO alpha-2 code         |
| certificateOfGoodStanding       | invalid_format (422)    | Error: certificateOfGoodStanding must be a valid URL                |
| businessLicense                 | invalid_format (422)    | Error: businessLicense must be a valid URL                          |
| registerShareholder             | invalid_format (422)    | Error: registerShareholder must be a valid URL                      |
| idShareholders                  | invalid_format (422)    | Error: idShareholders must be a valid URL                           |
| addressVerificationShareholders | invalid_format (422)    | Error: addressVerificationShareholders must be a valid URL          |

> **500 Internal Server Error** indica un problema interno de nuestro lado. Por favor, notifica al equipo de PayCaddy con la evidencia del payload para que podamos investigar prontamente.


---

## **Merchant User SR <font color="sky-blue">GET</font>**

**URL de la solicitud:** https://api.api-sandbox.paycaddy.dev/v2/SR/MerchantUsersSR/

La llamada GET de un merchantUser te permite conocer los datos almacenados de un userId en particular, especialmente el walletId de su billetera inicial y el estado de actividad del usuario en el campo "isActive". Ambos datos son cruciales para las demás llamadas a la API de NeoBank.

Esta llamada puede usarse para verificar el estado del usuario en cualquier punto del flujo.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v2/merchantUserSRs/{MERCHANT_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}```

---
