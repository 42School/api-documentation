
Create an application
---------------

In order to use the 42 API, you first need to create a "v2" application [here](https://profile.intrav2.42.fr/oauth/applications/new), and get this creation form.

![new_app](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/new_app.png?token=AC4978POSgxmEGCtDRFYW3Lx-D1zeEqFks5V9PaBwA%3D%3D)

You will need to configure a few things in order to make your application:

- **The name** of your application, wich needs to be explicit (please avoid names like "test" or "app").
- **The redirect URI(s)**. Theses URI(s) are needed if you app acts as a third tier between the 42 data and an user (this _flow_ is called `Web Application Flow`), and specify where the user need to be redirected after his authentication. If you plan to use your app just as a server-side app, without user interaction, you can set any valid adress, you'll not need theses URI.
- **The scopes** you'll need. A scope is an aera of access. By default, your application only have access to public data, it's your call to add more scopes. Try to *only add the scopes you'll really need*, you can change your application scopes later if you need more permissions. 

> Note: The complete description of the authentication process through the OAuth2 Web Application Flow is described in the [introduction section of theses guides](/apidoc/guides/introduction#web-application-flow).


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
  client = OAuth2::Client.new(UID, SECRET, site: "https://api.intrav2.42.fr")

  # Get an access token
  token = client.client_credentials.get_token
```

Now, we can fetch all the public data which don't need user authentication, like the list of the cursus in 42.
The [reference documentation](https://api.intrav2.42.fr/apidoc) gave us ([by the `Cursus` resource page](https://api.intrav2.42.fr/apidoc/2.0/cursus.html)) the endpoint `/v2/cursus`.


```ruby

token.get("/v2/cursus").parsed
# => [{"id"=>1, "created_at"=>"2014-11-02T17:43:38.480+01:00", "name"=>"42", "slug"=>"42", "users_count"=>1918, "users_url"=>"https://api.intrav2.42.fr/v2/cursus/42/users", "projects_url"=>"https://api.intrav2.42.fr/v2/cursus/42/projects", "topics_url"=>"https://api.intrav2.42.fr/v2/cursus/42/topics"}, ...]
```

Hooray ! We got our data ! And what about the users in the cursus `42` ?

```ruby

users_in_cursus = token.get("/v2/cursus/42/users").parsed
# => {"id"=>2, "login"=>"avisenti", "url"=>"https://api.intrav2.42.fr/v2/users/avisenti", "end_at"=>nil}, {"id"=>3, "login"=>"spariaud", "url"=>"https://api.intrav2.42.fr/v2/users/spariaud", "end_at"=>nil}, ...
users_in_cursus.count
# => 30
```

What the hell ? Only 30 users ? And what says the [documentation](https://api.intrav2.42.fr/apidoc/2.0/cursus_users/index.html) about that ?



Pagination
----------
The documentation says that the resource is paginated by 30 items, and that we can specify a `page` parameter, in order to navigate trough it.
Let's try to fetch the second page:

```ruby
second_page = token.get("/v2/cursus/42/users", params: {page: 2})
# => #<OAuth2::Response:0x007f9ba3b7eb98 @response=#<Faraday::Response:0x007f9ba3b949c0 @on_complete_callbacks=[], @env=#<Faraday::Env @method=:get @body="[{\"id\":35,\"login\":\"droger\",\"url\":\"https://api.intrav2.42.fr/v2/users/droger\",\"end_at\":null},{\"id\":36,\"login\":\"edelbe\",\"url\":\"https://api.intrav2.42.fr/v2/users/edelbe\"...
second_page.parsed
# => {"id"=>35, "login"=>"droger", "url"=>"https://api.intrav2.42.fr/v2/users/droger", "end_at"=>nil}, {"id"=>36, "login"=>"edelbe", "url"=>"https://api.intrav2.42.fr/v2/users/edelbe", "end_at"=>nil}, ...
```

Well, it seems to work ! But how can we know if there is a next page ? One simple solution is to go forward until the call returns an empty array, but if we need more informations, we can take a look on the `Link` HTTP response header.

```ruby
second_page.headers["Link"]
# => "<https://api.intrav2.42.fr/v2/cursus/42/users?page=3>; rel=\"next\", <https://api.intrav2.42.fr/v2/cursus/42/users?page=1>; rel=\"prev\", <https://api.intrav2.42.fr/v2/cursus/42/users?page=1>; rel=\"first\", <https://api.intrav2.42.fr/v2/cursus/42/users?page=64>; rel=\"last\""
```
We now have the links for the first, the next, the previous and the last pages.
The response headers contains a lot of more or less useful informations, like the name of your application, the id, the rates limits and the roles.

```http
x-frame-options: SAMEORIGIN
x-xss-protection: 1; mode=block
x-content-type-options: nosniff
x-ratelimit-limit: '100'
x-ratelimit-remaining: '12'
x-application-name: My first application
x-application-id: '1'
x-application-roles: None
link: <https://api.intrav2.42.fr/v2/cursus/42/users?page=3>; rel="next", <https://api.intrav2.42.fr/v2/cursus/42/users?page=1>;
  rel="prev", <https://api.intrav2.42.fr/v2/cursus/42/users?page=1>; rel="first", <https://api.intrav2.42.fr/v2/cursus/42/users?page=64>;
  rel="last"
content-type: application/json; charset=utf-8
etag: W/"78edc461d5186b60dec4e2fd515dda64"
cache-control: max-age=0, private, must-revalidate
x-runtime: '0.751982'
vary: Origin
x-rack-cors: preflight-hit; no-origin
connection: close
```


Limits
---------
By default, your application is "rate limited", and can only make a limited number of requests per hour. The `x-ratelimit-*` fields of the response header can give us some informations about that.

- The `x-ratelimit-limit` field shows the request limit per hour of our application.
- The `x-ratelimit-remaining` field shows the number of request remaining in this hour for our application.

Now, let's see what happens when we exceed the rate limit, for example by fetching 100 times the users in the 42 cursus.

```ruby
(1..100).each{|i| token.get("/v2/cursus/42/users", params: {page: i})}
# OAuth2::Error: Too many requests within the hour:
# => {"error":"Too many requests within the hour"}
```

Damn ! Okay, let's wait an hour...



Roles
---------
Applications can have **roles**, which grants particular privileges.
For example, applications recognized as being of public interest may have the role `Official App`, which eliminates any request limitation.
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

