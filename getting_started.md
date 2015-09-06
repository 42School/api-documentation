
Create an application
---------------

In order to use the 42 API, you first need to create a "v2" application [here](https://profile.intrav2.42.fr/oauth/applications/new), and get this creation form.

![new_app](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/new_app.png?token=AC4978POSgxmEGCtDRFYW3Lx-D1zeEqFks5V9PaBwA%3D%3D)

You will need to configure a few things in order to make your application:
- **The name** of your application, wich needs to be explicit (please avoid names like "test" or "app").
- **The redirect URI(s)**. Theses URI(s) are needed if you app acts as a third tier between the 42 data and an user (this _flow_ is called `Web Application Flow`), and specify where the user need to be redirected after his authentication. If you plan to use your app just as a server-side app, without user interaction, you can set any valid adress, you'll not need theses URI.
- **The scopes** you'll need. A scope is an aera of access. By default, your application only have access to public data, it's your call to add more scopes. Try to *only add the scopes you'll really need*, you can change your application scopes later if you need more permissions. 

> Note: The complete description of the authentication process through the OAuth2 Web Application Flow is described in the [introduction section of theses guides](/apidoc/guides/introduction#web-application-flow).


Get your application credentials
-------------------------------

Awesome ! You just created your first application !
Now, take a look on your application page, we got a lot of informations there, but the most important are:
- *The client uid*, an unique identifier for your application.
- *The client secret*, an secret passphrase for your application, which **must be kept secret**, and only used on server side, where users can't see it.


Start fetching data
--------------------
Now, you have all you need to setup a little basic script using the API trough your application. In this example, we will use ruby, with the [OAuth2 ruby wrapper](https://github.com/intridea/oauth2), but OAuth2 wrappers exists in most languages.

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
users_in_cursus.count
# => 30
```

What the hell ? Only 30 users ? And what says the [documentation](https://api.intrav2.42.fr/apidoc/2.0/cursus_users/index.html) about that ?
![doc_pagination](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/doc_pagination.png?token=AC4978POSgxmEGCtDRFYW3Lx-D1zeEqFks5V9PaBwA%3D%3D)
