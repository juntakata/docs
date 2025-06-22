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

## Step 2: Export the Certificate in BASE64 Format

Once the certificate is created, export it from the certificate store in BASE64 format. Open the exported file in Notepad and copy the certificate body (the part between -----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----).

## Step 3: Register the Certificate with Entra ID for Federation

Use the following PowerShell command to register the certificate as a token-signing certificate for a federated domain in Entra ID. Replace the domain and URIs as needed:

```powershell
New-MgDomainFederationConfiguration -DomainId "sts.contoso.com" -ActiveSignInUri "https://sts.contoso.com/adfs/services/trust/2005/usernamemixed" -DisplayName "Contoso" -IssuerUri "http://sts.contoso.com/adfs/services/trust" -PassiveSignInUri "https://sts.contoso.com/adfs/ls/" -SignOutUri "https://sts.contoso.com/adfs/ls/" -SigningCertificate "MIIDCDCCAfCgAwIBAgIQIoSM...yczX2cgr6m3j/Pmep" -PreferredAuthenticationProtocol "saml" -FederatedIdPMfaBehavior "acceptIfMfaDoneByFederatedIdp" -MetadataExchangeUri "https://sts.contoso.com/adfs/services/trust/mex"
```

Replace <BASE64_CERT_HERE> with the actual BASE64-encoded certificate string.

## Step 4: Create a .NET Console App to Sign SAML Assertions

In Visual Studio, create a .NET console application. This sample uses the SignedXml class from System.Security.Cryptography.Xml to sign SAML assertions.

> [!NOTE]
> The System.Security.Cryptography.Xml package must be added to your project manually via NuGet.

```csharp
In the sample code:

The green section intentionally generates a multipleauthn element for SAML 1.1 inside a SAML 2.0 assertion.
This is to test how Entra ID handles such a scenario.
Normally, you should remove the green section and use the yellow section instead.

using System.Security.Cryptography.X509Certificates;
using System.Security.Cryptography.Xml;
using System.Text;
using System.Xml;

class Program
{
    static void Main()
    {
        // Define inputs
        string issuerUrl = "http://sts.contoso.com/adfs/services/trust";
        string nameId = "gPqi4EiVx0OzaxbNb/QeyQ==";

        string certificatePath = "C:\\cert.pfx";
        string certPassword = "xxxxxxxx";

        // Load certificate
        X509Certificate2 cert = new X509Certificate2(certificatePath, certPassword);

        // Generate SAML response
        string samlResponse = GenerateSamlResponse(issuerUrl, nameId, cert);

        var bytes = Encoding.UTF8.GetBytes(samlResponse);
        var base64 = Convert.ToBase64String(bytes);
        Console.WriteLine("Generated SAML Response:\n" + base64);
    }

    static string GenerateSamlResponse(string issuer, string nameId, X509Certificate2 cert)
    {
        var xmlDoc = new XmlDocument();

        var samlResponseElem = xmlDoc.CreateElement("samlp", "Response", "urn:oasis:names:tc:SAML:2.0:protocol");
        
        samlResponseElem.SetAttribute("Version", "2.0");
        samlResponseElem.SetAttribute("IssueInstant", DateTime.UtcNow.ToString("o"));
        samlResponseElem.SetAttribute("Destination", "https://login.microsoftonline.com/login.srf");
        samlResponseElem.SetAttribute("ID", "_" + Guid.NewGuid().ToString());
        samlResponseElem.SetAttribute("Consent", "urn:oasis:names:tc:SAML:2.0:consent:unspecified");

        var issuerElem = xmlDoc.CreateElement("", "Issuer", "urn:oasis:names:tc:SAML:2.0:assertion");
        issuerElem.InnerText = issuer;
        samlResponseElem.AppendChild(issuerElem);

        var statusElem = xmlDoc.CreateElement("samlp", "Status", "urn:oasis:names:tc:SAML:2.0:protocol");
        var statusCodeElem = xmlDoc.CreateElement("samlp", "StatusCode", "urn:oasis:names:tc:SAML:2.0:protocol");
        statusCodeElem.SetAttribute("Value", "urn:oasis:names:tc:SAML:2.0:status:Success");
        statusElem.AppendChild(statusCodeElem);
        samlResponseElem.AppendChild(statusElem);

        var assertionElem = xmlDoc.CreateElement("", "Assertion", "urn:oasis:names:tc:SAML:2.0:assertion");
        assertionElem.SetAttribute("Version", "2.0");
        assertionElem.SetAttribute("IssueInstant", DateTime.UtcNow.ToString("o"));
        assertionElem.SetAttribute("ID", "_93654ac3-f056-4a83-bda9-63828536aa08"); // same value as SessionIndex in AuthnStatement

        var assertionIssuerElem = xmlDoc.CreateElement("", "Issuer", "urn:oasis:names:tc:SAML:2.0:assertion");
        assertionIssuerElem.InnerText = issuer;
        assertionElem.AppendChild(assertionIssuerElem);

        var subjectElem = xmlDoc.CreateElement("", "Subject", "urn:oasis:names:tc:SAML:2.0:assertion");
        var nameIdElem = xmlDoc.CreateElement("", "NameID", "urn:oasis:names:tc:SAML:2.0:assertion");
        nameIdElem.SetAttribute("Format", "urn:oasis:names:tc:SAML:2.0:nameid-format:persistent");
        nameIdElem.InnerText = nameId;
        var subjectConfirmationElem = xmlDoc.CreateElement("", "SubjectConfirmation", "urn:oasis:names:tc:SAML:2.0:assertion");
        subjectConfirmationElem.SetAttribute("Method", "urn:oasis:names:tc:SAML:2.0:cm:bearer");
        var subjectConfirmationDataElem = xmlDoc.CreateElement("", "SubjectConfirmationData", "urn:oasis:names:tc:SAML:2.0:assertion");
        subjectConfirmationDataElem.SetAttribute("Recipient", "https://login.microsoftonline.com/login.srf");
        subjectConfirmationDataElem.SetAttribute("NotOnOrAfter", DateTime.UtcNow.AddMinutes(-5).AddHours(1).ToString("o"));
        subjectConfirmationElem.AppendChild(subjectConfirmationDataElem);
        subjectElem.AppendChild(nameIdElem);
        subjectElem.AppendChild(subjectConfirmationElem);
        assertionElem.AppendChild(subjectElem);

        var conditionElem = xmlDoc.CreateElement("", "Conditions", "urn:oasis:names:tc:SAML:2.0:assertion");
        conditionElem.SetAttribute("NotBefore", DateTime.UtcNow.AddMinutes(-5).ToString("o"));
        conditionElem.SetAttribute("NotOnOrAfter", DateTime.UtcNow.AddMinutes(-5).AddHours(1).ToString("o"));
        var audienceRestrictionElem = xmlDoc.CreateElement("", "AudienceRestriction", "urn:oasis:names:tc:SAML:2.0:assertion");
        var audienceElem = xmlDoc.CreateElement("", "Audience", "urn:oasis:names:tc:SAML:2.0:assertion");
        audienceElem.InnerText = "urn:federation:MicrosoftOnline";
        audienceRestrictionElem.AppendChild(audienceElem);
        conditionElem.AppendChild(audienceRestrictionElem);
        assertionElem.AppendChild(conditionElem);

        var attributeStatementElem = xmlDoc.CreateElement("", "AttributeStatement", "urn:oasis:names:tc:SAML:2.0:assertion");

        // Attribute Element 1
        var attributeElem1 = xmlDoc.CreateElement("", "Attribute", "urn:oasis:names:tc:SAML:2.0:assertion");
        attributeElem1.SetAttribute("Name", "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID");
        var AttributeValueElem1 = xmlDoc.CreateElement("", "AttributeValue", "urn:oasis:names:tc:SAML:2.0:assertion");
        AttributeValueElem1.InnerText = nameId;
        attributeElem1.AppendChild(AttributeValueElem1);
        attributeStatementElem.AppendChild(attributeElem1);

        // Attribute Element 2...

        // Add an AuthnStatement.
        var authnStatementElem = xmlDoc.CreateElement("", "AuthnStatement", "urn:oasis:names:tc:SAML:2.0:assertion");
        authnStatementElem.SetAttribute("AuthnInstant", DateTime.UtcNow.AddMinutes(-5).ToString("o"));
        authnStatementElem.SetAttribute("SessionIndex", "_93654ac3-f056-4a83-bda9-63828536aa08"); // same value as SessionIndex in AuthnStatement
        var authnContextElem = xmlDoc.CreateElement("", "AuthnContext", "urn:oasis:names:tc:SAML:2.0:assertion");
        var authnContextClassRefElem = xmlDoc.CreateElement("", "AuthnContextClassRef", "urn:oasis:names:tc:SAML:2.0:assertion");
        authnContextClassRefElem.InnerText = "http://schemas.microsoft.com/claims/multipleauthn";
        authnContextElem.AppendChild(authnContextClassRefElem);
        authnStatementElem.AppendChild(authnContextElem);
        assertionElem.AppendChild(authnStatementElem);

        samlResponseElem.AppendChild(assertionElem);

        xmlDoc.AppendChild(samlResponseElem);

        // Sign the assertion
        SignXml(xmlDoc, cert);

        return xmlDoc.OuterXml;
    }

    static void SignXml(XmlDocument doc, X509Certificate2 cert)
    {
        SignedXml signedXml = new SignedXml(doc);
        signedXml.SigningKey = cert.GetRSAPrivateKey();

        string id = doc.GetElementsByTagName("Assertion")[0].Attributes["ID"].Value;
        Reference reference = new Reference();
        reference.Uri = "#" + id;

        // Only enveloped signature transform
        XmlDsigEnvelopedSignatureTransform env = new XmlDsigEnvelopedSignatureTransform();
        reference.AddTransform(env);

        XmlDsigExcC14NTransform excC14N = new XmlDsigExcC14NTransform();
        reference.AddTransform(excC14N);
        signedXml.AddReference(reference);

        // KeyInfo
        KeyInfo keyInfo = new KeyInfo();
        keyInfo.AddClause(new KeyInfoX509Data(cert));
        signedXml.KeyInfo = keyInfo;

        // Exclusive canonicalization for SignedInfo
        signedXml.SignedInfo.CanonicalizationMethod = SignedXml.XmlDsigExcC14NTransformUrl;

        signedXml.ComputeSignature();

        XmlElement signatureElement = signedXml.GetXml();
        if (signatureElement.GetAttribute("xmlns:ds") == "")
            signatureElement.SetAttribute("xmlns:ds", "http://www.w3.org/2000/09/xmldsig#");

        // Insert Signature after Issuer
        XmlElement assertionElem = (XmlElement)doc.GetElementsByTagName("Assertion")[0];
        XmlElement issuerElem = null;
        foreach (XmlNode child in assertionElem.ChildNodes)
        {
            if (child is XmlElement elem && elem.LocalName == "Issuer")
            {
                issuerElem = elem;
                break;
            }
        }
        if (issuerElem != null)
        {
            assertionElem.InsertAfter(doc.ImportNode(signatureElement, true), issuerElem);
        }
    }
}
```

User BASE64 converter to check the output. If you find the assertion looks good, now embed it to the following HTML file and save it as a file.

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
