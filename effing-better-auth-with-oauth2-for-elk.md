Don’t be misled by the title. You may be thinking, “Finally! Instructions on how to implement Oauth2 with X-pack.” Ok, it is about that, but first you should know that there is no support for Oauth2 with X-pack. Stop looking for it. This is about how you implement it yourself the hard way. Strap in folks. This is going to be a ride for the ages.

I encountered this problem personally and I could not find any solution in forums, documentation (Ha!), or blogs. I really 
looked a lot and hard. The best I could find are some forum posts that were closed with no resolution and 
[X-Pack with Oauth Authentication](https://discuss.elastic.co/t/x-pack-with-oauth-authentication/72015/2) which led me to this
gem [Integrating with Other Authentication Systems](https://www.elastic.co/guide/en/x-pack/current/custom-realms.html)

Those are just for reference. Observe and familiarize yourselves with the contents, but believe me when I say, 
> These are not the droids you are looking for

Some of you may be reading this and hadn't yet come across the links given above. Some of you already have and arrived at the
same conclusion as I have. If you are already familiar with the problem, just skip to [the answer](#the-answer). If you are new
this issue and need some explanation or at least some convincing about the solution, read on.

## Oauth2 Refresher

Before getting to the real meat of things, we first need to understand (big picture) how Oauth2 works and how it fits in with X-Pack
and the ELK stack.

## The Answer


