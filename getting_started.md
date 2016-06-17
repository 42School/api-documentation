
Create an application
---------------

In order to use the 42 API, you first need to create a "v2" application [here](https://profile.intra.42.fr/oauth/applications/new).

<!-- ![new_app](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/new_app.png?token=AC4978POSgxmEGCtDRFYW3Lx-D1zeEqFks5V9PaBwA%3D%3D) -->

You will need to configure a few things in order to make your application:

- **The name** of your application, wich needs to be explicit (please avoid names like "test" or "app").
- **The redirect URI(s)**. Theses URI(s) are needed if you app acts as a third tier between the 42 data and an user (this _flow_ is called `Web Application Flow`), and specify where the user need to be redirected after his authentication. If you plan to use your app just as a server-side app, without user interaction, you can set any valid adress, you'll not need theses URI.
- **The scopes** you'll need. A scope is an aera of access. By default, your application only have access to public data, it's your call to add more scopes. Try to *only add the scopes you'll really need*, you can change your application scopes later if you need more permissions. 
- **Public** set if your application is visible by other users or not.
- **All the other fields** are facultative, and can be set later.

> Note: The complete description of the authentication process through the OAuth2 Web Application Flow is described in [the next section of this guide](/apidoc/guides/web_application_flow).


Get your credentials
-------------------------------

Awesome ! You just created your first application !
Now, take a look on your application page, we got a lot of informations there, but the most important are:

- *The client uid*, an unique identifier for your application.
- *The client secret*, an secret passphrase for your application, which **must be kept secret**, and only used on server side, where users can't see it.


Make basic requests
--------------------
Now, you have all you need to setup a little basic script using the API trough your application. In this example, we will use the **Client Credentials Flow**, in ruby, with the [OAuth2 ruby wrapper](https://github.com/intridea/oauth2), but OAuth2 wrappers exists in most languages.
The Client Credentials flow is probably the most simple flow of OAuth 2 flows. The main difference from the others is that this flow is not associated with a user.
You can read more about this OAuth flow [directly from the reference documentation of OAuth2](https://tools.ietf.org/html/rfc6749#section-1.3.4). _RIP_.

First of all, we'll request an access token with our application credentials.

```ruby

require "oauth2"

UID = "Your application uid"
SECRET = "Your secret token"

# Create the client with your credentials
client = OAuth2::Client.new(UID, SECRET, site: "https://api.intra.42.fr")

# Get an access token
token = client.client_credentials.get_token
```

Requesting an access token with the client credentials flow is, in fact, just a POST request on the `/oauth/token` endpoint with a `grant_type` parameter set to `client_credentials`. If you wanted to do this with the command line, the equivalent Curl line will be:

<pre class="command-line language-bash" data-user="andral" data-host="api" data-output="2"><code class='language-bash'>curl -X POST --data "grant_type=client_credentials&client_id=MY_AWESOME_UID&client_secret=MY_AWESOME_SECRET" https://api.intra.42.fr/oauth/token</code></pre>

```json
{
  "access_token":"42804d1f2480c240f94d8f24b45b318e4bf42e742f0c06a42c6f4242787af42d",
  "token_type":"bearer",
  "expires_in":7200,
  "scope":"public",
  "created_at":1443451918
}
```


Now, we can fetch all the public data which don't need user authentication, like the list of the cursus in 42.
The [reference documentation](https://api.intra.42.fr/apidoc) gave us ([by the `Cursus` resource page](https://api.intra.42.fr/apidoc/2.0/cursus.html)) the endpoint `/v2/cursus`.


```ruby

token.get("/v2/cursus").parsed
# => [{"id"=>1, "created_at"=>"2014-11-02T17:43:38.480+01:00", "name"=>"42", "slug"=>"42", "users_count"=>1918, "users_url"=>"https://api.intra.42.fr/v2/cursus/42/users", "projects_url"=>"https://api.intra.42.fr/v2/cursus/42/projects", "topics_url"=>"https://api.intra.42.fr/v2/cursus/42/topics"}, ...]
```

Hooray ! We got our data ! And what about the users in the cursus `42` ?

```ruby

users_in_cursus = token.get("/v2/cursus/42/users").parsed
# => {"id"=>2, "login"=>"avisenti", "url"=>"https://api.intra.42.fr/v2/users/avisenti", "end_at"=>nil}, {"id"=>3, "login"=>"spariaud", "url"=>"https://api.intra.42.fr/v2/users/spariaud", "end_at"=>nil}, ...
users_in_cursus.count
# => 30
```

What the hell ? Only 30 users ? And what says the [documentation](https://api.intra.42.fr/apidoc/2.0/cursus_users/index.html) about that ?



Pagination
----------
The documentation says that the resource is paginated by 30 items by defaut, and that we can specify a `page[number]` parameter (or, more simpler, the `page` parameter), in order to navigate trough it.
Let's try to fetch the second page:

```ruby
second_page = token.get("/v2/cursus/42/users", params: {page: {number: 2}})
# => #<OAuth2::Response:0x007f9ba3b7eb98 @response=#<Faraday::Response:0x007f9ba3b949c0 @on_complete_callbacks=[], @env=#<Faraday::Env @method=:get @body="[{\"id\":35,\"login\":\"droger\",\"url\":\"https://api.intra.42.fr/v2/users/droger\",\"end_at\":null},{\"id\":36,\"login\":\"edelbe\",\"url\":\"https://api.intra.42.fr/v2/users/edelbe\"...
second_page.parsed
# => {"id"=>35, "login"=>"droger", "url"=>"https://api.intra.42.fr/v2/users/droger", "end_at"=>nil}, {"id"=>36, "login"=>"edelbe", "url"=>"https://api.intra.42.fr/v2/users/edelbe", "end_at"=>nil}, ...
```

Well, it seems to work ! But how can we know if there is a next page ? One simple solution is to go forward until the call returns an empty array, but if we need more informations, we can take a look on the `Link` HTTP response header.

```ruby
second_page.headers["Link"]
# => "<https://api.intra.42.fr/v2/cursus/42/users?page=3>; rel=\"next\", <https://api.intra.42.fr/v2/cursus/42/users?page=1>; rel=\"prev\", <https://api.intra.42.fr/v2/cursus/42/users?page=1>; rel=\"first\", <https://api.intra.42.fr/v2/cursus/42/users?page=64>; rel=\"last\""
```

We now have the links for the first, the next, the previous and the last pages.
The response headers contains a lot of more or less useful informations, like the name of your application, the id and the roles.

```http
GET https://api.intra.42.fr/v2/messages?page[number]=1

HTTP/1.1 200 OK
Cache-Control: max-age=0, private, must-revalidate
Content-Type: application/json; charset=utf-8
ETag: W/"b326132feb08f61b7de85a13ca83f264"
Link: <https://api.intra.42.fr/v2/messages?page=1>; rel="first", <https://api.intra.42.fr/v2/messages?page=1>; rel="prev", <https://api.intra.42.fr/v2/messages?page=586>; rel="last", <https://api.intra.42.fr/v2/messages?page=3>; rel="next"
Transfer-Encoding: chunked
Vary: Origin
X-Application-Id: 318
X-Application-Name: test-app
X-Application-Roles: None
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Meta-Request-Version: 0.4.0
X-Page: 2
X-Per-Page: 30
X-Rack-CORS: preflight-hit; no-origin
X-Request-Id: c763e95e-95a6-4307-88da-f441038be349
X-Runtime: 0.278242
X-Total: 17570
X-XSS-Protection: 1; mode=block

{...}
```

You can also increase the number of results returned by the request with the `page[size]` parameter (or the `per_page` parameter). Almost all the endpoints can return up to 100 results per page.


Limits
---------

There is not rate limiting anymore.


Roles
---------

Applications can have **roles**, which grants particular privileges.

There is a short list of the most common roles:

- Alpha: Unstable features
- Beta: Intranet beta-testers
- Official App: Approved application without rate limits
- Moderator: Moderate topics, messages and versions on the forum
- Basic Tutor: Manage projects, scales and all cursus related data
- Basic Staff: Member of the staff, can manage community services, closes, exams and
  access advanced student data

The roles of your application are present in the `x-application-roles` field of the response header.

> If your application is "production ready", public and useful, you can [send us a mail](mailto:intrateam@staff.42.fr) to request the `Official App` role.

That's it for now. If your want to go deeper, and allow users to use their 42 account from a third-party website, you can continue with the [web application flow tutorial](/apidoc/guides/web_application_flow).


Getting informations about your token
----------------

If you want to know more about your token, you can fetch **https://api.intra.42.fr/oauth/token/info**.

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" https://api.intra.42.fr/oauth/token/info

# {"resource_owner_id":74,"scopes":["public"],"expires_in_seconds":7174,"application":{"uid":"3089cd94d72cc9109800a5eea5218ed4c3e891ec1784874944225878b95867f9"},"created_at":1439460680}%
```

