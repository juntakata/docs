# How to start and gain Office 365 Management Activity log

## Create an app on Azure AD

1. Go to Azure portal
2. Go to Azure Active Directory blade
3. Select [App registrations]
4. Select [+ New registration]
5. Specify new name and select [Public client (mobile & desktop)]
6. Specify <https://localhost> as a redirect URI.
7. Select [Register]
8. Copy Application (client) ID
9. Go to [API permissions]
10. Select [+ Add a permission]
11. Select [Office 365 Management APIs]
12. Select [Delegated permissions]
13. Enable [ActivityFeed.Read], [ServiceHealth.Read], and [ActivityFeed.ReadDlp]
14. Select [Add permissions]
15. Press [Grand admin consent for {tenant name}]

## Acquire token

Request

```text
https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/token

password: Password
grant_type: password
username: testuser@contoso.onmicrosoft.com
resource: https://manage.office.com
client_id: 6e2c4902-3cfc-41b9-bd72-b58c753dbf7d
```

Response

```json
{
    "token_type": "Bearer",
    "scope": "ActivityFeed.Read ActivityFeed.ReadDlp ServiceHealth.Read",
    "expires_in": "3600",
    "ext_expires_in": "3600",
    "expires_on": "1559094135",
    "not_before": "1559090235",
    "resource": "https://manage.office.com",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...",
    "refresh_token": "AQABAAAAAADCoMpjJXrxTq9VG9te-..."
}
```

## Start Office 365 Maangement API Subscription

Request

```text
POST https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/subscriptions/start?contentType=Audit.AzureActiveDirectory
Content-Type:application/json
Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...
```

Response

```json
{
    "contentType": "Audit.AzureActiveDirectory",
    "status": "enabled",
    "webhook": null
}
```

## List current subscriptions

Request

```text
GET https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/subscriptions/list
Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...
```

Response

```json
[
    {
        "contentType": "Audit.AzureActiveDirectory",
        "status": "enabled",
        "webhook": null
    }
]
```

## List available content

Request

```text
GET https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/subscriptions/content?contentType=Audit.AzureActiveDirectory
Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...
```

Response

```json
[
    {
        "contentUri": "https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/audit/20190528031411916085508$20190528041503354028379$audit_azureactivedirectory$Audit_AzureActiveDirectory$apac0015",
        "contentId": "20190528031411916085508$20190528041503354028379$audit_azureactivedirectory$Audit_AzureActiveDirectory$apac0015",
        "contentType": "Audit.AzureActiveDirectory",
        "contentCreated": "2019-05-28T04:15:03.354Z",
        "contentExpiration": "2019-06-04T03:14:11.916Z"
    },
    {
        "contentUri": "https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/audit/20190528043252783155041$20190528055709816074185$audit_azureactivedirectory$Audit_AzureActiveDirectory$apac0015",
        "contentId": "20190528043252783155041$20190528055709816074185$audit_azureactivedirectory$Audit_AzureActiveDirectory$apac0015",
        "contentType": "Audit.AzureActiveDirectory",
        "contentCreated": "2019-05-28T05:57:09.816Z",
        "contentExpiration": "2019-06-04T04:32:52.783Z"
    }
]
```

## Retrieving content

Request

```text
GET https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/audit/20190528031411916085508$20190528041503354028379$audit_azureactivedirectory$Audit_AzureActiveDirectory$apac0015
Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...
```

Response

```json
[
    {
        "CreationTime": "2019-05-28T03:38:14",
        "Id": "c9664be9-63d6-4e05-9c6c-a7d6b89a4136",
        "Operation": "UserLoggedIn",
        "OrganizationId": "d7a85814-4d89-4baf-8a8a-f5be8306a959",
        "RecordType": 15,
        "ResultStatus": "Succeeded",
        "UserKey": "100320003FF3312A@contoso.onmicrosoft.com",
        "UserType": 0,
        "Version": 1,
        "Workload": "AzureActiveDirectory",
        "ClientIP": "167.220.232.102",
        "ObjectId": "00000002-0000-0000-c000-000000000000",
        "UserId": "Sync_FED-AADC-VM1_25f220adb992@contoso.onmicrosoft.com",
        "AzureActiveDirectoryEventType": 1,
        "ExtendedProperties": [
            {
                "Name": "UserAuthenticationMethod",
                "Value": "1"
            },
            {
                "Name": "RequestType",
                "Value": "WindowsAuthenticationController:usernamemixed"
            },
            {
                "Name": "ResultStatusDetail",
                "Value": "Success"
            }
        ],
        "ModifiedProperties": [],
        "Actor": [
            {
                "ID": "97c8bd93-afcd-401e-a5cd-0791e3351700",
                "Type": 0
            },
            {
                "ID": "Sync_FED-AADC-VM1_25f220adb992@contoso.onmicrosoft.com",
                "Type": 5
            },
            {
                "ID": "100320003FF3312A",
                "Type": 3
            }
        ],
        "ActorContextId": "d7a85814-4d89-4baf-8a8a-f5be8306a959",
        "ActorIpAddress": "167.220.232.102",
        "InterSystemsId": "48cd68f0-1f34-4cab-a335-d9a72b4f82c5",
        "IntraSystemId": "1fa2302f-ac82-474c-b22a-4bf2e3341000",
        "SupportTicketId": "",
        "Target": [
            {
                "ID": "00000002-0000-0000-c000-000000000000",
                "Type": 0
            }
        ],
        "TargetContextId": "d7a85814-4d89-4baf-8a8a-f5be8306a959",
        "ApplicationId": ""
    },
    ...
]
```

## Stop a subscription

Request

```text
POST https://manage.office.com/api/v1.0/d7a85814-4d89-4baf-8a8a-f5be8306a959/activity/feed/subscriptions/stop?contentType=Audit.AzureActiveDirectory
Authorization:Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSU...
```

Response

```text
HTTP/1.1 204 No Content
```
