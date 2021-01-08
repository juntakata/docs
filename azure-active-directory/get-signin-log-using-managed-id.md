# Get sign-in log using managed id

To get sign-in log using Azure AD managed ID, you need to assign permissions beforehand.

## Assigning permission to Managed ID

```powershell
# Connect Azure AD PowerShell
Connect-AzureAD

# Get a service principal of Microsoft Graph (use 00000002-0000-0000-c000-000000000000 instead if you want to use Azure AD Graph)
$graph = Get-AzureADServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

# Get AuditLog.Read.All AppRole
$auditReadPermission = $graph.AppRoles | Where-Object Value -Like "AuditLog.Read.All" -and AllowedMemberTypes -contains 'Application' | Select-Object -First 1

# Get Directory.Read.All AppRole
$directoryReadPermission = $graph.AppRoles | Where-Object Value -Like "Directory.Read.All" | Select-Object -First 1

# Get MSI service principal
$msi = Get-AzureADServicePrincipal -ObjectId 5459d6c2-6933-4eb3-b206-fb3a042f2639

# Assign AuditLog.Read.All permission to MSI service principal
New-AzureADServiceAppRoleAssignment -Id $auditReadPermission.Id -ObjectId $msi.ObjectId -PrincipalId $msi.ObjectId -ResourceId $graph.ObjectId

ObjectId                                    ResourceDisplayName PrincipalDisplayName
--------                                    ------------------- --------------------
wtZZVDNps06yBvs6BC8mORZwqwQDRfRIlo3Hv7h-DWo Microsoft Graph     ws2019-prod-vm1

# Assign Directory.Read.All permission to MSI service principal
New-AzureADServiceAppRoleAssignment -Id $directoryReadPermission.Id -ObjectId $msi.ObjectId -PrincipalId $msi.ObjectId -ResourceId $graph.ObjectId

ObjectId                                    ResourceDisplayName PrincipalDisplayName
--------                                    ------------------- --------------------
wtZZVDNps06yBvs6BC8mOS_gePK-bQlDiONpD7ztGm4 Microsoft Graph     ws2019-prod-vm1
```

## Getting access token and sign-in log

Once you finished assigning permission to MSI service principal, you can get access token as below.

```powershell
# Send POST request to MSI token endpoint specifying Graph API resource ID
$tokenResponse = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=00000003-0000-0000-c000-000000000000' -Method GET -Headers @{Metadata="true"}

$tokenContent = $tokenResponse.Content | ConvertFrom-Json 

$graphToken = $tokenContent.access_token 

# Get 1 entry of sign-in log
$signinResponse = (Invoke-WebRequest -Uri 'https://graph.microsoft.com/v1.0/auditLogs/signIns?$top=1' -Method GET -Headers @{Authorization="Bearer $graphToken"}).content

$signinContent = $signinResponse | ConvertFrom-Json 

$signinContent | fl

@odata.context  : https://graph.microsoft.com/v1.0/$metadata#auditLogs/signIns
@odata.nextLink : https://graph.microsoft.com/v1.0/auditLogs/signIns?$top=1&$skiptoken=7ee59f6a1b3facf15510fd409988586e
                  _1
value           : {@{id=77253a0f-2f6a-4d67-bf59-a2123cfd2800; createdDateTime=2019-12-30T17:31:59.139761Z;
                  userDisplayName=Takata Jun; userPrincipalName=jutakata@jutakata02.onmicrosoft.com;
                  userId=6753a736-7efc-4187-a758-f6fa9c06efad; appId=0000000c-0000-0000-c000-000000000000;
                  appDisplayName=Microsoft App Access Panel; ipAddress=167.220.2.4; clientAppUsed=Browser;
                  correlationId=67a5be47-0310-48ce-b307-439735056c89; conditionalAccessStatus=notApplied;
                  isInteractive=False; riskDetail=none; riskLevelAggregated=none; riskLevelDuringSignIn=none;
                  riskState=none; riskEventTypes=System.Object[]; resourceDisplayName=Windows Azure Active Directory;
                  resourceId=00000002-0000-0000-c000-000000000000; status=; deviceDetail=; location=;
                  appliedConditionalAccessPolicies=System.Object[]}}
```
