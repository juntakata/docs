# SAML Request Encoder

This is a small sample program to generate SAML Request. Make sure that you have configured a corresponding SAML application on Entra ID using the matching tenant ID, ACS URL, and Entity ID URL.

Run the command as below.

```powershell
.\SamlRequestEncoder.ps1 -RequestType SignIn `
  -TenantId 'tenant-id' `
  -Issuer 'app-entity-id' `
  -AcsUrl 'https://app.example.com/acs' `
  -PfxPath 'C:\certs\request-signing.pfx'
```

```powershell
.\SamlRequestEncoder.ps1 -RequestType SignOut `
  -TenantId 'tenant-id' -Issuer 'app-entity-id' `
  -NameId 'NameID-from-assertion' `
  -PfxPath 'C:\certs\request-signing.pfx'
```

Save below as SamlRequestEncoder.ps1.

```powershell
<#
.SYNOPSIS
Generates an optionally signed Microsoft Entra SAML HTTP-Redirect request URL.

.DESCRIPTION
SignIn is the default and requires AcsUrl. SignOut requires the NameID from
the successful sign-in assertion. Use the same TenantId and Issuer for both.
Optionally copy NameID Format from the assertion.
Supply original values, not XML-escaped values.

Supply PfxPath to sign either request type with RSA-SHA256. The PFX must contain
an RSA certificate and its private key. PfxPassword accepts a SecureString;
if omitted, the script prompts securely (press Enter for an empty password).
The key is loaded ephemerally, without importing it into a certificate store.
Signing uses the HTTP-Redirect query string, not an embedded XML signature.
Without PfxPath, the URL remains unsigned.

For Entra authentication-request verification, upload the matching public
certificate under the enterprise application's SAML Verification certificates
and enable Require verification certificates. Do not upload the PFX/private key.
Generating a signature does not by itself prove the identity provider verified it.

Open the generated URLs in the same browser session, completing sign-in first.
The script only generates URLs; it does not send requests or clear sessions.
Configure the application's Logout URL in Microsoft Entra ID and ensure that
the application handles SAML logout and clears its own session.

Verify that the LogoutResponse StatusCode is
urn:oasis:names:tc:SAML:2.0:status:Success and InResponseTo matches the printed
request ID. Also verify the application's session no longer grants access.
A later sign-in may silently authenticate via another SSO mechanism, so it
alone is not a logout check. With multiple active Entra sessions, the browser
may prompt for an account; NameID does not force selection of a session.

Reference: https://learn.microsoft.com/en-us/entra/identity-platform/single-sign-out-saml-protocol

.EXAMPLE
.\SamlRequestEncoder.ps1 -RequestType SignIn -TenantId 'tenant-id' -Issuer 'app-entity-id' -AcsUrl 'https://app.example.com/acs'

.EXAMPLE
.\SamlRequestEncoder.ps1 -RequestType SignOut -TenantId 'tenant-id' -Issuer 'app-entity-id' -NameId 'name-id-from-assertion'

.EXAMPLE
$password = Read-Host 'PFX password' -AsSecureString
.\SamlRequestEncoder.ps1 -RequestType SignOut -TenantId 'tenant-id' -Issuer 'app-entity-id' -NameId 'name-id-from-assertion' -PfxPath 'C:\certs\request-signing.pfx' -PfxPassword $password
#>
[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][string]$TenantId,
    [ValidateNotNullOrEmpty()][string]$AcsUrl,
    [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][string]$Issuer,
    [ValidateSet('SignIn', 'SignOut')][string]$RequestType = 'SignIn',
    [ValidateNotNullOrEmpty()][string]$NameId,
    [ValidateNotNullOrEmpty()][string]$NameIdFormat,
    [ValidateNotNullOrEmpty()][string]$PfxPath,
    [ValidateNotNull()][System.Security.SecureString]$PfxPassword
)

if ($RequestType -eq 'SignIn' -and [string]::IsNullOrWhiteSpace($AcsUrl)) {
    throw '-AcsUrl is required for -RequestType SignIn.'
}
if ($RequestType -eq 'SignOut' -and [string]::IsNullOrWhiteSpace($NameId)) {
    throw '-NameId from the sign-in assertion is required for -RequestType SignOut.'
}
if ($PSBoundParameters.ContainsKey('PfxPassword') -and -not $PSBoundParameters.ContainsKey('PfxPath')) {
    throw '-PfxPath is required when -PfxPassword is supplied.'
}

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

$id = 'id' + [Guid]::NewGuid().ToString('N')
$IssueInstant = [DateTime]::UtcNow.ToString("o")
$Endpoint = "https://login.microsoftonline.com/$([Uri]::EscapeDataString($TenantId))/saml2"
$XmlEndpoint = [System.Security.SecurityElement]::Escape($Endpoint)
$XmlIssuer = [System.Security.SecurityElement]::Escape($Issuer)

Write-Host "Request type: $RequestType"
Write-Host "Request ID: $id"
Write-Host "Tenant ID: $TenantId"
Write-Host "Entity ID URI: $Issuer"

if ($RequestType -eq 'SignIn') {
    $XmlAcsUrl = [System.Security.SecurityElement]::Escape($AcsUrl)
    Write-Host "Reply (ACS) URL: $AcsUrl"
    $SAMLRequest = @"
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="$id" Version="2.0" IssueInstant="$IssueInstant" Destination="$XmlEndpoint" AssertionConsumerServiceURL="$XmlAcsUrl">
  <saml:Issuer>$XmlIssuer</saml:Issuer>
  <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified"/>
</samlp:AuthnRequest>
"@
} else {
    $XmlNameId = [System.Security.SecurityElement]::Escape($NameId)
    $FormatAttribute = ''
    if ($PSBoundParameters.ContainsKey('NameIdFormat')) {
        $XmlNameIdFormat = [System.Security.SecurityElement]::Escape($NameIdFormat)
        $FormatAttribute = " Format=`"$XmlNameIdFormat`""
    }
    $SAMLRequest = @"
<samlp:LogoutRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="$id" Version="2.0" IssueInstant="$IssueInstant" Destination="$XmlEndpoint">
  <saml:Issuer>$XmlIssuer</saml:Issuer>
  <saml:NameID$FormatAttribute>$XmlNameId</saml:NameID>
</samlp:LogoutRequest>
"@
}

$EncodedString = GenerateSAMLRequestParam $SAMLRequest
$RequestQuery = "SAMLRequest=$EncodedString"

if ($PSBoundParameters.ContainsKey('PfxPath')) {
    $PfxFile = Get-Item -LiteralPath $PfxPath -ErrorAction Stop
    if ($PfxFile -isnot [System.IO.FileInfo]) {
        throw '-PfxPath must refer to a file.'
    }
    if (-not $PSBoundParameters.ContainsKey('PfxPassword')) {
        $PfxPassword = Read-Host 'PFX password (press Enter if none)' -AsSecureString
    }
    $Certificate = $null
    $PrivateKey = $null
    try {
        $Certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new(
            $PfxFile.FullName,
            $PfxPassword,
            [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::EphemeralKeySet
        )
        if (-not $Certificate.HasPrivateKey) {
            throw 'The PFX must contain a private key to sign the request.'
        }
        $PrivateKey = [System.Security.Cryptography.X509Certificates.RSACertificateExtensions]::GetRSAPrivateKey($Certificate)
        if ($null -eq $PrivateKey) {
            throw 'The PFX must contain an RSA private key for RSA-SHA256 signing.'
        }
        $SignatureAlgorithm = 'http://www.w3.org/2001/04/xmldsig-more#rsa-sha256'
        $EncodedAlgorithm = [System.Web.HttpUtility]::UrlEncode($SignatureAlgorithm)
        $RequestQuery += "&SigAlg=$EncodedAlgorithm"
        # Sign the exact encoded query bytes, before appending Signature.
        $Signature = $PrivateKey.SignData(
            [System.Text.Encoding]::ASCII.GetBytes($RequestQuery),
            [System.Security.Cryptography.HashAlgorithmName]::SHA256,
            [System.Security.Cryptography.RSASignaturePadding]::Pkcs1
        )
        $EncodedSignature = [System.Web.HttpUtility]::UrlEncode([Convert]::ToBase64String($Signature))
        $RequestQuery += "&Signature=$EncodedSignature"
    } finally {
        if ($null -ne $PrivateKey) { $PrivateKey.Dispose() }
        if ($null -ne $Certificate) { $Certificate.Dispose() }
    }
}

$RequestUrl = "$Endpoint`?$RequestQuery"

Write-Verbose $SAMLRequest
Write-Host "${RequestType}Url:"
Write-Output $RequestUrl
```
