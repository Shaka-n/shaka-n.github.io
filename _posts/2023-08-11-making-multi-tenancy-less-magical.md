---
title: "Making Multi-Tenancy Less Magical"
category: elixir
---
I was tasked by a friend to help implement multi-tenancy in his WISP platform in a way that is still secure, but more conducive to testing and generally more explicit.

Multi-tenancy is an architecture pattern where a single instance of an application serves multiple tenants/entities while keeping their respective data isolated from each other. Commonly used in SaaS platforms, the pattern helps keep costs low by allowing users to share resources and while simplifying updates and maintenance. In Elixir, there are two common approaches to multi-tenancy: database prefixing and foreign key associations. 

The crux of the [query prefixing](https://hexdocs.pm/ecto/multi-tenancy-with-query-prefixes.html) technique relies on adding a new database prefix for every tenant, which keeps their data completely siloed in separate Postgres schemas. In an Elixir app, you set up your Ecto config like so to change the default database prefix:

{% highlight elixir %}
query_args = ["SET search_path TO connection_prefix", []]

config :my_app, MyApp.Repo,
  username: "postgres",
  password: "postgres",
  database: "demo_dev",
  hostname: "localhost",
  pool_size: 10,
  after_connect: {Postgrex, :query!, query_args}
{% endhighlight %}

You'll then have to explicitly create a new prefix on your database, as Ecto won't do it for you.

Querying across prefixes can be done by passing the :prefix keyword at the query, struct, or join level:

{% highlight elixir %}
MyApp.Repo.all(Sample, prefix: "public")
...
from p in Post, prefix: "foo",
  join: c in Comment, prefix: "bar"
{% endhighlight %}

The drawbacks to this approach are that you need to add a new prefix for every tenant, and then must run database migrations for each prefix individually when making changes. This can get very costly and slow at scale, not to mention the headache it poses for just writing your average Ecto join. Further, this does require a bit more Postgres knowledge to work with, which imposes its own kind of overhead. There are hex packages that abstract the funky parts of this away, but the scaling costs remain the same. Not to mention, adding packages breaks with the spirit of my original mission of making things less magical.

By contrast, enforcing [multi-tenancy via foreign key](https://hexdocs.pm/ecto/multi-tenancy-with-foreign-keys.html) should appear much more familiar to the average dev. Let's say schemas are restricted by `org-id` such that only users from that organization can access its data. Just add this to all of your queries for tenanted schemas (i.e. data that should be siloed per user/client/tenant), and you're good to go:

{% highlight elixir %}
...
where(query, org_id: ^org_id)
...
{% endhighlight %}

To make this more reliable and less wildly insecure, a common approach is to leverage the callback `prepare_query/3` from Ecto.Query. Adding this callback to our `repo.ex` will ensure that it is run before any other Ecto Query, enforcing our requirement that a tenanted schema can only be accessed by a tenant with the proper org id. 

{% highlight elixir %}
defmodule MyApp.Repo do
  ...
  @impl true
  def prepare_query(_operation, %{from: %{source: {_, schema}}}=query, opts) do
    excepted? = Enum.member?(@excepted_schemas, schema)
    cond do
      opts[:skip_org_id] || opts[:schema_migration] ->
        {query, opts}

      org_id = opts[:org_id] ->
        {Ecto.Query.where(query, org_id: ^org_id), opts}

      true ->
        raise "expected org_id or skip_org_id to be set"
    end
  end
end
{% endhighlight %}

You'll still have to add `org_id` to every query though. Let's make that easier by setting `org_id` by default. First, add these to `repo.ex`:

{% highlight elixir %}
defmodule MyApp.Repo do
  ...

  @tenant_key {__MODULE__, :org_id}

  def put_org_id(org_id) do
    Process.put(@tenant_key, org_id)
  end

  def get_org_id() do
    Process.get(@tenant_key)
  end
{% endhighlight %}

These will give every calling process an easy way to keep the org_id on hand. To set the org_id as a default option in all repo operations, we implement the `default_options/1` callback like so:

{% highlight elixir %}
defmodule MyApp.Repo do
  ...

  @impl true
  def default_options(_operation) do
    [org_id: get_org_id()]
  end
end
{% endhighlight %}


Now `org_id` will always be set by default on every repo operation and we don't have to worry about specifying that we don't want to pollute our data store. But how do we set the `org_id`? Well, it can vary, but in my case, I opted to add it to the Plug pipeline:

{% highlight elixir %}
def call(%Plug.Conn{host: host} = conn, %{root_host: root_host} = _opts) do
    case extract_subdomain(host, root_host) do
      subdomain when byte_size(subdomain) > 0 ->
        Repo.find_and_put_org_id(subdomain)
        org_id = Repo.find_org_id(subdomain)
        conn
        |> fetch_session()
        |> put_session(:org_id, org_id)

      _ ->
        conn
    end
  end
{% endhighlight %}


Plugs are a powerful part of the Phoenix framework that I won't get into. Suffice to say, this will run when a user loads the page on their custom subdomain, and that subdomain will be used to retrieve their org id and set it in the process dictionary of our Repo module. They'll still need to authenticate as normal, so there's no opportunity for somebody to go to the wrong domain and login with the wrong org_id.

There is one last wrinkle however: what do you do when a schema doesn't have org_id and can't be joined to one? What if you have a schema in your ORM that is relatively remote from org_id? Are you forced to chain a long series of joins together every time? In my case there were two outliers like this: cron jobs and session tokens. 

For the cron jobs (we used the Oban package), it doesn’t make sense for a daily system level job to be associated with a specific tenant. So, recalling the earlier prefix technique, I chose to give Oban its own database prefix. This is pretty simple to do in config, and there's not much else to change but add an exception to `prepare_query/3`. Now the queries for Oban jobs skip right by the check and continue as normal.

For the session tokens, I opted for an explicit exceptions list. I'd rather not mess around with the authentication code, and since it didn't rely on my multi-tenancy solution to begin with, excepting it won't introduce a new attack surface. By and large I want all schemas to be tenanted in some way within this app, so having a short list of the exceptions makes some sense, as I'm unlikely to add to it frequently. If I do, all I have to do is alias the schema and add it to the module attribute that defines the list of exceptions. 

{% highlight elixir %}
defmodule MyApp.Repo do
  ...
 @excepted_schemas [UserToken]

  @impl true
  def prepare_query(_operation, %{from: %{source: {_, schema}}}=query, opts) do
    excepted? = Enum.member?(@excepted_schemas, schema)
    cond do
      opts[:skip_org_id] || opts[:schema_migration] || opts[:prefix] == "oban" || excepted? ->
        {query, opts}

      org_id = opts[:org_id] ->
        {Ecto.Query.where(query, org_id: ^org_id), opts}

      true ->
        raise "expected org_id or skip_org_id to be set"
    end
  end
 2nd
{% endhighlight %}


The combinations of approaches here are more explicit and more custom-tailored to the specific problem. This should make writing and maintaining tests much easier. While being perhaps somewhat less elegant, I think this will lend itself to simpler maintenance down the line as this project grows.

