# Motivation

After developing using the websocket api I have learned that it is very difficult to continually collect data through this interface if I want to build applications that depend on large amounts of data. For example, if I have 3 applications that are separate and all query account history, this can lead to gateway timeouts. It is far better for one application to read more account history and have the other applications feed off of it, but this leads to horrible and inefficient software design. What can I do to get around it?

## Aggregation

Of course, I could just read the data through and aggregate it, but to where? A database? I realized that would just be re-inventing the wheel. There has to be a better way. This is where distributed logs really shine. A distributed log like Kafka will allow me to dump blocks and transactions to a feed just like I am getting from the websocket api.

### Custom Events

One might ask, "Why not just use a witness RPC server?" True. Running a witness and syncing transactions would solve aggregation. I could then just point my apps to my local RPC server instead of to a remote server. Then I could have exclusive access to the streams.

However, there is an added advantage to running a more abstract source. I can add whatever I want to it. With a distributed log, I can now utilize reactive software development methodologies. I can even implement [event sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)

With event sourcing, comes custom events. This will be profoundly powerful for steem developers because now they can add events and link those events to transactions in the blockchain without affecting the blockchain directly.


# Technical Details

* Kafka for distributed log
* Golang websocket api frontend
* OpenAPI (Swagger)
* WebAssembly

# Implementation 

Kafka will be deployed through a docker container. Another service will stream transactions from steem and repost transactions to the kafka distributed log. Each steem plugin (`database_api`, `login_api`, etc...) will get an exclusive topic within the kafka distributed log. This will allow users to watch for events by topic.

An additional websocket api will be added to the distributed log to expose WebAssembly and OpenAPI schema'd API's.