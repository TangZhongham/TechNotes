# 80% Damus and Nostr Explained

> zap 对比
> 是否会有一种方法，对这些个协议进行数学逻辑推理论证？从而达到收敛的目的？
> Schnorr sigs 基本原理

TODO

基本差不多了，把剩下几篇再总结，然后整理一下就可以产出了。

## bulletpoints

- [ ] Why Nostr and Damus
- [ ] What is damus and Nostr
- [ ] How does it work?
- [ ] What should we expect from it? 优缺点
- [ ] Refs and continue to read...

## Why

首先我们应该了解，Nostr 和 Damus 被设计出来的目的是啥，官方给出的定义是：

Nostr is the simplest open protocol that is able to create a censorship-resistant global "social" network once and for all.

他的定语是 global "social" network，Nostr是为了打造一个去中心化的社交媒体而开发的开源协议，一个去中心化的 Twitter。而Damus 是官方写的ios 客户端。

主要由以下两方面实现：

1. 不依赖中心服务器 central server，hence resilient
2. based on cryptographic keys and signatures, hence tamperproof 不会被篡改。

基本实现：

Everybody runs a client. It can be a native client, a web client, etc. To publish something, you write a post, sign it with your key and send it to multiple relays (servers hosted by someone else, or yourself). To get updates from other people, you ask multiple relays if they know anything about these other people. Anyone can run a relay. A relay is very simple and dumb. It does nothing besides accepting posts from some people and forwarding to others. Relays don't have to be trusted. Signatures are verified on the client side.

每个人都在运行一个客户端。它可以是一个本地客户端，一个网络客户端，等等。要发布一些东西，你写一个帖子，用你的密钥签名，然后把它发送到多个中继站（由其他人或你自己托管的服务器）。要获得其他人的更新，你要问多个中继站是否知道这些其他人的情况。任何人都可以运行一个中继站。一个中继站是非常简单和愚蠢的。它除了接受一些人的帖子并转发给其他人外，什么也不做。中转站不需要被信任。签名是在客户端验证的。

技术角度的实现步骤：

There are two components: clients and relays. Each user runs a client. Anyone can run a relay.

Every user is identified by a public key. Every post is signed. Every client validates these signatures.

Clients fetch data from relays of their choice and publish data to other relays of their choice. A relay doesn't talk to another relay, only directly to users.

For example, to "follow" someone a user just instructs their client to query the relays it knows for posts from that public key.

On startup, a client queries data from all relays it knows for all users it follows (for example, all updates from the last day), then displays that data to the user chronologically.

A "post" can contain any kind of structured data, but the most used ones are going to find their way into the standard so all clients and relays can handle them seamlessly.


## How does it work? 技术解释

本质上利用了密码学原理。你在客户端通过密码学的原理，生成一对公🔑和私钥。

🏷️ A public key: Which will act as your username. This key can be shared and will be public to everyone.
🔑 A private key: This key is like your password. You need to keep it secret, hence the name. Also, this key will grant you access to your account in any platform that is powered by nostr.

公🔑等于你的身份证，私钥等于你的指纹。好玩的在于，任何经过私钥加密的消息，都可以通过公🔑验证，是不是该私钥产生的。所以可以理解这个私🔑就是你的指纹，不能被人复制了去。

然后客户端初始会带上几个boost relay 信息，向这些relay 要一些 posts 过来，发送一些posts出去。每个消息体都是用私🔑加密的Event(Schnorr sigs)，客户端使用公🔑解密确认消息的真实无篡改性。每个人都可以建立自己的relay。这些relay 只做分发，不对消息进行处理，也因为密码学无法对消息进行篡改。

传输层使用 websocket + json 的消息格式

所有消息都叫EVENT

```json
{
  "id": <32-bytes sha256 of the the serialized event data>
  "pubkey": <32-bytes hex-encoded public key of the event creator>,
  "created_at": <unix timestamp in seconds>,
  "kind": <integer>,
  "tags": [
    ["e", <32-bytes hex of the id of another event>, <recommended relay URL>],
    ["p", <32-bytes hex of the key>, <recommended relay URL>],
    ... // other kinds of tags may be included later
  ]
  "content": <arbitrary string>,
  "sig": <64-bytes signature of the sha256 hash of the serialized event data, which is the same as the "id" field>,
}
```

Events are always signed (using Schnorr sigs) and they contain structured data that can have semantic meanings. A Schnorr type XOnlyPubkeys as defined in BIP340 (and currently in use with Bitcoin Taproot) are used as "identities" throughout the protocol.

A nostr-client is an APP that can talk with a nostr-relay and can subscribe to any set of events using a Subscription Filter. The filter represents all the set of nostr Events that a client is interested in.

There is no sign-up or account creation for a client. Clients are identified with their pubkeys. Every time a client connects to a relay, it submits its subscription filters and the relay streams the "interested events" to the client as long as they are connected.

A relay can cache the client subscriptions but it doesn't have to. Clients are supposed to handle everything at the "client-side", and relays can be dumb as a rock.

Clients don't talk to each other. But relays can. This allows relays to fetch data for a client that it doesn't have. And Clients get to subscribe to events outside of their connected relays.

This at first glance gives the image of the uselessness of Nostr as a protocol, (why not just sign and dump raw JSON and let clients figure it out?), but on a deeper look, the "dumb-server, smart client" model can be found to have some massive engineering benefits, especially in decentralized protocol designs.

This document is an outline of how these dumb servers, smart clients, and the Bitcoin network, e2e encryption can come together to solve the problem of "Decentralized Social Networks", DSNs (a buzzword I just came up with).


## 优缺点

cons

有人说私钥一直签名会降低自己的安全性

足够简单，等待更多新功能被加上（有时候闷头开发出来，等到被破解则too big too fall）成本太大

暂时没有 incentive 机制去鼓励大家建立relay （目前打算引入bitcoin）

spam，follow，like 机制模糊

pros

足够简单而且work，其余的想象力由社区驱动


## 展望未来

Nostr, Bitcoin, and Encrypted Subscription sharing, what we have now is a very powerful and default private social network that can share data among participants using some very generic and global protocols.

This document is an outline of how these dumb servers, smart clients, and the Bitcoin network, e2e encryption can come together to solve the problem of "Decentralized Social Networks", DSNs (a buzzword I just came up with).

The examples of Gab and Mastodon clearly show, that just having the code open source is not enough. The building process and designing of the standards have to be open also.

What if instead of building the perfect social media, we just build the most basic lego it needs to create such things and let developer consensus emerge in open over this basic standard unit of the puzzle.

Specify the smallest unit of a social data format (an Event), and let agreement emerge naturally among devs to build on this. This defines the core of the protocol. The bare minimum backbone of things that everyone needs to agree on, to be part of this network.

Nostr defines these protocol rules as NIPs. And mentions a set of mandatory NIPs. Rules that need to be implemented to talk the Nostr protocol.

On top of these mandatory NIPs, optional NIPs can be defined by anyone. Relays are free to choose their set of supported NIPs.

The Event data can be extended in the tag field by having more tag items defined by future NIPs.

An Events can be thought of as a generic data store. There is no restriction on the content that can be put inside it.

As odd as it might seem, such a simple protocol is getting more dev attention than many of the "well designed" existing alternate social media.

## Ref

https://github.com/nostr-protocol/nostr

https://github.com/aljazceru/awesome-nostr

https://nostr.guide/

https://github.com/rajarshimaitra/rust-nostr/blob/main/VISION.md

https://usenostr.org/