Welcome to PayCaddy's API documentation, a comprehensive and flexible platform designed to enable businesses and developers to easily integrate banking and financial services into their applications and systems.

Our API is built as a REST interface. The main benefit is that it accepts form-encoded request bodies and returns JSON-encoded responses, using standard HTTP response codes which should make our API familiar to anyone with previous experience using APIs.

Use this documentation as a guide to execute tasks such as:

- Creating natural and legal type users. (Check **[Users](user.en.md)**)
- Creating wallets for each user you create. This option will be useful if you are going to integrate pre-funded credit type cards or if you want to manage more than one wallet per user. (Check **[Wallets](wallet.en.md)**)
- Managing balances of wallets created through PayIns, PayOuts, and Transfers between wallets. (Check **[Wallet Operations](wallet_ops.en.md)**)
- Creating cards linked to the created wallets. (Check **[Cards](card.en.md)**)
- Managing created cards. (Check **[Card Operations](card_ops.en.md)**)
- Registering a Callback URLs for event notifications (Check [**Notifications Enlist**](notificationsEnlist.en.md))
- Retrieving transactional data. (Check [**Transaction Data Retrieval**](transactionLists.en))


> Card creation is done through different endpoints for debit, credit, and prepaid cards.

> In the initial product exploration, our commercial team will gather the specifications of the *card product* to be issued. Our integration team will provide a related `clientCode` for each *card product* required, this is a unique code for your assigned card profile that should be included in the card creation calls.

---

## **Sandbox Access and Authentication**

To start an integration process, you must first request integration API Keys by contacting info@paycaddy.com. After initial scoping and onboarding processes, sandbox API keys will be delivered via email to the technical responsible party.

Handling the keys is of great importance and it is the responsibility of the key recipient to keep them safe. Secret keys must not be shared on any publicly accessible site such as GitHub, client-side code, etc.

Authentication to the API should be performed via X-API Key. It is necessary to provide the API Key as the basic user value, without the need to provide a password.‍

All calls must be made via HTTPS; any call made via HTTP or without authentication will fail‍.

With the integration credentials, you will be able to make calls to PayCaddy API testing environment directly.

Utilize our [Sandbox Swagger Interface](https://api.api-sandbox.paycaddy.dev/openapi/index.html) to test each of the endpoints of our API as you develop your code.

----
## **Entity Structure**

PayCaddy API has 3 fundamental entities with which you will interact each time you consume an endpoint to complete the various available flows. These entities identify the end users of the card issuance service, the wallets or virtual containers for money, and the cards associated with them.

![entity_diagram](./assets/imgs/entityRelationship.svg)
{class="img"}

- **UserID:** ‍Uniquely identifies an EndUser or MerchantUser interchangeably. The UserID is the primary entity of the NeoBank API. The process of creating an EndUser or MerchantUser always results in the creation of a UserID and, in turn, an associated WalletID.

- **WalletID:** The WalletID is the identifier of the electronic money container to which funds are credited and debited through PayIns, PayOuts, and/or transactions of the card(s) associated with it. A UserID will always be associated with at least one WalletID; however, it may contain multiple WalletIDs if required by the client's solution.

- **‍CardID:** ‍These are the unique identifiers of the card that also serve as an abstraction of the PAN of the issued cards, avoiding the need for storing sensitive card information.