Bienvenido a la documentación de la API de Paycaddy, una plataforma integral y flexible diseñada para permitir a empresas y desarrolladores integrar fácilmente servicios bancarios y financieros en sus aplicaciones y sistemas.

Nuestra API está construida como una interfaz REST. El principal beneficio es que acepta cuerpos de solicitud codificados en formularios y devuelve respuestas codificadas en JSON, utilizando códigos de respuesta HTTP estándar, lo que debería hacer que nuestra API sea familiar para cualquier persona con experiencia previa en el uso de APIs.

A partir de este punto y utilizando esta documentación como guía, deberías poder integrar y probar llamadas para:

- Crear usuarios de tipo natural y jurídico. (Consulta **[Usuarios](user.es.md)**)
- Crear billeteras para cada usuario que crees. Esta opción será útil si vas a integrar tarjetas de crédito prefinanciadas o si deseas gestionar más de una billetera por usuario. (Consulta **[Billeteras](wallet.es.md)**)
- Gestionar saldos de billeteras creadas a través de PayIns, PayOuts y transferencias entre billeteras. (Consulta **[Operaciones de Billetera](wallet_ops.es.md)**)
- Crear tarjetas vinculadas a las billeteras creadas. (Consulta **[Tarjetas](card.es.md)**)
- Gestionar las tarjetas creadas. (Consulta **[Operaciones de Tarjetas](card_ops.es.md)**)


>Es importante destacar que la creación de tarjetas tiene diferentes endpoints para tarjetas de débito, crédito y prepago.

>En la exploración inicial, nuestro equipo comercial te debe haber asignado los detalles específicos de los perfiles de tu programa de tarjetas, lo que definirá a qué endpoint(s) debes hacer llamadas para la creación de tus tarjetas, así como un código único para tu perfil asignado que debe incluirse en las llamadas de creación de tarjetas.

---

## **Acceso a Sandbox y Autenticación**

Para comenzar un proceso de integración, primero debes solicitar las claves de API de integración contactando (correo electrónico para este propósito). Las claves de API se entregarán por correo electrónico a la persona responsable técnica.

Manejar las claves es de gran importancia y es responsabilidad del receptor de las claves mantenerlas seguras. Las claves secretas no deben compartirse en ningún sitio accesible públicamente como GitHub, código del lado del cliente, etc.

La autenticación a la API debe realizarse mediante la clave X-API. Es necesario proporcionar la clave API como el valor de usuario básico, sin necesidad de proporcionar una contraseña.‍

Todas las llamadas deben realizarse a través de HTTPS; cualquier llamada realizada a través de HTTP o sin autenticación fallará‍.

Con las credenciales de integración, podrás realizar llamadas al entorno de pruebas de la API de NeoBank directamente. Nuestro equipo proporcionará herramientas para probar los endpoints mientras desarrollas tu código hacia nuestros entornos (puedes elegir entre Swagger o Postman).

---

## **Estructura de Entidades**

La API de NeoBank tiene 3 entidades fundamentales con las que interactuarás cada vez que consumas un endpoint para completar los diversos flujos disponibles. Estas entidades identifican a los usuarios finales del servicio de emisión de tarjetas, las billeteras o contenedores virtuales de dinero, y las tarjetas asociadas a ellos.

![entity_diagram](./assets/imgs/entitys.png){class="img"}

**UserID:** ‍Identifica de manera única a un EndUser o MerchantUser indistintamente. El UserID es la entidad principal de la API de NeoBank. El proceso de creación de un EndUser o MerchantUser siempre da como resultado la creación de un UserID y, a su vez, de un WalletID asociado.

‍
‍**WalletID:** El WalletID es el identificador del contenedor de dinero electrónico al cual se acreditan y debitan fondos a través de PayIns, PayOuts y/o transacciones de las tarjetas asociadas a él. Un UserID siempre estará asociado con al menos un WalletID; sin embargo, puede contener múltiples WalletIDs si lo requiere la solución del cliente.
‍

**‍CardID:** ‍Son los identificadores únicos de la tarjeta que también sirven como una abstracción del PAN de las tarjetas emitidas, evitando la necesidad de almacenar información sensible de las tarjetas.

