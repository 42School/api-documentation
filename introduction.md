The 42 API provide programmatic access to read and write 42's data. You can get and interact with the whole intranet data, and do things such creating a new message on the forum, read users profiles, get datas on a project, and more.
On his second version, it identifies 42 applications and users using OAuth, and responses are available in JSON.

Whether you're looking to build an official 42 integration for your service, or you just want to build something awesome, [we can help you get started](/apidoc/guides/getting_started).

Specifications
--------------

The API works over the https protocol. and accessed from the *api.intrav2.42.fr* domain.

- The current endpoint is https://api.intrav2.42.fr/v2.
- All data is sent and received as JSON.
- Blank fields are included as null instead of being omitted.
- All timestamps are returned in ISO 8601 format


Current version
----------------

The current API version is *2.0a*. The first version of the API is now deprecated, and we encourage all API v1 applications developers to migrate their apps to this version.


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
response_type|string | The response type. Ususally `code`.

All this things will make together a nice and understandable URI, like:

`https://api.intrav2.42.fr/oauth/authorize?client_id=your_very_long_client_id&redirect_uri=http%3A%2F%2Flocalhost%3A1919%2Fusers%2Fauth%2Fft%2Fcallback&response_type=code&scope=public&state=a_very_long_random_string_witchmust_be_unguessable'`

> *Small note*: when formatting the scopes parameters, be sure to read above about the distinction between application-level and token-level scopes. this has been a point of friction for some developers.

#### 2. 42 redirects back to your site

![auth_dialog](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/authorize_dialog.png?token=AC497ynhhrHdo8rZlplIQ_tb4Fd2wbT4ks5V1J_kwA%3D%3D) _The 42 auth dialog_

If the user grants the permission for your application to use the requested data (see [scopes](#scopes)), it will be redirected to your `redirect_uri` with a temporary code in a GET `code` parameter as well as the state you provided in the previous step in a `state` parameter.
> If the states don't match, the request has been created by a third party and the process should be aborted.

#### 3. Exchange your code for an access token

You're almost here !
The last thing to do is a POST request to the `https://api.intrav2.42.fr/oauth/token` endpoint, with your `client_id`, your `client_secret`, the previous `code` and your `redirect_uri`. **This request must be performed on server side, over a secure connexion**.

Useless note: This corresponds to the token endpoint, section 3.2 of the OAuth 2 RFC. Happy ?

##### Base url

    POST https://api.intrav2.42.fr/oauth/token

##### Parameters

Name | Type | Description
-----|------|--------------
grant_type | string |  **Required**. The grant type. In this case, it's `authorization_code`.
client_id | string |  **Required**. The client ID you received from 42 when you registered.
client_secret | string |  **Required**. The client secret you received from 42 when you registered.
code  | string |  **Required**. The code you received as a response to [Step 1](#1-redirect-users-to-request-42-access).
redirect_uri  | string |  The URL in your app where users will be sent after authorization.
state | string |  The unguessable random string you optionally provided in [Step 1](#1-redirect-users-to-request-42-access).

For example, with curl:

```bash
curl -F grant_type=authorization_code \
-F client_id=9b36d8c0db59eff5038aea7a417d73e69aea75b41aac771816d2ef1b3109cc2f \
-F client_secret=d6ea27703957b69939b8104ed4524595e210cd2e79af587744a7eb6e58f5b3d2 \
-F code=fd0847dbb559752d932dd3c1ac34ff98d27b11fe2fea5a864f44740cd7919ad0 \
-F redirect_uri=https://myawesomeweb.site/callback \
-X POST https://api.intrav2.42.fr/oauth/token
```

#### 4. Make API requests with your token

Include your token in all your requests in a authorization header:

```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

For example, you can fetch the current token owner, with curl:

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" https://api.intrav2.42.fr/v2/me

# {"id":23,"alias":"30_1@orga.42.fr","email":"30_1@orga.42.fr","login":"30_1","url":"http://localhost:12000/v2/users/30_1","mobile":null,"displayname":"Mathieu TRENTIN","image_url":"https://cdn.42.fr/userprofil/profilview/30_1.jpg","staff?":true,"wallet":0,"groups":[{"id":3,"name":"pixel"},{"id":1,"name":"staff"}],"user_data":{"correction_point":5,"location":null,"campus":{"id":1,"name":"Paris","created_at":"2015-05-19T12:53:31.459+02:00","updated_at":"2015-07-20T19:28:05.730+02:00","time_zone":"Paris","language_id":1,"slug":"paris"}},"cursus":[{"id":1,"created_at":"2014-11-02T17:43:38.480+01:00","name":"42","slug":"42","users_count":2024,"url":"http://localhost:12000/v2/cursus/42"}]}
```

> If you can't modify http headers, you can send your token as a `access_token` parameter.

--------------------------------------------------------------------------------------------------

If you want to know more about your token, you can fetch **https://api.intrav2.42.fr/oauth/token/info**.

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" https://api.intrav2.42.fr/oauth/token/info

# {"resource_owner_id":74,"scopes":["public"],"expires_in_seconds":7174,"application":{"uid":"3089cd94d72cc9109800a5eea5218ed4c3e891ec1784874944225878b95867f9"},"created_at":1439460680}%
```



## Errors

The 42 API uses the following error codes:

| Http Code | Error code | Meaning |
|-----------|-----------:|--------:|
| 400       |            | The request is malformed |
| 401       |            | Unauthorized |
| 403       |            | Forbidden|
| 404       |            | Page or resource is not found|
| 422       |            | Unprocessable entity|
| 500       |            | We have a problem with our server. Please try again later.|
| Connection refused       |            |Most likely cause is not using HTTPS. |




## Scopes

App can have different access scopes.

Authorization scopes are a way to determine to what extent the client can use resources located in the provider.

When the client requests the authorization it specifies in which scope they would like to be authorized. This information is then displayed to the user - resource owner - and they can decide whether or not they accept the given application to be able to act in specified scopes.





## Pagination

The 42 API paginates all indexes pages.
`TODO`



    




    




    




    





