# How to generate a SAML assertion and sign it

## Step 1: Create a Self-Signed Certificate Using PowerShell

You can generate a self-signed certificate using the following PowerShell script:

```powershell
$cert = New-SelfSignedCertificate `
    -Subject "CN=SelfSignedCert" `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyExportPolicy Exportable `
    -KeySpec Signature
$cert
```

## Step 2: Export the Certificate as .cer (BASE64 format) and .pfx

1. Once the certificate is created, run "certmgr.msc" and export the certfiicate as .cer (BASE64 format) and .pfx
2. Open the exported .cer file in Notepad and copy the certificate body (the part between -----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----).

## Step 3: Register the Certificate with Entra ID for Federation

Use the following PowerShell command to register the certificate as a token-signing certificate for a federated domain in Entra ID. Replace the domain and URIs as needed and replace the value of the -SigningCertificate option with the actual BASE64-encoded certificate string you collected in the step 2.

```powershell
Connect-MgGraph -Scopes "Domain.ReadWrite.All"

New-MgDomainFederationConfiguration -DomainId "sts.contoso.com" -ActiveSignInUri "https://sts.contoso.com/adfs/services/trust/2005/usernamemixed" -DisplayName "Contoso" -IssuerUri "http://sts.contoso.com/adfs/services/trust" -PassiveSignInUri "https://sts.contoso.com/adfs/ls/" -SignOutUri "https://sts.contoso.com/adfs/ls/" -SigningCertificate "MIIDCDCCAfCgAwIBAgIQIoSM...yczX2cgr6m3j/Pmep" -PreferredAuthenticationProtocol "saml" -FederatedIdPMfaBehavior "acceptIfMfaDoneByFederatedIdp" -MetadataExchangeUri "https://sts.contoso.com/adfs/services/trust/mex"
```

## Step 4: Create a .NET Console App to Sign SAML Assertions

Save the following PowerShell script as SamlResponseGenerator.ps1.

```powershell
param(
    [Parameter(Mandatory=$true)][string]$IssuerUrl,
    [Parameter(Mandatory=$true)][string]$NameId,
    [Parameter(Mandatory=$true)][string]$CertificatePath,
    [Parameter(Mandatory=$true)][string]$CertPassword
)

Add-Type -AssemblyName System.Security.Cryptography.Xml

function Sign-Xml([System.Xml.XmlDocument]$doc, [System.Security.Cryptography.X509Certificates.X509Certificate2]$cert) {
    $signedXml = [System.Security.Cryptography.Xml.SignedXml]::new($doc)
    $signedXml.SigningKey = [System.Security.Cryptography.X509Certificates.RSACertificateExtensions]::GetRSAPrivateKey($cert)

    $id = $doc.GetElementsByTagName("Assertion")[0].Attributes["ID"].Value
    $reference = [System.Security.Cryptography.Xml.Reference]::new()
    $reference.Uri = "#$id"

    # Enveloped signature transform
    $env = [System.Security.Cryptography.Xml.XmlDsigEnvelopedSignatureTransform]::new()
    $reference.AddTransform($env)

    # Exclusive canonicalization transform
    $excC14N = [System.Security.Cryptography.Xml.XmlDsigExcC14NTransform]::new()
    $reference.AddTransform($excC14N)
    $signedXml.AddReference($reference)

    # KeyInfo with X509 certificate data
    $keyInfo = [System.Security.Cryptography.Xml.KeyInfo]::new()
    $keyInfo.AddClause([System.Security.Cryptography.Xml.KeyInfoX509Data]::new($cert))
    $signedXml.KeyInfo = $keyInfo

    # Exclusive canonicalization for SignedInfo
    $signedXml.SignedInfo.CanonicalizationMethod = [System.Security.Cryptography.Xml.SignedXml]::XmlDsigExcC14NTransformUrl

    $signedXml.ComputeSignature()

    $signatureElement = $signedXml.GetXml()
    if ($signatureElement.GetAttribute("xmlns:ds") -eq "") {
        $signatureElement.SetAttribute("xmlns:ds", "http://www.w3.org/2000/09/xmldsig#")
    }

    # Insert Signature after Issuer inside the Assertion
    $assertionElem = [System.Xml.XmlElement]$doc.GetElementsByTagName("Assertion")[0]
    $issuerElem = $null
    foreach ($child in $assertionElem.ChildNodes) {
        if ($child -is [System.Xml.XmlElement] -and $child.LocalName -eq "Issuer") {
            $issuerElem = $child
            break
        }
    }
    if ($null -ne $issuerElem) {
        $assertionElem.InsertAfter($doc.ImportNode($signatureElement, $true), $issuerElem) | Out-Null
    }
}

function New-SamlResponse([string]$issuer, [string]$nameId, [System.Security.Cryptography.X509Certificates.X509Certificate2]$cert) {
    $xmlDoc = [System.Xml.XmlDocument]::new()
    $nsProtocol = "urn:oasis:names:tc:SAML:2.0:protocol"
    $nsAssertion = "urn:oasis:names:tc:SAML:2.0:assertion"

    # Response element
    $samlResponseElem = $xmlDoc.CreateElement("samlp", "Response", $nsProtocol)
    $samlResponseElem.SetAttribute("Version", "2.0")
    $samlResponseElem.SetAttribute("IssueInstant", [DateTime]::UtcNow.ToString("o"))
    $samlResponseElem.SetAttribute("Destination", "https://login.microsoftonline.com/login.srf")
    $samlResponseElem.SetAttribute("ID", "_" + [Guid]::NewGuid().ToString())
    $samlResponseElem.SetAttribute("Consent", "urn:oasis:names:tc:SAML:2.0:consent:unspecified")

    # Issuer
    $issuerElem = $xmlDoc.CreateElement("", "Issuer", $nsAssertion)
    $issuerElem.InnerText = $issuer
    $samlResponseElem.AppendChild($issuerElem) | Out-Null

    # Status
    $statusElem = $xmlDoc.CreateElement("samlp", "Status", $nsProtocol)
    $statusCodeElem = $xmlDoc.CreateElement("samlp", "StatusCode", $nsProtocol)
    $statusCodeElem.SetAttribute("Value", "urn:oasis:names:tc:SAML:2.0:status:Success")
    $statusElem.AppendChild($statusCodeElem) | Out-Null
    $samlResponseElem.AppendChild($statusElem) | Out-Null

    # Assertion
    $assertionElem = $xmlDoc.CreateElement("", "Assertion", $nsAssertion)
    $assertionElem.SetAttribute("Version", "2.0")
    $assertionElem.SetAttribute("IssueInstant", [DateTime]::UtcNow.ToString("o"))
    $assertionElem.SetAttribute("ID", "_93654ac3-f056-4a83-bda9-63828536aa08") # same value as SessionIndex in AuthnStatement

    # Assertion Issuer
    $assertionIssuerElem = $xmlDoc.CreateElement("", "Issuer", $nsAssertion)
    $assertionIssuerElem.InnerText = $issuer
    $assertionElem.AppendChild($assertionIssuerElem) | Out-Null

    # Subject
    $subjectElem = $xmlDoc.CreateElement("", "Subject", $nsAssertion)
    $nameIdElem = $xmlDoc.CreateElement("", "NameID", $nsAssertion)
    $nameIdElem.SetAttribute("Format", "urn:oasis:names:tc:SAML:2.0:nameid-format:persistent")
    $nameIdElem.InnerText = $nameId
    $subjectConfirmationElem = $xmlDoc.CreateElement("", "SubjectConfirmation", $nsAssertion)
    $subjectConfirmationElem.SetAttribute("Method", "urn:oasis:names:tc:SAML:2.0:cm:bearer")
    $subjectConfirmationDataElem = $xmlDoc.CreateElement("", "SubjectConfirmationData", $nsAssertion)
    $subjectConfirmationDataElem.SetAttribute("Recipient", "https://login.microsoftonline.com/login.srf")
    $subjectConfirmationDataElem.SetAttribute("NotOnOrAfter", [DateTime]::UtcNow.AddMinutes(-5).AddHours(1).ToString("o"))
    $subjectConfirmationElem.AppendChild($subjectConfirmationDataElem) | Out-Null
    $subjectElem.AppendChild($nameIdElem) | Out-Null
    $subjectElem.AppendChild($subjectConfirmationElem) | Out-Null
    $assertionElem.AppendChild($subjectElem) | Out-Null

    # Conditions
    $conditionElem = $xmlDoc.CreateElement("", "Conditions", $nsAssertion)
    $conditionElem.SetAttribute("NotBefore", [DateTime]::UtcNow.AddMinutes(-5).ToString("o"))
    $conditionElem.SetAttribute("NotOnOrAfter", [DateTime]::UtcNow.AddMinutes(-5).AddHours(1).ToString("o"))
    $audienceRestrictionElem = $xmlDoc.CreateElement("", "AudienceRestriction", $nsAssertion)
    $audienceElem = $xmlDoc.CreateElement("", "Audience", $nsAssertion)
    $audienceElem.InnerText = "urn:federation:MicrosoftOnline"
    $audienceRestrictionElem.AppendChild($audienceElem) | Out-Null
    $conditionElem.AppendChild($audienceRestrictionElem) | Out-Null
    $assertionElem.AppendChild($conditionElem) | Out-Null

    # AttributeStatement
    $attributeStatementElem = $xmlDoc.CreateElement("", "AttributeStatement", $nsAssertion)

    # Attribute: ImmutableID
    $attributeElem1 = $xmlDoc.CreateElement("", "Attribute", $nsAssertion)
    $attributeElem1.SetAttribute("Name", "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID")
    $attributeValueElem1 = $xmlDoc.CreateElement("", "AttributeValue", $nsAssertion)
    $attributeValueElem1.InnerText = $nameId
    $attributeElem1.AppendChild($attributeValueElem1) | Out-Null
    $attributeStatementElem.AppendChild($attributeElem1) | Out-Null
    $assertionElem.AppendChild($attributeStatementElem) | Out-Null

    # AuthnStatement
    $authnStatementElem = $xmlDoc.CreateElement("", "AuthnStatement", $nsAssertion)
    $authnStatementElem.SetAttribute("AuthnInstant", [DateTime]::UtcNow.AddMinutes(-5).ToString("o"))
    $authnStatementElem.SetAttribute("SessionIndex", "_93654ac3-f056-4a83-bda9-63828536aa08") # same value as Assertion ID
    $authnContextElem = $xmlDoc.CreateElement("", "AuthnContext", $nsAssertion)
    $authnContextClassRefElem = $xmlDoc.CreateElement("", "AuthnContextClassRef", $nsAssertion)
    $authnContextClassRefElem.InnerText = "http://schemas.microsoft.com/claims/multipleauthn"
    $authnContextElem.AppendChild($authnContextClassRefElem) | Out-Null
    $authnStatementElem.AppendChild($authnContextElem) | Out-Null
    $assertionElem.AppendChild($authnStatementElem) | Out-Null

    $samlResponseElem.AppendChild($assertionElem) | Out-Null
    $xmlDoc.AppendChild($samlResponseElem) | Out-Null

    # Sign the assertion
    Sign-Xml $xmlDoc $cert

    return $xmlDoc.OuterXml
}

# Load certificate
$cert = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($CertificatePath, $CertPassword)

# Generate SAML response
$samlResponse = New-SamlResponse $IssuerUrl $NameId $cert

$bytes = [System.Text.Encoding]::UTF8.GetBytes($samlResponse)
$base64 = [Convert]::ToBase64String($bytes)
Write-Host "Generated SAML Response:`n$base64"
```

To run the command:

```powershell
.\SamlResponseGenerator.ps1 -IssuerUrl "http://sts.contoso.com/adfs/services/trust" -NameId "<on-premises immutable ID>" -CertificatePath "<pfx path>" -CertPassword "<pfx password>"
```

Use BASE64 converter to check the output. If you find the assertion looks good, now embed it to the following HTML file and save it as a file.

```html
<html>

<head>
    <title>Working...</title>
</head>

<body>
    <form method="POST" name="hiddenform" action="https://login.microsoftonline.com/login.srf"><input type="hidden"
            name="SAMLResponse"
            value="PHNhbWxw...mVzcG9uc2U+" /><noscript>
            <p>Script is disabled. Click Submit to continue.</p><input type="submit" value="Submit" />
        </noscript></form>
    <script language="javascript" nonce='CPrO7H9eidpFY1Vn_JTYiQ'>document.forms[0].submit();</script>
</body>

</html>
```

By openning this file in a browser, it performs an IdP-initiated sign-on, directly sending the SAML assertion to Entra ID. If the certificate matches, the sign-in will succeed, typically showing either a speed bump (CMSI) or Keep Me Signed In (KMSI) prompt.
