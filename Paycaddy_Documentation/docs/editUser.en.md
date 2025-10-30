# **User Update Data Endpoints**

!!!info
    **Purpose:**  
    These endpoints allow PayCaddy clients to update the existing user data for both **natural persons** (`EndUser`, `EndUserSR`) and **legal entities** (`MerchantUser`, `MerchantUserSR`).  
    They are primarily used to manage **compliance-related information**, **contact updates**, or **minor corrections** to existing records.  
    Certain fields—such as identifiers, userId, and wallet relations—**cannot** be modified through this call.


## **V1/UserUpdateData <font color="green">POST</font>**

**Request URL:**  
`https://api.api-sandbox.paycaddy.dev/v1/UserUpdateData`

This endpoint updates user information for **EndUser** and **EndUserSR** records.  
The body replicates the schema of the creation endpoint but omits fields that cannot be modified (e.g., identifiers, wallet data, and creation timestamps).

=== "Request"
	```json
		{
		  "userId": "string",
		  "email": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": false,
		  "salary": 200000,
		  "telephone": "+50760001234",
		  "address": {
		    "addressLine1": "123 Main Street",
		    "addressLine2": "Tower B, Apt 12B",
		    "homeNumber": "12B",
		    "city": "Panama",
		    "region": "Panama Metropolitan Area",
		    "postalCode": "000000",
		    "country": "PA"
		  },
		  "nationality": "PA",
		  "countryOfOperations": "PA, US"
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string"
		}```


=== "Response"
	```json
		{
		  "userId": "string",
		  "email": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": false,
		  "salary": 200000,
		  "telephone": "+50760001234",
		  "address": {
		    "addressLine1": "123 Main Street",
		    "addressLine2": "Tower B, Apt 12B",
		    "homeNumber": "12B",
		    "city": "Panama",
		    "region": "Panama Metropolitan Area",
		    "postalCode": "000000",
		    "country": "PA"
		  },
		  "nationality": "PA",
		  "countryOfOperations": "PA, US"
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string"
		}```

> **Note:**
> 
> - The update request only changes provided fields.
>     
> - Any omitted fields remain unchanged.
>     
> - The system validates each updated field according to the same format and type rules described in the [User Creation chapter](https://chatgpt.com/c/userv2.en).
>     

---

### Editable Fields

|Field|Description|Editable|Notes|
|---|---|---|
|`email`|Contact email|Must follow RFC-5322 format|
|`firstName`, `lastName`|User's legal name|⚠️ Conditionally|Only allowed for compliance updates|
|`alias`|Emboss alias|Must comply with ITU-T.50 and ≤ 22 chars|
|`occupation`, `placeOfWork`|Job-related fields|May be required for updated KYC|
|`pep`|Politically exposed person flag|Boolean|
|`salary`|Updated salary in cents|Integer, USD cents|
|`telephone`|E.164 format|Example: +50760001234|
|`address.*`|All subfields|See creation schema|
|`nationality`|ISO alpha-2 code||
|`countryOfOperations`|ISO alpha-2, comma-separated||

---

### Non-Editable Fields

|Field|Reason|
|---|---|
|`userId`|Immutable primary key|
|`walletId`|System-generated|
|`kycUrl`|Tied to KYC process|
|`isActive`|Controlled by compliance logic|
|`creationDate`|Immutable|

---

### Validation & Error Handling

Errors follow the same format as user creation. See “Field Requirements” in the creation chapter for detailed validation.

|HTTP Code|Type|Description|
|---|---|---|
|`200`|OK|Update accepted|
|`400`|ValidationError|Field missing or invalid|
|`422`|Business Rule Violation|Attempt to modify restricted field|
|`500`|Internal Error|Unexpected error on server side|

=== "Sample Error"

```json
{
  "type": "https://docs.paycaddy.com/errors/PC-422-READONLY",
  "title": "Attempted update of read-only field: walletId",
  "status": 422,
  "traceId": "00-1133a5e8f93b83b4b29d61d91cb-1a140dcbf259a24d-00"
}
```

---

## **V1/MerchantUserUpdateData POST**

**Request URL:**  
`https://api.api-sandbox.paycaddy.dev/v1/MerchantUserUpdateData`

This endpoint updates user information for **MerchantUser** and **MerchantUserSR** (legal entity users).  
It allows for controlled modification of business or compliance information, maintaining immutable identifiers and KYB-linked attributes.

=== "Request"
	```json
	{
	  "merchantUserId": "string",
	  "email": "string",
	  "registeredName": "string",
	  "taxId": "string",
	  "legalRepresentation": "string",
	  "kindOfBusiness": "string",
	  "telephone": "+50760001234",
	  "address": {
	    "addressLine1": "Avenue 5, Building 14",
	    "addressLine2": "Suite 200",
	    "city": "Panama City",
	    "region": "Panama",
	    "postalCode": "000000",
	    "country": "PA"
	  },
	  "firstName": "John",
	  "lastName": "Smith",
	  "nationality": "PA",
	  "countryOfOperations": "PA, US",
	  "businessLicense": "https://cdn.server.com/docs/businessLicense.pdf",
	  "registerShareholder": "https://cdn.server.com/docs/registerShareholder.pdf"
	}
	```

=== "Response"
	```json
	{
	  "merchantUserId": "string",
	  "email": "string",
	  "registeredName": "string",
	  "taxId": "string",
	  "legalRepresentation": "string",
	  "kindOfBusiness": "string",
	  "telephone": "+50760001234",
	  "address": {
	    "addressLine1": "Avenue 5, Building 14",
	    "addressLine2": "Suite 200",
	    "city": "Panama City",
	    "region": "Panama",
	    "postalCode": "000000",
	    "country": "PA"
	  },
	  "firstName": "John",
	  "lastName": "Smith",
	  "nationality": "PA",
	  "countryOfOperations": "PA, US",
	  "businessLicense": "https://cdn.server.com/docs/businessLicense.pdf",
	  "registerShareholder": "https://cdn.server.com/docs/registerShareholder.pdf"
	}
	```

---

### Editable Fields

|Field|Description|Editable|Notes|
|---|---|---|---|
|`email`|Business email|✅|RFC-5322 format|
|`registeredName`|Legal entity name|⚠️|Requires compliance approval|
|`legalRepresentation`|Legal representative|✅|Must match ITU-T.50|
|`kindOfBusiness`|Business type or code|✅||
|`telephone`|Contact phone|✅|E.164 format|
|`address.*`|Address components|✅||
|`firstName`, `lastName`|Natural representative|✅|Same rules as creation|
|`nationality`, `countryOfOperations`|Country data|✅|ISO alpha-2 format|
|`businessLicense`, `registerShareholder`, `certificateOfGoodStanding`, `idShareholders`, `addressVerificationShareholders`|Document URLs|✅|HTTPS URLs (PDF/JPG/PNG) between 5kb–10mb|

---

### Non-Editable Fields

|Field|Reason|
|---|---|
|`merchantUserId`|Immutable primary key|
|`walletId`|System-generated|
|`isActive`|Controlled by KYB compliance|
|`creationDate`|Immutable timestamp|

---

### Validation & Error Handling

|HTTP Code|Type|Description|
|---|---|---|
|`200`|OK|Update accepted|
|`400`|ValidationError|Field missing or invalid|
|`422`|Business Rule Violation|Attempt to modify restricted field|
|`500`|Internal Error|Unexpected error on server side|

=== "Sample Error"

```json
{
  "type": "https://docs.paycaddy.com/errors/PC-422-READONLY",
  "title": "Attempted update of read-only field: taxId",
  "status": 422,
  "traceId": "00-77aa4e5b7f1e3b45c92f91f77aa4e5b-1a140dcbf259a24d-00"
}
```

---

### Notes & Recommendations

- Partial updates are allowed — only the provided keys are modified.
    
- Changes to **sensitive fields** (e.g., `registeredName`, `legalRepresentation`) may trigger **manual review** by PayCaddy’s compliance team.
    
- All updates are versioned internally and may be audited on request.
    
- Document URLs must remain valid for at least 24 hours post-update.
    

---

> **Compliance Reminder:**  
>  The purpose of these endpoints is to keep user data up-to-date for ongoing KYC/KYB compliance.  
> Any misuse or attempt to alter identity or ownership data outside authorized flows may result in rejection or account suspension.