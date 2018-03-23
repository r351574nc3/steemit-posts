# Blockchain Backed Issue Tracking

This idea came to me after I had been using @Utopian-io for some time. I realized, @Utopian-io feels a bit awkward as a development platform because it still feels like a blog. It doesn't feel like a tool I would use as a developer. It doesn't even feel like a development blog. 

It feels like a place I go to. I do my due diligence of project management overhead by creating a detailed (redundant) description of my project. This in my mind is really just my obligatory ticket (I think that's the best way to describe it), to get paid. The trouble is, I think a lot of people feel this way. Basically, people feel that they've already done the work (not just by building the contribution, but by also filling out the ticket), and now they're entitled a payout.

True, that's their fault for feeling that way, but I think Utopian is also partially to blame because it's a blog impersonating a development portal. 

## Let's be Constructive

This article is not about bashing @Utopian-io. The above is just my motivation. I am speaking from a place of constructive criticism. I do not wish to bash @Utopian-io. Rather, I see a need, and I want to fill that need. I think for @Utopian-io to really be a useful portal, there are a lot of things it needs, but highest in my list of priorities is integrated issue tracking.

## Requirements

### Github Integration

@Utopian-io already includes a considerable amount of additional metadata including GH information into the `json_metadata` of a post. The kind of GH integration I'm thinking of is.

#### GH pull requests as posts

When a PR is created in GH, a post about the pull request is automatically posted to the steemit blockchain. This allows us to upvote changes directly without a @utopian-io post. Comments to the PR are also replicated as comments on the steemit post and can be up/downvoted. This allows us to use steemit for code reviews and users on steemit to be more engaged in OSS.

#### GH issues as posts

Steemit users can also post issues to GH projects. When an issue is created a post about the issue is automatically posted to the steemit blockchain. This allows us to upvote issues directly without a @utopian-io post. Comments to the issue are also replicated as comments on steemit post.

## Technical Details

* Golang project
* Uses Steemit Distributed Log Passthru
* User interface using atlaskit
* Allow application access to GH posting

## Implementation

To implement this there will need to be 
* An automated process to watch for GH changes
* An application that a user can give access to through GH to impersonate them (this feature is builtin to GH already).
* A user interface to set preferences.

### User interface

The user interface is really just a dashboard for managing application access for GH. The automation will need to impersonate the user. For this to happen, it needs to be given access to GH.

### Automation (Bot)

When changes happen to a project (either issues or PRs), a notification is sent via github webhook to a bot. The bot then handles impersonation of the steemit user. The bot also reads changes from the steemit blockchain through the distributed log passthru. Comments about the issue/pr are then sent to the issue/pr in GH.

