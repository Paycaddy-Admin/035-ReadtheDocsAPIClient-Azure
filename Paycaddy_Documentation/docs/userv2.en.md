!!!Important
    Since October 15th 2025,  these endpoints have been made available as part of an effort to migrate all user creation from v1 into the below documented endpoints. Official communications will handle the exact timelines for adoption. If you're integrating PayCaddy's system for the first time, these are the currently accepted endpoints for user creation flow.



## **V2/End User  <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/endUsers

‍The creation of a new user for a natural person begins with a POST call in which an endpoint is consumed for sending the user's basic information:

=== "Request"
    ```json
        {
		  "email": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string"
		}
    ```
=== "Response"
    ```json
        {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "isActive": false,
		  "walletId": "string",
		  "kycUrl": "string",
		  "creationDate": "2025-01-01T00:00:00.000Z"
		}
    ```

#### Validation & Error Handling

PayCaddy validates the **format and presence** of key fields, while Business accuracy (e.g., “is this the person’s _real_ employer?”) and **deduplication across your own users** remain the client’s responsibility.

- The API **will not** reject on business accuracy or duplicates.
    
- The API **will** reject when format/validation rules are not met (see "Field Requirements" section below).
    
If the request passes validation and is processed, the API responds with **HTTP 200 OK** and returns the created `userId`, its initial `walletId`, and control fields such as `isActive=false` until KYC is completed.

>**Deduplication is Not performed by PayCaddy.** 
>Multiple POSTs with identical data will create **distinct** `userId` values. If you need duplicate control, implement it on your side before calling the API.

>**Spam Control**
>Identical Payloads (or the creation of users with identical emails) sent within 5 minutes of each other will result in a 422 error as a measure against erroneous spamming, the payload would become acceptable once again after the time is past.

---
#### User Name & Embossing Rules

1. **Sanitized character set**: `FirstName` and `LastName` **must** be sent already sanitized to [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en).
    
2. **22-char emboss limit**: `(FirstName + LastName)` **without spaces** must be **≤ 22** **after** sanitization.
    
3. **Alias requirement when > 22**:  
    If `(FirstName + LastName)` would exceed 22 after sanitization, you **must** provide an `alias` (max 22, ITU-T.50).
    
    - When a valid `alias` is provided, it is used for the card name line.
        
    - When the full name fits (≤ 22), `alias` is **not required** and may be ignored for embossing.
        
4. **No nicknames**: `alias` is strictly for embossing overflow cases; do **not** use unrelated nicknames as it may impact in-person acceptance.
	
5. **Real Name Utilization:** The fields of `FirstName` and `LastName` must match or resemble the real name found on IDs otherwise the KYC verification will fail repeatedly.
    

---

#### Field Requirements

| Field                  | Type    | Example Value                                      | Required | Rules                                                                                                                                                                                                       | Example Error Code                                                                                                                                                                                                    |
| ---------------------- | ------- | -------------------------------------------------- | -------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `firstName`            | string  | "John"                                             |      Yes | Non-null, non-empty, ITU-T.50 only; contributes to 22-char limit (with `LastName`)                                                                                                                          | {<br>        "type": "",<br>        "title": "Error: special characters present in first name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                   |
| `lastName`             | string  | "Smith"                                            |      Yes | Non-null, non-empty, ITU-T.50 only; contributes to 22-char limit (with `FirstName`)                                                                                                                         | {<br>        "type": "",<br>        "title": "Error: special characters present in last name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                    |
| `alias`                | string  | "J Smith"                                          |    Cond. | **Required only** if `(FirstName+LastName)` > 22 after sanitization; ITU-T.50; max 22                                                                                                                       | {<br>        "type": "",<br>        "title": "Error: special characters present in alias",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                        |
| `occupation`           | string  | `"21313"` Representing "Computer Systems Designer" |      Yes | **Code** from the provided occupation list (exact match)<br><br>Find full list of values in CSV or JSON [HERE](https://github.com/Paycaddy-Admin/PayCaddy-Integration)                                      | {<br>        "type": "",<br>        "title": "Error: value not in allowed occupation list",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                       |
| `email`                | string  | "jsmith@example.com"                               |      Yes | Non-null, standard email format (RFC-5322-compatible)                                                                                                                                                       | {<br>        "type": "",<br>        "title": "Invalid format for email",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                          |
| `telephone`            | string  | "+50760001234"                                     |      Yes | Must be **E.164** (`+` + country code + number)                                                                                                                                                             | {<br>        "type": "",<br>        "title": "Telephone value is incorrect, allowed format: E.164 (`+` + country code + number)",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    } |
| `placeOfWork`          | string  | "ACME, Inc."                                       |      Yes |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `pep`                  | boolean | `false`                                            |      Yes | `true` if the natural person is politically exposed; else `false`                                                                                                                                           |                                                                                                                                                                                                                       |
| `salary`               | number  | `200000` Representing USD 2,000.00 monthly salary  |      Yes | Monthly salary in **cents of USD**;                                                                                                                                                                         | {<br>        "type": "",<br>        "title": "Error: salary must be an integer value in cents USD",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                               |
| `address.addressLine1` | string  | "123 Main Street"                                  |      Yes |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `address.addressLine2` | string  | "PH Residential Tower"                             |       No | Optional                                                                                                                                                                                                    |                                                                                                                                                                                                                       |
| `address.homeNumber`   | string  | "12B"                                              |       No | If present, must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`)                                                                                                                                         | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |
| `address.city`         | string  | "Panama"                                           |      Yes |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `address.region`       | string  | "Panama Metropolitan Area"                         |      Yes |                                                                                                                                                                                                             |                                                                                                                                                                                                                       |
| `address.postalCode`   | string  | "000000"                                           |      Yes | Use correct Postal Code formatting for countries where it applies. Use "000000" as a bypass for countries that do not use a Postal Code. Per country logic can evolve and will be communicated accordingly. |                                                                                                                                                                                                                       |
| `address.country`      | string  | "PA"                                               |      Yes | Must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`)                                                                                                                                                     | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |
| `nationality`          | string  | "PA"                                               |      Yes | Must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`)                                                                                                                                                     |                                                                                                                                                                                                                       |
| `countryOfOperations`  | string  | "PA, US"                                           |      Yes | Must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`), if multiple values, separate them by commas in a single string.                                                                                    |                                                                                                                                                                                                                       |


> **PEP definition:** a natural person entrusted with prominent public responsibilities who, by virtue of position and influence, may present higher risk of bribery/corruption exposure.

> **KYC reminder:** After IDs are assigned, the end-user must complete KYC to activate the profile. Until then, `isActive` remains `false` and all procedures are disabled. See KYC Verification

---
### Common Errors


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "FirstName": [
	            "The FirstName field is required."
	        ]
	    }
	}
	```

| Input Field    | Code/Response           | Message                                                                                                                 |
| -------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| email          | invalid_format (422)    | Invalid format for email                                                                                                |
| firstName      | special_chars (422)     | Error: special characters present in first name                                                                         |
| lastName       | special_chars (422)     | Error: special characters present in last name                                                                          |
| alias          | special_chars (422)     | Error: special characters present in alias<br><br>The field Alias is required when First and LastName are above 22 char |
| telephone      | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number                                                     |
| country        | invalid_value (422)     | Country value is incorrect                                                                                              |
| occupation     | not_allowed_value (422) | Error: value not in allowed occupation list                                                                             |
| salary         | invalid_format (422)    | Error: salary must be an integer value in cents USD                                                                     |
| addressLine1   | min_length (422)        | Error: AddressLine1 must be at least 5 characters long                                                                  |
| region         | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                                                                     |



> **500 Internal Server Error** indicates an internal issue on our side. Please notify the PayCaddy team with the payload evidence so we can investigate promptly.

---
#### KYC Verification
In the user creation response (EndUser POST), a link to the identity validation associated with the userId in Metamap is presented at the field **kycUrl**, where the instructions and steps to follow will be presented to the end-user to complete the verification.
```json
    {
        "kycUrl": "https%3a%2f%2fsignup.getmati.com%2f%3fmerchantToken%
                  3d6046cc2a54816f001bedd641%26flowId%3d6046cc2a54816f0
                01bedd640%26metadata%3d%7b%22userid%22%3a%226955ea0f4
                f3-4254-b10a-0181f307298c%22%7d"
    }
```

It is important to consider that the kycURL is shared by applying URL encoding that allows the sending of metadata (userId) that links the validation with the created user. To ensure the activation of the user upon successfully completing the validation, it is necessary to ensure that the URL to which the user is redirected from the front-end interface loads the following structure:

>https://signup.getmati.com/?merchantToken=6046cc2a54816f001dedd641&flowId=6046cc2a54816f001bedd640&metadata={"userid":"a955ea0f-34f3-4254-b10a-0181f30729kd"}

For complete information on KYC, please find detailed information on this documentation's KYC chapter.

---

## **End User <font color="sky-blue">GET</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/endUsers/

‍The GET call for an endUser allows you to know the stored data of a particular userId, especially the walletId of their initial wallet and the activity status of this user in the "isActive" field. Both data are crucial for the other calls to the NeoBank API.

This call can be used to verify the user's status at any point in the flow.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v2/endUsers/${USER_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "isActive": false,
		  "walletId": "string",
		  "kycUrl": "string",
		  "creationDate": "2025-01-01T00:00:00.000Z"
		}
    ```

---

## **V2/Merchant User <font color="green">POST</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/merchantUsers

The creation of a new user for a legal entity begins with a POST call in which an endpoint is consumed for sending the basic data of the legal entity.

=== "Request"
    ```json```
        ```{
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string"
		}```
		

=== "Response"
    ```json```
    ```
		{
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}```



>It should be noted that the responsibility for validating the **accuracy and format** of the entered data falls on the **PayCaddy client**, meaning that our API will return a successful response **as long as the following parameters are met**, regardless of the factual accuracy of the information or duplication of the shared data.

>**Spam Control**
>Identical Payloads (or the creation of users with identical emails) sent within 5 minutes of each other will result in a 422 error as a measure against erroneous spamming, the payload would become acceptable once again after the time is past.

## Field Requirements

| Field                             | Type         | Required | Rules                                                                                                                                                                            | Example Error Response                                                                                                                                                  |
| --------------------------------- | ------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `email`                           | string       | Yes      | Must follow RFC-5322 format. Non-null, non-empty.                                                                                                                                | {  <br>"type": "",  <br>"title": "Invalid format for email",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                            |
| `registeredName`                  | string       | Yes      | Legal entity’s registered name. Must be sanitized ([ITU-T50](https://www.itu.int/rec/T-REC-T.50/en)), ≤ 22 chars, no emojis or symbols.                                          | {  <br>"type": "",  <br>"title": "Error: special characters present in registeredName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `taxId`                           | string       | Yes      | Unique taxpayer identifier (RUC, VATIN, CUIT, CNPJ, NIT, etc.). Alphanumeric + hyphens only.                                                                                     | {  <br>"type": "",  <br>"title": "Error: Tax ID must contain alphanumeric or hyphen characters only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}   |
| `legalRepresentation`             | string       | Yes      | Full name of legal representative. ITU-T.50 characters only, ≤ 60 chars.                                                                                                         | {  <br>"type": "",  <br>"title": "Error: special characters present in legalRepresentation",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}            |
| `kindOfBusiness`                  | string       | Yes      | Description or code of economic activity. Must match allowed list.<br><br>Find full list of values in CSV or JSON [HERE](https://github.com/Paycaddy-Admin/PayCaddy-Integration) | {  <br>"type": "",  <br>"title": "Error: value not in allowed kindOfBusiness list",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                     |
| `telephone`                       | string       | Yes      | Must follow E.164 format (`+` + country code + number).                                                                                                                          | {  <br>"type": "",  <br>"title": "Telephone value is incorrect, allowed format: +(country code)number",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>} |
| `address.addressLine1`            | string       | Yes      | Minimum 5 characters. Descriptive physical address.                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: AddressLine1 must be at least 5 characters long",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}              |
| `address.region`                  | string       | Yes      | Must be alphabetic-only, non-empty.                                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: Region must be non-empty and alphabetic-only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `address.country`                 | string       | Yes      | ISO-3166-1 alpha-2 code (e.g., `PA`, `US`, `BR`).                                                                                                                                | {  <br>"type": "",  <br>"title": "Country value is incorrect",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                          |
| `firstName`                       | string       | Yes      | First name of owner/representative. [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en) only, part of 22-char limit combined with `lastName`.                                       | {  <br>"type": "",  <br>"title": "Error: special characters present in firstName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `lastName`                        | string       | Yes      | Last name of owner/representative. [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en) only, part of 22-char limit combined with `firstName`.                                       | {  <br>"type": "",  <br>"title": "Error: special characters present in lastName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                       |
| `nationality`                     | string       | Yes      | Must be ISO-3166-1 alpha-2 country code.                                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: Invalid nationality code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                     |
| `countryOfOperations`             | string       | Yes      | Country where entity primarily operates; ISO-3166-1 alpha-2 code. <br><br>Multiple codes can be introduced as comma separated values, e.g.: "BR, PA"                             | {  <br>"type": "",  <br>"title": "Error: Invalid countryOfOperations code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                             |
| `certificateOfGoodStanding`       | string (URL) | Yes      | Must be valid URL to official document uploaded for KYB.                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: certificateOfGoodStanding must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                |
| `businessLicense`                 | string (URL) | Yes      | Must be valid URL to active business license document.                                                                                                                           | {  <br>"type": "",  <br>"title": "Error: businessLicense must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                          |
| `registerShareholder`             | string (URL) | Yes      | Must be valid URL to official shareholder register document.                                                                                                                     | {  <br>"type": "",  <br>"title": "Error: registerShareholder must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `idShareholders`                  | string (URL) | Yes      | Must be valid URL to scanned IDs of shareholders.                                                                                                                                | {  <br>"type": "",  <br>"title": "Error: idShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                           |
| `addressVerificationShareholders` | string (URL) | Yes      | Must be valid URL to proof of address documents for shareholders.                                                                                                                | {  <br>"type": "",  <br>"title": "Error: addressVerificationShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}          |

### KYB Activation & Additional Field Responsibilities

In addition to the format verifications, it is important to highlight the responsibility of the client to **consistently send accurate and verifiable business information** for KYB compliance review:

1. The **`taxId`** field must include the **official taxpayer identification number** of the entity (e.g., RUC, VATIN, CUIT, CNPJ, NIT, or RUT), depending on the jurisdiction.
    
2. The **`legalRepresentation`** field must contain the full legal name of the person authorized to represent the company, as registered in its incorporation or government record.
    
3. The fields **`firstName`** and **`lastName`** must correspond to the natural person associated with the **legal representation** or the individual aimed to become the cardholder. These must comply with the same ITU-T.50 sanitization and character restrictions used for natural persons. These will be the names printed in the card, so they must aim to identify the cardholder.
    
4. The fields **`certificateOfGoodStanding`**, **`businessLicense`**, **`registerShareholder`**, **`idShareholders`**, and **`addressVerificationShareholders`** should contain valid URLs or document references for the company’s compliance documentation.
    
    - Each uploaded document must be instantly accessible for programatic capture, the expected files must be at least 5kb and no more than 10mb in size and either a PDF, JPG or PNG filetype.
        
    - Missing or invalid links will result in a validation error and the user creation will fail.
    
5. PayCaddy's compliance team will review the shared information for approval and user activation. Ensuring the information is correctly shared diminishes rejection rates.


### Common Errors


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "RegisteredName": [
	            "The RegisteredName field is required."
	        ]
	    }
	}
	```

| Input Field                     | Code/Response           | Message                                                             |
| ------------------------------- | ----------------------- | ------------------------------------------------------------------- |
| email                           | invalid_format (422)    | Invalid format for email                                            |
| registeredName                  | special_chars (422)     | Error: special characters present in registeredName                 |
| taxId                           | invalid_format (422)    | Error: Tax ID must contain alphanumeric or hyphen characters only   |
| legalRepresentation             | special_chars (422)     | Error: special characters present in legalRepresentation            |
| kindOfBusiness                  | not_allowed_value (422) | Error: value not in allowed kindOfBusiness list                     |
| telephone                       | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number |
| address.addressLine1            | min_length (422)        | Error: AddressLine1 must be at least 5 characters long              |
| address.region                  | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                 |
| address.country                 | invalid_value (422)     | Country value is incorrect                                          |
| firstName                       | special_chars (422)     | Error: special characters present in firstName                      |
| lastName                        | special_chars (422)     | Error: special characters present in lastName                       |
| nationality                     | invalid_value (422)     | Error: nationality must be a valid ISO alpha-2 code                 |
| countryOfOperations             | invalid_value (422)     | Error: countryOfOperations must be a valid ISO alpha-2 code         |
| certificateOfGoodStanding       | invalid_format (422)    | Error: certificateOfGoodStanding must be a valid URL                |
| businessLicense                 | invalid_format (422)    | Error: businessLicense must be a valid URL                          |
| registerShareholder             | invalid_format (422)    | Error: registerShareholder must be a valid URL                      |
| idShareholders                  | invalid_format (422)    | Error: idShareholders must be a valid URL                           |
| addressVerificationShareholders | invalid_format (422)    | Error: addressVerificationShareholders must be a valid URL          |

> **500 Internal Server Error** indicates an internal issue on our side. Please notify the PayCaddy team with the payload evidence so we can investigate promptly.


---

## **Merchant User <font color="sky-blue">GET</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/merchantUsers/

The GET call for a merchantUser allows you to know the stored data of a particular userId, especially the walletId of their initial wallet and the activity status of this user in the "isActive" field. Both data are crucial for the other calls to the NeoBank API.

This call can be used to verify the user's status at any point in the flow.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v2/merchantUsers/{MERCHANT_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}```

---

## **V2/End User SR  <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/endUserSRs

‍The creation of a new user with Delegated KYC type for a natural person begins with a POST call in which an endpoint is consumed for sending the user's basic information, in the case of End User SR (Subject to Regulation), users are created active by default.
This endpoint is not openly available, it must have been enabled by PayCaddy's compliance team during the integration process

=== "Request"
    ```json
        {
		  "email": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "alias": "string",
		  "occupation": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string"
		}
    ```
=== "Response"
    ```json
        {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string",
		  "alias": "string",
		  "occupation": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "creationDate": "2025-10-14T19:24:26.919Z"
		}
    ```

#### Validation & Error Handling

PayCaddy validates the **format and presence** of key fields, while Business accuracy (e.g., “is this the person’s _real_ employer?”) and **deduplication across your own users** remain the client’s responsibility.

- The API **will not** reject on business accuracy or duplicates.
    
- The API **will** reject when format/validation rules are not met (see "Field Requirements" section below).
    
If the request passes validation and is processed, the API responds with **HTTP 200 OK** and returns the created `userId`, its initial `walletId`, and control fields such as `isActive=true` for this type of user with Delegated KYC flow. 

>**Deduplication is Not performed by PayCaddy.** 
>Multiple POSTs with identical data will create **distinct** `userId` values. If you need duplicate control, implement it on your side before calling the API.

>**Spam Control**
>Identical Payloads (or the creation of users with identical emails) sent within 5 minutes of each other will result in a 422 error as a measure against erroneous spamming, the payload would become acceptable once again after the time is past.

---
#### User Name & Embossing Rules

1. **Sanitized character set**: `FirstName` and `LastName` **must** be sent already sanitized to ITU-T.50.
    
2. **22-char emboss limit**: `(FirstName + LastName)` **without spaces** must be **≤ 22** **after** sanitization.
    
3. **Alias requirement when > 22**:  
    If `(FirstName + LastName)` would exceed 22 after sanitization, you **must** provide an `alias` (max 22, ITU-T.50).
    
    - When a valid `alias` is provided, it is used for the card name line.
        
    - When the full name fits (≤ 22), `alias` is **not required** and may be ignored for embossing.
        
4. **No nicknames**: `alias` is strictly for embossing overflow cases; do **not** use unrelated nicknames as it may impact in-person acceptance.
	
5. **Real Name Utilization:** The fields of `FirstName` and `LastName` must match or resemble the real name found on IDs to ensure future compliance checks don't flag users for misuse.
    

---

#### Field Requirements

| Field                  | Type    | Example Value                                                     | Required | Rules                                                                                                                                                                                                                                                                                        | Example Error Code                                                                                                                                                                                                    |     |
| ---------------------- | ------- | ----------------------------------------------------------------- | -------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `firstName`            | string  | "John"                                                            |      Yes | Non-null, non-empty, ITU-T.50 only; contributes to 22-char limit (with `LastName`)                                                                                                                                                                                                           | {<br>        "type": "",<br>        "title": "Error: special characters present in first name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                   |     |
| `lastName`             | string  | "Smith"                                                           |      Yes | Non-null, non-empty, ITU-T.50 only; contributes to 22-char limit (with `FirstName`)                                                                                                                                                                                                          | {<br>        "type": "",<br>        "title": "Error: special characters present in last name",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                    |     |
| `alias`                | string  | "J Smith"                                                         |    Cond. | **Required only** if `(FirstName+LastName)` > 22 after sanitization; ITU-T.50; max 22                                                                                                                                                                                                        | {<br>        "type": "",<br>        "title": "Error: special characters present in alias",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                        |     |
| `occupation`           | string  | `"21313"` Representing "Computer Systems Designer"                |      Yes | **Code** from the provided occupation list (exact match)<br><br>Find full list of values in CSV or JSON [HERE](https://github.com/Paycaddy-Admin/PayCaddy-Integration)                                                                                                                       | {<br>        "type": "",<br>        "title": "Error: value not in allowed occupation list",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                       |     |
| `email`                | string  | "jsmith@example.com"                                              |      Yes | Non-null, standard email format (RFC-5322-compatible)                                                                                                                                                                                                                                        | {<br>        "type": "",<br>        "title": "Invalid format for email",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                          |     |
| `telephone`            | string  | "+50760001234"                                                    |      Yes | Must be **E.164** (`+` + country code + number)                                                                                                                                                                                                                                              | {<br>        "type": "",<br>        "title": "Telephone value is incorrect, allowed format: E.164 (`+` + country code + number)",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    } |     |
| `placeOfWork`          | string  | "ACME, Inc."                                                      |      Yes |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `pep`                  | boolean | `false`                                                           |      Yes | `true` if the natural person is politically exposed; else `false`                                                                                                                                                                                                                            |                                                                                                                                                                                                                       |     |
| `salary`               | number  | `200000` Representing USD 2,000.00 monthly salary                 |      Yes | Monthly salary in **cents of USD**;                                                                                                                                                                                                                                                          | {<br>        "type": "",<br>        "title": "Error: salary must be an integer value in cents USD",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                               |     |
| `address.addressLine1` | string  | "123 Main Street"                                                 |      Yes |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `address.addressLine2` | string  | "PH Residential Tower"                                            |       No | Optional                                                                                                                                                                                                                                                                                     |                                                                                                                                                                                                                       |     |
| `address.homeNumber`   | string  | "12B"                                                             |       No | If present, must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`)                                                                                                                                                                                                                          | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |     |
| `address.city`         | string  | "Panama"                                                          |      Yes |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `address.region`       | string  | "Panama Metropolitan Area"                                        |      Yes |                                                                                                                                                                                                                                                                                              |                                                                                                                                                                                                                       |     |
| `address.postalCode`   | string  | "000000"                                                          |      Yes | Use correct Postal Code formatting for countries where it applies. Use "000000" as a bypass for countries that do not use a Postal Code. Per country logic can evolve and will be communicated accordingly.                                                                                  |                                                                                                                                                                                                                       |     |
| `address.country`      | string  | "PA"                                                              |      Yes | Must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`)                                                                                                                                                                                                                                      | {<br>        "type": "",<br>        "title": "Country value is incorrect",<br>        "status": 0,<br>        "detail": "",<br>        "instance": ""<br>    }                                                        |     |
| `nationality`          | string  | "PA"                                                              |      Yes | Must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`)                                                                                                                                                                                                                                      |                                                                                                                                                                                                                       |     |
| `countryOfOperations`  | string  | "PA, US"                                                          |      Yes | Must be ISO-3166-1 **alpha-2** (e.g., `PA`, `US`, `BR`), if multiple values, separate them by commas in a single string.                                                                                                                                                                     | }                                                                                                                                                                                                                     |     |
| `idUrlFront`           | string  | "https://cdn.prod.server-files.com/directory/user_idFile.jpg"     |      Yes | URL must direct to a real, reachable file, either `PDF`,`JPG`or `PNG` with between 5kb and 10mb size. It's expected that this file will contain the correct accepted document types. <br><br>PayCaddy's team reserves the right to regularly block users that present misuse of these fields | {<br>		  "type": "",<br>		  "title": "The value provided for the IdUrlFront presents some issues in the process of validation",<br>		  "status": 0,<br>		  "detail": "",<br>		  "instance": ""<br>		}```              |     |
| `idUrlBack`            | string  | "https://cdn.prod.server-files.com/directory/user_idFileBack.jpg" |      Yes | URL must direct to a real, reachable file, either `PDF`,`JPG`or `PNG` with between 5kb and 10mb size. It's expected that this file will contain the correct accepted document types. <br><br>PayCaddy's team reserves the right to regularly block users that present misuse of these fields | {<br>		  "type": "",<br>		  "title": "The value provided for the IdUrlBack presents some issues in the process of validation",<br>		  "status": 0,<br>		  "detail": "",<br>		  "instance": ""<br>		}```               |     |
| `residenceProof`       | string  | "https://cdn.prod.server-files.com/directory/residenceProof.jpg"  |      Yes | URL must direct to a real, reachable file, either `PDF`,`JPG`or `PNG` with between 5kb and 10mb size. It's expected that this file will contain the correct accepted document types. <br><br>PayCaddy's team reserves the right to regularly block users that present misuse of these fields | {<br>		  "type": "",<br>		  "title": "The value provided for the residenceProof presents some issues in the process of validation",<br>		  "status": 0,<br>		  "detail": "",<br>		  "instance": ""<br>		}```          |     |


> **PEP definition:** a natural person entrusted with prominent public responsibilities who, by virtue of position and influence, may present higher risk of bribery/corruption exposure.

> **Delegated KYC:** Users of the EndUserSR type are used exclusively by clients who've been assigned said privilege. These users are create with their activity status as "Active" and do not require a KYC verification link.

---
### Common Errors


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "FirstName": [
	            "The FirstName field is required."
	        ]
	    }
	}
	```

=== "Document Access Issues"
	```json
		{
		  "type": "",
		  "title": "The value provided for the IdUrlBack presents some issues in the process of validation",
		  "status": 0,
		  "detail": "",
		  "instance": ""
		}```



| Input Field    | Code/Response           | Message                                                                                                                 |
| -------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| email          | invalid_format (422)    | Invalid format for email                                                                                                |
| firstName      | special_chars (422)     | Error: special characters present in first name                                                                         |
| lastName       | special_chars (422)     | Error: special characters present in last name                                                                          |
| alias          | special_chars (422)     | Error: special characters present in alias<br><br>The field Alias is required when First and LastName are above 22 char |
| telephone      | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number                                                     |
| country        | invalid_value (422)     | Country value is incorrect                                                                                              |
| occupation     | not_allowed_value (422) | Error: value not in allowed occupation list                                                                             |
| kindOfBusiness | not_allowed_value (422) | Error: value not in allowed kindOfBusiness list                                                                         |
| salary         | invalid_format (422)    | Error: salary must be an integer value in cents USD                                                                     |
| addressLine1   | min_length (422)        | Error: AddressLine1 must be at least 5 characters long                                                                  |
| region         | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                                                                     |



> **500 Internal Server Error** indicates an internal issue on our side. Please notify the PayCaddy team with the payload evidence so we can investigate promptly.

---
### Delegated KYC (SR) – Document URLs

In SR, there is **no `kycUrl`** for post-creation KYC verification. Instead, you **must** provide **within the creation request** the following documents:

```json
{
  "idUrlFront": "https://... (front of ID)",
  "idUrlBack": "https://... (back of ID)",
  "residenceProofUrl": "https://... (proof of residence)"
}
```

- URLs **must be HTTPS** and **publicly reachable** by PayCaddy’s backend (accessible for at least 24hs, no IP-locked links).
    
- Content **must be** an image (`image/*`) or `application/pdf`.
    
- Ensure links remain valid for at least 24 hours. 
    
- If any URL is invalid, unreachable, or file type is unsupported, the request will be rejected with **422**.
    


> **Activation:** SR users are created **`isActive=true`** immediately upon success. Downstream actions remain subject to standard compliance checks and ongoing monitoring.


---

## **End User SR <font color="sky-blue">GET</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/SR/EndUserSRs/

The GET call for an EndUserSR allows you to know the stored data of a particular userId, especially the walletId of their initial wallet and the activity status of this user in the "isActive" field. Both data are crucial for the other calls to the NeoBank API.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs/{USER_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "email": "string",
		  "telephone": "string",
		  "placeOfWork": "string",
		  "pep": true,
		  "salary": 0,
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "idUrlFront": "string",
		  "idUrlBack": "string",
		  "residenceProofUrl": "string",
		  "alias": "string",
		  "occupation": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "creationDate": "2025-10-14T19:24:26.919Z"
	}
    ```


---

## **V2/Merchant User SR <font color="green">POST</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/merchantUserSRs

The creation of a new user for a legal entity with Delegated KYB begins with a POST call in which an endpoint is consumed for sending the basic data of the legal entity, in the case of Merchant User SR (Subject to Regulation), users are created active by default.
This endpoint is not openly available, it must have been enabled by PayCaddy's compliance team during the integration process

>Previous to the release of our V2 branch for user creation, Merchant User SR was not available and all Merchant Users created followed a Delegated KYB flow. This differentiation is enforced since October 2025 with the launch of the V2 endpoints.

=== "Request"
    ```json```
        ```{
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string"
		}```
		

=== "Response"
    ```json```
    ```
		{
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}```



>It should be noted that the responsibility for validating the **accuracy and format** of the entered data falls on the **PayCaddy client**, meaning that our API will return a successful response **as long as the following parameters are met**, regardless of the factual accuracy of the information or duplication of the shared data.

>**Spam Control**
>Identical Payloads (or the creation of users with identical emails) sent within 5 minutes of each other will result in a 422 error as a measure against erroneous spamming, the payload would become acceptable once again after the time is past.

## Field Requirements

| Field                             | Type         | Required | Rules                                                                                                                                                                            | Example Error Response                                                                                                                                                  |
| --------------------------------- | ------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `email`                           | string       | Yes      | Must follow RFC-5322 format. Non-null, non-empty.                                                                                                                                | {  <br>"type": "",  <br>"title": "Invalid format for email",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                            |
| `registeredName`                  | string       | Yes      | Legal entity’s registered name. Must be sanitized (ITU-T.50), ≤ 22 chars, no emojis or symbols.                                                                                  | {  <br>"type": "",  <br>"title": "Error: special characters present in registeredName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `taxId`                           | string       | Yes      | Unique taxpayer identifier (RUC, VATIN, CUIT, CNPJ, NIT, etc.). Alphanumeric + hyphens only.                                                                                     | {  <br>"type": "",  <br>"title": "Error: Tax ID must contain alphanumeric or hyphen characters only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}   |
| `legalRepresentation`             | string       | Yes      | Full name of legal representative. ITU-T.50 characters only, ≤ 60 chars.                                                                                                         | {  <br>"type": "",  <br>"title": "Error: special characters present in legalRepresentation",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}            |
| `kindOfBusiness`                  | string       | Yes      | Description or code of economic activity. Must match allowed list.<br><br>Find full list of values in CSV or JSON [HERE](https://github.com/Paycaddy-Admin/PayCaddy-Integration) | {  <br>"type": "",  <br>"title": "Error: value not in allowed kindOfBusiness list",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                     |
| `telephone`                       | string       | Yes      | Must follow E.164 format (`+` + country code + number).                                                                                                                          | {  <br>"type": "",  <br>"title": "Telephone value is incorrect, allowed format: +(country code)number",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>} |
| `address.addressLine1`            | string       | Yes      | Minimum 5 characters. Descriptive physical address.                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: AddressLine1 must be at least 5 characters long",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}              |
| `address.region`                  | string       | Yes      | Must be alphabetic-only, non-empty.                                                                                                                                              | {  <br>"type": "",  <br>"title": "Error: Region must be non-empty and alphabetic-only",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                 |
| `address.country`                 | string       | Yes      | ISO-3166-1 alpha-2 code (e.g., `PA`, `US`, `BR`).                                                                                                                                | {  <br>"type": "",  <br>"title": "Country value is incorrect",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                          |
| `firstName`                       | string       | Yes      | First name of owner/representative. ITU-T.50 only, part of 22-char limit combined with `lastName`.                                                                               | {  <br>"type": "",  <br>"title": "Error: special characters present in firstName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `lastName`                        | string       | Yes      | Last name of owner/representative. ITU-T.50 only, part of 22-char limit combined with `firstName`.                                                                               | {  <br>"type": "",  <br>"title": "Error: special characters present in lastName",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                       |
| `nationality`                     | string       | Yes      | Must be ISO-3166-1 alpha-2 country code.                                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: Invalid nationality code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                                     |
| `countryOfOperations`             | string       | Yes      | Country where entity primarily operates; ISO-3166-1 alpha-2 code. <br><br>Multiple codes can be introduced as comma separated values, e.g.: "BR, PA"                             | {  <br>"type": "",  <br>"title": "Error: Invalid countryOfOperations code",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                             |
| `certificateOfGoodStanding`       | string (URL) | Yes      | Must be valid URL to official document uploaded for KYB.                                                                                                                         | {  <br>"type": "",  <br>"title": "Error: certificateOfGoodStanding must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                |
| `businessLicense`                 | string (URL) | Yes      | Must be valid URL to active business license document.                                                                                                                           | {  <br>"type": "",  <br>"title": "Error: businessLicense must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                          |
| `registerShareholder`             | string (URL) | Yes      | Must be valid URL to official shareholder register document.                                                                                                                     | {  <br>"type": "",  <br>"title": "Error: registerShareholder must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                      |
| `idShareholders`                  | string (URL) | Yes      | Must be valid URL to scanned IDs of shareholders.                                                                                                                                | {  <br>"type": "",  <br>"title": "Error: idShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}                           |
| `addressVerificationShareholders` | string (URL) | Yes      | Must be valid URL to proof of address documents for shareholders.                                                                                                                | {  <br>"type": "",  <br>"title": "Error: addressVerificationShareholders must be a valid URL",  <br>"status": 0,  <br>"detail": "",  <br>"instance": ""  <br>}          |

### Additional Field Responsibilities

In addition to the format verifications, it is important to highlight the responsibility of the client to **consistently send accurate and verifiable business information** for KYB compliance review:

1. The **`taxId`** field must include the **official taxpayer identification number** of the entity (e.g., RUC, VATIN, CUIT, CNPJ, NIT, or RUT), depending on the jurisdiction.
    
2. The **`legalRepresentation`** field must contain the full legal name of the person authorized to represent the company, as registered in its incorporation or government record.
    
3. The fields **`firstName`** and **`lastName`** must correspond to the natural person associated with the **legal representation** or the individual aimed to become the cardholder. These must comply with the same ITU-T.50 sanitization and character restrictions used for natural persons. These will be the names printed in the card, so they must aim to identify the cardholder.
    
4. The fields **`certificateOfGoodStanding`**, **`businessLicense`**, **`registerShareholder`**, **`idShareholders`**, and **`addressVerificationShareholders`** should contain valid URLs or document references for the company’s compliance documentation.
    
    - Each uploaded document must be instantly accessible for programatic capture, the expected files must be at least 5kb and no more than 10mb in size and either a PDF, JPG or PNG filetype.
        
    - Missing or invalid links will result in a validation error and the user creation will fail.
    
5. PayCaddy's compliance team will review the shared information for approval and user activation. Ensuring the information is correctly shared diminishes rejection rates.


### Common Errors


=== "Missing fields"
	```json
	{
	    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
	    "title": "One or more validation errors occurred.",
	    "status": 400,
	    "traceId": "00-68857a5c83b83b498f4c49d8a61d91cb-1a140dcbf259a24d-00",
	    "errors": {
	        "RegisteredName": [
	            "The RegisteredName field is required."
	        ]
	    }
	}
	```

| Input Field                     | Code/Response           | Message                                                             |
| ------------------------------- | ----------------------- | ------------------------------------------------------------------- |
| email                           | invalid_format (422)    | Invalid format for email                                            |
| registeredName                  | special_chars (422)     | Error: special characters present in registeredName                 |
| taxId                           | invalid_format (422)    | Error: Tax ID must contain alphanumeric or hyphen characters only   |
| legalRepresentation             | special_chars (422)     | Error: special characters present in legalRepresentation            |
| kindOfBusiness                  | not_allowed_value (422) | Error: value not in allowed kindOfBusiness list                     |
| telephone                       | invalid_value (422)     | Telephone value is incorrect, allowed format: +(country code)number |
| address.addressLine1            | min_length (422)        | Error: AddressLine1 must be at least 5 characters long              |
| address.region                  | invalid_format (422)    | Error: Region must be non-empty and alphabetic-only                 |
| address.country                 | invalid_value (422)     | Country value is incorrect                                          |
| firstName                       | special_chars (422)     | Error: special characters present in firstName                      |
| lastName                        | special_chars (422)     | Error: special characters present in lastName                       |
| nationality                     | invalid_value (422)     | Error: nationality must be a valid ISO alpha-2 code                 |
| countryOfOperations             | invalid_value (422)     | Error: countryOfOperations must be a valid ISO alpha-2 code         |
| certificateOfGoodStanding       | invalid_format (422)    | Error: certificateOfGoodStanding must be a valid URL                |
| businessLicense                 | invalid_format (422)    | Error: businessLicense must be a valid URL                          |
| registerShareholder             | invalid_format (422)    | Error: registerShareholder must be a valid URL                      |
| idShareholders                  | invalid_format (422)    | Error: idShareholders must be a valid URL                           |
| addressVerificationShareholders | invalid_format (422)    | Error: addressVerificationShareholders must be a valid URL          |

> **500 Internal Server Error** indicates an internal issue on our side. Please notify the PayCaddy team with the payload evidence so we can investigate promptly.


---

## **Merchant User SR <font color="sky-blue">GET</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v2/merchantUserSRs/

The GET call for a merchantUser allows you to know the stored data of a particular userId, especially the walletId of their initial wallet and the activity status of this user in the "isActive" field. Both data are crucial for the other calls to the NeoBank API.

This call can be used to verify the user's status at any point in the flow.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v2/merchantUserSRs/{MERCHANT_ID}
    ```
=== "Response"
    ```json
    {
		  "id": "string",
		  "email": "string",
		  "registeredName": "string",
		  "taxId": "string",
		  "legalRepresentation": "string",
		  "kindOfBusiness": "string",
		  "telephone": "string",
		  "address": {
		    "addressLine1": "string",
		    "addressLine2": "string",
		    "homeNumber": "string",
		    "city": "string",
		    "region": "string",
		    "postalCode": "string",
		    "country": "string"
		  },
		  "isActive": true,
		  "walletId": "string",
		  "firstName": "string",
		  "lastName": "string",
		  "nationality": "string",
		  "countryOfOperations": "string",
		  "certificateOfGoodStanding": "string",
		  "businessLicense": "string",
		  "registerShareholder": "string",
		  "idShareholders": "string",
		  "addressVerificationShareholders": "string",
		  "creationDate": "2025-10-13T21:04:11.970Z"
		}```

---
