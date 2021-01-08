# Creating and accessing Azure Key Vault managed storage account

## Prerequisites

1. Azure CLI or Azure PowerShell Installed on the machine
2. A Storage Account created on Azure

## How to create Key Vault managed Storage Account Keys with with Azure CLI

1. Login with Azure CLI.

    ```text
    az login
    az account set --subscription <ID>
    ```

2. Get the resource ID of the storage account

    ```text
    az storage account show -n storageaccountname 
    ```

    ```text
    /subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/ResourceGroup/providers/Microsoft.Storage/storageAccounts/StorageAccountName
    ```

3. Find an object id of Azure Key Vault service principal. cfa8b339-82a2-471a-a3c9-0fc0be7a4093 is an application ID of Key Vault on Azure public cloud.

    ```text
    az ad sp show --id cfa8b339-82a2-471a-a3c9-0fc0be7a4093
    ```

    ```text
    "objectId": "bffbdcb7-0c5c-4765-96ad-1b78655a6ce1"
    ```

4. Set permission to the service principal object of Azure Key Vault

    ```text
    az role assignment create --role "Storage Account Key Operator Service Role" --assignee-object-id "bffbdcb7-0c5c-4765-96ad-1b78655a6ce1" --scope "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa"
    ```

    ```json
    {
      "canDelegate": null,
      "id": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa/providers/Microsoft.Authorization/roleAssignments/55a893ca-ce0e-44bc-a6dc-7a034bec112a",
      "name": "55a893ca-ce0e-44bc-a6dc-7a034bec112a",
      "principalId": "bffbdcb7-0c5c-4765-96ad-1b78655a6ce1",
      "resourceGroup": "keyvlt-prod-rg",
      "roleDefinitionId": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/providers/Microsoft.Authorization/roleDefinitions/81a9662b-bebf-436f-a333-f67b29880f12",
      "scope": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    ```

5. Create a Key Vault Managed Storage Account.

    ```text
    az keyvault storage add --vault-name keyvlt-prod-kv -n juntakatakvsa --active-key-name key1 --auto-regenerate-key --regeneration-period P90D --resource-id "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa"
    ```

    ```json
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

    ```text
    az storage account generate-sas --expiry 2020-01-01 --permissions rw --resource-types sco --services bfqt --https-only --account-name juntakatakvsa --account-key 00000000
    ```

    ```text
    "se=2020-01-01&sp=***gbeBWJaFUYLpbPa9hWP9HPA4Tno%3D"
    ```

7. Create a SAS Definition

    ```text
    az keyvault storage sas-definition create --vault-name keyvlt-prod-kv --account-name juntakatakvsa -n juntakatasasdefinition --validity-period P90D --sas-type account --template-uri "se=2020-01-01&sp=***gbeBWJaFUYLpbPa9hWP9HPA4Tno%3D"
    ```

    ```json
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

8. Run a command as below to regenerate storage account key

    ```text
    az keyvault storage regenerate-key --vault-name keyvlt-prod-kv --name juntakatakvsa --key-name key1
    ```

    ```json
    {
        "activeKeyName": "key1",
        "attributes": {
        "created": "2019-04-30T08:45:00+00:00",
        "enabled": true,
        "recoveryLevel": "Purgeable",
        "updated": "2019-04-30T08:45:00+00:00"
        },
        "autoRegenerateKey": true,
        "id": "https://keyvlt-prod-kv.vault.azure.net/storage/juntakatakvsa",
        "regenerationPeriod": "P90D",
        "resourceId": "/subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakvsa",
        "tags": null
    }
    ```

You can now see the Key Vault secret as below.

```text
az keyvault secret show --vault-name keyvlt-prod-kv --name juntakatakvsa-juntakatasasdefinition
```

```json
{
  "attributes": {
    "created": null,
    "enabled": true,
    "expires": "2019-09-16T07:36:38+00:00",
    "notBefore": null,
    "recoveryLevel": "Purgeable",
    "updated": null
  },
  "contentType": "application/vnd.ms-sastoken-storage",
  "id": "https://keyvlt-prod-kv.vault.azure.net/secrets/juntakatakvsa-juntakatasasdefinition",
  "kid": null,
  "managed": true,
  "tags": null,
  "value": "?sv=2018-03-28&***uJPV1lBdb8kk7EPXyU%3D"
}
```

## How to create Key Vault managed Storage Account Keys with PowerShell

1. Login with Azure PowerShell.

    ```text
    > Connect-AzAccount
    > Set-AzContext -SubscriptionId <ID>
    ```

2. Get a storage account

    ```text
    > $resourceGroupName = "keyvlt-prod-rg"
    > $storageAccountName = "juntakatakeyvltsa"
    > $storageAccountKey = "key1"
    > $keyVaultName = "keyvlt-prod-kv"
    > $keyVaultSpAppId = "cfa8b339-82a2-471a-a3c9-0fc0be7a4093"

    > $storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName
    ```

3. Set permission to the service principal object of Azure Key Vault

    ```text
    > New-AzRoleAssignment -ApplicationId $keyVaultSpAppId -RoleDefinitionName 'Storage Account Key Operator Service Role' -Scope $storageAccount.Id

    RoleAssignmentId   : /subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakeyvltsa/providers/Microsoft.Authorization/roleAssignments/b909a59b-9fe0-4312-8fe8-5b74b27d4563
    Scope              : /subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakeyvltsa
    DisplayName        : Azure Key Vault
    SignInName         :
    RoleDefinitionName : Storage Account Key Operator Service Role
    RoleDefinitionId   : 81a9662b-bebf-436f-a333-f67b29880f12
    ObjectId           : bffbdcb7-0c5c-4765-96ad-1b78655a6ce1
    ObjectType         : ServicePrincipal
    CanDelegate        : False
    ```

    ```text
    > Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName jutakata@jutakata02.onmicrosoft.com -PermissionsToStorage get, list, listsas, delete, set, update, regeneratekey, recover, backup, restore, purge, setsas
    ```

4. Create a Key Vault Managed Storage Account.

    ```text
    > $regenPeriod = [System.Timespan]::FromDays(90)

    > Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -RegenerationPeriod $regenPeriod


    Id                  : https://keyvlt-prod-kv.vault.azure.net:443/storage/juntakatakeyvltsa
    Vault Name          : keyvlt-prod-kv
    AccountName         : juntakatakeyvltsa
    Account Resource Id : /subscriptions/xxxxxxxx-4516-450e-a37f-76e65755465f/resourceGroups/keyvlt-prod-rg/providers/Microsoft.Storage/storageAccounts/juntakatakeyvltsa
    Active Key Name     : key1
    Auto Regenerate Key : True
    Regeneration Period : 3.00:00:00
    Enabled             : True
    Created             : 2019/06/18 9:32:08
    Updated             : 2019/06/18 9:32:08
    Tags                :
    ```

5. Create a SAS token

    ```text
    > $storagecontext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey key1

    > New-AzStorageAccountSASToken -ExpiryTime 2020-01-01 -Permission rw -ResourceType Service,Container,Object -Service Blob,File,Table,Queue -Protocol HttpsOnly -Context $storagecontext

    ?sv=2018-03-28&sig=***3A00Z&srt=sco&ss=bfqt&sp=rw
    ```

6. Create a SAS Definition

    ```text
    > Set-AzKeyVaultManagedStorageSasDefinition -VaultName $keyVaultName -StorageAccountName $storageAccountName -Name juntakatakeyvltsasdefinition -ValidityPeriod ([System.Timespan]::FromDays(30)) -SasType 'account' -TemplateUri "?sv=2018-03-28&sig=***3A00Z&srt=sco&ss=bfqt&sp=rw"


    Id          : https://keyvlt-prod-kv.vault.azure.net:443/storage/juntakatakeyvltsa/sas/juntakatakeyvltsasdefinition
    Secret Id   : https://keyvlt-prod-kv.vault.azure.net/secrets/juntakatakeyvltsa-juntakatakeyvltsasdefinition
    Vault Name  : keyvlt-prod-kv
    AccountName : juntakatakeyvltsa
    Name        : juntakatakeyvltsasdefinition
    Parameter   :
    Enabled     : True
    Created     : 2019/06/18 10:00:13
    Updated     : 2019/06/18 10:00:13
    Tags        :
    ```

Now the SAS definition is stored in Key Vault secret. You can see the Key Vault secret as below.

```text
> $value = Get-AzKeyVaultSecret -VaultName $keyVaultName -Name juntakatakeyvltsa-juntakatakeyvltsasdefinition

> $value.SecretValueText

?sv=2018-03-28&ss=***ADBr1wifv3CXZ7Rw%3D
```

## Access sotrage blob from PowerShell

```text
$context = New-AzStorageContext -StorageAccountName "juntakatakeyvltsa" -SasToken $value.SecretValueText
$context | Get-AzStorageBlob -Container "kvblob"

   Container Uri: https://juntakatakvsa.blob.core.windows.net/kvblob

Name                 BlobType  Length          ContentType                    LastModified         AccessTier SnapshotTime         IsDeleted
----                 --------  ------          -----------                    ------------         ---------- ------------         ---------
test.txt             BlockBlob 4               text/plain                     2019-12-05 17:41:28Z Hot                             False
   
$context | Get-AzStorageBlobContent -Container "kvblob" -Blob "test.txt" -Destination "C:\temp"
```

## Access storage blob using C# .NET Core Console Application

1. Open Visual Studio and create a new project.
2. From Tools menu, go to NuGet Package Manager and Package Manager Console
3. Run following commands

    ```text
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
