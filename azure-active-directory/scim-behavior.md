# Testing SCIM behavior

SCIM の動作を確認しました。

## ヘッダーの情報

各 HTTP リクエストのヘッダーには以下が Authorization Bearer として入っています。

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjJLVmN1enFBaWRPTHFXU2FvbDd3Z0ZSR0NZbyIsImtpZCI6IjJLVmN1enFBaWRPTHFXU2FvbDd3Z0ZSR0NZbyJ9.eyJhdWQiOiIwMDAwMDAwMi0wMDAwLTAwMDAtYzAwMC0wMDAwMDAwMDAwMDAiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9kN2E4NTgxNC00ZDg5LTRiYWYtOGE4YS1mNWJlODMwNmE5NTkvIiwiaWF0IjoxNTA4MjAzOTgxLCJuYmYiOjE1MDgyMDM5ODEsImV4cCI6MTUwODIwNzg4MSwiYWlvIjoiWTJWZ1lGQ2Z0S1RBakczRCswUGRIWTFWcTVYekFBPT0iLCJhcHBpZCI6IjAwMDAwMDE0LTAwMDAtMDAwMC1jMDAwLTAwMDAwMDAwMDAwMCIsImFwcGlkYWNyIjoiMiIsImVfZXhwIjoyNjI4MDAsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0L2Q3YTg1ODE0LTRkODktNGJhZi04YThhLWY1YmU4MzA2YTk1OS8iLCJvaWQiOiJkMmRmYWJmOC0xM2Y1LTRiMmUtYjE4Ni05YjI0NGUxZTZkOTkiLCJzdWIiOiJkMmRmYWJmOC0xM2Y1LTRiMmUtYjE4Ni05YjI0NGUxZTZkOTkiLCJ0ZW5hbnRfcmVnaW9uX3Njb3BlIjoiQVMiLCJ0aWQiOiJkN2E4NTgxNC00ZDg5LTRiYWYtOGE4YS1mNWJlODMwNmE5NTkiLCJ1dGkiOiJmZkRlaEs5X19VdUtkcWlFX3c4TkFBIiwidmVyIjoiMS4wIn0.m1O_cKvTrc8yOSAaI8WPNPpTNo4mq_pSnUc06AytnbkRyecX3hDRKN2yFN9aL1ofLU8PBTfhj_u-HgX0CUa9wOqxBfjFDMZQy-wDqdu6RxIvkDDmICgjcyBAo9G4UaFB3xkHdGzMEygeIgLgG7y8CTWvuxmPHC11DmIGHJTaKvqS2NIDNUWNsDo1YJPl60nLJBeZxC43IuaOMCML3tAOW7w3Dk2d-AirvlZEGmYZotUtiia8vGL7i_C2fhOClAEzx-gpfUtEWhefJQVfUm9XhKDfaQ7hiHfXxKHN1BsXLq85oXkChtP2TP8h_1EWym9euM_MAPTr4HlFWz48p6vc3w
```

ヘッダーとペイロードの部分をデコードすると以下です。

```json
{
    "typ": "JWT",
    "alg": "RS256",
    "x5t": "2KVcuzqAidOLqWSaol7wgFRGCYo",
    "kid": "2KVcuzqAidOLqWSaol7wgFRGCYo"
}
{
    "aud": "00000002-0000-0000-c000-000000000000",
    "iss": "https://sts.windows.net/d7a85814-4d89-4baf-8a8a-f5be8306a959/",
    "iat": 1508203981,
    "nbf": 1508203981,
    "exp": 1508207881,
    "aio": "Y2VgYFCftKTAjG3D+0PdHY1Vq5XzAA==",
    "appid": "00000014-0000-0000-c000-000000000000",
    "appidacr": "2",
    "e_exp": 262800,
    "idp": "https://sts.windows.net/d7a85814-4d89-4baf-8a8a-f5be8306a959/",
    "oid": "d2dfabf8-13f5-4b2e-b186-9b244e1e6d99",
    "sub": "d2dfabf8-13f5-4b2e-b186-9b244e1e6d99",
    "tenant_region_scope": "AS",
    "tid": "d7a85814-4d89-4baf-8a8a-f5be8306a959",
    "uti": "ffDehK9__UuKdqiE_w8NAA",
    "ver": "1.0"
}
```

以下にネットワーク パケット (HTTP で取得) したものの抜粋を記載します。

## ユーザーを割り当てに追加した時

まず、作成したいユーザーの GET が行われます。

```
1021    43.329574    52.161.102.217    10.2.2.4    HTTP    1448    GET    /scim/Users?filter=externalId+eq+%22Sync_AADC01_bddd63a128a5%22 HTTP/1.1
```

```
1025    43.569273    10.2.2.4    52.161.102.217    HTTP    326    HTTP/1.1200 OK  (application/jwt){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
    ],
    "totalResults": 0,
    "itemsPerPage": 0,
    "startIndex": null,
    "Resources": []
}
```

```
1105    46.747452    52.161.102.217    10.2.2.4    HTTP    1434    GET /scim/Users?filter=externalId+eq+%22testuser01%22 HTTP/1.1
```

```
1108    46.859896    10.2.2.4    52.161.102.217    HTTP    326    HTTP/1.1200 OK  (application/jwt){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
    ],
    "totalResults": 0,
    "itemsPerPage": 0,
    "startIndex": null,
    "Resources": []
}
```

```
1156    48.596648    52.161.102.217    10.2.2.4    HTTP    1432    GET /scim/Users?filter=externalId+eq+%22jutakata%22 HTTP/1.1
```

```
1159    48.707073    10.2.2.4    52.161.102.217    HTTP    326    HTTP/1.1200 OK  (application/jwt){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
    ],
    "totalResults": 0,
    "itemsPerPage": 0,
    "startIndex": null,
    "Resources": []
}
```

```
1168    48.963864    52.161.102.217    10.2.2.4    HTTP    1435    GET /scim/Users?filter=externalId+eq+%22clouduser02%22 HTTP/1.1
```

```
1170    49.071858    10.2.2.4    52.161.102.217    HTTP    326    HTTP/1.1200 OK  (application/jwt){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
    ],
    "totalResults": 0,
    "itemsPerPage": 0,
    "startIndex": null,
    "Resources": []
}
```

```
1220    49.747711    52.161.102.217    10.2.2.4    HTTP    1435    GET /scim/Users?filter=externalId+eq+%22clouduser03%22 HTTP/1.1
```

```
1222    49.854414    10.2.2.4    52.161.102.217    HTTP    326    HTTP/1.1200 OK  (application/jwt){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
    ],
    "totalResults": 0,
    "itemsPerPage": 0,
    "startIndex": null,
    "Resources": []
}
```

ここまでで作成したいユーザーの GET が行われます。この後 Create に相当する POST が行われます。

```
1257    51.105724    52.161.102.217  10.2.2.4    HTTP    462    POST /scim/Users HTTP/1.1  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser03",
    "userName": "clouduser03@jutakata02.onmicrosoft.com",
    "active": true,
    "displayName": "clouduser03",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "clouduser03@jutakata02.onmicrosoft.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser03"
    },
    "roles": []
}
```

```
1260    51.258460    10.2.2.4    52.161.102.217    HTTP    820 HTTP/1.1200 OK  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser03",
    "id": "b70ee734-710e-41ca-98d7-9e99d096d590",
    "userName": null,
    "active": true,
    "displayName": "clouduser03",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "clouduser03@jutakata02.onmicrosoft.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser03",
        "familyName": "clouduser03",
        "givenName": null
    },
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": null
}
```

```
1272    51.920655    52.161.102.217  10.2.2.4    HTTP    466    POST /scim/Users HTTP/1.1  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser01",
    "userName": "clouduser01@jutakata02.onmicrosoft.com",
    "active": true,
    "displayName": "clouduser01",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "clouduser01@jutakata02.msftonlinerepro.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser01"
    },
    "roles": []
}
```

```
1273    51.922832    10.2.2.4    52.161.102.217    HTTP    824    HTTP/1.1200 OK  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser01",
    "id": "b3800b90-6a19-4a0d-b3b4-5a2000d380f5",
    "userName": null,
    "active": true,
    "displayName": "clouduser01",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "clouduser01@jutakata02.msftonlinerepro.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser01",
        "familyName": "clouduser01",
        "givenName": null
    },
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": null
}
```

```
1304    52.364885    52.161.102.217  10.2.2.4    HTTP    462    POST /scim/Users HTTP/1.1  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser02",
    "userName": "clouduser02@jutakata02.onmicrosoft.com",
    "active": true,
    "displayName": "clouduser02",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "clouduser02@jutakata02.onmicrosoft.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser02"
    },
    "roles": []
}
```

```
1305    52.366397    10.2.2.4    52.161.102.217    HTTP    820    HTTP/1.1200 OK  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser02",
    "id": "a7c6f072-db63-456c-b34b-2b70c4e725ef",
    "userName": null,
    "active": true,
    "displayName": "clouduser02",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "clouduser02@jutakata02.onmicrosoft.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser02",
        "familyName": "clouduser02",
        "givenName": null
    },
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": null
}
```

```
1334    52.890417    52.161.102.217  10.2.2.4    HTTP    736    POST /scim/Users HTTP/1.1  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "jutakata",
    "userName": "jutakata@jutakata02.onmicrosoft.com",
    "active": true,
    "addresses": [
        {
            "primary": false,
            "type": "work",
            "country": "JP",
            "formatted": null,
            "locality": null,
            "postalCode": null,
            "region": null,
            "streetAddress": null
        }
    ],
    "displayName": "TakataJun",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "jutakata@jutakata02.onmicrosoft.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "Jun Takata",
        "familyName": "Takata",
        "givenName": "Jun"
    },
    "phoneNumbers": [
        {
            "primary": true,
            "type": "work",
            "value": "03-4535-5907"
        }
    ],
    "preferredLanguage": "ja-JP",
    "roles": []
}
```

```
1335    52.915987    10.2.2.4    52.161.102.217    HTTP    1055    HTTP/1.1200 OK  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "jutakata",
    "id": "7da83c92-9cb6-401e-bf20-1eb8e0004a94",
    "userName": null,
    "active": true,
    "addresses": [
        {
            "primary": false,
            "type": "work",
            "country": "JP",
            "formatted": null,
            "locality": null,
            "postalCode": null,
            "region": null,
            "streetAddress": null
        }
    ],
    "displayName": "TakataJun",
    "emails": [
        {
            "primary": true,
            "type": "work",
            "value": "jutakata@jutakata02.onmicrosoft.com"
        }
    ],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "Jun Takata",
        "familyName": "jutakata",
        "givenName": "Jun"
    },
    "phoneNumbers": [
        {
            "primary": true,
            "type": "work",
            "value": "03-4535-5907"
        }
    ],
    "preferredLanguage": "ja-JP",
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": null
}
```

## ユーザーを割り当てから削除した時

Active が false になり、Delete ではなく Patch になります。

```
13671    1290.770818    52.176.0.176    10.2.2.4    HTTP    307    PATCH /scim/Users/5c85ceed-867c-4fd0-8ed1-4ad1a870a685 HTTP/1.1  (application/scim+json)
{
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ],
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": [
                {
                    "$ref": null,
                    "value": "False"
                }
            ]
        },
        {
            "op": "Add",
            "path": "userName",
            "value": [
                {
                    "$ref": null,
                    "value": "clouduser01@jutakata02.onmicrosoft.com"
                }
            ]
        }
    ]
}
```

```
13676    1291.197201    10.2.2.4    52.176.0.176    HTTP    168    HTTP/1.1 204 No Content 
```

## ユーザーを割り当てに再度追加した時

```
26406    2441.865001    52.161.9.156    10.2.2.4    HTTP    306    PATCH /scim/Users/5c85ceed-867c-4fd0-8ed1-4ad1a870a685 HTTP/1.1  (application/scim+json)
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ],
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": [
                {
                    "$ref": null,
                    "value": "True"
                }
            ]
        },
        {
            "op": "Add",
            "path": "userName",
            "value": [
                {
                    "$ref": null,
                    "value": "clouduser01@jutakata02.onmicrosoft.com"
                }
            ]
        }
    ]
}
```

```
26412    2442.115630    10.2.2.4    52.161.9.156    HTTP    168    HTTP/1.1 204 No Content 
```

## ユーザーを割り当てられた状態から削除した時

```
39218    3817.093553    52.176.0.176    10.2.2.4    HTTP    307    PATCH /scim/Users/5c85ceed-867c-4fd0-8ed1-4ad1a870a685 HTTP/1.1  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ],
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": [
                {
                    "$ref": null,
                    "value": "False"
                }
            ]
        },
        {
            "op": "Add",
            "path": "userName",
            "value": [
                {
                    "$ref": null,
                    "value": "clouduser01@jutakata02.onmicrosoft.com"
                }
            ]
        }
    ]
}
```

```
39236    3817.452837    10.2.2.4    52.176.0.176    HTTP    168    HTTP/1.1 204 No Content 
```

## ユーザーがアプリに割り当てられプロビジョニングされている状態でハードデリートした場合

Hard-delete で DELETE のコマンドが出ます。ユーザーがアプリに割り当てられ、プロビジョニングされている状態で以下を実行しました。

```powershell
Remove-MsolUser -UserPrincipalName clouduser01@jutakata02.onmicrosoft.com
Remove-MsolUser -UserPrincipalName clouduser01@jutakata02.onmicrosoft.com -RemoveFromRecycleBin
```

パケットのやり取りは以下のとおりです。

```
3080    346.212611    52.173.199.78    10.2.2.4    HTTP    1455    DELETE /scim/Users/5c85ceed-867c-4fd0-8ed1-4ad1a870a685 HTTP/1.1 
```

```
3093    346.368298    10.2.2.4    52.173.199.78    HTTP    168    HTTP/1.1 204 No Content 
```

続いて、ユーザーの割り当てを行います。

```
1698    212.963855    52.161.102.217    10.2.2.4    HTTP    371    POST /scim/Users HTTP/1.1  (application/scim+json)
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser01",
    "userName": "clouduser01@jutakata02.onmicrosoft.com",
    "active": true,
    "displayName": "clouduser01",
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser01"
    },
    "roles": []
}
```

```
1699    212.975355    10.2.2.4    52.161.102.217    HTTP    729    HTTP/1.1 200 OK  (application/scim+json)
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    ],
    "externalId": "clouduser01",
    "id": "eec92b60-57d8-4c92-9413-081f5a2165eb",
    "userName": null,
    "active": true,
    "displayName": "clouduser01",
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "clouduser01",
        "familyName": "clouduser01",
        "givenName": null
    },
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": null
}
```

その後割り当てからユーザーを削除 (deactivated) します。

```
16977    1544.961677    52.173.199.78    10.2.2.4    HTTP    1433    GET /scim/Users/eec92b60-57d8-4c92-9413-081f5a2165eb HTTP/1.1 
```

```
16989    1545.166682    10.2.2.4    52.173.199.78    HTTP    617    HTTP/1.1 200 OK  (application/jwt)
```

```
17065    1545.975130    52.173.199.78    10.2.2.4    HTTP    307    PATCH /scim/Users/eec92b60-57d8-4c92-9413-081f5a2165eb HTTP/1.1  (application/scim+json){
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ],
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": [
                {
                    "$ref": null,
                    "value": "False"
                }
            ]
        },
        {
            "op": "Add",
            "path": "userName",
            "value": [
                {
                    "$ref": null,
                    "value": "clouduser01@jutakata02.onmicrosoft.com"
                }
            ]
        }
    ]
}
```

```
17077    1546.312533    10.2.2.4    52.173.199.78    HTTP    168    HTTP/1.1 204 No Content 
```

Remove-MsolUser -UserPrincipalName clouduser01@jutakata02.onmicrosoft.com を実行します。
なぜか patch が来ているが特に変化はありません。

```
25593    2612.349086    52.161.102.217    10.2.2.4    HTTP    1433    GET /scim/Users/eec92b60-57d8-4c92-9413-081f5a2165eb HTTP/1.1 
```

```
25595    2612.473090    10.2.2.4    52.161.102.217    HTTP    618    HTTP/1.1 200 OK  (application/jwt)
```

```
25607    2613.019331    52.161.102.217    10.2.2.4    HTTP    234    PATCH /scim/Users/eec92b60-57d8-4c92-9413-081f5a2165eb HTTP/1.1  (application/scim+json)
{
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ],
    "Operations": [
        {
            "op": "Add",
            "path": "userName",
            "value": [
                {
                    "$ref": null,
                    "value": "clouduser01@jutakata02.onmicrosoft.com"
                }
            ]
        }
    ]
}
```

```
25609    2613.349816    10.2.2.4    52.161.102.217    HTTP    168    HTTP/1.1 204 No Content 
```

Remove-MsolUser -UserPrincipalName clouduser01@jutakata02.onmicrosoft.com -RemoveFromRecycleBin を実行します。
割り当てにはいないのに削除されます。

```
36933    3831.786161    23.99.54.23    10.2.2.4    HTTP    1455    DELETE /scim/Users/eec92b60-57d8-4c92-9413-081f5a2165eb HTTP/1.1 
```

```
36934    3831.813622    10.2.2.4    23.99.54.23    HTTP    168    HTTP/1.1 204 No Content 
```