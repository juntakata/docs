# Export Key Vault Certificate

You can programatically download certificates on Azure Key Vault with samples below. 

## C# sample code

This sample uses Client Credentials grant flow. You need to register an application on Azure AD before running this sample. Change client_id and client_secret according to your application.

```csharp
using System;
using System.Security.Cryptography.X509Certificates;
using System.Threading.Tasks;
using Microsoft.Azure.KeyVault;
using Microsoft.IdentityModel.Clients.ActiveDirectory;

namespace KeyVaultCertificate
{
    class Program
    {
        public static async Task<string> GetToken(string authority, string resource, string scope)
        {
            //string authority = "https://login.microsoftonline.com/contoso.onmicrosoft.com";
            string client_id = "aeddf8c8-acad-45ef-a4e6-a36eb7d0ea69";
            string client_secret = "kkv0tVS[Z-S";

            var authContext = new AuthenticationContext(authority);
            ClientCredential clientCred = new ClientCredential(client_id, client_secret);
            AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

            if (result == null)
                throw new InvalidOperationException("Failed to obtain the JWT token");

            return result.AccessToken;
        }

        static void Main(string[] args)
        {
            var keyVaultClient = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(GetToken));

            var certBundle = keyVaultClient.GetCertificateAsync("https://keyvlt-prod-kv.vault.azure.net", "test").Result;

            var secret = keyVaultClient.GetSecretAsync(certBundle.SecretIdentifier.Identifier).Result;
            var pfxb = Convert.FromBase64String(secret.Value);

            var certPriv = new X509Certificate2(rawData: pfxb,
                        password: "",
                        keyStorageFlags: X509KeyStorageFlags.MachineKeySet);

            var certPubl = new X509Certificate2(certBundle.Cer);
        }
    }
}
```

## PowerShell sample code

```powershell
$certName = "test"
$vaultName = "keyvlt-prod-kv1"
$password = "Password"

# Connect to Azure
Connect-AzAccount

# Get a certificate
$cert = Get-AzKeyVaultCertificate -VaultName $vaultName -Name $certName

# Build a partial secret id
$split = $cert.SecretId.split("/")
$secretIdParts = $split[4] + "/" + $split[5]

# Get a certificate secret
$secret = Get-AzKeyVaultSecret -VaultName $vaultName -Name $secretIdParts

$secretByte = [Convert]::FromBase64String($secret.SecretValueText)
$x509Cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2
$x509Cert.Import($secretByte, "", "Exportable,PersistKeySet")
$type = [System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx
$pfxFileByte = $x509Cert.Export($type, $password)

# Write to a file
[System.IO.File]::WriteAllBytes("KeyValt.pfx", $pfxFileByte)
```
