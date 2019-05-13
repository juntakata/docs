# Access EWS manually with authorization code grant flow

## Create an app on Azure AD

1. Go to Azure portal
2. Go to Azure Active Directory blade
3. Select [App registrations]
4. [+ New registration]
5. Specify new name and select [Public client (mobile & desktop)]
6. Specify https://localhost as a redirect URI.
7. Select [Register]
8. Copy Application (client) ID
9. Go to [API permissions]
10. Select [+ Add a permission]
11. Select [Exchange] from supported legacy APIs
12. Select [Delegated permissions]
13. Enable [EWS.AccessAsUser.All]
14. Select [Add permissions]

## Compose authorization request to gain authorization code

### Request

#### URL

Replace the client_id with the one you copied above on Azure portal and access the URL with a browser.

```
GET https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/authorize?client_id=8ed26df8-25dd-41a5-bf0e-81940e3984f5&nonce=defaultNonce&redirect_uri=https%3A%2F%2Flocalhost&scope=openid%20offline_access&response_type=code&prompt=login&grant_type=authorization_code&resource=https%3A%2F%2Foutlook.office365.com
```

### Response

After the authentication, you will be shown a consent screen and redirected URL below.

```
GET https://localhost/?code=AQABAAIAAADCoMpjJXrxTq9VG9te<omit>gHdxcBncIqj3VvQ0mPCAnXNyB8IAA&session_state=0e11bd4c-4065-47a6-b7e4-d4c1312d1054
```

## Access token endpoint to gain access token

### Request

Copy code in the response and start a POST request as below

#### URL

```
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/token
```

#### Body

```
code:AQABAAIAAADCoMpjJXrxTq9VG9te<omit>gHdxcBncIqj3VvQ0mPCAnXNyB8IAA
grant_type:authorization_code
client_id:8ed26df8-25dd-41a5-bf0e-81940e3984f5
scope:openid offline_access
redirect_uri:https://localhost
resource:https://outlook.office365.com
```

### Response

```json
{
    "token_type": "Bearer",
    "scope": "EWS.AccessAsUser.All User.Read",
    "expires_in": "3599",
    "ext_expires_in": "3599",
    "expires_on": "1557715749",
    "not_before": "1557711849",
    "resource": "https://outlook.office365.com",
    "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6<omit>uDt48b786GLm1W9eZAGLWK6fLKw",
    "refresh_token": "AQABAAAAAADCoMpjJXrxTq9VG9te<omit>SXuURa73EO9tdXCEAqhLRl_RIAA",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJS<omit>_5cqhIhFOm2cGYV52imejarWU_Q"
}
```

## Access EWS with access token

Send POST request to URL below to gain results.

### Request

#### URL

```
POST https://outlook.office365.com/ews/exchange.asmx 
```

#### Header

```
Content-Type:text/xml
Authorization:Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6<omit>uDt48b786GLm1W9eZAGLWK6fLKw
```

#### Body

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:m="http://schemas.microsoft.com/exchange/services/2006/messages" xmlns:t="http://schemas.microsoft.com/exchange/services/2006/types" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Header>
    <t:RequestServerVersion Version="Exchange2013" />
  </soap:Header>
  <soap:Body>
    <m:FindFolder Traversal="Shallow">
      <m:FolderShape>
        <t:BaseShape>AllProperties</t:BaseShape>
      </m:FolderShape>
      <m:IndexedPageFolderView MaxEntriesReturned="10" Offset="0" BasePoint="Beginning" />
      <m:ParentFolderIds>
        <t:DistinguishedFolderId Id="root" />
      </m:ParentFolderIds>
    </m:FindFolder>
  </soap:Body>
</soap:Envelope>
```

### Response

```xml
<?xml version="1.0" encoding="utf-8"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
    <s:Header>
        <h:ServerVersionInfo MajorVersion="15" MinorVersion="20" MajorBuildNumber="1878" MinorBuildNumber="26" Version="V2018_01_08" xmlns:h="http://schemas.microsoft.com/exchange/services/2006/types" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"/>
    </s:Header>
    <s:Body>
        <m:FindFolderResponse xmlns:m="http://schemas.microsoft.com/exchange/services/2006/messages" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:t="http://schemas.microsoft.com/exchange/services/2006/types">
            <m:ResponseMessages>
                <m:FindFolderResponseMessage ResponseClass="Success">
                    <m:ResponseCode>NoError</m:ResponseCode>
                    <m:RootFolder IndexedPagingOffset="10" TotalItemsInView="81" IncludesLastItemInRange="false">
                        <t:Folders>
                            <t:SearchFolder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAABTvAAA=" ChangeKey="BwAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAAABcP"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:FolderClass>IPF.Note</t:FolderClass>
                                <t:DisplayName>AllContacts</t:DisplayName>
                                <t:TotalCount>25</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:SearchFolder>
                            <t:SearchFolder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAcwffSAAA=" ChangeKey="BwAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAcwplO"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:FolderClass>IPF</t:FolderClass>
                                <t:DisplayName>AllItems</t:DisplayName>
                                <t:TotalCount>282</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>66</t:UnreadCount>
                            </t:SearchFolder>
                            <t:SearchFolder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAcwfu/AAA=" ChangeKey="BwAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAcwp5a"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:FolderClass>IPF.Note</t:FolderClass>
                                <t:DisplayName>AllPersonMetadata</t:DisplayName>
                                <t:TotalCount>24</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:SearchFolder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAcwfP7AAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAcwsc2"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>ApplicationDataRoot</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>21</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAErAAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAAAAmC"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>BrokerSubscriptions</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAFkkiYcAAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAFkjuZj"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>BulkActions</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEuAAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAAAAml"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>CalendarItemSnapshots</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAFMAAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAAAArc"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>CalendarSharingCacheCollection</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEGAAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAAAACU"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>Common Views</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                            <t:Folder>
                                <t:FolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAk8UAjAAA=" ChangeKey="AQAAABYAAAA/T+2wxc9XSpYSK/qYMKhcAAAk8hV/"/>
                                <t:ParentFolderId Id="AAMkAGQ2NGQzMTU1LWZlZDQtNGZjZC1hZDc4LTE2YWY0YzA2NmFiYQAuAAAAAAALC96ZGFlcSpQS8PhbCp0RAQA/T+2wxc9XSpYSK/qYMKhcAAAAAAEBAAA=" ChangeKey="AQAAAA=="/>
                                <t:DisplayName>ComplianceMetadata</t:DisplayName>
                                <t:TotalCount>0</t:TotalCount>
                                <t:ChildFolderCount>0</t:ChildFolderCount>
                                <t:EffectiveRights>
                                    <t:CreateAssociated>true</t:CreateAssociated>
                                    <t:CreateContents>true</t:CreateContents>
                                    <t:CreateHierarchy>true</t:CreateHierarchy>
                                    <t:Delete>true</t:Delete>
                                    <t:Modify>true</t:Modify>
                                    <t:Read>true</t:Read>
                                    <t:ViewPrivateItems>true</t:ViewPrivateItems>
                                </t:EffectiveRights>
                                <t:UnreadCount>0</t:UnreadCount>
                            </t:Folder>
                        </t:Folders>
                    </m:RootFolder>
                </m:FindFolderResponseMessage>
            </m:ResponseMessages>
        </m:FindFolderResponse>
    </s:Body>
</s:Envelope>
```