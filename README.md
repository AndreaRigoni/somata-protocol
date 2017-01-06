# Somata Protocol

Somata is a framework for building networked microservices, supporting both remote procedure call (RPC) and publish-subscribe models of communication. 

* [Overview](#)
    * [Service vs. Client](#)
    * [Service discovery](#)
    * [Message passing](#)
* [Service lifecycle](#)
    * [Registration](#)
    * [Client heartbeating](#)
* [Client lifecycle](#)
    * [Lookup](#)
    * [Service heartbeating](#)
* [Messages](#)
    * [Sent by Clients](#)
        * [Method](#)
        * [Subscribe](#)
        * [Unsubscribe](#)
        * [Ping](#)
    * [Sent by Services](#)
        * [Response](#)
        * [Error](#)
        * [Event](#)
        * [Pong](#)
* [Classes](#)
    * [Service](#)
    * [Client](#)
    * [Binding](#)
    * [Connection](#)

## Overview

To start using Somata you need to understand three things: the **Service**, the **Client**, and the **Registry**.

### Service vs. Client

The two main building blocks of Somata are the **Service** and **Client**. These are represented in somata-node as the `somata.Service` and `somata.Client` classes.

A **Service** has a name and exposes a set of methods, and may publish events.

```js
var service = new somata.Service(name, methods)
```

A **Client** manages connections to one or more Services, to call methods and subscribe to events.

```js
var  client = new somata.Client()
client.remote(service_name, method, args..., cb)
```

![](https://i.imgur.com/PiiuL7a.png)

### Registry

Service discovery is managed by the [Somata Registry](https://github.com/somata/somata-registry), which is itself a specialized Service. Other Services will send registration information (i.e. their name and binding port) to the Registry when started. Before a Client makes a connection to a Service, it must ask the Registry for connection details.

![](https://i.imgur.com/ejzIx4f.png)


### Message passing

Clients and Services communicate by passing JSON encoded messages over ZeroMQ sockets. Every message has an `id` and a `kind`, further attributes depend on the kind.

The purpose of the Somata Protocol is to define which kinds of messages are sent when.

![](https://i.imgur.com/PhcM5QA.png)

## Messages

### Sent by Client

#### Method

To call a remote method on a Service, a Client sends a *method* message with strings `service` and `method` and an array of arguments `args`. It expects exactly one *response* or *error* message in return.

```json
{"id": "1", "kind": "method", "service": "hello", "method": "sayHello", "args": ["world"]}
```

#### Subscribe

To subscribe to events from a Service, a Client sends a *subscribe* message with strings `service` and `type`. It expects 0 or more *event* messages in return.

```json
{"id": "2", "kind": "subscribe", "service": "hello", "type": "hi"}
```

#### Ping

When a Client first connects to a Service it sends a *ping* message with `ping: "hello"`. It expects exactly one *pong* message `pong: "welcome"`.

After the hello & welcome exchange, the Client continues the ping loop, sending a *ping* message with `ping: "ping"` and expecting one *pong* with `pong: "pong"`.

```json
{"id": "3", "kind": "ping", "service": "hello", "ping": "hello"}
```

### Sent by Service

#### Response

In response to a *method* message from a Client, a Service may send a *response* message with any object `response`, using the same `id`.

```json
{"id": "1", "kind": "response", "response": "Hello, world!"}
```

#### Error

In response to a *method* message from a Client, a Service may send an *error* message with a string `error`, using the same `id`.

```json
{"id": "1", "kind": "error", "error": "No such method 'sayEhllo'"}
```

#### Event

After a *subscribe* message from a Client, a Service may send 0 or more *event* messages with any object `event`, using the same `id`.

```json
{"id": "2", "kind": "event", "event": "Just saying hi."}
```

#### Pong

After a *ping* message from a Client, a Service should send a *pong* message with the appropriate `pong` value, using the same `id`.

If a Client has sent a *ping* message with `ping: "hello"`, the Service should send a *pong* with `pong: "welcome"`.

If a Client has sent a *ping* message with `ping: "ping"`, the Service should send a *pong* with `pong: "pong"`.

```json
{"id": "3", "kind": "pong", "pong": "welcome"}
```

---

## Classes

### Service

### Client

### Binding

### Connection
