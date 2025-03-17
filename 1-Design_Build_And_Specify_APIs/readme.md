# Chapter 1 - Design Build and Specify APIs

## Working Examples

- gRPC implementation in go: https://grpc.io/docs/languages/go/quickstart/

## Notes

### RPC Introduction

#### gRPC

https://grpc.io/docs/what-is-grpc/core-concepts

##### Protocol Buffers

- Uses protocol buffers (Google's open source mechanism for serializing structured data)
- Define strucure for data you want to serialize in .proto file

```proto
message Person {
  string name = 1;
  int32 id = 2;
  bool has_swag = 3;
}
```

- Then, once you have specified your data structures, you use the protocol buffer compiler `protoc` to generate data access classes in your preferred language(s)
- These provide simple accessors for each field, like name() and set_name(), as well as methods to serialize/parse the whole structure to/from raw bytes.
- So, for instance, if your chosen language is C++, running the compiler on the example above will generate a class called Person. You can then use this class in your application to populate, serialize, and retrieve Person protocol buffer messages.

```proto
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

##### Types of RPCs

- Unary: Single Request to server with single response back
- Server Streaming: Client sends request to server and gets a stream to read a sequence of messages back. The client reads from the returned stream until therea are no more messages.
- Client Streaming: Client writes sequence of messages and sends them to server, again using a provided stream. Once the client has finished writing messages, it waits for the server to read them and return its response.
- Bidirectional Streaming: Both sides send messages using read-write stream. 2 streams operate independently so clients and servers can read and write in whatever order they like

##### Using the API

Starting from a service definition in a .proto file, gRPC provides protocol buffer compiler plugins that generate client and server side code. gRPC users typically call thes APIs on the client side and implement the corresponding API on the server side.

- On the server side, the server implements the methods declared by the service and runs a gRPC server to handle client calls. The gRPC infrastructure decodes incoming requests, executes service methods, and encodes service responses.
- On the client side, the client has a local object known as stub (for some languages, the preferred term is client) that implements the same methods as the service. The client can then just call those methods on the local object, and the methods wrap the parameters for the call in the appropriate protocol buffer message type, send the requests to the server, and return the server’s protocol buffer responses.

##### Synchronous vs. asynchronous

Synchronous RPC calls that block until a response arrives from the server are the closest thing to the abstraction of a procedure call that RPC aspires to. However, networks are inherently asynchronous and in many scenarios it is useful to be able to start RPCs without blocking the current thread. gRPC programming API in most languages comes in both synchronous and asynchronus flavors.

##### RPC Life Cycle

###### Unary RPC

1. Client calls stub method, the server is notified that RPC has been invoked with clients metadata for the call, the method name, and the specified deadline if applicable.
2. The server can then either send back its own metadata straight away, or wait for clients request message. Which happens first depends on the application.
3. Once the server has the clients request message, it does whatever is necessary to create and populate a response. The response is then returned to the client together with status details (status code and optional status message) and optional trailing metadata.
4. If the response status is OK, then the client gets the response, which completes the call on the client side.

###### Server Streaming RPC

Similar to unary, except that server returns a stream of messages in response to clients request. After sending all messages, the server's status details and optional trailing metadata are sent to the client. This completes processing on the server side. The client completes once it has all the servers messages.

###### Client Streaming RPC

A client-streaming RPC is similar to a unary RPC, except that the client sends a stream of messages to the server instead of a single message. The server responds with a single message (along with its status details and optional trailing metadata), typically but not necessarily after it has received all the client’s messages.

###### Bidirectional Streaming RPC

In a bidirectional streaming RPC, the call is initiated by the client invoking the method and the server receiving the client metadata, method name, and deadline. The server can choose to send back its initial metadata or wait for the client to start streaming messages.

Client- and server-side stream processing is application specific. Since the two streams are independent, the client and server can read and write messages in any order. For example, a server can wait until it has received all of a client’s messages before writing its messages, or the server and client can play “ping-pong” – the server gets a request, then sends back a response, then the client sends another request based on the response, and so on.

###### Deadlines / Timeouts

gRPC allows clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with a DEADLINE_EXCEEDED error. On the server side, the server can query to see if a particular RPC has timed out, or how much time is left to complete the RPC.

Specifying a deadline or timeout is language specific: some language APIs work in terms of timeouts (durations of time), and some language APIs work in terms of a deadline (a fixed point in time) and may or may not have a default deadline.

###### RPC Termination

In gRPC, both client and server make independent and local determinations of the success of the call, and the conclusions may not match.is means that, for example, you could have an RPC that finishes successfully on the server side (“I have sent all my responses!”) but fails on the client side (“The responses arrived after my deadline!”). It’s also possible for a server to decide to complete before a client has sent all its requests.

###### Cancelling an RPC

Either the client or the server can cancel an RPC at any time. A cancellation terminates the RPC immediately so that no further work is done.
WARNING: Changes made before cancellation are not rolled back.

###### Metadata

Metadata is information about a particular RPC call (such as authentication details) in the form of a list of key-value pairs, where the keys are strings and the values are typically strings, but can be binary data.
Keys are case insensitive and consist of ASCII letters, digits, and special characters -, \_, . and must not start with grpc- (which is reserved for gRPC itself). Binary-valued keys end in -bin while ASCII-valued keys do not.
User-defined metadata is not used by gRPC, which allows the client to provide information associated with the call to the server and vice versa.
Access to metadata is language dependent.

###### Channels

A gRPC channel provides a connection to a gRPC server on a specified host and port. It is used when creating a client stub. Clients can specify channel arguments to modify gRPC’s default behavior, such as switching message compression on or off. A channel has state, including connected and idle.
How gRPC deals with closing a channel is language dependent. Some languages also permit querying channel state.

#### gRPC Go Quick Start

Cloned from repo, added function, and regenerated code.
Update client and server code to send message.

```bash
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```
