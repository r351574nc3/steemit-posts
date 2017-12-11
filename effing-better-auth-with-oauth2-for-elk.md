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

This is exactly why it cannot work with Oauth2.


### OAuth2 is for Client/User Authorization

That is, it requires a user on one end to provide information. You can do just server-to-server authorization; however, we are trying to add OAuth2 to kibana. It will not serve use to user server-to-server. 

[Some solutions](https://www.elastic.co/blog/user-impersonation-with-x-pack-integrating-third-party-auth-with-kibana) actually suggest this as an answer by layering *impersonation* on top of that. How does server-to-server authz with user impersonation not sound like absolute h4x0ry? It's just silly. Let's pretend we didn't hear that silliness.

## The Answer


