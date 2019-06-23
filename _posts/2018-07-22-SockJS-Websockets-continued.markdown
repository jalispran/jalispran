---
title: "SockJS - Websockets continued"
layout: post
date: 2018-07-22
tag: 
    - Websocket
    - HTTP Vs Websocket
    - STOMP
    - SockJs
description: "This is the follow up blog post for the earlier blog on Websocket protocol."
category: blog
author: pranjal
---

## Who should read this?
This is a follow up blog post after [Websockets - All you need to know][1]. If you have not checked it out, I recommend going through that first unless you have your basics clear and just want to refresh **SockJS**.

The goal of SockJS is to let applications use a WebSocket API but fall back to non-WebSocket alternatives when necessary at runtime, i.e. without the need to change application code.

> This blog uses an example from [callicoder.com](https://www.callicoder.com) to show SockJS in action. The link to the example can be found at the bottom of this post.

## What you should expect?
1. [How does SockJS work?](#how-does-sockjs-work)
2. [IE8 & 9 Case](#ie8--9-case)
3. [Authentication](#authentication)
4. [User destinations](#user-destinations)
5. [Performance](#performance)
6. [Points to remember](#points-to-remember)
7. [Spring specifics](#spring-specifics)
8. [Further reading](#further-reading)

## How does SockJS work?
#### Step 1: GET /info - to obtain basic information from the server
This is a request for information that can influence the client’s choice of transports. After that it must decide what transport to use. If possible WebSocket is used. If not, in most browsers there is at least one HTTP streaming option and if not then HTTP (long) polling is used.

![Markdown Image](../assets/websockets-all-you-need-to-know/snapshot2.png)
<figcaption class="caption">Step 1 /info message sent by client</figcaption>

All transport requests have the following URL structure:
{% highlight html %}
http://host:port/myApp/myEndpoint/{server-id}/{session-id}/{transport}
{% endhighlight %}

**server-id** - useful for routing requests in a cluster but not used otherwise <br/>
**session-id** - correlates HTTP requests belonging to a SockJS session <br/>
**transport** - indicates the transport type, e.g. *websocket*, *xhr-streaming*, etc. <br/>

The session between the client and the server is always initialized by the client. The client chooses **server_id**, which should be a three digit number: *000 to 999*. It can be provided by the user or it can be generated randomly. The main reason for this parameter is to make it easier to configure load balancer - and enable sticky sessions based on first part of the url.

Second parameter **session_id** must be a random string, unique for every session. It is undefined what happens when two clients share the same **session_id**. It is client's responsibility to choose an identifier with enough entropy.

![Markdown Image](../assets/websockets-all-you-need-to-know/snapshot3.png)
<figcaption class="caption">Server response showing the server_id and the session_id</figcaption>

*SockJS* adds minimal message framing. For example the server sends **open frame** initially. Messages are sent as a ["message1","message2"] (JSON-encoded array). **Heartbeat frame** is sent if no messages flow for 25 seconds by default, and **close frame** to close the session. 

##### Note
* It is important to note that neither server nor client API's can expose *session_id* to the application. This field must be protected from the app. The server must accept any value in server and session fields.
* A session is identified by only *session_id*. Whereas, *server_id* is a parameter for load balancer and must be ignored by the server.

#### Step 2: Framing accepted by client
1. Open Frame (o) - Every time a new session is established, the server must immediately send the *open frame*. This is required, as some protocols (mostly polling) can't distinguish between a properly established connection and a broken one - we must convince the client that it is indeed a valid url and it can be expecting further messages in the future on that url.
2. Heartbeat frame (h) -  Most loadbalancers have arbitrary timeouts on connections. In order to keep connections from breaking, the server must send a *heartbeat frame* every now and then. <u>The typical delay is 25 seconds and should be configurable</u>.
3. Array (a) -  of json-encoded messages. For example: a["message"].
4. Close frame (c) - This frame is send to the browser every time the client asks for data on closed connection. This may happen multiple times. *Close frame* contains a code and a string explaining a reason of closure, like: c[3000,"Go away!"]

![Markdown Image](../assets/websockets-all-you-need-to-know/snapshot6.png)
<figcaption class="caption">Showcasing various frames exchanged between client and server</figcaption>

#### Step 3: Framing accepted by server
SockJS server does not have any framing defined. All incoming data is treated as incoming messages, either single json-encoded messages or an array of json-encoded messages, depending on transport.

![Markdown Image](../assets/websockets-all-you-need-to-know/snapshot7.png)
<figcaption class="caption">Summary of the entire process. Notice blue comments scribbled in the image?</figcaption>

<div class="breaker"></div>

## IE8 & 9 Case
The *SockJS* client supports *Ajax/XHR* streaming in IE 8 and 9 via Microsoft’s *XDomainRequest*. That works across domains but does not support sending cookies. Cookies are very often essential for Java applications. However since the *SockJS* client can be used with many server types (not just Java ones), it needs to know whether cookies matter.
The very first */info* request from the SockJS client is a request for information that can influence the client’s choice of transports. One of those details is whether the server application relies on cookies?

<div class="breaker"></div>

## Authentication
Every STOMP over WebSocket messaging session begins with an HTTP request.

Web applications already have authentication and authorization in place to secure HTTP requests. Therefore for a WebSocket handshake, or for SockJS HTTP transport requests, typically there will already be an authenticated user.

> In short there is nothing special a typical web application needs to do above and beyond what it already does for security

Note that the STOMP protocol does have a *login* and *passcode* headers on the **CONNECT** frame. Those were originally designed for and are still needed, for example, for STOMP over TCP.

### Token authentication (JWT)
JWT can be used as the authentication mechanism in Web applications including STOMP over WebSocket interactions by maintaining identity through a cookie-based Websocket/SOCKJS session.
However, not all web applications see it fit to maintain cookie-based sessions on the server. Also, for mobile applications where its common to use headers for authentication instead of cookie-based sessions.

> The WebSocket protocol [RFC 6455][2] "doesn’t prescribe any particular way that servers can authenticate clients during the WebSocket handshake."

Therefore applications that wish to avoid the use of cookies may not have any good alternatives for authentication at the HTTP authentication level. Instead of using cookies you may prefer to authenticate with headers at the STOMP messaging protocol level at **CONNECT** message.

## User destinations
For sending messages to a perticular user you will have to go through the documentation of your specific STOMP Client as WebSocket/STOMP being protocols just mention how messages are sent and received. And they do not talk about any specific use cases.

<div class="breaker"></div>

## Performance
In a messaging application messages are passed through channels for asynchronous executions backed by thread pools. Configuring such an application requires good knowledge of the channels and the flow of messages.

> Check out Spring specific flow of messages [here][3]

However, here are some recommendations:
1. If the handling of messages is mainly CPU bound then the number of threads in the inbound channel should remain close to the number of processors. 
2. If the work they do is more IO bound and requires blocking or waiting on a database or other external system then the thread pool size will need to be increased.
3. On the outboud channel it is all about sending messages to WebSocket clients. If clients are on a fast network then the number of threads should remain close to the number of available processors. If they are slow or on low bandwidth they will take longer to consume messages and put a burden on the thread pool. Therefore increasing the thread pool size will be necessary.

<div class="breaker"></div>

## Points to remember
* WebSocket is a computer communications protocol. It is a low-level transport protocol.
* Keys in request and response handshake are different.
* If a WebSocket server is running behind a web server (e.g. nginx) you will likely need to configure the web server to pass WebSocket upgrade requests on to the WebSocket server. Likewise if the application runs in a cloud environment, check the instructions of the cloud provider related to WebSocket support.
* WebSocket clients and servers can negotiate the use of a higher-level, messaging protocol (e.g. STOMP), via the "Sec-WebSocket-Protocol" header on the HTTP handshake request, or in the absence of that they need to come up with their own conventions.
* The WebSocket transport needs only a single HTTP request to do the WebSocket handshake. All messages thereafter are exchanged on that socket.
* The SockJS client supports Ajax/XHR streaming in IE 8 and 9 via Microsoft’s XDomainRequest.
* SockJS client can be used with many server types (not just Java ones)
* If you do use an iframe-based transport, and in any case, it is good to know that browsers can be instructed to block the use of IFrames on a given page by setting the HTTP response header X-Frame-Options to DENY, SAMEORIGIN. This is used to prevent *clickjacking*.
* Although *STOMP* is a text-oriented protocol, message payloads can be either text or binary.
* STOMP Commands used by client - **SEND** and **SUBSCRIBE**
* STOMP Commands used by server - **MESSAGE** (to broadcast messages to all subscribers)
* Long polling is similar except it ends the current request after each server-to-client send.

<div class="breaker"></div>

## Spring specifics
* On the server side you may want to enable *TRACE* logging for **org.springframework.web.socket**.
* For generating a Server Key (Sec-WebSocket-Key):
{% highlight html %}
  byte[] nonce = new byte[16];
  random.nextBytes(nonce);
  this.key = ByteString.of(nonce).base64();
{% endhighlight %}
* For *STOMP* over *WebSocket* by default Spring ignores authorization headers at the STOMP protocol level and assumes the user is already authenticated at the HTTP transport level and expects that the WebSocket or SockJS session contain the authenticated user. Use *ChannelInterceptor* to provide authentication using STOMP headers.
* *STOMP* messages whose destination header starts with */app* may be routed to **@MessageMapping** methods in annotated controllers, while */topic* and */queue* messages may be routed directly to the message broker.
* When using Spring Security’s authorization for messages, at present you will need to ensure that the authentication **ChannelInterceptor** config is ordered ahead of Spring Security’s. This is best done by declaring the custom interceptor in its own implementation of **WebSocketMessageBrokerConfigurer** marked with **@Order(Ordered.HIGHEST_PRECEDENCE + 99)**.

<div class="breaker"></div>

## Further reading
1. <https://sockjs.github.io/sockjs-protocol/sockjs-protocol-0.3.3.html>
2. <https://tools.ietf.org/html/rfc6455>
3. <https://stomp.github.io/stomp-specification-1.2.html#Abstract>
4. <https://www.callicoder.com/spring-boot-websocket-chat-example/>


---

[1]: {{site.url}}/Websockets-all-you-need-to-know
[2]: https://tools.ietf.org/html/rfc6455#section-10.5
[3]: https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow