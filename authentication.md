
## Authentication

The authentication on the 42 API works with [OAuth2](http://oauth.net/2/).

OAuth2 is a protocol that lets external apps request authorization to private details in a user’s 42 account without getting their password. This is preferred over a basic authentication because tokens can be limited to specific types of data, and can be revoked by users at any time.

All developers need to [register their application](https://profile.intrav2.42.fr/oauth/applications/new) before getting started. A registered OAuth application is assigned a unique Client ID and Client Secret. The Client Secret should not be shared.

<!-- Actually, the API support two authentication flows:
- *Web application flow*, for web-based applications. It allow 3rd party websites to log users trought their 42 account.
- *Non-web application flow*, for server based usage. -->


### Client side web Application Flow

This is a description of the OAuth2 flow from 3rd party web sites.

#### 1. Redirect users to request 42 access

    GET https://api.intrav2.42.fr/oauth/authorize

#### Parameters

Name | Type | Description
-----|------|--------------
`client_id`|`string` | **Required**. The client ID you received from 42 when you [registered](https://profile.intrav2.42.fr/oauth/applications/new).
`redirect_uri`|`string` | The URL in your app where users will be sent after authorization. See details below about [redirect urls](#redirect-urls).
`scope`|`string` | A space separated list of [scopes](#scopes). If not provided, `scope` defaults to an empty list of scopes for users that don't have a valid token for the app. For users who do already have a valid token for the app, the user won't be shown the OAuth authorization page with the list of scopes. Instead, this step of the flow will automatically complete with the same scopes that were used last time the user completed the flow.
`state`|`string` | An unguessable random string. It is used to protect against cross-site request forgery attacks.

For example:

  curl 'https://api.intrav2.42.fr/oauth/authorize?client_id=your_very_long_client_id&redirect_uri=http%3A%2F%2Flocalhost%3A1919%2Fusers%2Fauth%2Fft%2Fcallback&response_type=code&scope=public&state=a_very_long_random_string_witchmust_be_unguessable'

### 2. 42 redirects back to your site

<!-- If the user accepts your request, 42 redirects back to your site
with a temporary code in a `code` parameter as well as the state you provided in
the previous step in a `state` parameter. If the states don't match, the request
has been created by a third party and the process should be aborted.

Exchange this for an access token:

    POST https://api.intrav2.42.fr/oauth/access_token

### Parameters

Name | Type | Description
-----|------|---------------
`client_id`|`string` | **Required**. The client ID you received from 42 when you [registered](https://github.com/settings/applications/new).
`client_secret`|`string` | **Required**. The client secret you received from 42 when you [registered](https://github.com/settings/applications/new).
`code`|`string` | **Required**. The code you received as a response to [Step 1](#redirect-users-to-request-github-access).
`redirect_uri`|`string` | The URL in your app where users will be sent after authorization. See details below about [redirect urls](#redirect-urls).
`state`|`string` | The unguessable random string you optionally provided in [Step 1](#redirect-users-to-request-github-access).

### Response

By default, the response will take the following form:

    access_token=e72e16c7e42f292c6912e7710c838347ae178b4a&scope=user%2Cgist&token_type=bearer

You can also receive the content in different formats depending on the Accept
header:

    Accept: application/json
    {"access_token":"e72e16c7e42f292c6912e7710c838347ae178b4a", "scope":"repo,gist", "token_type":"bearer"}

    Accept: application/xml
    <OAuth>
      <token_type>bearer</token_type>
      <scope>repo,gist</scope>
      <access_token>e72e16c7e42f292c6912e7710c838347ae178b4a</access_token>
    </OAuth>


#### Requested scopes vs. granted scopes

The `scope` attribute lists scopes attached to the token that were granted by
the user. Normally, these scopes will be identical to what you requested.
However, users [will soon be able to edit their scopes][oauth changes blog], effectively
granting your application less access than you originally requested. Also, users
will also be able to edit token scopes after the OAuth flow completed.
You should be aware of this possibility and adjust your application's behavior
accordingly.

It is important to handle error cases where a user chooses to grant you
less access than you originally requested. For example, applications can warn
or otherwise communicate with their users that they will see reduced
functionality or be unable to perform some actions.

Also, applications can always send users back through the flow again to get
additional permission, but don’t forget that users can always say no.

Check out the [Basics of Authentication guide][basics auth guide] which
provides tips on handling modifiable token scopes.

#### Normalized scopes

When requesting multiple scopes, the token will be saved with a normalized list
of scopes, discarding those that are implicitly included by another requested
scope. For example, requesting `user,gist,user:email` will result in a
token with `user` and `gist` scopes only since the access granted with
`user:email` scope [is included](#scopes) in the `user` scope.

### 3. Use the access token to access the API

The access token allows you to make requests to the API on a behalf of a user.

    GET https://api.github.com/user?access_token=...

You can pass the token in the query params like shown above, but a
cleaner approach is to include it in the Authorization header

    Authorization: token OAUTH-TOKEN

For example, in curl you can set the Authorization header like this:

    curl -H "Authorization: token OAUTH-TOKEN" https://api.github.com/user -->
