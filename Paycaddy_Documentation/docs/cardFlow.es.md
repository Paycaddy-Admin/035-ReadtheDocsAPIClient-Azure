La API de PayCaddy cuenta con llamadas diferenciadas para la creación de tarjetas débito, crédito y prepago. A través de estas llamadas es posible crear una tarjeta física o virtual para un **UserID** existente vinculada al balance disponible en un **WalletID** específico de ese usuario.

Todas las tarjetas se crean utilizando un código de parametrización único para cada producto de tarjeta, el cual debe solicitarse previamente al equipo de integración de PayCaddy.

La creación de una tarjeta inicia con la llamada POST [Debit Card POST](card.es.md#debit-card-post), [Credit Card POST](card.es.md#credit-card-post) o [Prepaid Card POST](card.es.md#prepaid-card-post), según el servicio de emisión contratado.

> Es importante considerar el tipo de usuario para el cual se ha parametrizado la tarjeta. Las tarjetas parametrizadas para personas naturales solo pueden crearse asociándolas a UserIDs que representen EndUsers o EndUserSRs, mientras que aquellas parametrizadas para personas jurídicas solo pueden crearse asociándolas a MerchantUsers.

Una vez que hayas recibido un código de creación de tarjeta, puedes comenzar a probar la creación de tarjetas en el entorno de pruebas.

![cardflow](./assets/imgs/cardFlow.svg)

> Las tarjetas físicas se crean con un estado predeterminado de `PendingAck` por motivos de seguridad durante la logística de entrega. Consulta [Ack Reception](card_ops.es.md#ack-reception-post) para detalles sobre la activación de tarjetas físicas.