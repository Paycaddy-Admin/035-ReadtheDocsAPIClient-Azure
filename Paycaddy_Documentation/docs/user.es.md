!!!Importante
	**Aviso de Depreciación:**  
	Desde el 15 de octubre de 2025, estos endpoints han iniciado su proceso de depreciación como parte del esfuerzo por migrar toda la creación de usuarios de v1 a nuestros [endpoints v2](./userv2.es.md). Las comunicaciones oficiales informarán los plazos exactos para la adopción. Si estás integrando el sistema de PayCaddy por primera vez, consulta nuestros [endpoints v2](./userv2.es.md) para conocer los esquemas actualmente aceptados para nuevos proyectos.
	

## **End User <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/endUsers](https://api.api-sandbox.paycaddy.dev/v1/endUsers)

‍La creación de un nuevo usuario para una **persona natural** comienza con una llamada **POST** en la que se consume un endpoint para enviar la información básica del usuario:

=== "Request"
	```json
	{
	    "email": "string",
	    "firstName": "string",
	    "lastName": "string",
	    "occupation": "string",
	    "placeofwork": "string",
	    "pep": "bool",
	    "salary": "integer",
	    "telephone": "string",
	    "address": {
	        "addressLine1": "string",
	        "addressLine2": "string",
	        "homeNumber": "string",
	        "city": "string",
	        "region": "string",
	        "postalCode": "string",
	        "country": "string"
	    }
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
	    "isActive": false,
	    "walletId": "string",
	    "kycUrl": "string",
	    "creationDate": "2022-07-13T21:07:21.166Z"
	}
	```

Se debe tener en cuenta que la **responsabilidad** de validar la veracidad y el formato de los datos ingresados recae en el cliente de PayCaddy, por lo que nuestra API devolverá una respuesta exitosa siempre que se cumplan los siguientes parámetros, sin importar la exactitud de la información ni la duplicidad de los datos compartidos:

1. Los campos **First Name** y **Last Name** deben sumar un máximo total de **22 caracteres** que deben enviarse en formato saneado, sin caracteres especiales y limitándose al rango [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en), que es un subconjunto ligeramente más restrictivo de ASCII.
    
2. Ninguno de los campos debe enviarse como **NULL**.
    
3. El campo **email** debe seguir un formato estándar de correo electrónico.
    

Además de las verificaciones de formato, es importante destacar la responsabilidad del cliente de enviar coherentemente el resto de los campos con información correcta:

- El campo **"salary"** debe incluir el salario mensual del usuario en USD.
    
- El campo **"pep"** indica si la persona natural para la que se crea el usuario es políticamente expuesta.
    
    - **PEP**: Persona a la que se le ha confiado una función pública prominente. Por lo general, presenta mayor riesgo de participación potencial en sobornos y corrupción debido a su cargo e influencia.
        

El control sobre la **duplicación de usuarios** debe ser gestionado por el cliente de PayCaddy. Nuestra API generará múltiples **userId** distintos incluso si los datos compartidos son idénticos en llamadas duplicadas.

Si el evento no genera errores, el sistema responderá con un **HTTP 200 OK**. La respuesta exitosa replica los datos ingresados y agrega la información de **isActive**, el **walletId** inicial asociado al userId y la marca de tiempo (creationDate) de dicho usuario.

Si se produce un error, el sistema responderá con uno de los siguientes mensajes con código **400**:

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

=== "Character Limit"
	```json
	{
	    "type": "",
	    "title": "The value provided for the name exceeds the maximum value",
	    "status": 0,
	    "detail": "",
	    "instance": ""
	}
	```

=== "Formato de email"
	```json
	{
	    "type": "",
	    "title": "Invalid format for email",
	    "status": 0,
	    "detail": "",
	    "instance": ""
	}
	```

> En caso de recibir un mensaje de error **500**, podría tratarse de un error interno del servidor. Si esto ocurre, te pedimos que informes al equipo de PayCaddy para que investiguemos y resolvamos el problema con eficiencia.

Una vez asignados los identificadores de **usuario** y **wallet**, el usuario final debe completar la **verificación KYC** para finalizar la creación del perfil. Hasta que esta verificación se complete, el campo de control **"isActive"** permanecerá en `false`, deshabilitando cualquier operación.

En la respuesta de creación de usuario (EndUser POST) se presenta en el campo **kycUrl** un enlace a la verificación de identidad en Metamap asociada al userId, donde se detallan las instrucciones y pasos a seguir:

```json
{
    "kycUrl": "https%3a%2f%2fsignup.getmati.com%2f%3fmerchantToken%
              3d6046cc2a54816f001bedd641%26flowId%3d6046cc2a54816f0
              01bedd640%26metadata%3d%7b%22userid%22%3a%226955ea0f4
              f3-4254-b10a-0181f307298c%22%7d"
}
```

Es importante considerar que el **kycUrl** se comparte aplicando URL-encoding que permite enviar metadata (userId) y vincular la verificación con el usuario creado. Para asegurar la activación del usuario tras completar la verificación, es necesario que la URL a la que se redirige al usuario desde el front-end cargue la siguiente estructura:

> [https://signup.getmati.com/?merchantToken=6046cc2a54816f001dedd641&flowId=6046cc2a54816f001bedd640&metadata={"userid":"a955ea0f-34f3-4254-b10a-0181f30729kd"}](https://signup.getmati.com/?merchantToken=6046cc2a54816f001dedd641&flowId=6046cc2a54816f001bedd640&metadata=%7B%22userid%22:%22a955ea0f-34f3-4254-b10a-0181f30729kd%22%7D)

Para información completa sobre KYC, consulta el capítulo correspondiente en esta documentación.

---

## **End User <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/endUsers/](https://api.api-sandbox.paycaddy.dev/v1/endUsers/)

‍La llamada **GET** para un endUser permite conocer los datos almacenados de un userId en particular, especialmente el **walletId** de su wallet inicial y el estado de actividad en el campo **"isActive"**. Ambos datos son cruciales para otras llamadas a la API de NeoBank.

Esta llamada puede usarse para verificar el estado del usuario en cualquier momento del flujo.

=== "Request"
	```
	https://api.api-sandbox.paycaddy.dev/v1/endUsers/${USER_ID}
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
	    "kycUrl": "string",
	    "creationDate": "2023-03-22T19:13:46.178Z"
	}
	```

---

## **Merchant User <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/merchantUsers](https://api.api-sandbox.paycaddy.dev/v1/merchantUsers)

La creación de un nuevo usuario para una **persona jurídica** comienza con una llamada **POST** en la que se consume un endpoint para enviar los datos básicos de la entidad legal:

=== "Request"
	```json
	{
	    "email": "string",
	    "registeredName": "string",
	    "numOfRegistration": "string",
	    "ruc": "string",
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
	    }
	}
	```

=== "Response"
	```json
	{
	    "id": "string",
	    "email": "string",
	    "registeredName": "string",
	    "numOfRegistration": "string",
	    "ruc": "string",
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
	    "creationDate": "2022-07-18T22:29:45.914Z"
	}
	```

Se debe tener en cuenta que la **responsabilidad** de validar la veracidad y el formato de los datos recae en el cliente de PayCaddy, por lo que nuestra API devolverá una respuesta exitosa siempre que se cumplan los siguientes parámetros, sin importar la exactitud de la información ni la duplicidad:

1. Los campos **First Name** y **Last Name** deben sumar un máximo de **22 caracteres** en formato saneado, solo caracteres [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en).
    
2. Ningún campo debe enviarse como **NULL**.
    
3. El campo **email** debe seguir un formato estándar.
    

Adicionalmente, el cliente debe enviar coherentemente el resto de la información:

1. El campo **"RUC"** debe incluir el número del Registro Único de Contribuyentes o su equivalente internacional (VATIN, CUIT, CNPJ, NIT, RUT, etc.).
    
2. El campo **"numOfRegistration"** debe incluir el número de registro de la persona jurídica según la autoridad competente. Puede replicar el RUC si no aplica en tu jurisdicción.
    
3. El campo **"kindOfBusiness"** debe describir brevemente la actividad económica de la entidad. Aunque la API no valida el detalle, es crucial incluir información exacta pues la usará el equipo de compliance.
    

Si el evento no genera errores, el sistema responderá con **HTTP 200 OK**, replicando los datos ingresados y agregando **isActive**, **walletId** y la fecha de creación.

> El control de duplicidad de usuarios debe ser gestionado por los socios de integración. Nuestra API generará múltiples userId distintos aun cuando los datos sean idénticos.

Si se produce un error, el sistema responderá con uno de los siguientes mensajes **400**:

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

=== "Character Limit"
	```json
	{
	    "type": "",
	    "title": "The value provided for the name exceeds the maximum value",
	    "status": 0,
	    "detail": "",
	    "instance": ""
	}
	```

=== "Formato de email"
	```json
	{
	    "type": "",
	    "title": "Invalid format for email",
	    "status": 0,
	    "detail": "",
	    "instance": ""
	}
	```

> En caso de recibir un error **500**, informa al equipo de PayCaddy para que investiguemos y resolvamos el problema con rapidez.

---

## **Merchant User <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/merchantUsers/](https://api.api-sandbox.paycaddy.dev/v1/merchantUsers/)

La llamada **GET** para un merchantUser permite conocer los datos almacenados de un userId específico, en particular el **walletId** de su wallet inicial y el estado **"isActive"**. Ambos datos son esenciales para otras llamadas a la API.

=== "Request"
	```
	https://api.api-sandbox.paycaddy.dev/v1/merchantUsers/{MERCHANT_ID}
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
	    "kycUrl": "string",
	    "creationDate": "2023-03-22T19:13:46.178Z"
	}
	```

---

## **End User SR <font color="green">POST</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs](https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs)

La creación de un nuevo usuario para una **persona natural con flujo KYC delegado** comienza con una llamada **POST** en la que se envían los datos básicos del usuario:

=== "Request"
	```json
	{
	    "email": "string",
	    "firstName": "string",
	    "lastName": "string",
	    "occupation": "string",
	    "placeofwork": "string",
	    "pep": "bool",
	    "salary": "integer",
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
	    "creationDate": "2022-07-13T21:07:21.166Z"
	}
	```

Los campos **"idUrlFront"**, **"idUrlBack"** y **"residenceProofUrl"** deben contener URLs únicas donde se almacenen las imágenes del anverso y reverso de la cédula de identidad del usuario y una prueba de domicilio. Los parámetros específicos para subir estos documentos deben haber sido acordados previamente con el equipo de compliance de PayCaddy.

Los usuarios creados bajo este flujo nacen con estado **activo**, pero podrán ser bloqueados si la información KYC enviada es rechazada por el equipo de compliance.

Requisitos de formato idénticos a los descritos para End User POST:

1. **First Name + Last Name** ≤ 22 caracteres, solo ASCII.
    
2. Ningún campo **NULL**.
    
3. **email** en formato estándar.
    

Además:

1. **"salary"**: salario mensual en USD.
    
2. **"pep"**: indica si la persona es políticamente expuesta (PEP).
    

> El control de duplicidad de usuarios debe ser gestionado por el cliente de PayCaddy. Nuestra API generará múltiples userId distintos incluso si los datos compartidos son idénticos.

Si no hay errores, se devuelve **HTTP 200 OK** con los datos ingresados, **isActive**, **walletId** y la fecha de creación.

---

## **End User SR <font color="sky-blue">GET</font>**

**URL de la solicitud:** [https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs/](https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs/)

La llamada **GET** para un EndUserSR permite consultar los datos almacenados de un userId concreto, el **walletId** inicial y el estado **"isActive"**.

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
	    "creationDate": "2024-11-12T11:57:38.644Z"
	}
	```
