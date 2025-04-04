## **Visión General de los Flujos**

Los flujos para el uso de la API de NeoBank están categorizados según la entidad principal que registran, modifican o consultan. A un alto nivel, los flujos se correlacionan como se explica a continuación:
![entity_diagram](./assets/imgs/flow_entitys.png){class="img img-thin"}

---

## **Flujo de Usuarios**

La creación de nuevos UserIDs en la API de NeoBank sigue dos flujos separados según el tipo de persona que se va a ingresar en el sistema.

EndUser se refiere a los usuarios creados para personas físicas.
MerchantUser se refiere a los usuarios creados para entidades jurídicas.


>Es importante tener en cuenta que hay endpoints separados para la creación de EndUsers y MerchantUsers.

>Durante la exploración inicial, nuestro equipo de ventas debería haberte asignado los detalles específicos de los perfiles de tu programa de tarjetas, lo que definirá qué endpoint(s) debes llamar para la creación de usuarios y las obligaciones KYC pertinentes.

![entity_diagram](./assets/imgs/user_flow.png){class="img"}

---

## **Flujo de Tarjetas**

La API de NeoBank tiene llamadas diferenciadas para la creación de tarjetas de débito, crédito y prepago. A través de estas llamadas, es posible crear una tarjeta física o virtual para un UserID existente vinculado al saldo disponible en un WalletID específico de ese usuario.

Todas las tarjetas se crean utilizando un código de parametrización único para cada producto de tarjeta, que debe ser solicitado previamente al equipo de integración de PayCaddy.

La creación de una tarjeta comienza con la llamada post debitCard POST, creditCard POST o prepaidCard POST, dependiendo del servicio de emisión adquirido.


> Es importante considerar el tipo de usuario para el cual se ha parametrizado la tarjeta. Las tarjetas parametrizadas para personas físicas solo pueden crearse asociándolas con UserIDs que representen EndUsers, mientras que aquellas parametrizadas para entidades jurídicas solo pueden crearse asociándolas con MerchantUsers.


Una vez que se te haya otorgado un código de creación de tarjetas, podrás comenzar a probar la creación de tarjetas en el entorno de pruebas.

```mermaid

flowchart TB
    A([Start]) --> B{clientCode: OK<br>userId: OK<br>isActive: true?}
    B -- "No" --> X([Card cannot be created<br>4XX Error])
    B -- "Yes" --> D{Which endpoint?<br>credit / debit / prepaid}

    D -- "Debit" --> E{Wallet type == 0?}
    D -- "Prepaid" --> E
    D -- "Credit" --> F{Wallet type == 1?}

    E -- "No" --> X
    E -- "Yes" --> G([Invoke Card<br>API call])

    F -- "No" --> X
    F -- "Yes" --> G

    G --> H{API returns<br>200 OK?}
    H -- "No" --> X
    H -- "Yes" --> I[Card Created<br>cardId]

    I --> J{Physical<br>or Virtual?}
    J -- "Virtual" --> K[isActive = true<br>status = Active]
    J -- "Physical" --> L[isActive = false<br>status = PendingAck]

    L --> M([Card in Hands check<br>+ ackReception POST])
    M --> N[Physical Card<br>isActive = true<br>status = Active]


```

---
