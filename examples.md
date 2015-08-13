## Examples


#### Simple server based example with ruby

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






    




    




    




    





