---
title: "Developing a Personal Project, Part 3: How to Build a Discord Bot Using Nostrum"
category: elixir
---

*In the first three entries in this series I waxed poetic on my approach to personal coding projects. In this entry I'll get down to business describing how to set up a Discord bot using Nostrum.*

Before starting, you'll need to have a Discord account and the latest version of Elixir. I would also recommend starting a new Discord server/guild to test your bot in with out risk of bothering your users.

**Registering a new Discord Application**

Step 1: Log into the [Discord Developer Portal](https://discord.com/developers/) and register a new application.

Step 2: Under Oath2, generate an invitation URL for your application using the URL Generator. Select "bot" and "applications.commands" from the top list. This will bring up another list of permissions that your bot user can have. If you know exactly which permissions your bot needs, you can specify them now. Personally, I find it useful to give the bot Administrator permissions so that you can quickly move into development and worry about config best practices later.
Step 3: Open the generated URL in your browser. This will bring up a window that will let you select which server you'd like to add it to. Add it to your test server.

Step 4: In the "Bot" tab in the Discord Developer Portal, copy the token near the top of the page.

Step 5: Store your token as an environment variable. I use Oh My Zsh on Mac, so in my case I would open my ~.zshrc file with nvim, and add a line to the file like so:
```
DISCORD_BOT_TOKEN: MTExMzUxMDk5MjgzMjM3Mjc0Ng.999999.mtkJ7zfEHTZoD5ne5xS_999999_SZvtUYZxxDg
```

Aftwards, entering `source ~.zshrc` will ensure the environment variable is loaded. You can check by just typing `env` to list all of your set variables.

With that, we're basically done with the Discord side of things, for a while anyways.

**Bootstrapping the Project**

I recommend starting a fresh Phoenix app for this because it gives us a directory and config structure that will be useful for getting up and running quickly. Nostrum implements a GenStage architecture (more on that later), which requires a Supervision Tree. If you're comfortable starting a vanilla Elixir project and Supervision tree yourself, by all means go for it.

Step 1: Open your IDE of choice and a terminal, and enter `mix phx.new my_app --no-html --no-assets`. Starting the project this way will omit the HTML and Javascript bits, giving us purely an API for our bot user. If you plan to have some sort of UI for your app, just use `mix phx.new my_app`.

Step 2: In `mix.exs` add Nostrum to your dependecies list:
```
def deps do
  [{:nostrum, git: "https://github.com/Kraigie/nostrum.git"}]
end
```
Step 3: In `config/runtime.exs` add the following:
```
config :nostrum,
    token: System.get_env("HANEKAWA_BOT_TOKEN"),
    gateway_intents: :all
```
This will allow Nostrum to pick up your bots API key at runtime. By default Nostrum uses the Gateway API, which requires you to define which intents (i.e. which payloads you want to receive from Discord). As with permissions, it can be useful to set this to all for now, but it is certainly recommended to only receive those intents that you need.

**Implementing Nostrum**

Now that we've spun up our project and registered a new application, we're ready to implement the last pieces.
 
Step 1: Make a new file `lib/my_app/consumer_supervisor.ex`. Implement the `start_link/1` and `init/1` callbacks, like so:

```
defmodule MyApp.ConsumerSupervisor do
  use Supervisor

  def start_link(args) do
    Supervisor.start_link(__MODULE__, args, name: __MODULE__)
  end

  @impl true
  def init(_args) do
    children =
      for n <- 1..System.schedulers_online(),
          do: Supervisor.child_spec({MyApp.Consumer, []}, id: {:my_app, :consumer, n})

    Supervisor.init(children, strategy: :one_for_one)
  end
end
```

Step 2: Make a new file `lib/my_app/consumer.ex`, and implement the `start_link/1` callback:
```
defmodule Hanekawa.Consumer do
  @moduledoc """
    This module defines the consumer agent that acts as a gateway event handler for
    events passed from the Discord WebSocket connection.
  """
  use Nostrum.Consumer

  alias Nostrum.Api

  def start_link do
    Consumer.start_link(__MODULE__)
  end
end
```

Step 3: Implement event handlers. When an event comes through from Discord, Nostrum will attempt to match against the intent of the event (e.g. :MESSAGE_CREATE). We need a default handler to prevent the consumer from crashing when it receives an event that we haven't explicitly told it how to handle.

```
 def handle_event({:MESSAGE_CREATE, msg, _ws_state}) do
    # IO.inspect(msg)
    case msg.content do
      "!ping" ->
        Api.create_message(msg.channel_id, "pong!")
      _ ->
      	:ignore
    end
 end

 def handle_event(_event) do
    :noop
  end
```

Step 4: Within `lib/my_app/application.exs` add your Consumer Supervisor to your start function. This will start our implementation of the Nostrum ConsumerSupervisor within our Supervision tree.

```
  @impl true
  def start(_type, _args) do
    children = [
      # Start the Telemetry supervisor
      MyAppWeb.Telemetry,
      # Start the Ecto repository
      MyApp.Repo,
      # Start the PubSub system
      {Phoenix.PubSub, name: MyApp.PubSub},
      # Start Finch
      {Finch, name: MyApp.Finch},
      # Start the Endpoint (http/https)
      MyAppWeb.Endpoint,
      MyApp.ConsumerSupervisor
      # Start a worker by calling: MyApp.Worker.start_link(arg)
      # {MyApp.Worker, arg}
    ]

    # See https://hexdocs.pm/elixir/Supervisor.html
    # for other strategies and supported options
    opts = [strategy: :one_for_one, name: MyApp.Supervisor]
    Supervisor.start_link(children, opts)
  end
  ```

Step 5: Test your application. Start it up with `iex -S mix phx.server`, and send your test message "!ping" from Discord. If configured properly, you should see the response "!pong" returned by the bot user to the channel.


That's the bare necessities, as they are. There's much more to both Nostrum and the Discord API, but this is all you should need to get up and running. If you found you had any trouble with these instructions, or you think I made a mistake, by all means drop me a DM on [Twitter](https://twitter.com/StefanSahagian).