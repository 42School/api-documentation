
Create an application
---------------

In order to use the 42 API, you first need to create a "v2" application [here](https://profile.intrav2.42.fr/oauth/applications/new), and get this creation form.

![new_app](https://raw.githubusercontent.com/lambda2/42-API-Documentation/master/images/new_app.png?token=AC4978POSgxmEGCtDRFYW3Lx-D1zeEqFks5V9PaBwA%3D%3D)

You will need to configure a few things in order to make your application:
- **The name** of your application, wich needs to be explicit (please avoid names like "test" or "app").
- **The redirect URI(s)**. Theses URI(s) are needed if you app acts as a third tier between the 42 data and an user (this _flow_ is called `Web Application Flow`), and specify where the user need to be redirected after his authentication. The complete description of the authentication process through the OAuth2 Web Application Flow is described in the [introduction section of theses guides](/apidoc/guides/introduction#web-application-flow). If you plan to use your app just as a server-side app, without user interaction, you can set any valid adress, you'll not need theses URI.