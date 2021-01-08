# SAML Request Encoder

This is just a small sample program to generate SAML Request. This app takes 3 command line arguments and generate SAML Request parameter. Make sure that you have configured a corresponding SAML application on Azure AD using the matching tenant ID, ACS URL, and Entity ID URL.

```csharp
using System;
using System.IO;
using System.IO.Compression;
using System.Text;
using System.Web;

namespace SAMLRequestEncoder
{
    class Program
    {
        static void Main(string[] args)
        {
            Guid guidValue = Guid.NewGuid();
            byte[] gb = Guid.NewGuid().ToByteArray();
            long id = BitConverter.ToInt64(gb, 0);

            DateTime dtUtcNow = DateTime.UtcNow;
            string IssueInstant = dtUtcNow.ToString("o");

            if (args.Length < 2)
            {
                Console.WriteLine("Example: SAMLRequestEncoder.exe <Tenant ID> <Reply (ACS) URL> <Entity ID URI>");
                Console.WriteLine("Example: SAMLRequestEncoder.exe contoso.onmicrosoft.com https://www.contoso.com/acs https://contoso.onmicrosoft.com/samlapp01");
            }
            else
            {
                Console.WriteLine("Tenant ID: " + args[0]);
                Console.WriteLine("Reply (ACS) URL: " + args[1]);
                Console.WriteLine("Entity ID URI: " + args[2]);
            }

            string tenant = args[0];
            string acsurl = args[1];
            string issuer = args[2];

            string SigninSAMLRequest =
                "<samlp:AuthnRequest" +
                " xmlns=\"urn:oasis:names:tc:SAML:2.0:metadata\"" +
                " xmlns:saml=\"urn:oasis:names:tc:SAML:2.0:assertion\"" +
                " ID=\"id" + id + "\"" +
                " Version=\"2.0\" IssueInstant=\"" + IssueInstant + "\"" +
                " xmlns:samlp=\"urn:oasis:names:tc:SAML:2.0:protocol\"" +
                " AssertionConsumerServiceURL=\"" + acsurl + "\">" +
                "  <Issuer xmlns=\"urn:oasis:names:tc:SAML:2.0:assertion\">" + issuer + "</Issuer>" +
                "  <samlp:NameIDPolicy Format=\"urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified\"/>" +
                "</samlp:AuthnRequest>";

            string SigninEncodedString = GenerateSAMLRequestParam(SigninSAMLRequest);
            string Endpoint = "https://login.microsoftonline.com/" + tenant + "/saml2";
            string SigninUrl = Endpoint + "?SAMLRequest=" + SigninEncodedString;

            Console.WriteLine("SigninUrl: ");
            Console.WriteLine(SigninUrl);
        }

        private static string GenerateSAMLRequestParam(string SAMLRequest)
        {
            var saml = string.Format(SAMLRequest, Guid.NewGuid());
            var bytes = Encoding.UTF8.GetBytes(saml);
            using (var output = new MemoryStream())
            {
                using (var zip = new DeflateStream(output, CompressionMode.Compress))
                {
                    zip.Write(bytes, 0, bytes.Length);
                }
                var base64 = Convert.ToBase64String(output.ToArray());
                return HttpUtility.UrlEncode(base64);
            }
        }
    }
}
```
