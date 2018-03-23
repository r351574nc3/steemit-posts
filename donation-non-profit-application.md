# Non-profit Donation Application

I was inspired recently by [Sneaky Ninja Supports You Are Hope!!](https://steemit.com/steemit/@sneaky-ninja/sneaky-ninja-supports-you-are-hope). I was inspired because @youarehope is a growing organization committed to helping the less fortunate world-wide and a bot is contributing. I thought to myself, "How many others want to contribute and are off-put because they have to send a memo." What if users, just want to send 5% of their earnings on every post and/or comment?

There is always the option of delegating though, right? Delegating is an easy way to give without actually giving. You're giving influence. It's not real money, but it can become money through curation. That is, if someone runs a campaign for @youarehope, then the account can upvote which will contribute as well as gain curation rewards. It's a win win, so in that sense delegation works. 

I think though that when it comes to giving, the more options, the better. Normally, I think choices are quite bad, but in this case it's more inclusive to have choices.

So far we have:
* Memos
* Delegation

As means to donate. 

# What about Beneficiaries?
----
What if you could assign beneficiaries on posts? Here are a couple use cases.

## Donate 5% from all posts

> I realize I already mentioned this one.

What if you could just donate 5% of all your earnings from posts and/or comments to @youarehope? 

## Donate by tag

Suppose you want any posts you tag with #youarehope will add @youarehope as a beneficiary to that post and give a percentage of the payouts?

## Donate by Campaign

Maybe you want to run a special campaign for your organization that will give a percentage to @youarehope

Normally, campaigns would require a payout and then the person/organization running the campaign would have to send a memo. What if you could just have a portion of the payout directed to @youarehope automatically?

### What's a Campaign 

Campaigns can be managed/tracked through the blockchain. A campaign can span several posts and even tie-in comments and memos.

What really defines a campaign is metadata stored on a series of posts that tie them together.

#### Displaced Orphans of Ghana Campaign

This is just an example. However, the point is that this campaign can span several posts, initiatives and even accounts. Being able to define a campaign on a post would ensure that funds stay related to the campaign and are not appropriated for other things.

## Requirements

* Campaigns store metadata in steemit blockchain
* Campaign management through a user interface
* Automation that tracks campaign, tag, and other details about posts to add beneficiaries to posts upon their creation.
* Use steemconnect to provide application access for users
* Persist preferences for giving
    * Beneficiary percentage
    * Campaign management
    * User access control

## Technical Details

* User interface built with nodejs and atlaskit
* Persistance outside of the blockchain for preferences in a postgreSQL database.
* Container cluster deployment.
* Independent worker process.

## Implementation

### Enrollment

Users can sign up with the application through steemconnect. Enrollment gives users access to a preferences dashboard as well as storing access control information in a database.

### Campaign Management

The preferences dashboard will have a campaign management interface that allows the users to build/track campaigns.

### Beneficiary Assignment

An automated process (bot) waits for criteria according to preferences and assigns beneficiaries according to preferences on designated posts.

### Campaign information

The aforementioned automated process also stores campaign metadata in posts for tracking.
