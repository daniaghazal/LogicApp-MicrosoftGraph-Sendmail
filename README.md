# LogicApp-MicrosoftGraph-Sendmail

## Description
This repository is a walkthrough that illustrates how to access Microsoft Graph API using Client Credentials Flow from a Logic App

## Prerequisites
Set up a [Microsoft 365 developer subscription](https://docs.microsoft.com/en-us/office/developer-program/microsoft-365-developer-program-get-started) by joining the [Microsoft 365 Developer Program](https://docs.microsoft.com/en-us/office/developer-program/microsoft-365-developer-program). You need this in order to ensure that users in your tenant are set up with the required O365 licenses and mailboxes for Outlook/Exchange.

## Keep in Mind
- Please note that the steps executed below are done via **Global Admin** user in the Azure AD tenant. The Azure AD tenant is available for you through M365 E5/E3 subscription
- You should be able to complete the steps with any user in that tenant, however, you will require **admin consent** when adding Microsoft Graph required permissions for your app
- You need to deploy Azure Key Vault API connection first before deploying the Logic App. You will need to pass the output of the former as an input parameter in the latter

## Steps

## 1. Register your app in Azure AD and add Microsoft Graph permissions

- In the Azure portal -> App registrations -> New registration. Let's call the app **test-graph-api**.
![msgraph-registered-app](/images/msgraph-registered-app.png)
![register-app](/images/register-app-in-AAD.png)
- In **test-graph-api**, go to API permissions -> Add a permission -> Microsoft Graph -> Application permissions -> Mail.Send -> Add permissions.
![api-permissions](/images/api-permissions.png)
- Click on *Grant admin consent*.
![admin-consent](/images/admin-consent.png)

## 2. Create a client secret for your app

- In **test-graph-api**, go to Certificates & secrets -> Client secrets -> New client secret -> *my-secret*. Copy the value of this secret as you will need to reference it later from Azure Key Vault.
![my-secret](/images/my-secret.png)
![client-secret](/images/client-secret.png)

## 3. Create Logic App flow to send email

1. Trigger the logic app on an HTTP call
2. Initialize clientId, clientSecret, tenantId variables and store values in them by referencing Azure Key Vault.

![logicapp-flow](/images/logicapp-flow.png)

  

  3. Add an action that makes an HTTP call to authenticate to Azure AD and obtain a bearer token
  
  ![http-get-token](/images/http-get-token.png)
  
   **URI**
    ```
    POST https://login.microsoftonline.com/<tenantId>/oauth2/v2.0/token
    ```

   **Headers**
    ```
    Content-Type: application/x-www-form-urlencoded
    ```

   **Body**
    ```
    client_id=<clientId>&scope=https://graph.microsoft.com/.default&client_secret=<clientSecret>&grant_type=client_credentials
    ```
  
  4. Parse HTTP response to retrieve token value from JSON object
  5. Add an action that makes an HTTP call to SendMail using Microsoft Graph API
  
  ![sendmail-msgraph](/images/sendmail-msgraph.png)
  
   **URI**
    ```
    https://graph.microsoft.com/v1.0/users/daniaghazal@contoso.com/sendMail
    ```

   **Headers**
    ```
    Authorization: concat('Bearer ', body('Parse_JSON')?['access_token'])
    Content-Type: application/json
    ```

   **Body**
    ```
    {
    "message": {
    "body": {
        "content": "The new cafeteria is open.",
        "contentType": "Text"
     },
    "subject": "Meet for lunch?",
    "toRecipients": [
      {
        "emailAddress": {
          "address": "daniaghazal@contoso.com"
        }
      }
     ]
    },
    "saveToSentItems": "false"
    }
    ```

## Output

Email received:
![email-received](/images/email-received.png)



## Resources
- [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program)
- [Use your developer subscription to build Microsoft 365 solutions](https://docs.microsoft.com/en-us/office/developer-program/build-microsoft-365-solutions)
- [Microsoft identity platform and the OAuth 2.0 client credentials flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
- [Microsoft Graph - Get access without a user](https://docs.microsoft.com/en-us/graph/auth-v2-service)
- [Microsoft Graph developer portal](https://developer.microsoft.com/en-us/graph/graph-explorer)
- [Azure Resource Manager (ARM)](https://docs.microsoft.com/en-us/azure/azure-resource-manager)
- [Azure Templates - Microsoft.Web connections](https://docs.microsoft.com/en-us/azure/templates/microsoft.web/connections?tabs=json)
