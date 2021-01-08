# Creating appid and password programatically

You can create appid and password using Azure CLI programatically. See below how to create appid specifying password and how to sign-in to the app.

1. Sign in using "az login" with administrative account.
2. Run a command below to create an app ('Password' is used in this example.)

    ```text
    az ad app create --display-name 'ContosoAzCliApp' --native-app false --password 'Password' --identifier-uris 'https://contoso.onmicrosoft.com/ContosoAzCliApp'
    ```

    ```json
    {
        "acceptMappedClaims": null,
        "addIns": [],
        "allowGuestsSignIn": null,
        "allowPassthroughUsers": null,
        "appId": "1282bdfd-c03d-4dbe-ad70-9fca4f7fd418",
        "appLogoUrl": null,
        "appPermissions": null,
        "appRoles": [],
        "applicationTemplateId": null,
        "availableToOtherTenants": false,
        "deletionTimestamp": null,
        "displayName": "ContosoAzCliApp",
        "errorUrl": null,
        "groupMembershipClaims": null,
        "homepage": null,
        "identifierUris": [
            "https://contoso.onmicrosoft.com/ContosoAzCliApp"
        ],
        ...
    }
    ```

3. Copy the value of "appid"
4. Run a command below to create a corresponding service principal.

    ```text
    az ad sp create --id '1282bdfd-c03d-4dbe-ad70-9fca4f7fd418'
    ```

Now you can sign in to the app using the appid and password. Try a command below.

```text
az login --service-principal --password 'Password' --username '1282bdfd-c03d-4dbe-ad70-9fca4f7fd418' --tenant 'd7a85814-4d89-4baf-8a8a-f5be8306a959' --allow-no-subscriptions
```

See URLs below for reference.

az ad app create  
<https://docs.microsoft.com/en-us/cli/azure/ad/app?view=azure-cli-latest#az-ad-app-create>

az ad sp create  
<https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create>

az bot create  
<https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest#az-bot-create>
