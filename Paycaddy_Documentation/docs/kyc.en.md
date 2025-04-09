
## **Integrated KYC Notifications**

As part of our Integrated KYC flow for Natural Persons, after users complete the identity verification steps at the URL shared in the **kycURL** field **(see EndUser POST)**, our identity verification provider's system processes the submitted data and compares it with blacklists and does AML checks. Our API leverages these verifications and provides status notifications via webhook.

To properly receive notifications of KYC verifications, it is necessary to maintain an URL for receiving messages from the PayCaddy API via webhook. You must contact the PayCaddy integration team to configure the sending of notifications to that URL.

### **KYC Verification Webhooks**

The KYC (Know Your Customer) Validation webhook is a notification sent to customers with relevant information about the status of the KYC verification process. The information is delivered in JSON format and may include different states and descriptions, depending on the verification cases.

| **Field**      | **Description**      |
| ------------- | ------------- |
| **metadata** | Crucial information to identify the user in the form of their **userId** |
| **status** | The status of the KYC Verification, which can be "verified", "rejected" or "reviewNeeded" |
| **description** | A detailed description of the verification status. |
| **fullName** | The user's full name |
| **age** | The user's age |
| **timestamp** | The ISO 8601 timestamp of the webhooks response |

```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "string",
        "description": "string",
        "fullName": "string",
        "age": "string",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
```

### **Verification States and Descriptions**

The webhook flow for a validation follows the diagram described below.



![kycflow](./assets/imgs/kycflow.svg)
{class="img"}

Following this process flow, below are the webhooks corresponding to each state.
Each of these webhooks provide a description of the corresponding state, including rejection reasons in case of failed verifications. The corresponding descriptions are detailed below for your reference.

**‍Verification Inputs Completed:** This state indicates that a user has successfully completed KYC data capture through the **kycURL** and will have the structure and information shown below:

```json
    {
        "metadata": {
            "userId": "unique user identifier"
        },
        "status": "verification_inputs_completed",
        "description": "Ongoing verification",
        "fullName": "Document OCR capture ongoing",
        "age": "Document OCR capture ongoing",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
```

**Verified:** This state refers to when a user has successfully passed the KYC verification process. This indicates that the user has been successfully activated in our database and can proceed with other card creation and operations flows. The webhook response will have the following structure and information:

```json
    {
        "metadata": {
            "userId": "ID único del usuario"
        },
        "status": "verified",
        "description": "Verification signed",
        "fullName": "User's full name",
        "age": "User's age",
        "timeStamp": "ISO 8601 format timestamp"
    }
```

**Rejected:** This state refers to when a user has not passed the KYC verification process. The different rejection cases are detailed below.

=== "Input mismatch"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "Document doesn’t match input data",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
        }
    ```

=== "AML Checks Rejected"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "AML checks rejected",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "User underage"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "The user is underage",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "Document Expired"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "The document uploaded is expired",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "Negligence"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "The document uploaded has issues",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "Others"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "rejected",
        "description": "Validation Failed. User cannot be verified",
        "fullName": "User's complete name",
        "age": "null",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

**ReviewNeeded:** This state refers to when a user has not passed the KYC verification process. The different rejection cases are detailed below.

=== "Negligence"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "The document uploaded has issues",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```

=== "AML Checks Review"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "AML checks need to be reviewed",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```
=== "System Inoperative"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "Government validation system inoperative",
        "fullName": "User's complete name",
        "age": "User's age",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```
=== "Others"
    ```json
    {
        "metadata": {
            "userId": "string"
        },
        "status": "reviewNeeded",
        "description": "Validation failed. Pending compliance review",
        "fullName": "User's complete name",
        "age": "null",
        "timeStamp": "2023-03-16T09:42:18.8338086Z"
    }
    ```
