
# Sign-in with Managed ID from Azure cLI with Fiddler

## Prerequisite
To capture a Fiddler trace for Azure CLI, you need to set some environment variables. These variables apply for all Azure CLI environments such as Windows, Linux, and etc.

```
set AZURE_CLI_DISABLE_CONNECTION_VERIFICATION=1
set HTTPS_PROXY=https://127.0.0.1:8888
set HTTP_PROXY=http://127.0.0.1:8888
```

If you do not want to disable connection verification, you can add your local proxy certificate to a file below.

```
C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem
```

After capturing the trace, disable each variables as below.

```
set AZURE_CLI_DISABLE_CONNECTION_VERIFICATION=
set HTTPS_PROXY=
set HTTP_PROXY=
```

## Debug output when Sing-in with Managed ID

Run following command to sign-in with Managed ID.

```
az login --identity --allow-no-subscription --debug --verbose

Command arguments: ['login', '--identity', '--allow-no-subscription', '--debug', '--verbose']
Event: Cli.PreExecute []
Event: CommandParser.OnGlobalArgumentsCreate [<function CLILogging.on_global_arguments at 0x02F4FBB8>, <function OutputProducer.on_global_arguments at 0x02F968E8>, <function CLIQuery.on_global_arguments at 0x02FB69C0>]
Event: CommandInvoker.OnPreCommandTableCreate []
Installed command modules ['acr', 'acs', 'advisor', 'ams', 'appservice', 'backup', 'batch', 'batchai', 'billing', 'botservice', 'cdn', 'cloud', 'cognitiveservices', 'configure', 'consumption', 'container', 'cosmosdb', 'deploymentmanager', 'dla', 'dls', 'dms', 'eventgrid', 'eventhubs', 'extension', 'feedback', 'find', 'hdinsight', 'interactive', 'iot', 'iotcentral', 'keyvault', 'kusto', 'lab', 'maps', 'monitor', 'natgateway', 'network', 'policyinsights', 'privatedns', 'profile', 'rdbms', 'redis', 'relay', 'reservations', 'resource', 'role', 'search', 'security', 'servicebus', 'servicefabric', 'signalr', 'sql', 'sqlvm', 'storage', 'vm']
Loaded module 'acr' in 0.016 seconds.
Loaded module 'acs' in 0.006 seconds.
Loaded module 'advisor' in 0.004 seconds.
Event: CommandLoader.OnLoadCommandTable []
Loaded module 'ams' in 0.011 seconds.
Loaded module 'appservice' in 0.023 seconds.
Loaded module 'backup' in 0.006 seconds.
Event: CommandLoader.OnLoadCommandTable []
Loaded module 'batch' in 0.017 seconds.
Loaded module 'batchai' in 0.014 seconds.
Loaded module 'billing' in 0.004 seconds.
Loaded module 'botservice' in 0.007 seconds.
Event: CommandLoader.OnLoadCommandTable []
Loaded module 'cdn' in 0.006 seconds.
Loaded module 'cloud' in 0.003 seconds.
Loaded module 'cognitiveservices' in 0.004 seconds.
Loaded module 'configure' in 0.003 seconds.
Loaded module 'consumption' in 0.007 seconds.
Loaded module 'container' in 0.005 seconds.
Loaded module 'cosmosdb' in 0.008 seconds.
Loaded module 'deploymentmanager' in 0.006 seconds.
Loaded module 'dla' in 0.008 seconds.
Loaded module 'dls' in 0.014 seconds.
Loaded module 'dms' in 0.004 seconds.
Loaded module 'eventgrid' in 0.005 seconds.
Loaded module 'eventhubs' in 0.007 seconds.
Loaded module 'extension' in 0.002 seconds.
Loaded module 'feedback' in 0.002 seconds.
Loaded module 'find' in 0.010 seconds.
Loaded module 'hdinsight' in 0.004 seconds.
Loaded module 'interactive' in 0.003 seconds.
Loaded module 'iot' in 0.007 seconds.
Loaded module 'iotcentral' in 0.005 seconds.
Loaded module 'keyvault' in 0.009 seconds.
Loaded module 'kusto' in 0.010 seconds.
Loaded module 'lab' in 0.007 seconds.
Loaded module 'maps' in 0.005 seconds.
Loaded module 'monitor' in 0.010 seconds.
Loaded module 'natgateway' in 0.004 seconds.
Loaded module 'network' in 0.042 seconds.
Loaded module 'policyinsights' in 0.007 seconds.
Loaded module 'privatedns' in 0.009 seconds.
Loaded module 'profile' in 0.003 seconds.
Loaded module 'rdbms' in 0.020 seconds.
Loaded module 'redis' in 0.004 seconds.
Loaded module 'relay' in 0.006 seconds.
Loaded module 'reservations' in 0.005 seconds.
Loaded module 'resource' in 0.016 seconds.
Loaded module 'role' in 0.008 seconds.
Loaded module 'search' in 0.003 seconds.
Loaded module 'security' in 0.005 seconds.
Loaded module 'servicebus' in 0.016 seconds.
Loaded module 'servicefabric' in 0.004 seconds.
Loaded module 'signalr' in 0.004 seconds.
Loaded module 'sql' in 0.011 seconds.
Loaded module 'sqlvm' in 0.005 seconds.
Event: CommandLoader.OnLoadCommandTable []
Loaded module 'storage' in 0.050 seconds.
Loaded module 'vm' in 0.026 seconds.
Loaded all modules in 0.511 seconds. (note: there's always an overhead with the first module loaded)
Extensions directory: 'C:\Users\testuser\.azure\cliextensions'
Event: CommandInvoker.OnPreCommandTableTruncate [<function AzCliLogging.init_command_file_logging at 0x0300AD68>]
az_command_data_logger : command args: login --identity --allow-no-subscription --debug --verbose
metadata file logging enabled - writing logs to 'C:\Users\testuser\.azure\commands'.
Event: CommandInvoker.OnPostCommandTableCreate [<function register_global_subscription_argument.<locals>.add_subscription_parameter at 0x03048540>, <function register_ids_argument.<locals>.add_ids_arguments at 0x03048588>, <function register_cache_arguments.<locals>.add_cache_arguments at 0x03048780>]
Event: CommandInvoker.OnCommandTableLoaded []
Event: CommandInvoker.OnPreParseArgs [<function _documentdb_deprecate at 0x0399E7C8>]
Event: CommandInvoker.OnPostParseArgs [<function OutputProducer.handle_output_argument at 0x02F96930>, <function CLIQuery.handle_query_parameter at 0x02FB6A08>, <function register_ids_argument.<locals>.parse_ids_arguments at 0x03048738>, <function handler at 0x03A5E5D0>]
urllib3.connectionpool : Starting new HTTP connection (1): 127.0.0.1:8888
urllib3.connectionpool : http://127.0.0.1:8888 "GET http://169.254.169.254/metadata/identity/oauth2/token?resource=https%3A%2F%2Fmanagement.core.windows.net%2F&api-version=2018-02-01 HTTP/1.1" 200 1611
msrestazure.azure_active_directory : MSI: Retrieving a token from http://169.254.169.254/metadata/identity/oauth2/token, with payload {'resource': 'https://management.core.windows.net/', 'api-version': '2018-02-01'}
msrestazure.azure_active_directory : MSI: Token retrieved
MSI: token was retrieved. Now trying to initialize local accounts...
msrest.universal_http.requests : Configuring retry: max_retries=4, backoff_factor=0.8, max_backoff=90
Connection verification disabled by environment variable AZURE_CLI_DISABLE_CONNECTION_VERIFICATION
msrest.async_paging : Paging async iterator protocol is not available for SubscriptionPaged
msrest.universal_http : Configuring redirects: allow=True, max=30
msrest.universal_http : Configuring request: timeout=100, verify=False, cert=None
msrest.universal_http : Configuring proxies: ''
msrest.universal_http : Evaluate proxies against ENV settings: True
urllib3.connectionpool : Starting new HTTPS connection (1): management.azure.com:443
C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\lib\site-packages\urllib3\connectionpool.py:847: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
urllib3.connectionpool : https://management.azure.com:443 "GET /subscriptions?api-version=2016-06-01 HTTP/1.1" 200 133
Event: CommandInvoker.OnTransformResult [<function _resource_group_transform at 0x03041AE0>, <function _x509_from_base64_to_hex_transform at 0x03041B28>]
Event: CommandInvoker.OnFilterResult []
[
  {
    "environmentName": "AzureCloud",
    "id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "isDefault": true,
    "name": "N/A(tenant level account)",
    "state": "Enabled",
    "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "user": {
      "assignedIdentityInfo": "MSI",
      "name": "systemAssignedIdentity",
      "type": "servicePrincipal"
    }
  }
]
Event: Cli.PostExecute []
az_command_data_logger : exit code: 0
telemetry.save : Save telemetry record of length 2509 in cache
telemetry.check : Negative: The C:\Users\testuser\.azure\telemetry.txt was modified at 2019-06-30 07:12:43.300420, which in less than 600.000000 s
command ran in 2.955 seconds.
```

## Fiddler traces when Sing-in with Managed ID

See below for HTTPS requests and responses when the command above is executed.

### Request

```
GET http://169.254.169.254/metadata/identity/oauth2/token?resource=https%3A%2F%2Fmanagement.core.windows.net%2F&api-version=2018-02-01 HTTP/1.1
Host: 169.254.169.254
User-Agent: python/3.6.6 (Windows-10-10.0.17763-SP0) msrest/0.6.7 msrest_azure/0.6.1
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Metadata: true
```

### Response

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Server: Microsoft-IIS/10.0
Date: Sun, 30 Jun 2019 07:15:13 GMT
Content-Length: 1611
{
    "access_token": "eyJ0eXAiOiJKV1QiL...l71Jic0_JP5TaVHiFHIeLaju9w",
    "client_id": "ae96967e-8561-4d22-bc3e-b449656a11b8",
    "expires_in": "28800",
    "expires_on": "1561903890",
    "ext_expires_in": "28800",
    "not_before": "1561874790",
    "resource": "https://management.core.windows.net/",
    "token_type": "Bearer"
}
```

### Request

```
GET https://management.azure.com/subscriptions?api-version=2016-06-01 HTTP/1.1
Host: management.azure.com
User-Agent: python/3.6.6 (Windows-10-10.0.17763-SP0) msrest/0.6.7 msrest_azure/0.6.1 subscriptionclient/2.1.0 Azure-SDK-For-Python
Accept-Encoding: gzip, deflate
Accept: application/json
Connection: keep-alive
Authorization: Bearer eyJ0eXAiOiJKV1QiL...l71Jic0_JP5TaVHiFHIeLaju9w
x-ms-client-request-id: ce370eb4-9b06-11e9-83d4-000d3a94b3cf
accept-language: en-US
```

### Response

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Type: application/json; charset=utf-8
Expires: -1
Vary: Accept-Encoding
x-ms-ratelimit-remaining-tenant-reads: 11999
x-ms-request-id: fc81aae2-2fff-4dd7-960e-bbce2a9bb380
x-ms-correlation-request-id: fc81aae2-2fff-4dd7-960e-bbce2a9bb380
x-ms-routing-request-id: CENTRALUS:20190630T071516Z:fc81aae2-2fff-4dd7-960e-bbce2a9bb380
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
Date: Sun, 30 Jun 2019 07:15:15 GMT
Content-Length: 12

{"value":[]}
```

