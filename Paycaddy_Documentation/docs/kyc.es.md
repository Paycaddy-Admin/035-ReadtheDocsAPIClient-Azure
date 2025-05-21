## **Notificaciones de KYC Integrado**

Como parte de nuestro **flujo de KYC Integrado** para Personas Naturales, después de que los usuarios completen los pasos de verificación de identidad en la URL suministrada en el campo **kycURL** (consulta **EndUser POST**), el proveedor procesa los datos, los contrasta con listas negras y realiza chequeos AML. PayCaddy recibe ese resultado y envía un webhook con el estado.

Para recibirlos, expón una URL y pídele al equipo de integración que la asocie a tu proyecto.

### **Webhook de Verificación KYC**

|Campo|Descripción|
|---|---|
|`metadata`|Identificador del usuario (`userId`)|
|`status`|`"verified"`, `"rejected"` o `"reviewNeeded"`|
|`description`|Detalle del estado|
|`fullName`|Nombre completo del usuario|
|`age`|Edad del usuario|
|`timeStamp`|Marca ISO 8601 de la respuesta|

```json
{
    "metadata": { "userId": "string" },
    "status": "string",
    "description": "string",
    "fullName": "string",
    "age": "string",
    "timeStamp": "2023-03-16T09:42:18.8338086Z"
}
```

### **Estados y Descripciones de Verificación**

El flujo de un proceso de validación sigue el diagrama descrito a continuación.

![kycflow](assets/imgs/kycflow.svg)
{class="img"}

### Estados posibles
A continuación se muestran los webhooks correspondientes a cada estado, con sus descripciones y, en caso de rechazo, los motivos específicos.

**‍Verification Inputs Completed:** Este estado muestra que el usuario ha completado exitosamente la captura de datos de KYC a través del **kycURL** y tendrá la estructura y  información mostrada debajo.
```json
	{
	    "metadata": { "userId": "unique user identifier" },
	    "status": "verification_inputs_completed",
	    "description": "Ongoing verification",
	    "fullName": "Document OCR capture ongoing",
	    "age": "Document OCR capture ongoing",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

**Verified:** Este estado indica que el usuario ha completado exitosamente la  verificación de KYC y cuenta con los datos y estructura necesarios. Esto también significa que el usuario está activado en nuestra base de datos y puede realizar otros procesos de creación de tarjetas y operaciones adicionales. La respuesta del webhook incluirá la estructura y información requerida.

```json
	{
	    "metadata": { "userId": "ID único del usuario" },
	    "status": "verified",
	    "description": "Verification signed",
	    "fullName": "User's full name",
	    "age": "User's age",
	    "timeStamp": "ISO 8601 timestamp"
	}
	```


**Rejected:** Este estado se refiere a cuando el usuario no ha pasado la verificación de KYC. Los casos se detallan abajo.

=== "Input mismatch"
	```
	{
	    "metadata": { "userId": "string" },
	    "status": "rejected",
	    "description": "Document doesn’t match input data",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "AML Checks Rejected"
	```
	{
	    "metadata": { "userId": "string" },
	    "status": "rejected",
	    "description": "AML checks rejected",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "User underage"
	```
	{
	    "metadata": { "userId": "string" },
	    "status": "rejected",
	    "description": "The user is underage",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "Document Expired"
	```
	{
	    "metadata": { "userId": "string" },
	    "status": "rejected",
	    "description": "The document uploaded is expired",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "Negligence"
	```
	{
	    "metadata": { "userId": "string" },
	    "status": "rejected",
	    "description": "The document uploaded has issues",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "Others"
	```
	{
	    "metadata": { "userId": "string" },
	    "status": "rejected",
	    "description": "Validation Failed. User cannot be verified",
	    "fullName": "User's complete name",
	    "age": "null",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

**ReviewNeeded:** Este estado se refiere a cuando un usuario no ha pasado correctamente el proceso de verificación KYC y requiere una revisión manual para su posible activación. Los distintos casos se detallan abajo

=== "Negligence"
	```json
	{
	    "metadata": { "userId": "string" },
	    "status": "reviewNeeded",
	    "description": "The document uploaded has issues",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "AML Checks Review"
	```json
	{
	    "metadata": { "userId": "string" },
	    "status": "reviewNeeded",
	    "description": "AML checks need to be reviewed",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "System Inoperative"
	```json
	{
	    "metadata": { "userId": "string" },
	    "status": "reviewNeeded",
	    "description": "Government validation system inoperative",
	    "fullName": "User's complete name",
	    "age": "User's age",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```

=== "Others"
	```json
	{
	    "metadata": { "userId": "string" },
	    "status": "reviewNeeded",
	    "description": "Validation failed. Pending compliance review",
	    "fullName": "User's complete name",
	    "age": "null",
	    "timeStamp": "2023-03-16T09:42:18.8338086Z"
	}
	```


---
