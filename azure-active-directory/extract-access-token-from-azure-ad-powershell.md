# Extract access token from Azure AD PowerShell

You can extract an access token from Azure AD PowerShell. See an example below how to do it and access Azure AD Graph. Azure AD PowerShell acquires an access token for Azure AD Graph API, not for Microsoft Graph API.

The sample PowerShell script extracts an access token from Azure AD PowerShell and executes Azure AD Graph API to change service principal settings. It enables oauth2AllowIdTokenImplicitFlow and changes Reply URLs.

1. Install Azure AD PowerShell

    > Install-Module -Name AzureAD

2. Create a PowerShell script like below. Change first 4 lines for your environment.

    ```powershell
    $objectId = "6ad4eea4-65e8-4274-a5a8-d7a1c34a4c6f"
    $replyUrl1 = "https://contoso.azurewebsites.net/.auth/login/aad/callback"
    $replyUrl2 = "https://jwt.ms"
    $replyUrl3 = "https://localhost"

    Connect-AzureAD
    $token = [Microsoft.Open.Azure.AD.CommonLibrary.AzureSession]::AccessTokens['AccessToken']
    $headers = @{"Authorization" = "Bearer " + $token.AccessToken}

    $method = "PATCH"
    $contenttype = "application/json"
    $body = "
    {
        `"oauth2AllowIdTokenImplicitFlow`": `"true`",
        `"replyUrls`": [`"$replyUrl1`", `"$replyUrl2`", `"$replyUrl3`"]
    }"

    $url = "https://graph.windows.net/myorganization/applications/" + $objectId + "?api-version=1.6" 
    $responce = Invoke-WebRequest -Uri $url -Method $method -Headers $headers -ContentType $contenttype -Body $body
    ConvertFrom-Json $responce | ConvertTo-Json
    ```

3. If there is no output, the script completed successfully.