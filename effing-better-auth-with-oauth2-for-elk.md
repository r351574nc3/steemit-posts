Don’t be misled by the title. You may be thinking, “Finally! Instructions on how to implement Oauth2 with X-pack.” Ok, it is about that, but first you should know that there is no support for Oauth2 with X-pack. Stop looking for it. This is about how you implement it yourself the hard way. Strap in folks. This is going to be a ride for the ages.

I encountered this problem personally and I could not find any solution in forums, documentation (Ha!), or blogs. I really 
looked a lot and hard. The best I could find are some forum posts that were closed with no resolution and 
[X-Pack with OAuth Authentication](https://discuss.elastic.co/t/x-pack-with-oauth-authentication/72015/2) which led me to this
gem [Integrating with Other Authentication Systems](https://www.elastic.co/guide/en/x-pack/current/custom-realms.html)

Those are just for reference. Observe and familiarize yourselves with the contents, but believe me when I say, 
> These are not the droids you are looking for

Some of you may be reading this and hadn't yet come across the links given above. Some of you already have and arrived at the
same conclusion as I have. If you are already familiar with the problem, just skip to [the answer](#the-answer). If you are new
this issue and need some explanation or at least some convincing about the solution, read on.

## Oauth2 Refresher

Before getting to the real meat of things, we first need to understand (big picture) how OAuth2 works and how it fits in with X-Pack and the ELK stack.

Rather than get into technical details an history of what OAuth2 is and why to use it, I'm going to go straight into the flow to illustrate what exactly is happening during OAuth2 authentication process.

### OAuth2 is *Not* Authentication

OAuth2 flow is actually what occurs after authentication. Authentication is the process of being challenged for credentials (a login screen), then verifying those credentials. Oauth2 does not use standard credentials. Access is managed through a session token.

### OAuth2 Flow

Due to the nature of the web OAuth2 is connection-less. That is, it isn't something that just stays open like a VPN. Instead, numerous requests and responses are made for verification. This is referred to as the *oauth2 flow*. 

1. **Auth Initialization** request an oauth2 auth from the application to be accessed. The auth request usually consists of (grant type, code, and request uri)
1. **Authentication** this is actually outside of the Oauth2 flow, but we are going to start here. Normally, you authenticate some way (Oauth2 is not authentication, it's for authorization).
1. **Start flow** now that authentication is complete flow begins.
1. **Client Verification Request** is what the application will now try to do. It will actually send a request back to the client (*Note: Remember this part. It will be important later.*). The request contains a *code* (the same one sent from the client originally to the oauth2 provider.)
1. **Client Verification Response** will now take the same *code* given in the previous step and compare it against the code that was sent.
1. **Token Request** once the code is verified and everyone agrees they are who they say they are (sounds like some shady backroom deal), the application will request a token from the Oauth2 provider. It will make a request with the client id, client secret, credentials type, and scope as parameters for the token. The client id and client secret are specific to the application. The user has no notions about what this is or what it's for. It is kept hidden from the user.
1. **Token Response** the Oauth2 provider will not respond back to the client with an access token as part of the response payload. This token can be used within a `Authorization: Bearer` HTTP header to whatever service within the Oauth2 domain that it needs access to. This token can be saved to the client session, and reused as long as the user remains logged in.

## X-Pack: How it works

Now that we understand a bit how OAuth2 works, let's have a look into X-Pack and see exactly why these two are in conflict.

### X-Pack Resides in Elasticsearch

X-Pack is actually a plugin for Elasticsearch. X-Pack has its own set of extensions that can be installed; however, it is basically just Elasticsearch. That means it runs within Netty and produces/consumes HTTP requests.

For reference, here is a link to the [X-Pack Security API](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/security-api-authenticate.html). From the examples, you can see that logging in with X-Pack is as simple as `_xpack/security/_authenticate`. Unlike OAuth2, it is only one request. X-Pack only supports Basic Authorization which means you simply make a `GET` request with the `Authorization: Basic` header. If your response is anything other than status `200`, it failed authentication. Pretty simple.

### Too Simple

This is exactly why it cannot work with Oauth2. The normal login process for kibana is a user provides Username/Password credentials at a login screen. Instead of `POST` of this information to a form

1. A REST request is sent to (not X-Pack) Kibana containing the Username/Password credentials
1. Kibana's builtin X-Pack client then sends a request to X-Pack for authentication at `_xpack/security/_authenticate`
1. The response is handled and authz details are then stored in session state
1. Kibana then re-requests the `/` main page which loads session data and skips the login

> **To recap** The login page doesn't make a `POST`. Instead it makes a REST call, then reloads the main page. This is important because it does not follow a flow. It just loops itself. It's also important to note that a REST call is what's actually authorizing. This means, the login page basically throws auth over a wall and waits to see if anything gets thrown back.

![](https://memegenerator.net/img/images/600x600/2726051/face-palm.jpg)

^^ **EXACTLY WHY OAUTH2 CAN'T WORK**

> Read on for more details.

### OAuth2 is for Client/User Authorization

That is, it requires a user on one end to provide information. You can do just server-to-server authorization; however, we are trying to add OAuth2 to kibana. It will not serve use to user server-to-server. 

[Some solutions](https://www.elastic.co/blog/user-impersonation-with-x-pack-integrating-third-party-auth-with-kibana) actually suggest this as an answer by layering *impersonation* on top of that. How does server-to-server authz with user impersonation not sound like absolute h4x0ry? It's just silly. Let's pretend we didn't hear that silliness and try to do things correctly just this once.

### X-pack is for Service Authentication and Authorization

X-Pack handles simple credentials over HTTP because it's assuming communication from another service (like the X-pack client embedded into kibana).

It can't work because 
1. X-pack is Authentication and Authorization (Oauth2 is just Authorization)
1. Kibana has no way to get a response from X-pack other than the session. 
1. Third-party auth providers (SSO for example) cannot gain access to X-pack
1. OAuth2 flow requires redirect to the Authentication provider for user input
1. OAuth2 flow requires client verification by sending a request back to a redirect uri
1. OAuth2 flow returns an authorization token in the HTTP response (not a cookie)

## The Answer

Ok, now that we're all familiar with the problem... It seems hopeless, right? It's not. The solution is actually very simple. Here's why:

### Kibana is built on HapiJS

[HapiJS](https://github.com/hapijs/hapi) is a web application toolkit created by the folks at Walmart Labs. It comes with a plugin (Kibana is actually using it for auth) called [Bell](https://github.com/hapijs/bell) for Authorization and Authentication. It's rather brilliant. Most importantly, it supports OAuth2 out-of-the box. This is great because you can write a plugin for Kibana that instantly will hook you up with Oauth2.

### What about X-pack? 

This is actually very simple. We just make our Oauth2 plugin an X-pack client. Now upon completing the Oauth2 flow, we can authenticate with X-pack. I will explain how, but first we will tackle the Kibana Plugin.

### Custom Kibana OAuth2 Plugin

Here is the [Example Project Source](https://github.com/r351574nc3/sample-kibana-oauth-plugin). 

#### Template

The first thing I did was create a base kibana plugin from the [kibana template](https://github.com/elastic/template-kibana-plugin). It uses SAO.js. It works really well and will get your project started. I just did:

```
npm install -g sao
sao oauth2-kibana-plugin
```

Now we have a base project.

#### Moar Project Setup

Before moving on, I want to make sure I have all the requisite plugins/module for OAuth.

```
npm install -S bell@8.7.0 hapi@14.2.0 hapi-auth-cookie@6.1.1
```

I choose these because at the time of my writing this blog post, these are the libraries that kibana uses and their respective versions.

#### Custom Plugin Guts

Just to get it out of the way, here are my imports that I'm using:

```javascript
const Querystring = require('querystring');
const Bell = require('bell');
const Boom = require('boom');
const uuidv4 = require('uuid/v4');
const Hydra = require('./lib/providers/hydra');
const Wreck = require('wreck');
```

Next, I setup my plugin boilerplate and model. It should look something like this:
```
export default function (kibana) {
  return new kibana.Plugin({
    require: ['kibana', 'elasticsearch', 'security'],
    name: 'oauth-kibana-plugin',
    uiExports: {

    },
    config(Joi) {
      return Joi.object({
        enabled: Joi.boolean().default(true),
        provider: Joi.string().default('hydra'),
        isSecure: Joi.boolean().default(true),
        clientId: Joi.string(),
        clientSecret: Joi.string(),
        internalName: Joi.string(),
        cookieName: Joi.string().default('sid')
      }).default();
    },
    init: function (server, options) {
      const config = server.config();
    }
  })
}


```
The above just sets up a plugin that basically does nothing and defines a model for the plugin that does nothing. This model is important because it actually integrates with the `kibana.yaml`. The properties of the model are mirrored in the `kibana.yaml` as follows:

```yaml
oauth-kibana-plugin:
    enabled: true
    provider: google
    clientId: blah
    clientSecret: moarblah
    internalName: local DNS name for kibana
```

At this time, `init` points to a function that simply loads the config. Now, I will add some initialization function for the plugin to actually do something.

```
server.register([Bell], function (err) {
    if (err) {
        throw err;
    }

    server.auth.strategy(options.provider, 'bell', {
        config: {
            authHost: options.providerHost,
            userInfoUrl: options.userInfoUrl
        },
        location: options.redirectUri,
        password: options.password,
        provider: options.provider,
        clientId: options.clientId,
        clientSecret: options.clientSecret,
        skipProfile: false,
        scope: ['profile']
    });
});
```

The above will register my provider as the default auth strategy. This is important. In our case, I have chosen `'google'` as the provider. This will automatically set everything up for oauth2 with google. Pretty easy? If only it were that simple. As explained earlier, this is just the beginning.

Next, I setup managed state. `Bell` and `Kibana` already manage state through a session cookie with `hapi-auth-cookie`. I am going to reuse this cookie to store additional information for the session. 

```
// Setup Session cookie
server.state('credentials', {
    isSecure: true,
    ttl: null,
    isHttpOnly: true,
    encoding: 'base64json'
});
```

Now, a new state variable `'credentials'` will store session information for auth. This is important because we will use it to access credential information later when passing it to and from xpack.

Now, when accessing kibana, you are immediately redirected to the standard kibana authorization page; however, we are going to bypass this page in order to access our SSO or oauth credentials page. To do this, we use the following:

```
server.ext('onRequest', (req, reply) => {
    if (req.path() == '/login') {
        reply.redirect('/auth/login');
    }
});
```

The above will redirect to `/auth/login` instead of `/login`.

#### Defining the route

Now that everything is mostly setup, now we need to configure the actual auth and communication with x-pack. To do this we setup a custom route. Our new route is going to be `/auth/login`.

```javascript
server.route({
    method: ['GET', 'POST'],
    path: '/auth/login'
});
```

The route doesn't really do anything yet. First, I need to setup the config
```javascript
    config: {
        auth: options.provider
    }
```

This config just sets up `google` as my chosen auth provider for `Bell` and basically by doing so flips on the switch for `Bell`. 

Next, I need to setup my route handler. This is the logic for what happens server-side when the route is accessed.
```javascript
    handler: function (request, reply) {
        return reply()
    }
```

Just returning an empty reply for now. Doesn't really do anything. First, we want to fetch credentials from the request.

```javascript
console.log('Received OAuth Call back.');
const credentials = request.auth.credentials;

if (!request.auth.isAuthenticated) {
    return reply('Authentication failed due to: ' + request.auth.error.message);
}
```

Now that we have our auth creds, we're going to forward them on to x-pack to connect with elasticsearch using oauth. In order to do that, we will prepare our request to x-path.

First, we will need to setup our `Authorization` header. At the time of my writing this, elasticsearch/x-pack only support Auth Basic. The header will look something like this:
`Authorization: Basic Base64encoded<<username>:<password>>`. What is really interesting about this and why it's so important is that the `credentials.token` is the actual `oauth` token that is being passed through to x-pack.

Here, I construct that header:
```javascript
const base64Auth = new Buffer(`${credentials.profile.username}:${credentials.token}`).toString('base64');
const requestOptions = {
    payload: {
        username: credentials.profile.username,
        password: credentials.token
    },
    headers: {
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=UTF-8',
        'DNT': 1,
        'kbn-name': 'kibana',
        'kbn-version': '5.6.2',
        'Referer': options.redirectUri + '/auth/login',
        'Origin': options.redirectUri,
        'X-Hydra-Authorization': new Buffer(`Basic ${base64Auth}`)
    }
};
```

Next, I need to construct the call. Kibana already comes equipped with an elasticsearch client that is fully capable of auth. There is no need for me to write a new one. In fact, it even has a web interface that I can utilize. I just make an internal call to kibana, and let it negotiate the communication for me.

```javascript
var response = '';
const loginUrl = 'http://' + options.internalName + '/api/security/v1/login';
console.log('Received OAuth Call back - redirecting authentication to XPac Security to url:', loginUrl + ' for user ' + credentials.profile.username);

Wreck.post(loginUrl, requestOptions, (err, res, payload) => {
if (err ||
    res.statusCode < 200 ||
    res.statusCode > 299) {
    return reply(Boom.unauthorized('Authorization with elasticsearch failed'));
}
```

Next, we need to handle that response. Was it successful? Who knows? Part of handling the response is adding the response to the current kibana session. This way, kibana registers the successful auth through x-pack. To do this, we use a poor man's cookie-passthru.

```javascript
response = reply().code(302).header('Location', '/')
    .state('credentials', credentials)

var sidcookie = '';

if (res.headers['set-cookie'] !== null
    && res.headers['set-cookie'].length > 0) {
    sidcookie = res.headers['set-cookie'][0];

}
response = response.header('Set-Cookie', sidcookie);

return response;
});
```

#### Custom Providers

If you are so inclined to use a custom OAuth2 provider because your client has their own, that's no problem. Here is how you will create your own provider.

First, create a path for your provider. I used `lib/providers/`. Then, add a source file for your provider. 
> In this example, I've created my own custom [Hydra provider](https://github.com/ory/hydra) (`hydra.js`).

Mine looks something like this:
```javascript
'use strict';

const Querystring = require('querystring');
const Url = require('url');
const Boom = require('boom');
const Cryptiles = require('cryptiles');
const Crypto = require('crypto');
const Hoek = require('hoek');
const Wreck = require('wreck');

const internals = {};

exports = module.exports = function (options) {
    options = options || {};

    return {
        name: 'hydra',
        protocol: 'oauth2',
        useParamsAuth: false,
        auth: 'https://' + options.authHost + '/oauth2/auth',
        token: 'https://' + options.authHost + '/oauth2/token',
        scope: ['profile'],
        profile: function (credentials, params, get, callback) {
            get(options.userInfoUrl, null, (profile) => {
                credentials.profile = {
                    username: profile.username,
                    displayName: profile.name,
                    groups: profile.groups,
                    email: profile.email,
                    raw: profile
                };
                return callback();
            });
        }
    };
};
```

You can find other examples for providers at [the Bell Source Repository](https://github.com/hapijs/bell/tree/master/lib/providers).

Additionally, I have to replace the plugin `init` function `server.register` call with the following:

```javascript
Bell.providers.hydra = Hydra;
server.auth.strategy(options.provider, 'bell', {
    config: {
        authHost: options.providerHost,
        userInfoUrl: options.userInfoUrl
    },
    location: options.redirectUri,
    password: options.password,
    provider: options.provider,
    clientId: options.clientId,
    clientSecret: options.clientSecret,
    skipProfile: false,
    scope: ['profile']
});
```

Where `options.provider` ultimately is `"hydra"`.

#### Building

To build this awesome plugin, just use:
```shell
npm run build
```

The file will then be located in the `build` directory.

### Custom X-Pack Realm Extension

Here is the [Example Project Source](https://github.com/r351574nc3/sample-xpack-oauth-plugin)

This is the second part of this. Once we have built a `kibana` plugin to communicate with x-pack upon oauth, we now need x-pack to comply. For this, I created a custom realm as explained in the [x-pack custom realm documentation](https://www.elastic.co/guide/en/x-pack/current/custom-realms.html).

> Notice: Before installing this plugin, please make sure you have taken appropriate security measures as outlined here https://discuss.elastic.co/t/how-to-customize-plugin-security-policy-for-custom-realm/71570/5

Once again, we have to setup the plugin. I do this by modifying `src/main/resources/x-pack-extension-descriptor.properties` with the following:
```
description=Kibana Custom Realm Extension
version=${version}
name=customrealm
classname=com.github.r351574nc3.realm.CustomRealmExtension
java.version=${java.version}
xpack.version=${xpack.version}
```

I also setup `src/main/resources/x-pack-extension-security.policy`

```
grant {
  permission java.lang.RuntimePermission "createClassLoader";
  permission java.lang.RuntimePermission "getClassLoader";
  permission java.lang.RuntimePermission "accessDeclaredMembers";
  permission java.lang.RuntimePermission "accessClassInPackage.sun.reflect";
  permission java.lang.RuntimePermission "accessClassInPackage.jdk.internal.reflect";
  permission java.lang.reflect.ReflectPermission "suppressAccessChecks";
  permission java.net.SocketPermission "*", "resolve,connect";
  permission java.net.URLPermission "${kibana.userInfoUrl}", "POST:Accept-EncodingUser-Agent,GET";
  
  // Standard set of classes
  permission org.elasticsearch.script.ClassPermission "<<STANDARD>>";
  permission org.elasticsearch.script.ClassPermission "sun.reflect.ConstructorAccessorImpl";
  permission org.elasticsearch.script.ClassPermission "sun.reflect.MethodAccessorImpl";
  permission org.elasticsearch.script.ClassPermission "jdk.internal.reflect.ConstructorAccessorImpl";
  permission org.elasticsearch.script.ClassPermission "jdk.internal.reflect.MethodAccessorImpl";
};
```

Now to actually change stuff. I modify my `CustomRealmExtension.java` and add the following method:

```java
@Override
public Collection<String> getRestHeaders() {
    return Arrays.asList("authorization", "Authorization");
}
```

This specifies to let the `Authorization` header through with rest calls to x-pack. It allows my realm to pickup the `Authorization` header. 

Next, I need to handle this header. It contains my `oauth` token. 

In my `CustomRealm.java`, I make sure I have the following:
```java
@Override
public boolean supports(AuthenticationToken token) {
    log.debug("Checking to see if " + token + " is supported");
    return token instanceof UsernamePasswordToken;
}
```

Then I add
```java
@Override
public UsernamePasswordToken token(final ThreadContext threadContext) {
    final String authStr = threadContext.getHeader(AUTH_HEADER);

    if (authStr == null) {
        log.debug("Authorization again: " + threadContext.getHeader("Authorization"));
        final UsernamePasswordToken retval = usernamePasswordToken(threadContext);
        log.debug("Using token: " + retval);
        return retval;
    }

    if (authStr.lastIndexOf(" ") < 0) {
        throw new RuntimeException("Unable to verify token from header: " + authStr);
    }

    final String authB64 = authStr.substring(authStr.lastIndexOf(" "), 1);
    final String[] authArr = new String(Base64.getDecoder().decode(authB64)).split(":");
    final String user = authArr[0];
    final String token = authArr[1];

    return new UsernamePasswordToken(user, new SecureString(token.toCharArray()));
}
```

This is important because what it does is it parses the `Authorization` header and strips the token value to be used later.

Once the token is retrieved, it is verified through the `authenticate` method. I have that overridden here:
```java
public void authenticate(AuthenticationToken authenticationToken, ActionListener<User> listener) {
    final UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
    try {
        listener.onResponse(new User(token.principal(), getGroupsFor(token.credentials())));
    } 
    catch (Exception e) {
        listener.onFailure(e);
    }
}
```

`getGroupsFor` is important because kibana determines access through roles. `getGroupsFor` could be whatever you want to call it. Roles are just an array of strings anyway. Specify the roles you want and this will determine the access the user gets. It's intended that `getGroupsFor` will communicate with the `oauth` provider.

#### Building

To build this, just run
```shell
gradle clean buildZip
```

