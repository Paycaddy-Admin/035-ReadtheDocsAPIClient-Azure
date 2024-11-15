## **End User <font color="green">POST</font>**

**URL:** https://api.paycaddy.dev/v1/endUsers

‍La creación de un nuevo usuario para una persona física comienza con una llamada POST en la que se consume un endpoint para enviar la información básica del usuario:

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

Cabe señalar que la responsabilidad de validar la exactitud y el formato de los datos ingresados recae sobre el cliente de PayCaddy, lo que significa que nuestra API devolverá una respuesta exitosa siempre que se cumplan los siguientes parámetros, independientemente de la precisión de la información o duplicación de los datos compartidos:

1. Los campos de Nombre y Apellido deben sumar un total máximo de 22 caracteres y deben enviarse en un formato depurado, eliminando caracteres especiales y limitándose al rango ASCII.
2. Ninguno de los campos debe enviarse como NULL.
3. El campo de correo electrónico debe seguir un formato estándar de correo electrónico.


Además de las verificaciones de formato, es importante destacar la responsabilidad del cliente de enviar consistentemente el resto de los campos con la información correcta:

- El campo **"salario"** debe incluir el salario mensual del usuario en USD.
- El campo **"pep"** indica si la persona natural para la que se está creando el usuario es una persona políticamente expuesta.
- **PEP:** Alguien que ha sido confiado con una responsabilidad pública prominente. Típicamente, una PEP presenta un mayor riesgo de posible implicación en soborno y corrupción debido a su posición e influencia.

>El control sobre la duplicación de usuarios debe ser gestionado por el cliente de PayCaddy. Nuestra API generará múltiples **userIds** distintos, independientemente de si los datos compartidos son idénticos en llamadas duplicadas.

Si este evento no genera errores, el sistema responderá con un mensaje **HTTP 200 OK**. La respuesta exitosa replica los datos ingresados y agrega información sobre los parámetros **isActive** y el **walletId** inicial asociado con el userId, así como una marca de tiempo de la fecha de creación de dicho usuario.

Si ocurre un error, el sistema responderá con uno de los siguientes mensajes de error con código de estado 400:


=== "Campos pendientes"
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
=== "Limite de caracteres"
    ```json
    {
        "type": "",
        "title": "The value provided for the name exceeds the maximum value",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```
=== "Formato de correo electronico"
    ```json
    {
        "type": "",
        "title": "Invalid format for email",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```

> En caso de que encuentres un mensaje de error 500, esto podría indicar un error interno del servidor en el sistema. Si te enfrentas a esta situación, te pedimos que informes al equipo de PayCaddy para que podamos investigar y resolver el problema de manera eficiente.

Una vez que se hayan asignado los identificadores de usuario y billetera, el usuario final deberá completar la verificación KYC para finalizar el procedimiento de creación del perfil. Hasta que se complete esta verificación, el campo de control "isActive" permanecerá asignado como falso, deshabilitando cualquier procedimiento.

En la respuesta de creación de usuario (EndUser POST), se presenta un enlace a la validación de identidad asociada con el userId en Metamap en el campo **kycUrl**, donde se presentarán las instrucciones y los pasos que el usuario final debe seguir para completar la verificación.

```json
    {
        "kycUrl": "https%3a%2f%2fsignup.getmati.com%2f%3fmerchantToken%
                  3d6046cc2a54816f001bedd641%26flowId%3d6046cc2a54816f0
                01bedd640%26metadata%3d%7b%22userid%22%3a%226955ea0f4
                f3-4254-b10a-0181f307298c%22%7d"
    }
```

Es importante considerar que el kycURL se comparte aplicando codificación de URL, lo que permite el envío de metadatos (userId) que vinculan la validación con el usuario creado. Para asegurar la activación del usuario al completar exitosamente la verificación, es necesario que la URL a la cual se redirige al usuario desde la interfaz front-end cargue la siguiente estructura:

>https://signup.getmati.com/?merchantToken=6046cc2a54816f001dedd641&
flowId=6046cc2a54816f001bedd640&metadata={"userid":"a955ea0f-34f3-4254-b10a-0181f30729kd"}

Para obtener información completa sobre KYC, puedes consultar el capítulo detallado de KYC en esta documentación.

---

## **End User <font color="sky-blue">GET</font>**

**URL:** https://api.paycaddy.dev/v1/endUsers/

El metodo GET para un endUser permite conocer los datos almacenados de un userId concreto, especialmente el walletId de su monedero inicial y el estado de actividad de este usuario en el campo "isActive". Ambos datos son cruciales para las demás llamadas a la API de NeoBank.

Esta llamada puede utilizarse para verificar el estado del usuario en cualquier punto del flujo.


=== "Request"
    ```
     https://api.paycaddy.dev/v1/endUsers/${USER_ID}
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

**URL:** https://api.paycaddy.dev/v1/merchantUsers

La creación de un nuevo usuario para una persona jurídica comienza con una llamada POST en la que se consume un endpoint para el envío de los datos básicos de la persona jurídica.


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

Cabe señalar que la responsabilidad de validar la exactitud y el formato de los datos ingresados recae sobre el cliente de PayCaddy, lo que significa que nuestra API devolverá una respuesta exitosa siempre que se cumplan los siguientes parámetros, independientemente de la precisión de la información o duplicación de los datos compartidos:

1. Los campos de Nombre y Apellido deben sumar un total máximo de 22 caracteres y deben enviarse en un formato depurado, eliminando caracteres especiales y limitándose al rango ASCII.
2. Ninguno de los campos debe enviarse como NULL.
3. El campo de correo electrónico debe seguir un formato estándar de correo electrónico.


Además de las verificaciones de formato, es importante destacar la responsabilidad del cliente de enviar consistentemente el resto de los campos con la información correcta:

- El campo **"salario"** debe incluir el salario mensual del usuario en USD.
- El campo **"pep"** indica si la persona natural para la que se está creando el usuario es una persona políticamente expuesta.
- **PEP:** Alguien que ha sido confiado con una responsabilidad pública prominente. Típicamente, una PEP presenta un mayor riesgo de posible implicación en soborno y corrupción debido a su posición e influencia.

>El control sobre la duplicación de usuarios debe ser gestionado por el cliente de PayCaddy. Nuestra API generará múltiples **userIds** distintos, independientemente de si los datos compartidos son idénticos en llamadas duplicadas.

Si este evento no genera errores, el sistema responderá con un mensaje **HTTP 200 OK**. La respuesta exitosa replica los datos ingresados y agrega información sobre los parámetros **isActive** y el **walletId** inicial asociado con el userId, así como una marca de tiempo de la fecha de creación de dicho usuario.

Si ocurre un error, el sistema responderá con uno de los siguientes mensajes de error con código de estado 400:

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
=== "Email Format"
    ```json
    {
        "type": "",
        "title": "Invalid format for email",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```

> En caso de que encuentres un mensaje de error 500, esto podría indicar un error interno del servidor en el sistema. Si te enfrentas a esta situación, te pedimos que informes al equipo de PayCaddy para que podamos investigar y resolver el problema de manera eficiente.

---

## **Merchant User <font color="sky-blue">GET</font>**

**URL:** https://api.paycaddy.dev/v1/merchantUsers/

El metodo GET para un merchantUser permite conocer los datos almacenados de un userId concreto, especialmente el walletId de su monedero inicial y el estado de actividad de este usuario en el campo «isActive». Ambos datos son cruciales para las demás llamadas a la API de NeoBank.

Esta llamada puede utilizarse para verificar el estado del usuario en cualquier punto del flujo.

=== "Request"
    ```
     https://api.paycaddy.dev/v1/merchantUsers/{MERCHANT_ID}
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

**URL:**  https://api.paycaddy.dev/v1/SR/endUserSRs/

La creación de un nuevo usuario para una persona física con un flujo KYC delegado comienza con una llamada POST en la que se consume un endpoint para el envío de los datos básicos del usuario:


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
            "country": "string",
        }
        "idUrlFront": "string",
        "idUrlBack": "string",
        "residenceProofUrl": "string",
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
            "homeNumber" : "string",
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

Los campos "idUrlFront", "idUrlBack" y "residenceProofUrl" deben enviarse con URL únicas donde se almacenan las imágenes del anverso y reverso del documento nacional de identidad del usuario y una prueba de residencia. Los parámetros específicos para subir estos documentos deben haber sido acordados previamente con el departamento de cumplimiento de PayCaddy.

Los usuarios creados bajo este flujo nacen con un estado activo pero están sujetos a bloqueos en caso de que la información KYC enviada en la llamada de creación sea rechazada por el equipo de cumplimiento de PayCaddy.

Cabe señalar que la responsabilidad de validar la exactitud y el formato de los datos introducidos recae en el cliente de PayCaddy, lo que significa que nuestra API devolverá una respuesta correcta siempre y cuando se cumplan los siguientes parámetros, independientemente de la exactitud de la información o la duplicación de los datos compartidos:

1. Los campos Nombre y Apellidos deben sumar un total máximo de 22 caracteres que deben ser enviados de forma saneada, eliminando caracteres especiales y limitándose al rango ASCII.
2. Ninguno de los campos debe enviarse como NULL.
3. El campo email debe seguir un formato de email estándar.

Además de las verificaciones de formato, es importante destacar la responsabilidad del cliente de enviar consistentemente el resto de los campos con la información correcta:

- El campo **"salario"** debe incluir el salario mensual del usuario en USD.
- El campo **"pep"** indica si la persona natural para la que se está creando el usuario es una persona políticamente expuesta.
- **PEP:** Alguien que ha sido confiado con una responsabilidad pública prominente. Típicamente, una PEP presenta un mayor riesgo de posible implicación en soborno y corrupción debido a su posición e influencia.

>El control sobre la duplicación de usuarios debe ser gestionado por el cliente de PayCaddy. Nuestra API generará múltiples **userIds** distintos, independientemente de si los datos compartidos son idénticos en llamadas duplicadas.

Si este evento no genera errores, el sistema responderá con un mensaje HTTP 200 OK. La respuesta correcta replica los datos introducidos y añade información sobre los parámetros **isActive** y el **walletId** inicial asociado al **userId**, además de cargar una marca de tiempo de la fecha de creación de dicho usuario.

---

## **End User SR <font color="sky-blue">GET</font>** 

**URL:** https://api.paycaddy.dev/v1/SR/endUserSRs/

El metodo GET a un EndUserSR permite conocer los datos almacenados de un userId concreto, especialmente el walletId de su monedero inicial y el estado de actividad de este usuario en el campo «isActive». Ambos datos son cruciales para las demás llamadas a la API de NeoBank.


=== "Request"
    ```
     https://api.paycaddy.dev/v1/SR/EndUserSRs/{USER_ID}
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