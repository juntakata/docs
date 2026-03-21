# SAML Request Encoder

This is just a small sample program to generate SAML Request. This app takes 3 command line arguments and generate SAML Request parameter. Make sure that you have configured a corresponding SAML application on Azure AD using the matching tenant ID, ACS URL, and Entity ID URL.

Run the command as below.

```powershell
.\SAMLRequestEncoder.ps1 -TenantId "contoso.onmicrosoft.com" -AcsUrl "https://www.contoso.com/acs" -Issuer "https://contoso.onmicrosoft.com/samlapp01"
```

Save below as SamlRequestEncoder.ps1.

```powershell
param(
    [Parameter(Mandatory=$true)][string]$TenantId,
    [Parameter(Mandatory=$true)][string]$AcsUrl,
    [Parameter(Mandatory=$true)][string]$Issuer
)

function GenerateSAMLRequestParam([string]$SAMLRequest) {
    $bytes = [System.Text.Encoding]::UTF8.GetBytes($SAMLRequest)
    $output = New-Object System.IO.MemoryStream
    try {
        $zip = New-Object System.IO.Compression.DeflateStream($output, [System.IO.Compression.CompressionMode]::Compress)
        try {
            $zip.Write($bytes, 0, $bytes.Length)
        } finally {
            $zip.Dispose()
        }
        $base64 = [Convert]::ToBase64String($output.ToArray())
        return [System.Web.HttpUtility]::UrlEncode($base64)
    } finally {
        $output.Dispose()
    }
}

Add-Type -AssemblyName System.Web

$id = [BitConverter]::ToInt64([Guid]::NewGuid().ToByteArray(), 0)
$IssueInstant = [DateTime]::UtcNow.ToString("o")

Write-Host "Tenant ID: $TenantId"
Write-Host "Reply (ACS) URL: $AcsUrl"
Write-Host "Entity ID URI: $Issuer"

$SigninSAMLRequest = @"
<samlp:AuthnRequest xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="id$id" Version="2.0" IssueInstant="$IssueInstant" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" AssertionConsumerServiceURL="$AcsUrl">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">$Issuer</Issuer>
  <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified"/>
</samlp:AuthnRequest>
"@

$SigninEncodedString = GenerateSAMLRequestParam $SigninSAMLRequest
$Endpoint = "https://login.microsoftonline.com/$TenantId/saml2"
$SigninUrl = "$Endpoint`?SAMLRequest=$SigninEncodedString"

Write-Host "SigninUrl:"
Write-Host $SigninUrl
```
