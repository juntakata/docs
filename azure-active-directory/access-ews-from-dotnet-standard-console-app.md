# Access EWS from .NET Standard application

## Create an app on Azure AD

1. Go to Azure portal
2. Click [Azure Active Directory] and App registrations
3. Click [+ New registration]
4. Set display name
5. In Redirect URI, select [Public client (mobile & desktop)] and set "urn:ietf:wg:oauth:2.0:oob".
6. Press [Register] button
7. In Authentication blade, select "Yes" for Default client type option to allow ROPC flow.
8. Press [Save] button
9. Go to API permissions blade
10. Press [+ Add a permission]
11. Find Exchange from Supported legacy APIs section and press it
12. Select [Delegated permission]
13. press [Grant consent].
14. Enalbe [EWS.AccessAsUser.All]
15. Select [Add permissions]
16. Press [Grant admin consent for {your tenant name}]

## Build sample .NET Standard console application

```csharp
using System;
using Microsoft.Exchange.WebServices.Data;
using Microsoft.IdentityModel.Clients.ActiveDirectory;

namespace EwsOAuthApp
{
    class Program
    {
        static void Main(string[] args)
        {
            string authority = "https://login.microsoftonline.com/jutakata02.onmicrosoft.com";
            string clientId = "d00a4142-9b68-44f6-ac11-52659876e9d6";
            string resource = "https://outlook.office365.com";
            AuthenticationResult authenticationResult = null;
            AuthenticationContext authenticationContext = new AuthenticationContext(authority, false);

            UserPasswordCredential userCred = new UserPasswordCredential("clouduser@contoso.onmicrosoft.com", "password");


            string errorMessage = null;
            try
            {
                Console.WriteLine("Trying to acquire token");
                authenticationResult = authenticationContext.AcquireTokenAsync(resource, clientId, userCred).Result;
            }
            catch (AdalException ex)
            {
                errorMessage = ex.Message;
                if (ex.InnerException != null)
                {
                    errorMessage += "\nInnerException : " + ex.InnerException.Message;
                }
            }
            catch (ArgumentException ex)
            {
                errorMessage = ex.Message;
            }
            if (!string.IsNullOrEmpty(errorMessage))
            {
                Console.WriteLine("Failed: {0}" + errorMessage);
                return;
            }

            Console.WriteLine("\nMaking the protocol call\n");

            ExchangeService exchangeService = new ExchangeService(ExchangeVersion.Exchange2013);
            exchangeService.Url = new Uri(resource + "/ews/exchange.asmx");
            exchangeService.TraceEnabled = true;
            exchangeService.TraceFlags = TraceFlags.All;
            exchangeService.Credentials = new OAuthCredentials(authenticationResult.AccessToken);
            FindFoldersResults result = exchangeService.FindFolders(WellKnownFolderName.Root, new FolderView(10));

            // Process each item.
            foreach (Folder myFolder in result.Folders)
            {
                if (myFolder is SearchFolder)
                {
                    Console.WriteLine("Search folder: " + (myFolder as SearchFolder).DisplayName);
                }

                else if (myFolder is ContactsFolder)
                {
                    Console.WriteLine("Contacts folder: " + (myFolder as ContactsFolder).DisplayName);
                }

                else if (myFolder is TasksFolder)
                {
                    Console.WriteLine("Tasks folder: " + (myFolder as TasksFolder).DisplayName);
                }

                else if (myFolder is CalendarFolder)
                {
                    Console.WriteLine("Calendar folder: " + (myFolder as CalendarFolder).DisplayName);
                }
                else
                {
                    // Handle a generic folder.
                    Console.WriteLine("Folder: " + myFolder.DisplayName);
                }
            }

            // Determine whether there are more folders to return.
            if (result.MoreAvailable)
            {
                // Make recursive calls with offsets set for the FolderView to get the remaining folders in the result set.

            }
        }
    }
}
```
