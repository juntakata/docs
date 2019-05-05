# Creating and accessing Azure Key Vault managed storage account with Azure CLI

## Prerequisites

1. Azure CLI Installed on the machine
2. A Storage Account created on Azure

## Instructions on how to use Key Vault to manage Storage Account Keys

1. Login with Azure CLI.

    ```
    $ az login
    $ az account set --subscription <ID>
    ```
2.  Get the resource ID of the storage account

    ```
    $ az storage account show -n storageaccountname 
    ```
    
    ```
    /subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/ResourceGroup/providers/Microsoft.Storage/storageAccounts/StorageAccountName
    ```

3. Find an object id of Azure Key Vault service principal. cfa8b339-82a2-471a-a3c9-0fc0be7a4093 is an application ID of Key Vault on Azure public cloud.

    ```
    $ az ad sp show --id cfa8b339-82a2-471a-a3c9-0fc0be7a4093
    ```
    ```
    "objectId": "bffbdcb7-0c5c-4765-96ad-1b78655a6ce1"
    ```

4. Set permission to the service principal object of Azure Key Vault

    ```
    $ az role assignment create --role "Storage Account Key Operator Service Role" --assignee-object-id "bffbdcb7-0c5c-4765-96ad-1b78655a6ce1" --scope "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa"
    ```
    ```
    {
      "canDelegate": null,
      "id": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa/providers/Microsoft.Authorization/roleAssignments/55a893ca-ce0e-44bc-a6dc-7a034bec112a",
      "name": "55a893ca-ce0e-44bc-a6dc-7a034bec112a",
      "principalId": "bffbdcb7-0c5c-4765-96ad-1b78655a6ce1",
      "resourceGroup": "keyvlt-prod-rg",
      "roleDefinitionId": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/providers/Microsoft.Authorization/roleDefinitions/81a9662b-bebf-436f-a333-f67b29880f12",
      "scope": "/subscriptions/543e42fb-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    ```

5. Create a Key Vault Managed Storage Account.

    ```
    $ az keyvault storage add --vault-name keyvlt-prod-kv1 -n juntakatakvsa --active-key-name key1 --auto-regenerate-key --regeneration-period P90D --resource-id "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa"
    ```
    ```
    {
      "activeKeyName": "key1",
      "attributes": {
        "created": "2019-04-30T08:43:00+00:00",
        "enabled": true,
        "recoveryLevel": "Purgeable",
        "updated": "2019-04-30T08:43:00+00:00"
      },
      "autoRegenerateKey": true,
      "id": "https://keyvlt-prod-kv.vault.azure.net/storage/juntakatakvsa",
      "regenerationPeriod": "P90D",
      "resourceId": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa",
      "tags": null
    }
    ```

6. Create a SAS token

    ```
    $ az storage account generate-sas --expiry 2020-01-01 --permissions rw --resource-types sco --services bfqt --https-only --account-name juntakatakvsa --account-key 00000000
    ```
    ```
    "se=2020-01-01&sp=***gbeBWJaFUYLpbPa9hWP9HPA4Tno%3D"
    ```

7. Create a SAS Definition

    ```
    $ az keyvault storage sas-definition create --vault-name keyvlt-prod-kv --account-name juntakatakvsa -n juntakatasasdefinition --validity-period P90D --sas-type account --template-uri "se=2020-01-01&sp=***gbeBWJaFUYLpbPa9hWP9HPA4Tno%3D"
    ```
    ```
    {
      "attributes": {
        "created": "2019-05-01T00:34:16+00:00",
        "enabled": true,
        "recoveryLevel": "Purgeable",
        "updated": "2019-05-01T00:34:16+00:00"
      },
      "id": "https://keyvlt-prod-kv.vault.azure.net/storage/juntakatakvsa/sas/juntakatasasdefinition",
      "sasType": "account",
      "secretId": "https://keyvlt-prod-kv.vault.azure.net/secrets/juntakatakvsa-juntakatasasdefinition",
      "tags": null,
      "templateUri": "se=2020-01-01&sp=***gbeBWJaFUYLpbPa9hWP9HPA4Tno%3D",
      "validityPeriod": "P90D"
    }
    ```

## Create C# .NET Core Console Application

1. Open Visual Studio and create a new project.
2. From Tools menu, go to NuGet Package Manager and Package Manager Console
3. Run following commands

    ```
    Install-Package Microsoft.Azure.KeyVault -Version 3.0.3
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    Install-Package WindowsAzure.Storage
    ```

4. Copy and paste a code below.

    ```csharp
    using Microsoft.Azure.KeyVault;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using System;
    using System.Threading.Tasks;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;

    namespace FetchSasTokens
    {
        class Program
        {
            static string authority = "https://login.microsoftonline.com/contoso.onmicrosoft.com";
            static string resource = "https://vault.azure.net";
            static string clientId = "xxxxxxxx-e1f5-4081-9b6e-0ce278355b5b";
            static string secret = "W@i3[UXv0***vB.4u1D";

            static void Main(string[] args)
            {
                // Once you have a security token from one of the above methods, then create KeyVaultClient with vault credentials
                var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback((authority, resource, scope) =>
                            GetAccessToken(authority, resource, scope, clientId, secret)));

                // Get a SAS token for our storage from Key Vault. SecretUri is of the format https://<VaultName>.vault.azure.net/secrets/<ExamplePassword>
                var sasToken = kv.GetSecretAsync("https://keyvlt-prod-kv.vault.azure.net/secrets/juntakatakvsa-juntakatasasdefinition").Result;

                // Create new storage credentials using the SAS token.
                var accountSasCredential = new StorageCredentials(sasToken.Value);

                // Use the storage credentials and the Blob storage endpoint to create a new Blob service client.
                var accountWithSas = new CloudStorageAccount(accountSasCredential, new Uri("https://juntakatakvsa.blob.core.windows.net/"), null, null, null);

                var blobClientWithSas = accountWithSas.CreateCloudBlobClient();
                var container = blobClientWithSas.GetContainerReference("kvblob");
                var blockBlob = container.GetBlockBlobReference("test.txt");

                string txt = blockBlob.DownloadTextAsync().Result;
                Console.WriteLine("Contents of the file: " + txt);
            }

            private static async Task<string> GetAccessToken(string authority, string resource, string scope, string cliantId, string secret)
            {
                var authContext = new AuthenticationContext(authority);
                ClientCredential clientCred = new ClientCredential(cliantId, secret);
                AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

                if (result == null)
                {
                    throw new InvalidOperationException("Failed to obtain the JWT token");
                }

                return result.AccessToken;
            }
        }
    }
    ```