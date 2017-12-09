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

### OAuth2 Flow

Due to the nature of the web OAuth2 is connection-less. That is, it isn't something that just stays open like a VPN. Instead, numerous requests and responses are made for verification.

<div class="alert alert-warning" role="alert">
  <h4 class="alert-heading">Well done!</h4>
  <p>Aww yeah, you successfully read this important alert message. This example text is going to run a bit longer so that you can see how spacing within an alert works with this kind of content.</p>
  <hr>
  <p class="mb-0">Whenever you need to, be sure to use margin utilities to keep things nice and tidy.</p>
</div>

## X-Pack: How it works

Now that we understand a bit how OAuth2 works, let's have a look into X-Pack and see exactly why these two are in conflict.

## The Answer


