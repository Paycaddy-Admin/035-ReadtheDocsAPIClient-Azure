Bienvenido a la documentación de la API de PayCaddy, una plataforma completa y flexible diseñada para permitir que empresas y desarrolladores integren fácilmente servicios bancarios y financieros en sus aplicaciones y sistemas.

Nuestra API está construida como una interfaz REST. El principal beneficio es que acepta cuerpos de solicitud codificados como formularios y devuelve respuestas codificadas en JSON, utilizando códigos de respuesta HTTP estándar, lo que debería hacer que nuestra API resulte familiar para cualquiera con experiencia previa en el uso de APIs.

Utiliza esta documentación como guía para ejecutar tareas tales como:

- Crear usuarios de tipo natural y jurídico. (Consulta **[Usuarios](user.es.md)**)  
- Crear wallets para cada usuario que crees. Esta opción será útil si vas a integrar tarjetas de crédito prefundadas o si deseas gestionar más de una wallet por usuario. (Consulta **[Wallets](wallet.es.md)**)  
- Gestionar los saldos de las wallets creadas a través de PayIns, PayOuts y transferencias entre wallets. (Consulta **[Operaciones de Wallet](wallet_ops.es.md)**)  
- Crear tarjetas vinculadas a las wallets creadas. (Consulta **[Tarjetas](card.es.md)**)  
- Gestionar las tarjetas creadas. (Consulta **[Operaciones de Tarjetas](card_ops.es.md)**)  
- Registrar URLs de Callback para notificaciones de eventos (Consulta [**Enlistado de Notificaciones**](notificationsEnlist.es.md))  
- Recuperar datos transaccionales. (Consulta [**Obtención de Datos de Transacción**](transactionLists.es.md))

> La creación de tarjetas se realiza mediante distintos endpoints para tarjetas débito, crédito y prepago.

> Durante la exploración inicial del producto, nuestro equipo comercial recopilará las especificaciones del *producto de tarjeta* que se emitirá. Nuestro equipo de integración proporcionará un `clientCode` relacionado para cada *producto de tarjeta* requerido; este es un código único para el perfil de tarjeta asignado que debe incluirse en las llamadas de creación de tarjetas.

---

## **Acceso a Sandbox y Autenticación**

Para iniciar un proceso de integración, primero debes solicitar las API Keys de integración contactando a info@paycaddy.com. Después de la fase inicial de alcance y procesos de incorporación, las API Keys de sandbox se enviarán por correo electrónico a la persona técnica responsable.

El manejo de las claves es de gran importancia y es responsabilidad del receptor mantenerlas seguras. Las claves secretas no deben compartirse en ningún sitio de acceso público como GitHub, código del lado del cliente, etc.

La autenticación en la API debe realizarse mediante X-API Key. Es necesario proporcionar la API Key como el valor de usuario básico, sin necesidad de proporcionar una contraseña.

Todas las llamadas deben realizarse a través de HTTPS; cualquier llamada hecha vía HTTP o sin autenticación fallará.

Con las credenciales de integración, podrás realizar llamadas directamente al entorno de pruebas (testing) de la API de PayCaddy.

Utiliza nuestra [Interfaz Swagger del Sandbox](https://api.api-sandbox.paycaddy.dev/openapi/index.html) para probar cada uno de los endpoints de nuestra API mientras desarrollas tu código.

----

## **Estructura de Entidades**

La API de PayCaddy cuenta con 3 entidades fundamentales con las que interactuarás cada vez que consumas un endpoint para completar los distintos flujos disponibles. Estas entidades identifican a los usuarios finales del servicio de emisión de tarjetas, las wallets o contenedores virtuales de dinero y las tarjetas asociadas a ellos.

![entity_diagram](./assets/imgs/entityRelationship.svg)
{class="img"}

- **UserID:** Identifica de forma única a un EndUser o MerchantUser indistintamente. El UserID es la entidad primaria de la API de NeoBank. El proceso de creación de un EndUser o MerchantUser siempre genera la creación de un UserID y, a su vez, de un WalletID asociado.

- **WalletID:** El WalletID es el identificador del contenedor de dinero electrónico al que se acreditan y debitan fondos mediante PayIns, PayOuts y/o transacciones de la(s) tarjeta(s) asociada(s). Un UserID siempre estará asociado a al menos un WalletID; sin embargo, puede contener múltiples WalletIDs si así lo requiere la solución del cliente.

- **CardID:** Son los identificadores únicos de la tarjeta que también sirven como una abstracción del PAN de las tarjetas emitidas, evitando la necesidad de almacenar información sensible de la tarjeta.
