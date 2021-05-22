%{
  title: "Absinthe GraphQL Elixir",
  author: "Luiz Cattani",
  tags: ~w(absinthe graphql elixir),
  description: "This is my notes about using Absinte graphql in Elixir"
}
---

# GRAPHQL
Graph Query Language / Data Query Language

GraphQL is a new API standard that provides a more efficient, powerful and flexible
alternative to REST.

It was developed by open-sourced by Facebook, and is now maintained by a large community
of companies and individual from all over the world.

As it's core, GraphQL enables declarative data fetching where a client can specify
exactly what data it needs from an API.

Instead of multiple endpoints that returns fixed data structures, a GraphQL server
only exposes a single endpoint and responds with precisely the data a client asked for.

GraphQL minimizes the amount of data that needs to be transferred over the network

Flexibility and efficiency

## Overfetching
- Downloading unnecessary data

## Underfetching
- An endpoint doesn't return enough of the right information
- Need to send multiple requests(N+1 request problem).

## Rapid Product Iterations
- REST structure endpoints according to client's data need
- No need to adjust API when product requirements and design change
- Faster feedback cycles and product iterations

## Insightful Analytics
- Fine-grained info about what data is read by clients
- Enables evolving API and deprecating unneeded API features
- Great opportunity for instrumenting and performance monitoring
- It's possible to gain a deep understanding of how the available data is being used, helping
to evolve the API and deprecating specific fields that are not requested by any client any more
- Resolver functions collect the data that's requested by a client, instrumenting and measuring performance on these provides crucial insights about bottlenecks in your system

## Schema & Types
- GraphQL uses strong type system to define capabilities of an API
- All the types that are exposed in an API are written down in a schema using the GraphQL Schema Definition Language(SDL)
- The Schema serves as contract between client and server
- Frontend and Backend teams can work completely independent from each other

## Advantages to use GraphQL
1. GraphQL API's have a strongly typed schema
2. No more overfetching and underfetching
3. GraphQL enables rapid product development
4. Composing GraphQL API's
5. Rich open-source ecosystem and a amazing community behind the project

Companies using GraphQL
- Coursera
- Twitter
- Shopify
- Yelp
- Meteor
- Pinterest
- The New York Times
- Github

## The Schema Definition Language - SDL
Definie simple types

```graphql
type Person {

}
```

## Defining the Schema
```elixir
# lib/community_web/schema.ex
defmodule CommunityWeb.Schema do
  use Absinthe.Schema

  alias CommunityWeb.NewsResolver

  object :link do
    field :id, non_null(:id)
    field :url, non_null(:string)
    field :description, non_null(:string)
  end

  query do
    @desc "Get all links"
    field :all_links, non_null(list_of(non_null(:link)))
  end
end
```

## Query Resolver
The query is now defined, but the server still doesn’t know how to handle it. To do that you will now write your first resolver. Resolvers are just functions mapped to GraphQL fields, with their actual behavior. You specify the field for a resolver by using the resolve macro and passing it a function:

```elixir
# lib/community_web/schema.ex
query do
  @desc "Get all links"
  field :all_links, non_null(list_of(non_null(:link))) do
    resolve(&NewsResolver.all_links/3)
  end
end

# /lib/community_web/resolvers/news_resolver.ex
defmodule CommunityWeb.NewsResolver do
  alias Community.News

  def all_links(_root, _args, _info) do
    {:ok, News.list_links()}
  end
end
```

# Testing with Playground
```elixir
# lib/community_web/router.ex

defmodule CommunityWeb.Router do
  use CommunityWeb, :router

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/" do
    pipe_through :api

    forward "/graphiql", Absinthe.Plug.GraphiQL,
      schema: CommunityWeb.Schema,
      interface: :simple,
      context: %{pubsub: CommunityWeb.Endpoint}
  end

end
```

# Start the server
```bash
$ iex -S mix phx.server
```

Access `localhost:4000/graphiql`

# Using Query
```graphql
{
  allLinks {
    id
    url
    description
  }
}
```

# Mutations

```elixir
# lib/community_web/schema.ex
mutation do
  @desc "Create a new link"
  field :create_link, :link do
    arg :url, non_null(:string)
    arg :description, non_null(:string)

    resolve &NewsResolver.create_link/3
  end
end
```

# Resolver with arguments
```elixir
# lib/community_web/resolvers/news_resolver.ex
def create_link(_root, args, _info) do
  # TODO: add detailed error message handling later
  case News.create_link(args) do
    {:ok, link} ->
      {:ok, link}
    _error ->
      {:error, "could not create link"}
  end
end
```
# Using Mutation
```graphql
mutation {
  createLink(
    url: "http://npmjs.com/package/graphql-tools",
    description: "Best Tools!",
  ) {
    id
    url
    description
  }
}
```
# self documenting
# Documentation as code
# Built in versioning
# Strong Typed
# Elixir Absinthe
Elixir provides a abstraction to implement the GraphQL protocol.


# DataLoader
Batched resolutions via DataLoader (N+1 problem)
Multiple databases

# Safe Deprecations fields

# GraphiQL
Plyaground to ensure querys on GraphQL schemas


# Context for authorization


# References Links
Sam Davies - A shot of Absinthe: from zero to GraphQL in 40 minutes - Code Elixir LDN 2018
https://www.youtube.com/watch?v=IJJQvJRoXy8

Bruce Williams - A GraphQL-on-Elixir Primer - Code Beam SF 2018
https://www.youtube.com/watch?v=enbksvAko98

# OPTIONS IMPLEMENTATiONS

1. Dream World
Graphql | Database

2. GraphQL Aggregation
Graphql API | rest API | rest API | etc

3. Abstract Data Access
rest API | Graphql(directly) | Database


# Query
Requests information

# Mutation
Modifies data, requests information in response

# Subscription
Subscribes to an event
Requests information be sent later(over WS, etc)
Live Data (using websockets)
Scalable live data with elixir because in top of OTP, clusters, we have Phoenix PubSub / CRT's
I can use to publish a event to Kafka or RabbitMQ
Can do file upload over javascript built

# Websockets
Using channels from Phoenix

# Schemas
objects Entities - (user / Posts / Comments)
Fields - Associations or transitions that exist between those objects types ( user: {name, id, email}, post: {id, title, text})

# Root objects types are entry point for GraphQL operations
User -> Post
User is the query root

# Instrospection
When we create a schema we can document every piece of schema
We can document every single type
We can document every single field
We can get all this information using a normal GraphQL query that we send to the server
Has some special semantics, we need to use double underscore fields names
are built in fields that we get for free and we can Instrospect the schema


# Standard GraphQL
Parsing -> Validation -> Execution

## Parsing
We always need parsing something

## Validation
Check arguments and fields
if you don't pass that nothing gets executed

## Execution
Does the work

Absinthe brokes the three standards in 40+ small phases


Build a GraphQL API in Elixir with Phoenix and Absinthe, part 1: Overview

https://www.youtube.com/watch?v=uoCFQu9gQHE&list=PLw7bfDlTRWbgiApK7X1bRKJJ03xoDU3hm
