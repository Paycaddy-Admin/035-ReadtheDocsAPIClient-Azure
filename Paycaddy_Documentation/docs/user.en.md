!!!Important
    **Future Deprecation Notice:**
    Since October 15th 2025,  these endpoints have initiated their deprecation process as part of an effort to migrate all user creation from v1 into our [v2 endpoints](./userv2.en.md). Official communications will handle the exact timelines for adoption. If you're integrating PayCaddy's system for the first time, refer to our [v2 endpoints](./userv2.en.md) for the currently accepted schemas for new projects.

## **End User <font color="green">POST</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/endUsers

‍The creation of a new user for a natural person begins with a POST call in which an endpoint is consumed for sending the user's basic information:

=== "Request"
    ```json
        {
            "email": "string",
            "firstName": "string",
            "lastName": "string",
            "occupation": "string",
            "placeofwork": "string",
            "pep": "bool",
            "salary": "integer",
            "telephone": "string",
            "address": {
            "addressLine1": "string",
            "addressLine2": "string",
            "homeNumber": "string",
            "city": "string",
            "region": "string",
            "postalCode": "string",
            "country": "string"
            }
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
            "isActive": false,
            "walletId": "string",
            "kycUrl": "string",
            "creationDate": "2022-07-13T21:07:21.166Z"
        }
    ```
It should be noted that the responsibility for validating the accuracy and format of the entered data falls on the PayCaddy client, meaning that our API will return a successful response as long as the following parameters are met, regardless of the accuracy of the information or duplication of the shared data:

1. The First Name and Last Name fields must add up to a maximum total of 22 characters that must be sent in a sanitized form, removing special characters and limiting themselves to the [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en) range, which is a slightly more restrictive subset of ASCII.
2. None of the fields should be sent as NULL.
3. The email field must follow a standard email format.

In addition to the format verifications, it is important to highlight the responsibility of the client to consistently send the remaining fields with correct information:

- The **"salary"** field should include the user's monthly salary in USD.
- The **"pep"** field indicates whether the natural person for whom the user is being created is politically exposed.
  - **PEP**: Someone who has been entrusted with a prominent public responsibility. Typically, a PEP presents a higher risk of potential involvement in bribery and corruption by virtue of their position and influence.

The control over user duplication must be managed by the PayCaddy client. Our API will generate multiple distinct **userIds** regardless of whether the shared data is identical in duplicate calls.

If this event does not generate errors, the system will respond with an **HTTP 200 OK** message. The successful response replicates the entered data and adds information about the **isActive** parameters and the initial **walletId** associated with the userId, as well as loading a timestamp of the creation date of said user.

If there is an error, the system will respond with one of the following error messages with status code 400:

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
=== "Character Limit"
    ```json
    {
        "type": "",
        "title": "The value provided for the name exceeds the maximum value",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```
=== "Email Format"
    ```json
    {
        "type": "",
        "title": "Invalid format for email",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```
>In case you encounter an error message 500, this could indicate an internal server error in the system. If you face this situation, we ask that you inform the PayCaddy team so that we can investigate and solve the problem efficiently.

Once the user and wallet identifiers have been assigned, the end-user must complete the KYC verification in order to complete the profile creation procedure. Until this verification is completed, the "isActive" control field will remain assigned as false, disabling any procedures.

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

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/endUsers/

‍The GET call for an endUser allows you to know the stored data of a particular userId, especially the walletId of their initial wallet and the activity status of this user in the "isActive" field. Both data are crucial for the other calls to the NeoBank API.

This call can be used to verify the user's status at any point in the flow.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v1/endUsers/${USER_ID}
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
        "kycUrl": "string",
        "creationDate": "2023-03-22T19:13:46.178Z"
    }
    ```

---

## **Merchant User <font color="green">POST</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/merchantUsers

The creation of a new user for a legal entity begins with a POST call in which an endpoint is consumed for sending the basic data of the legal entity.

=== "Request"
    ```json
        {
            "email": "string",
            "registeredName": "string",
            "numOfRegistration": "string",
            "ruc": "string",
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
            }
        }

    ```
=== "Response"
    ```json```
    {
        "id": "string",
        "email": "string",
        "registeredName": "string",
        "numOfRegistration": "string",
        "ruc": "string",
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
        "creationDate": "2022-07-18T22:29:45.914Z"
    }
    ```

It should be noted that the responsibility for validating the accuracy and format of the entered data falls on the PayCaddy client, meaning that our API will return a successful response as long as the following parameters are met, regardless of the accuracy of the information or duplication of the shared data:

1. The First Name and Last Name fields must add up to a maximum total of 22 characters that must be sent in a sanitized form, removing special characters and limiting themselves to the [ITU-T50](https://www.itu.int/rec/T-REC-T.50/en) range, which is a slightly more restrictive subset of ASCII.
2. None of the fields should be sent as NULL.
3. The email field must follow a standard email format.

In addition to the format verifications, it is important to highlight the responsibility of the client to consistently send the remaining fields with correct information:

1. The field **"RUC"** must include the number of the Unique Taxpayer Registry or its international counterpart for tax identification such as VATIN, CUIT, CNPJ, NIT, or RUT.
2. The **"numOfRegistration"** field must include the registration number of the legal entity as determined by the institution. This field can replicate the data from "RUC" in case the information is not applicable in your jurisdiction.
3. The **"kindOfBusiness"** field should enter a brief description of the economic activity performed by the legal entity. The NeoBank API does not validate the detailed information provided in this field; however, it is crucial to ensure the submission of accurate information as it will be used by the compliance team in evaluating the user.

If this event does not generate errors, the system will respond with an HTTP 200 OK message. The successful response replicates the entered data and adds information about the isActive parameters and the initial walletId associated with the userId, as well as loading a timestamp of the creation date of said user.

>Controls for user duplication must be managed by integrating partners. Our API will generate multiple distinct userIds, regardless of whether the shared data is identical in duplicate calls.

If there is an error, the system will respond with one of the following error messages with status code 400:

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
=== "Character Limit"
    ```json
    {
        "type": "",
        "title": "The value provided for the name exceeds the maximum value",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```
=== "Email Format"
    ```json
    {
        "type": "",
        "title": "Invalid format for email",
        "status": 0,
        "detail": "",
        "instance": ""
    }
    ```

>In case you encounter an error message 500, this could indicate an internal server error in the system. If you face this situation, we ask that you inform the PayCaddy team so that we can investigate and solve the problem efficiently.

---

## **Merchant User <font color="sky-blue">GET</font>**

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/merchantUsers/

The GET call for a merchantUser allows you to know the stored data of a particular userId, especially the walletId of their initial wallet and the activity status of this user in the "isActive" field. Both data are crucial for the other calls to the NeoBank API.

This call can be used to verify the user's status at any point in the flow.

=== "Request"
    ```
     https://api.api-sandbox.paycaddy.dev/v1/merchantUsers/{MERCHANT_ID}
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
        "kycUrl": "string",
        "creationDate": "2023-03-22T19:13:46.178Z"
    }
    ```

---

## **End User SR <font color="green">POST</font>** 

**Request URL:**  https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs

The creation of a new user for a natural person with a delegated KYC flow begins with a POST call in which an endpoint is consumed for sending the basic data of the user:

=== "Request"
    ```json
    {
        "email": "string",
        "firstName": "string",
        "lastName": "string",
        "occupation": "string",
        "placeofwork": "string",
        "pep": "bool",
        "salary": "integer",
        "telephone": "string",
        "address": {
            "addressLine1": "string",
            "addressLine2": "string",
            "homeNumber": "string",
            "city": "string",
            "region": "string",
            "postalCode": "string",
            "country": "string",
        }
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
            "homeNumber" : "string",
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
        "creationDate": "2022-07-13T21:07:21.166Z"
    }
    ```

The fields "idUrlFront", "idUrlBack", and "residenceProofUrl" must be sent with unique URLs where images of the front and back of the user's national identity card and a proof of residence are stored. The specific parameters for uploading these documents must have been previously agreed with the compliance department of PayCaddy.

Users created under this flow are born with an active status but are subject to blockages in case the KYC information sent in the creation call is rejected by the compliance team of PayCaddy.

It should be noted that the responsibility for validating the accuracy and format of the entered data falls on the PayCaddy client, meaning that our API will return a successful response as long as the following parameters are met, regardless of the accuracy of the information or duplication of the shared data:

1. The First Name and Last Name fields must add up to a maximum total of 22 characters that must be sent in a sanitized form, removing special characters and limiting themselves to the ASCII range.
2. None of the fields should be sent as NULL.
3. The email field must follow a standard email format.

In addition to the format verifications, it is important to highlight the responsibility of the client to consistently send the remaining fields with correct information:

1. The "salary" field should include the user's monthly salary in USD.
2. The "pep" field indicates whether the natural person for whom the user is being created is politically exposed.

PEP: Someone who has been entrusted with a prominent public responsibility. Typically, a PEP presents a higher risk of potential involvement in bribery and corruption by virtue of their position and influence.

>The control over user duplication must be managed by the PayCaddy client. Our API will generate multiple distinct userIds regardless of whether the shared data is identical in duplicate calls.

If this event does not generate errors, the system will respond with an HTTP 200 OK message. The successful response replicates the entered data and adds information about the **isActive** parameters and the initial **walletId** associated with the **userId**, as well as loading a timestamp of the creation date of said user.

---

## **End User SR <font color="sky-blue">GET</font>** 

**Request URL:** https://api.api-sandbox.paycaddy.dev/v1/SR/EndUserSRs/

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
        "creationDate": "2024-11-12T11:57:38.644Z"
    }
    ```
