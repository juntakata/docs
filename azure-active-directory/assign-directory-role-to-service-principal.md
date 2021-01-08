# Assign directory roles to a service principal

## Create an Azure service principal with Azure PowerShell

If you want to use password for sign-in credentials, use below.

```powershell
Import-Module Az.Resources # Imports the PSADPasswordCredential object
$credentials = New-Object Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential -Property @{ StartDate=Get-Date; EndDate=Get-Date -Year 2024; Password=<Choose a strong password>}
$sp = New-AzAdServicePrincipal -DisplayName ServicePrincipalName -PasswordCredential $credentials
```

For certificate-based authentication, use below.

```powershell
$cert = <public certificate as base64-encoded string>
$sp = New-AzADServicePrincipal -DisplayName ServicePrincipalName -CertValue $cert
```

The default role for a service principal is Contributor. This role has full permissions to read and write to an Azure account. The Reader role is more restrictive, with read-only access.

```powershell
New-AzRoleAssignment -ApplicationId <service principal application ID> -RoleDefinitionName "Reader"
Remove-AzRoleAssignment -ApplicationId <service principal application ID> -RoleDefinitionName "Contributor"
```

## Sign in using a service principal

To sign-in using a password, run a command below.

```powershell
$credentials = Get-Credential
Connect-AzAccount -ServicePrincipal -Credential $credentials -Tenant <tenant ID>
```

Certificate-based authentication requires that Azure PowerShell can retrieve information from a local certificate store based on a certificate thumbprint.

```powershell
Connect-AzAccount -ServicePrincipal -TenantId $tenantId -CertificateThumbprint <thumbprint>
```

## Assign directory roles

A service principal has no directory role by default. The Directory Reader role is restrictive, with read-only access.

```powershell
# Install Azure AD PowerShell
Install-Module -Name AzureAD

# Sign in with an administrator account
Connect-AzureAD

# Get the Id of the "Directory Readers" role
$roleId = (Get-AzureADDirectoryRole | where-object {$_.DisplayName -eq "Directory Readers"}).Objectid

# Get the Service Principal Object ID
$spObjectId = (Get-AzureADServicePrincipal -SearchString "spName").ObjectId

# Add service principal to the "Directory Readers" role
Add-AzureADDirectoryRoleMember -ObjectId $roleId -RefObjectId $spObjectId

# Check if SP is assigned to the Directory Readers role
Get-AzureADDirectoryRoleMember -ObjectId $roleId | Where-Object {$_.ObjectId -eq $spObjectId}
```

If you want to remove the Service Principal from the role at a later stage

```powershell
Remove-AzureADDirectoryRoleMember -ObjectId $roleId -MemberId $spObjectId
```
