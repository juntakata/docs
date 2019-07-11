# Sign in Azure AD B2C users and call Web API from a JavaScript single-page application (SPA)

## Create Azure AD B2C application

1. Go to Azure AD B2C blade.
2. Select [Application] and [+ Add]
3. Enter a name for the application. For example, b2cwebapp.
4. For Include web app/ web API and Allow implicit flow, select Yes.
5. For Reply URL, enter an endpoint where Azure AD B2C should return any tokens that your application requests.
6. Click Create.

## Create user flows

1. Go to Azure AD B2C blade.
2. Select [User flows (policies)] and then select [+ New user flow].
3. On the Recommended tab, select the Sign up and sign in user flow
4. Enter a Name for the user flow. For example, SignUpAndSignIn.
5. For Identity providers, select Email signup.
6. For User attributes and claims, choose the claims and attributes that you want to collect and send from the user during sign-up. 
7. Cick Create to add the user flow. A prefix of B2C_1 is automatically appended to the name.

## Create a html file

```html
<html>

<head>
    <title>Calling a Web API as a user authenticated with Msal.js app</title>
    <style>
        .hidden {
            visibility: hidden
        }

        .visible {
            visibility: visible
        }

        .response {
            border: solid;
            border-width: thin;
            background-color: azure;
            padding: 2px;
        }
    </style>
</head>

<body>
    <!-- bluebird only needed if this page needs to run on Internet Explorer -->
    <!-- msal.min.js can be used in the place of msal.js; included msal.js to make debug easy -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/bluebird/3.3.4/bluebird.min.js" class="pre"></script>
    <script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.2/js/msal.js"></script>
    <script src="https://code.jquery.com/jquery-3.2.1.min.js" class="pre"></script>

    <h2>Getting an access token with Azure AD B2C and calling a Web API</h2>
    <div>
        <div id="label">Sign-in with Microsoft Azure AD B2C</div>
        <h4 id="WelcomeMessage"></h4>
        <button id="auth" onclick="login()">Login</button><br /><br />
        <pre id="json"></pre>
    </div>

    <pre class="response"></pre>

    <script>
        // configuration to initialize msal
        var msalConfig = {
            auth: {
                clientId: "779c0c56-ef9b-4eb4-b9ad-3aad0e7d259e", //This is your client ID
                authority: "https://contosob2c.b2clogin.com/contosob2c.onmicrosoft.com/B2C_1_SignUpAndSignIn", //This is your tenant info
                validateAuthority: false
            },
            cache: {
                cacheLocation: "localStorage",
                storeAuthStateInCookie: true
            }
        };

        var apiConfig = {
            apiEndpoint: "https://your.api.example.com/v1.0/"
        };

        var requestObj = {
            scopes: ["https://contosob2c.onmicrosoft.com/b2cwebapp/user.read"]
        };

        function callWebApi(theUrl, accessToken, callback) {
            var xmlHttp = new XMLHttpRequest();
            xmlHttp.onreadystatechange = function () {
                if (this.readyState == 4 && this.status == 200)
                    callback(JSON.parse(this.responseText));
            }
            xmlHttp.open("GET", theUrl, true); // true for asynchronous
            xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
            xmlHttp.send();
        }

        function authRedirectCallBack(error, response) {
            if (error) {
                console.log(error);
                logMessage(error);
            }
            else {
                if (response.tokenType === "id_token") {
                    console.log("token type is:" + response.tokenType);
                } else if (response.tokenType === "access_token") {
                    console.log("token type is:" + response.tokenType);
                    callWebApi(apiConfig.apiEndpoint, response.accessToken, apiCallback);
                } else {
                    console.log("token type is:" + response.tokenType);
                }
            }
        }

        function apiCallback(data) {
            document.getElementById("json").innerHTML = JSON.stringify(data, null, 2);
        }

        function acquireTokenRedirectAndCallWebApi() {
            console.log("acquireTokenRedirectAndCallWebApi: Entered");
            //Always start with acquireTokenSilent to obtain a token in the signed in user from cache
            clientApplication.acquireTokenSilent(requestObj).then(function (tokenResponse) {
                console.log("acquireTokenRedirectAndCallWebApi: acquireTokenSilent is successful");
                callWebApi(apiConfig.apiEndpoint, tokenResponse.accessToken, apiCallback);
            }).catch(function (error) {
                console.log(error);
                logMessage(error);
                // Upon acquireTokenSilent failure (due to consent or interaction or login required ONLY)
                // Call acquireTokenRedirect
                if (requiresInteraction(error.errorCode)) {
                    clientApplication.acquireTokenRedirect(requestObj);
                }
            });
        }

        function requiresInteraction(errorCode) {
            if (!errorCode || !errorCode.length) {
                return false;
            }
            return errorCode === "consent_required" ||
                errorCode === "interaction_required" ||
                errorCode === "login_required";
        }

        var clientApplication = new Msal.UserAgentApplication(msalConfig);
        clientApplication.handleRedirectCallback(authRedirectCallBack);

        function login() {
            clientApplication.loginRedirect(requestObj);
        }

        function showWelcomeMessage() {
            var userName = clientApplication.getAccount().name;
            logMessage("User '" + userName + "' logged-in");

            var authButton = document.getElementById('auth');
            authButton.innerHTML = 'logout';
            authButton.setAttribute('onclick', 'logout();');
        }

        function logout() {
            // Removes all sessions, need to call AAD endpoint to do full logout
            clientApplication.logout();
        }

        function logMessage(s) {
            document.body.querySelector('.response').appendChild(document.createTextNode('\n' + s));
        }

        if (clientApplication.getAccount() && !clientApplication.isCallback(window.location.hash)) { // avoid duplicate code execution on page load in case of iframe and popup window.
            showWelcomeMessage();
            acquireTokenRedirectAndCallWebApi();
        }

    </script>
</body>

</html>
```