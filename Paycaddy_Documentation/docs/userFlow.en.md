This guide explains the logic and API interaction required to create and activate users in the PayCaddy ecosystem. Users can represent either individuals or businesses, and the process varies depending on the type of user and the KYC (Know Your Customer) flow in use.

### User Types

There are **three types** of users in PayCaddy’s system:

| User Type        | Description                            | KYC Flow                                                                |
| ---------------- | -------------------------------------- | ----------------------------------------------------------------------- |
| `EndUser`        | A natural person using Integrated KYC. | PayCaddy handles KYC via a hosted link.                                 |
| `EndUserSR`      | A natural person using Delegated KYC.  | The client collects and verifies KYC, then sends verified data via API. |
| `MerchantUser`   | A business entity.                     | Delegated KYB (Know Your Business), provided by the client.             |

To determine the correct endpoint for a particular scenario, refer to the following flow. For more detailed information on each specific flow, please find the respective section in this chapter.

![general_user_flow](./assets/imgs/generaluserflow.svg)
{class="img"}

### EndUser (Integrated KYC)

This flow is used when PayCaddy is responsible for handling identity verification via our 3rd party KYC provider.

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
    
    - You can confirm the user's activation by querying [EndUser GET](user.en.md#end-user-get)

A visualization of the above described steps can be seen in the following flow:


![enduserflow](./assets/imgs/enduser.svg)


>*** Based on the KYC notification webhook, you can decide whether to offer users a retry option, as per your UX/UI preferences. For pricing details related to integrated KYC verifications, consult our commercial team.

---

### EndUserSR (Delegated KYC)

Used when your system (or your client’s) already handles the KYC process independently and sends verified data to PayCaddy.

>This option is reserved to **regulated entities** within their country of jurisdiction. 
>
>Access to the creation of this type of user must be requested and approved by PayCaddy's compliance team during onboarding or upgrade cycles.

**Steps:**

1. **Verify user externally**
    
    - Use your KYC provider to collect documents and validate identity.
        
2. **Create the user**
    
    - Payload schema is detailed in [EndUserSR POST](user.en.md#end-user-sr-post)
        
    - Payload must include temporary* URLs to access the document files data.
	    - Details regarding exact document files data capture parameters will be discussed with Compliance Team during onboarding.
        
3. **User becomes active immediately**
    
    - Since verification is delegated, no further KYC is needed from PayCaddy.
    - Periodically KYC information will be required to be updated. Review [Edit User Data](editUserData.en.md)

A visualization of the above described steps can be seen in the following flow:

![endusersrflow](./assets/imgs/EndUserSR.svg)


>** For specific periodicity details on scheduled KYC checks, consult with PayCaddy's compliance team during the integration phase.

---

### MerchantUser (Delegated KYB)

This user type represents a business entity. KYB is fully delegated.

**Steps:**

1. **Collect business documents**
    
    - Gather legal documents, beneficial owner info, tax IDs, etc.
        
2. **CONDITIONAL - Submit for pre-approval*** ***  
    
    - Some card programs require pre-approval of MerchantUsers prior to their creation.
	    
    - Failure to do so will incur in the blocking of non-approved users during scheduled checks.
	    
    - For particular card programs, this condition can be lifted during onboarding definitions.
	    
3. **Create the merchant**
    
    - Schema detailed in [MerchantUser POST](user.en.md#merchant-user-post)
        
4. **Merchant becomes active**
    
    - Since verification is delegated, no further KYC is needed from PayCaddy.
    - Periodically KYC information will be required to be updated. Review [Edit User Data](editUserData.en.md)

>* For specifics on which verification checks are done, review with Compliance team during onboarding process.
>
>** For specific periodicity details on scheduled KYC checks, consult with PayCaddy's compliance team during the integration phase.
>
>*** All card programs for Merchant type users will undergo a definition process with PayCaddy’s compliance team to establish file formats and pre-approval conditions if applicable.

A visualization of the above described steps can be seen in the following flow:

![merchantuserflow](./assets/imgs/merchantuser.svg)

---

###  Summary Table

| User Type      | API Endpoint                                       | KYC Mode   | Activation Trigger       |
| -------------- | -------------------------------------------------- | ---------- | ------------------------ |
| `EndUser`      | [EndUser POST](user.en.md#end-user-post)           | Integrated | MetaMap webhook          |
| `EndUserSR`    | [EndUserSR POST](user.en.md#end-user-sr-post)      | Delegated  | Immediate after creation |
| `MerchantUser` | [MerchantUser POST](user.en.md#merchant-user-post) | Delegated  | After document approval  |

>During the initial exploration, our sales team should have assigned you the specific details of your card program profiles, which will define which endpoint(s) you should call for user creation and the relevant KYC obligations.


