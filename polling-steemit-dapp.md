# Motivation

A new application with a user interface for creating polls/straw polls for bloggers that want this kind of feedback. The user interface will allow users to create polls like
```
**what paints to use?**
[] water color
[] acrylic
[] oil
[] latex
```

Then users will be able to respond. The goal is to provide a mostly seamless interface between the condenser and the polling app. This means whether you view the poll in the polling app or in the condenser UI, it will still make sense. There will be more features in the polling app that make it more desirable to use, but the condenser interface will not be confusing to those that stumble across polls. It will still be usable as well for condenser viewers.

## Technical Details

* VueJS
* Typescript

## Implementation

Application parse standard markdown and interpret markdown details as a specification for the poll. Markdown conceals details of 
* Poll title
* Poll options
* Poll result

### Votes

Voting on a poll uses markdown as convention again to maintain seamlessness with the condenser.

### Blockchain Storage

Polling information is calculated and persisted within the blockchain as `json_metadata` of the post. Below is the schema:
```
poll: {

}
```


