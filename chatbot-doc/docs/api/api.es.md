# API
## <h2><strong>Intro</strong></h2>

Bienvenido a la documentación de la API de Paycaddy, una plataforma integral y flexible diseñada para permitir a empresas y desarrolladores integrar fácilmente servicios bancarios y financieros en sus aplicaciones y sistemas.

Nuestra API está construida como una interfaz REST. El principal beneficio es que acepta cuerpos de solicitud codificados en formularios y devuelve respuestas codificadas en JSON, utilizando códigos de respuesta HTTP estándar, lo que debería hacer que nuestra API sea familiar para cualquier persona con experiencia previa en el uso de APIs.

A partir de este punto y utilizando esta documentación como guía, deberías poder integrar y probar llamadas para:

- Crear usuarios de tipo natural y jurídico. (Consulta **[Usuarios](#usuarios)**)
- Crear billeteras para cada usuario que crees. Esta opción será útil si vas a integrar tarjetas de crédito prefinanciadas o si deseas gestionar más de una billetera por usuario. (Consulta **[Billeteras](#billeteras)**)
- Gestionar saldos de billeteras creadas a través de PayIns, PayOuts y transferencias entre billeteras. (Consulta **[Operaciones de Billetera](#operaciones-de-billetera)**)
- Crear tarjetas vinculadas a las billeteras creadas. (Consulta **[Tarjetas](#flujo-de-tarjetas)**)
- Gestionar las tarjetas creadas. (Consulta **[Operaciones de Tarjetas](#operaciones-de-tarjeta)**)

<div class="highlighted">
Es importante destacar que la creación de tarjetas tiene diferentes endpoints para tarjetas de débito, crédito y prepago. En la exploración inicial, nuestro equipo comercial te debe haber asignado los detalles específicos de los perfiles de tu programa de tarjetas, lo que definirá a qué endpoint(s) debes hacer llamadas para la creación de tus tarjetas, así como un código único para tu perfil asignado que debe incluirse en las llamadas de creación de tarjetas.
</div>
---
## <h2><strong>Acceso a Sandbox y Autenticación</strong></h2>
Para comenzar un proceso de integración, primero debes solicitar las claves de API de integración contactando (correo electrónico para este propósito). Las claves de API se entregarán por correo electrónico a la persona responsable técnica.

Manejar las claves es de gran importancia y es responsabilidad del receptor de las claves mantenerlas seguras. Las claves secretas no deben compartirse en ningún sitio accesible públicamente como GitHub, código del lado del cliente, etc.

La autenticación a la API debe realizarse mediante la clave X-API. Es necesario proporcionar la clave API como el valor de usuario básico, sin necesidad de proporcionar una contraseña.‍

Todas las llamadas deben realizarse a través de HTTPS; cualquier llamada realizada a través de HTTP o sin autenticación fallará‍.

Con las credenciales de integración, podrás realizar llamadas al entorno de pruebas de la API de NeoBank directamente. Nuestro equipo proporcionará herramientas para probar los endpoints mientras desarrollas tu código hacia nuestros entornos (puedes elegir entre Swagger o Postman).

----
## <h2><strong>Estructura de Entidades</strong></h2>

La API de NeoBank tiene 3 entidades fundamentales con las que interactuarás cada vez que consumas un endpoint para completar los diversos flujos disponibles. Estas entidades identifican a los usuarios finales del servicio de emisión de tarjetas, las billeteras o contenedores virtuales de dinero, y las tarjetas asociadas a ellos.

![entity_diagram](../assets/imgs/entitys.png){class="img"}

**UserID:** ‍Identifica de manera única a un EndUser o MerchantUser indistintamente. El UserID es la entidad principal de la API de NeoBank. El proceso de creación de un EndUser o MerchantUser siempre da como resultado la creación de un UserID y, a su vez, de un WalletID asociado.

‍
‍**WalletID:** El WalletID es el identificador del contenedor de dinero electrónico al cual se acreditan y debitan fondos a través de PayIns, PayOuts y/o transacciones de las tarjetas asociadas a él. Un UserID siempre estará asociado con al menos un WalletID; sin embargo, puede contener múltiples WalletIDs si lo requiere la solución del cliente.
‍

**‍CardID:** ‍Son los identificadores únicos de la tarjeta que también sirven como una abstracción del PAN de las tarjetas emitidas, evitando la necesidad de almacenar información sensible de las tarjetas.

---

## <h2><strong>Visión General de los Flujos</strong></h2>

Los flujos para el uso de la API de NeoBank están categorizados según la entidad principal que registran, modifican o consultan. A un alto nivel, los flujos se correlacionan como se explica a continuación:
![entity_diagram](../assets/imgs/flow_entitys.png){class="img img-thin"}

---

## <h2><strong>Flujo de Usuarios</strong></h2>

La creación de nuevos UserIDs en la API de NeoBank sigue dos flujos separados según el tipo de persona que se va a ingresar en el sistema.

EndUser se refiere a los usuarios creados para personas físicas.  
MerchantUser se refiere a los usuarios creados para entidades jurídicas.

<div class="highlighted">
Es importante tener en cuenta que hay endpoints separados para la creación de EndUsers y MerchantUsers.
<br><br>
Durante la exploración inicial, nuestro equipo de ventas debería haberte asignado los detalles específicos de los perfiles de tu programa de tarjetas, lo que definirá qué endpoint(s) debes llamar para la creación de usuarios y las obligaciones KYC pertinentes.
</div>

![entity_diagram](../assets/imgs/user_flow.png){class="img"}

---

## <h2><strong>Flujo de Tarjetas</strong></h2>
La API de NeoBank tiene llamadas diferenciadas para la creación de tarjetas de débito, crédito y prepago. A través de estas llamadas, es posible crear una tarjeta física o virtual para un UserID existente vinculado al saldo disponible en un WalletID específico de ese usuario.

Todas las tarjetas se crean utilizando un código de parametrización único para cada producto de tarjeta, que debe ser solicitado previamente al equipo de integración de PayCaddy.

La creación de una tarjeta comienza con la llamada post debitCard POST, creditCard POST o prepaidCard POST, dependiendo del servicio de emisión adquirido.

<div class="highlighted">
Es importante considerar el tipo de usuario para el cual se ha parametrizado la tarjeta. Las tarjetas parametrizadas para personas físicas solo pueden crearse asociándolas con UserIDs que representen EndUsers, mientras que aquellas parametrizadas para entidades jurídicas solo pueden crearse asociándolas con MerchantUsers.
</div>

Una vez que se te haya otorgado un código de creación de tarjetas, podrás comenzar a probar la creación de tarjetas en el entorno de pruebas.

![entity_diagram](../assets/imgs/card_flow.png)

---
## <h2><strong>Flujo de Crédito</strong></h2>
Para los programas de emisión de tarjetas de crédito que utilizan líneas de crédito, se usarán los endpoints a continuación para gestionar los fondos disponibles en las billeteras asociadas a cada línea de crédito.

<div class="highlighted">
Si las tarjetas son de crédito prepago, por favor consulta el endpoint <strong>wallet POST</strong> para crear billeteras que tengan la capacidad de crear tarjetas de crédito.
</div>
La operación de crédito sigue el flujo a continuación y utiliza los endpoints detallados en esta sección.

![entity_diagram](../assets/imgs/credit_flow.png){class = ".img"}

## <h2><strong>Operaciones de Billetera</strong></h2>
## <h2><strong>Operaciones de Tarjeta</strong></h2>