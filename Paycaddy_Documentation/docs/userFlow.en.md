This guide explains the logic and API interaction required to create and activate users in the PayCaddy ecosystem. Users can represent either individuals or businesses, and the process varies depending on the type of user and the KYC (Know Your Customer) flow in use.

### User Types

There are **three types** of users in PayCaddy’s system:

| User Type      | Description                            | KYC Flow                                                                |
| -------------- | -------------------------------------- | ----------------------------------------------------------------------- |
| `EndUser`      | A natural person using Integrated KYC. | PayCaddy handles KYC via a hosted link.                                 |
| `EndUserSR`    | A natural person using Delegated KYC.  | The client collects and verifies KYC, then sends verified data via API. |
| `MerchantUser` | A business entity.                     | Delegated KYB (Know Your Business), provided by the client.             |


![entity_diagram](./assets/imgs/user_flow.svg)
{class="img"}

### EndUser (Integrated KYC)

This flow is used when PayCaddy is responsible for handling identity verification via a KYC provider like MetaMap.

**Steps:**

1. **Create the user**
    
    - Payload schema is detailed in [EndUser POST](user.en.md)
        
    - PayCaddy returns a KYC verification session link (e.g., MetaMap URL).
        
2. **User completes KYC**
    
    - The user is redirected or sent the KYC link.
        
    - KYC & AML 3rd party solution (Metamap) handles the identity capture and verification.
        
3. **Verification callback**
    
    - Positive verification status webhook is sent to callback URL.
        
    - The user’s KYC status is updated to `approved`.
        
4. **User becomes active**
    
    - You can confirm the user's activation by querying [EndUser GET](user.en.md#get)

---

### EndUserSR (Delegated KYC)

Used when your system (or your client’s) already handles the KYC process independently and sends verified data to PayCaddy.

**Steps:**

1. **Verify user externally**
    
    - Use your KYC provider to collect documents and validate identity.
        
2. **Create the user**
    
    - Payload schema is detailed in [EndUserSR POST](userSR.en.md)
        
    - Payload must include temporary* URLs to access the document files data.
	    - Details regarding exact document files data capture parameters will be discussed with Compliance Team during onboarding.
        
3. **User becomes active immediately**
    
    - Since verification is delegated, no further KYC is needed from PayCaddy.
    - Periodically KYC information will be required to be updated. Review [Edit User Data](editUserData.en.md)


---

### MerchantUser (Delegated KYB)

This user type represents a business entity. KYB is fully delegated.

**Steps:**

1. **Collect business documents**
    
    - Gather legal documents, beneficial owner info, tax IDs, etc.
        
2. **Create the merchant**
    
    - Schema detailed in [MerchantUser POST](merchant.en.md)
        
3. **CONDITIONAL - Submit for preapproval**
    
    - Some card programs require pre-approval of MerchantUsers prior to their creation.
	    
    - Failure to do so will incur in the blocking of non-approved users.
	    
    - For particular card programs, this condition can be lifted during onboarding definitions.
        
4. **Merchant becomes active**
    
    - Since verification is delegated, no further KYC is needed from PayCaddy.
    - Periodically KYC information will be required to be updated. Review [Edit User Data](editUserData.en.md)


---

###  Summary Table

| User Type      | API Endpoint      | KYC Mode   | Activation Trigger       |
| -------------- | ----------------- | ---------- | ------------------------ |
| `EndUser`      | `POST /users`     | Integrated | MetaMap webhook          |
| `EndUserSR`    | `POST /users`     | Delegated  | Immediate after creation |
| `MerchantUser` | `POST /merchants` | Delegated  | After document approval  |

>During the initial exploration, our sales team should have assigned you the specific details of your card program profiles, which will define which endpoint(s) you should call for user creation and the relevant KYC obligations.


