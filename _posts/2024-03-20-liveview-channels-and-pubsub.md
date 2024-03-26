---
title: "LiveView, Channels, and PubSub"
tags: elixir
---

I was recently experimenting with Phoenix Channels and Liveview to learn a bit more about Phoenix. I'm working on another social platform project right now, so I went for a chat app. I noticed that most tutorials used either Phoenix Channels, the official Javascript client and a JS frontend, or Liveview on its own. I wanted to see if I could combine LiveView with Channels. Here's what I learned. Warning: this isn't really a tutorial, more of a rambling recapitulation of what's in the docs, except pointing out what I didn't get the first time around.

TL;DR/Oversimplification: [Channels](https://hexdocs.pm/phoenix/channels.html) let things outside your Phoenix app connect to your PubSub system via websocket. One Channel process per client, a Channel process for every client. A [LiveView](https://hexdocs.pm/phoenix_live_view/welcome.html) is an Elixir process as well that maintains a websocket connection and sends server-rendered html to a browser client. LiveViews can also communicate with Channels via [PubSub](https://hexdocs.pm/phoenix_pubsub/2.1.3/Phoenix.PubSub.html). These processes communicate via broadcast and then render (or don't) the message data to their respective clients.

Right off the bat, this is not a normal way of doing things. Phoenix Channels and Liveview are two great tools that were built for two adjacent but different things and you wouldn't necessarily want to use them side by side like this. A Phoenix Channel is a high-level abstraction of the PubSub (Publisher/Subscriber) system paired with a websocket connection that allows clients to connect with your Phoenix application. LiveView is a framework for delivering real-time, server-rendered HTML to browser clients via websocket connection. Under the hood, a LiveView is an Elixir process with some additional callbacks. A Channel is an Elixir process with additional callbacks. Note that when I say process, I really mean [GenServer](https://hexdocs.pm/elixir/1.12/GenServer.html), but I find the term can make it hard to explain things to people who are new to Elixir. Channels use websockets. LiveViews use websockets. Channels have clients. LiveViews have clients. Though aimed at different use cases, these two concepts are interesting and not wholly imcompatible, and that's reason enough to experiment with them.

One thing to note is that when a client connects to a Channel it usually must authenticate via some logic in the Channel's join/3 callback. I find the verb "join" somewhat misleading, particularly in the chat application example scenario. The client is "join"-ing the conversation of the chosen PubSub topic, but there is only one Channel process per client. Multiple clients are **not** connecting to a single Channel. In contrast, The LiveView process, being native to the application, does not need to authenticate to "join" the chat, it simply subscribes to the same PubSub topic. To be more specific, it is to be assumed that if a user is able to view the LiveView page then it follows that their session would have already been authenticated in some way. Even if your application had the concept of a private or channel that required additional authentication to view from a LiveView, that authentication would be done via the Plug pipeline and not when the LiveView renders. This may seem obvious, but I found it a little confusing at first.

The key for my understarnding was getting to better know the PubSub system and how a Channel interacts with it. To repeat, a Channel is a process that acts as a link between a client and the rest of an Elixir application. A client could be a JavaScript application running in a browser, a desktop application, an IoT device, etc. It's any program that is not part of the Elixir application itself. When a client connects (which it will do so through some kind of client library), it will first hit the application's Endpoint (yet another GenServer) which will then route the client to the requested Channel. A new Channel process is then spawned for that client. One channel process per client. The Channel process will maintain any state regarding the connection and subscribe to the PubSub topic initially requested by the client. The Channel process will then monitor for any messages that are broadcast on that topic. Depending on how the event handlers are implemented, the Channel will determine what to do with each message it receives, and can push them back down to the client. When a Channel process receives a message from the client, it can then broadcast that data through the PubSub system to subscribers of that topic, whether they be other Channel processes, LiveView processes, or any other process in the application.

Liveview on the other hand is relatively simple. It renders an initial html response when the page loads, then upgrades to a stateful view (via websocket) when the load is complete. When socket assigns (i.e. variables) or the stateful view changes the view re-renders those parts that change. Event handlers implemented in the serverside determine the specific behavior of the stateful view. It can also do all the things that any nother Elixir process can do, meaning send and receive messages. Via PubSub subscription the LiveView can listen for events broadcast on a chosen topic and then handle those events as you see fit, typically by updating socket assigns and triggering a re-paint of the affected area of the DOM. Unhandled events will be ignored and dropped through the default GenServer implementations that LiveView (and Channels) are built on. 

Something that I kept tripping over is the fact that every LiveView and Channel is on equal footing. When you broadcast an event to the PubSub you are not trying to send an event to the Channel or the LiveView to then re-broadcast it. You are simply telling the PubSub "hey, tell all subscribers of this topic that this event happened." If you don't want the sending process to get its own message back, you can use `broadcast_from/5`.

In this one, I mostly learned how *not* to use PubSub. Shoutout to these DWYL tutorials ([Channels tutorial](https://github.com/dwyl/phoenix-chat-example), [LiveView tutorial](https://github.com/dwyl/phoenix-liveview-chat-example)) that I Frankenstein-ed together for this one. If you're wondering why I didn't just follow these two more closely, I find that I learn best from taking the unhappy path. You can [check out the code here](https://github.com/Shaka-n/demo-chat-app), it's a mess and I'm not sorry. Tune in next time, whenever that is, as I add in authentication, room navigation, and potentially a non-browser client.