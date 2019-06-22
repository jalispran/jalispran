---
title: "Websocket - All you need to know"
layout: post
date: 2018-07-18
tag: 
    - Websocket
    - HTTP Vs Websocket
    - STOMP
    - SockJs
description: "This is my first blog post about the concepts of websockets. This has to have another follow up blog post."
category: blog
author: pranjal
---

## Who should read this?
If you are like me who likes to get familiar with the theoretical aspect of a topic before proceeding with implementation, you have arrived at the right place. This is where I try to compile theory about websocket protocol, from various sources, and try to make it as simple and understandable as possible.

>If you want to see websockets in action, stay tuned in for the next blog on SockJS.

### What is this about?
* Websocket protocol
* HTTP Vs Websocket
* STOMP
* When to use websocket

---

### Websocket Protocol:
The WebSocket protocol provides a standardized way to establish a full-duplex, two-way communication channel between client and server over a single TCP connection.

The protocol consists of an opening handshake followed by basic message framing, layered over TCP. The goal of this technology is to provide a mechanism for browser-based applications that need two-way communication with  servers that does not rely on opening multiple HTTP connections (e.g., using XMLHttpRequest / AJAX or <pre><iframe>s</pre> and long polling).

It is a different TCP protocol from HTTP but is designed to work over HTTP, using ports 80 and 443 and allowing re-use of existing firewall rules. A WebSocket interaction begins with an HTTP request that uses the HTTP “Upgrade” header to upgrade, or in this case to switch, to the WebSocket protocol.

> The WebSocket Protocol enables two-way communication between a client running untrusted code in a controlled environment to a remote host that has opted-in to communications from that code.

{% highlight raw %}
    GET /my-websocket-demo HTTP/1.1
    Host: localhost:8183
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
    Sec-WebSocket-Protocol: v10.stomp, v11.stomp
    Sec-WebSocket-Version: 13
    Origin: http://localhost:8183
{% endhighlight %}

![Markdown Image](../assets/websockets-all-you-need-to-know/snapshot5.png)
<figcaption class="caption">Request Headers</figcaption>

<div class="breaker"></div>

Instead of the usual 200 status code, a server with WebSocket protocol returns:

{% highlight raw %}
    **HTTP/1.1 101 Switching Protocols**
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
    Sec-WebSocket-Protocol: v10.stomp
{% endhighlight %}

![Markdown Image](../assets/websockets-all-you-need-to-know/snapshot4.png)
<figcaption class="caption">Response Headers</figcaption>

<div class="breaker"></div>

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;border-color:#ccc;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#f0f0f0;}
.tg .tg-1wig{font-weight:bold;text-align:left;vertical-align:top}
.tg .tg-buh4{background-color:#f9f9f9;text-align:left;vertical-align:top}
.tg .tg-0lax{text-align:left;vertical-align:top}
@media screen and (max-width: 767px) {.tg {width: auto !important;}.tg col {width: auto !important;}.tg-wrap {overflow-x: auto;-webkit-overflow-scrolling: touch;}}</style>
<div class="tg-wrap"><table class="tg">
  <tr>
    <th class="tg-1wig">HTML</th>
    <th class="tg-1wig">Websocket</th>
  </tr>
  <tr>
    <td class="tg-buh4">An application is modeled as many URLs</td>
    <td class="tg-buh4">There is usually just one URL for the initial connect</td>
  </tr>
  <tr>
    <td class="tg-0lax">Request-response style architecture</td>
    <td class="tg-0lax">Asynchronous, event-driven, messaging architecture</td>
  </tr>
  <tr>
    <td class="tg-buh4">There are prescribed semantics to the content of messages (request methods, request header fields, response status codes, and response header fields, along with the payload of messages and mechanisms for content negotiation)</td>
    <td class="tg-buh4">No prescribed semantics to the content of the message (That means you can not write message controllers and handlers unless client and server agree on the message semantics.) Typically client and server negotiate on a sub-protocol for message semantics eg. STOMP</td>
  </tr>
  <tr>
    <td class="tg-0lax">Being a uni-directional and stateless protocol, the client needs to initiate communication to the server by sending a request with predefined semantics; only then the server can communicate</td>
    <td class="tg-0lax">Bi-directional, full duplex over TCP. There are no pre-defined message patterns such as request/response. Either client or server can send a message to the other party.</td>
  </tr>
  <tr>
    <td class="tg-buh4">HTTP require more requests. Its a chatty protocol.<br>Ajax/XHR streaming for example relies on one long-running request for server-to-client messages and additional HTTP POST requests for client-to-server messages.</td>
    <td class="tg-buh4">The WebSocket needs only a single HTTP request to do the WebSocket handshake. All messages thereafter are exchanged on that socket.</td>
  </tr>
</table></div>
<figcaption class="caption">Comparison between HTTP and WebSocket Communications Protocol</figcaption>

<div class="breaker"></div>

### STOMP
The WebSocket protocol defines two types of messages, text and binary, but their content is undefined. STOMP defines a mechanism for client and server to negotiate a sub-protocol i.e. a higher level messaging protocol, to use on top of WebSocket to define what kind of messages each can send.

STOMP (simple, text-oriented messaging protocol) is a simple inter-operable protocol designed for asynchronous message passing between clients via mediating servers. It defines a text based wire-format for messages passed between these clients and servers. It is designed to address a minimal subset of commonly used messaging patterns and can be used over any reliable, 2-way streaming network protocol such as TCP and WebSocket.

STOMP is a frame based protocol whose frames are modeled on HTTP. The structure of a STOMP frame.

{% highlight raw %}
    COMMAND
    header1:value1
    header2:value2

    [Body]
{% endhighlight %}

> Stay tuned in for the next blog on SockJS where I give examples of various commands and how they are used

SEND or SUBSCRIBE commands are used to send or subscribe for messages along with a “destination” header that describes what the message is about and who should receive it.

#### Important Headers:
**subscription-id** – A server cannot send unsolicited messages. All messages from a server must be in response to a specific client subscription, and the *subscription-id* header of the server message must match the “id” header of the client subscription.

**destination** – It can be any string, and it’s entirely up to STOMP servers to define the semantics and the syntax of the destinations that they support. It is very common, however, for destinations to be path-like strings where */topic/..* implies publish-subscribe (one-to-many) and */queue/* implies point-to-point (one-to-one) message exchanges.

### When to use Websocket Protocol?
*Short Answer?* <br/>
Low latency, high frequency and high volume that make the best case for the use WebSocket.

#### What are my alternatives?
A combination of Ajax and HTTP streaming and/or long polling could provide a simple and effective solution, just like websocket. For example news, mail, and social feeds need to update dynamically but it may be perfectly okay to do so every few minutes. Collaboration, games, and financial apps on the other hand need to be much closer to real time.

#### What if Websockets are not supported by the client?
Over the public Internet, restrictive proxies outside your control may preclude WebSocket interactions either because they are not configured to pass on the Upgrade header or because they close long lived connections that appear idle.
The solution to this problem is WebSocket emulation, i.e. attempting to use WebSocket first and then falling back on HTTP-based techniques that emulate a WebSocket interaction and expose the same application-level API.

> This is where SockJS comes into play.

### Where do I go next?
Wherever! This is a free world, my friend! However, if you want to see what happens with SockJS, stay tuned.