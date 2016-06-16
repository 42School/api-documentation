The 42 API provide programmatic access to read and write 42's data. You can get and interact with the whole intranet data, and do things such creating a new message on the forum, read users profiles, get datas on a project, and more.
On his second version, it identifies 42 applications and users using OAuth, and responses are available in JSON.

Whether you're looking to build an official 42 integration for your service, or you just want to build something awesome, [we can help you get started](/apidoc/guides/getting_started).


Specifications
--------------

The API works over the https protocol. and accessed from the *api.intra.42.fr* domain.

- The current endpoint is https://api.intra.42.fr/v2.
- All data is sent and received as JSON.
- Blank fields are included as null instead of being omitted.
- All timestamps are returned in ISO 8601 format

Current version
----------------

The current API version is *2.0*.

Authentication
----------------

The authentication on the 42 API works with [OAuth2](http://oauth.net/2/).

OAuth2 is a protocol that lets external apps request authorization to private details in a user’s 42 account without getting their password. This is preferred over a basic authentication because tokens can be limited to specific types of data, and can be revoked by users at any time.

All developers need to [register their application](https://profile.intra.42.fr/oauth/applications/new) before getting started. A registered OAuth application is assigned a unique Client ID and Client Secret. The Client Secret should not be shared.

Errors
----------------

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




Scopes
----------------

App can have different access scopes.

Authorization scopes are a way to determine to what extent the client can use resources located in the provider.

When the client requests the authorization it specifies in which scope they would like to be authorized. This information is then displayed to the user - resource owner - and they can decide whether or not they accept the given application to be able to act in specified scopes.

Requesting a resource with wrong or insufficient scopes will return a `403 Forbidden` response, with more details in the `WWW-Authenticate` response header. For example, for an application without the `projects` scope:

```sh
GET https://api.intra.42.fr/v2/me/slots
403
Cache-Control:max-age=0, private, must-revalidate
Content-Type:application/json; charset=utf-8
ETag:W/"4d4c5e9e9191ebd5b384aca5e6c5b5b7"
Transfer-Encoding:chunked
Vary:Origin
WWW-Authenticate:Bearer realm="42 API", error="insufficient scope", error_description="The action need the following scopes: [projects]"
```




Pagination
----------------

The 42 API paginates all resources on the "index" method.

Requests that return multiple items will be paginated to 30 items by default. You can specify further pages with the `page[number]` parameter. You can also set a custom page size (up to 100) with the `page[size]` parameter. Note that for technical reasons not all endpoints can go up to 100 on the the `page[size]` parameter.

The `Link` HTTP response header contains pagination data with `first`, `previous`, `next` and `last` raw pages links when available, under the format

```
link: <http://api.intra.42.fr/v2/{Resource}?page=X+1>; rel="next", <http://api.intra.42.fr/v2/{Resource}?page=X-1>; rel="prev", <http://api.intra.42.fr/v2/{Resource}?page=1>; rel="first", <http://api.intra.42.fr/v2/{Resource}?page=X+n>; rel="last"
```

There is also:
- A `X-Page` header field, which contains the current page.
- A `X-Per-Page` header field, which contains the current pagination length.
- A `X-Total` header field, which contains the count of pages.





