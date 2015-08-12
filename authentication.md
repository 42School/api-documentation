## Authentication

The authentication on the 42 API works with [OAuth2](http://oauth.net/2/).

OAuth2 is a protocol that lets external apps request authorization to private details in a user’s 42 account without getting their password. This is preferred over a basic authentication because tokens can be limited to specific types of data, and can be revoked by users at any time.

All developers need to [register their application](https://profile.intrav2.42.fr/oauth/applications/new) before getting started. A registered OAuth application is assigned a unique Client ID and Client Secret. The Client Secret should not be shared.

### Web Application Flow

This is a description of the OAuth2 flow from 3rd party web sites.

#### 1. Redirect users to request 42 access

That's the first step. Link or redirect users to the API authorize url: `https://api.intrav2.42.fr/oauth/authorize`.
This must be properly formatted for your application and will return a permissions screen for the user to authorize. For convenience, a formatted authorize URL including your client_id is provided for each application in the [apps page](https://profile.intrav2.42.fr/oauth/applications).

##### Base url

    GET https://api.intrav2.42.fr/oauth/authorize

##### Parameters

Name | Type | Description
-----|------|--------------
client_id|string | **Required**. The client ID you received from 42 when you [registered](https://profile.intrav2.42.fr/oauth/applications/new).
redirect_uri|string | The URL in your app where users will be sent after authorization. See details below about [redirect urls](#redirect-urls).
scope|string | A space separated list of [scopes](#scopes). If not provided, `scope` defaults to an empty list of scopes for users that don't have a valid token for the app. For users who do already have a valid token for the app, the user won't be shown the OAuth authorization page with the list of scopes. Instead, this step of the flow will automatically complete with the same scopes that were used last time the user completed the flow.
state|string | An unguessable random string. It is used to protect against cross-site request forgery attacks.

All this things will make together a nice and understandable URI, like:

    https://api.intrav2.42.fr/oauth/authorize?client_id=your_very_long_client_id&redirect_uri=http%3A%2F%2Flocalhost%3A1919%2Fusers%2Fauth%2Fft%2Fcallback&response_type=code&scope=public&state=a_very_long_random_string_witchmust_be_unguessable'

> *Small note*: when formatting the scopes parameters, be sure to read above about the distinction between application-level and token-level scopes. this has been a point of friction for some developers.

#### 2. 42 redirects back to your site

![auth_dialog](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/authorize_dialog.png?token=AC497ynhhrHdo8rZlplIQ_tb4Fd2wbT4ks5V1J_kwA%3D%3D) _The 42 auth dialog_

If the user grants the permission for your application to use the requested data (see [scopes](#scopes)), it will be redirected to your `redirect_uri`, with is the url provided...

-----------

## Simple server based examples

### With ruby

Simple example using the [oauth2 ruby wrapper](https://github.com/intridea/oauth2) with simple token flow. In this example, you only have access to public resources, which don't need user credentials.

```ruby

require "oauth2"

UID = "Your application uid"
SECRET = "Your secret token"

# Create the client with your credentials
client = OAuth2::Client.new(UID, SECRET, site: "https://api.intrav2.42.fr")

# Get an access token
token = client.client_credentials.get_token

# Make your requests
# Don't forget the "/v2" namespace, or your will request the first version of the 42 API
token.get("/v2/cursus").parsed
```






    




    




    




    





