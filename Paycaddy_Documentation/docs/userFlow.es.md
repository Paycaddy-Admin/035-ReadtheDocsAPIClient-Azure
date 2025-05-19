Esta guía explica la lógica y la interacción con la API necesarias para crear y activar usuarios en el ecosistema de PayCaddy. Los usuarios pueden representar tanto personas naturales como empresas, y el proceso varía según el tipo de usuario y el flujo [KYC (_Know Your Customer_)](./kyc.es.md) empleado.

### Tipos de usuario

En el sistema de PayCaddy existen **tres tipos** de usuarios:

| Tipo de usuario | Descripción                            | Flujo KYC                                                                         |
| --------------- | -------------------------------------- | --------------------------------------------------------------------------------- |
| `EndUser`       | Persona natural que usa KYC Integrado. | PayCaddy gestiona el KYC mediante un enlace _hosted_.                             |
| `EndUserSR`     | Persona natural con KYC Delegado.      | El cliente recopila y verifica el KYC, luego envía los datos verificados vía API. |
| `MerchantUser`  | Entidad empresarial.                   | KYB (_Know Your Business_) Delegado, proporcionado por el cliente.                |

Para determinar el endpoint correcto en un escenario particular, consulta el siguiente diagrama de flujo. Para información detallada de cada flujo específico, revisa la sección correspondiente en este capítulo.

![general_user_flow](./assets/imgs/generaluserflow.svg)  
{class="img"}

### EndUser (KYC Integrado)

Este flujo se utiliza cuando PayCaddy se encarga de la verificación de identidad mediante nuestro proveedor externo de KYC.

**Pasos:**

1. **Crear el usuario**
    
    - El esquema del payload se detalla en [EndUser POST](user.es.md).
        
    - PayCaddy devuelve un enlace de sesión de verificación KYC (por ejemplo, URL de MetaMap).
        
2. **El usuario completa el KYC**
    
    - El usuario es redirigido o recibe el enlace KYC.
        
    - La solución externa de KYC & AML (Metamap) realiza la captura y verificación de identidad.
        
3. **Callback de verificación**
    
    - Se envía un webhook con estado positivo de verificación a la Callback URL.
        
    - El estado KYC del usuario se actualiza a `approved`.
        
4. **El usuario queda activo**
    
    - Puedes confirmar la activación consultando [EndUser GET](user.es.md#end-user-get).
        

A continuación se muestra un diagrama de este flujo:

![enduserflow](./assets/imgs/enduser.svg)

> *** Con base en el webhook de notificación KYC, puedes decidir ofrecer un reintento a los usuarios según tu UX/UI. Para información de precios sobre verificaciones KYC integradas, contacta a nuestro equipo comercial.

---

### EndUserSR (KYC Delegado)

Se usa cuando tu sistema (o el de tu cliente) ya gestiona el proceso KYC de forma independiente y envía los datos verificados a PayCaddy.

> Esta opción está reservada para **entidades reguladas** en su país de jurisdicción.  
> El acceso para crear este tipo de usuario debe ser solicitado y aprobado por el equipo de compliance de PayCaddy durante la etapa de onboarding o ciclos de mejora.

**Pasos:**

1. **Verificar al usuario externamente**
    
    - Utiliza tu proveedor de KYC para recopilar documentos y validar la identidad.
        
2. **Crear el usuario**
    
    - El esquema del payload se detalla en [EndUserSR POST](user.es.md#end-user-sr-post).
        
    - El payload debe incluir URLs _temporales_ para acceder a los archivos de documentos.
        
        - Los parámetros exactos para la captura de documentos se definirán con el equipo de Compliance en el onboarding.
            
3. **El usuario queda activo inmediatamente**
    
    - Al ser verificación delegada, PayCaddy no realiza más KYC.
        
    - Periódicamente se requerirá actualización de la información KYC. Revisa [Edit User Data](editUserData.es.md).
        

Diagrama de este flujo:

![endusersrflow](./assets/imgs/EndUserSR.svg)

> ** Para detalles de periodicidad en controles KYC programados, consulta con el equipo de compliance de PayCaddy durante la integración.

---

### MerchantUser (KYB Delegado)

Este tipo de usuario representa una entidad empresarial. El KYB es completamente delegado.

**Pasos:**

1. **Recopilar documentos de la empresa**
    
    - Reúne documentos legales, datos de beneficiarios finales, identificaciones fiscales, etc.
        
2. **CONDICIONAL – Enviar para preaprobación*** ***
    
    - Algunos programas de tarjeta requieren la preaprobación de _MerchantUsers_ antes de su creación.
        
    - No hacerlo puede bloquear usuarios no aprobados en controles programados.
        
    - Para ciertos programas, esta condición puede eliminarse durante la definición en onboarding.
        
3. **Crear el comercio**
    
    - Esquema detallado en [MerchantUser POST](user.es.md#merchant-user-post).
        
4. **El comercio queda activo**
    
    - Al ser verificación delegada, PayCaddy no realiza más KYC.
        
    - Periódicamente se requerirá actualización de la información KYC. Revisa [Edit User Data](editUserData.es.md).
        

> - Para detalles de qué verificaciones se realizan, consúltalo con el equipo de Compliance en el onboarding.  
>     ** Para periodicidad específica de verificaciones KYC programadas, contacta al equipo de compliance de PayCaddy.  
>     *** Todos los programas de tarjeta para usuarios Merchant se definirán con el equipo de compliance de PayCaddy para establecer formatos de archivo y condiciones de preaprobación cuando corresponda.
>     

Diagrama de este flujo:

![merchantuserflow](./assets/imgs/merchantuser.svg)

---

### Tabla resumen

| Tipo de usuario | Endpoint API                                       | Modalidad KYC | Disparador de activación                                     |
| --------------- | -------------------------------------------------- | ------------- | ------------------------------------------------------------ |
| `EndUser`       | [EndUser POST](user.es.md#end-user-post)           | Integrado     | Webhook de MetaMap                                           |
| `EndUserSR`     | [EndUserSR POST](user.es.md#end-user-sr-post)      | Delegado      | Inmediato tras la creación                                   |
| `MerchantUser`  | [MerchantUser POST](user.es.md#merchant-user-post) | Delegado      | Inmediato tras la creación, con pre-aprobación de documentos |

> Durante la fase inicial, nuestro equipo de ventas debió asignarte los perfiles específicos de tu programa de tarjeta, lo que definirá qué endpoint(s) debes llamar para la creación de usuarios y las obligaciones KYC correspondientes.