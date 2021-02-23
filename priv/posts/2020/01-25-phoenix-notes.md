%{
  title: "Phoenix Framework Elixir",
  author: "Luiz Cattani",
  tags: ~w(phoenix elixir),
  description: "Basic concepts using Phoenix Framework in Elixir"
}
---

# Install Elixir and Phoenix
```bash
$ brew install elixir
$ elixir --version

$ brew install nvm
$ nvm install v14.4

$ mix local.hex
$ mix archive.install hex phx_new 1.5.7

$ brew install postgresql
```

# create simple application
```bash
$ mix phx.new blog
$ cd blog
$ mix ecto.create
$ mix phx.server
```
# Plug

# Environments
we can find the config files of our application into the config folder.

Phoenix supports a master configuration file "config.exs" plus an additional file for each environment.
this file contains application wide configuration concerns, logging and endpoint, and a file that has secret passwords, we need keep out of version control.

The environments supported by default is development: "dev.exs", test: "test.exs", and production: "prod.exs", but we can add any other custom config file.

The file "prod.secret.exs" is usually populated by deployment tasks,

We can switch of environments via MIX_ENV environment variable

```elixir
# change the environment

iex> MIX_ENV=:dev
iex> MIX_ENV=:test
iex> MIX_ENV=:prod

# verify the environment, returns a boolean

iex> Mix.env(:dev)
iex> Mix.env(:test)
iex> Mix.env(:prod)
```

# Endpoint
The first contact of external world wit our application

Are the chain of functions at the beginning of each request.

Chain of functions, or plugs, does typical things that almost all productions web servers need to do.
Deal with static content, log requests, parse parameters, etc..

# Mix.exs and Mix.lock files
Each Elixir project has a configuration File, "mix.exs", containing basic information about the Project
that supports tasks like compiling files, starting the server, and managing dependencies.

When we add dependencies in our project, we'll need to make sure they show up in the "mix.exs" file.

After we compile the project, "mix.lock" file will include the specific versions of the libraries we depend on,
so we guarantee that our production machines use exactly the same versions we used during development
and in our build servers

# The Router Flow
Routers - Distribute requests to application

To perform a common set of tasks, or transformations for some logical group of functions, we can use
"plug" for each transformation step, and group these plugs into "pipelines"

## Pipeline
is just a bigger plug that takes the "conn" struct and returns on too.

Example in the router file, we have two pipelines, browser and api.
For each type of request we can perform a set of tasks in each pipeline before go to the controller.

```elixir
connection
  |> endpoint
  |> router
  |> pipeline
  |> controller
```

- The endpoint has functions that happens for every requests
- The connections goes through a named pipeline, which has common functions for each major type of request
- The controller invoke the model and renders a template through a view


# Controllers
Call services and set up intermediate data for views

Controller pulls the strings for the application, making data available in the connection
for consumption by the view.

It potentially fetches database data to stash in the connection and then redirects or renders
a view.

The view substitutes values for a template.

```elixir
defmodule UserController do
  def create(conn, %{"room" => room_params}) do
    %User{}
    |> User.changeset(room_params)
    |> Repo.insert()
    |> case do
      {:ok, room} ->
        conn
        |> put_flash(:info, "Room created successfully.")
        |> redirect(to: room_path(conn, :index))

      {:error, %Ecto.Changeset{} = changeset} ->
        render(conn, "new.html", changeset: changeset)
    end
  end
end
```

# Pattern Matching
```elixir
defmodule Place do
  def city(%{city: city}), do: city
  def texas?(%{state: "TX"}), do: true
  def texas?(_), do: false
  def has_state?(%{state: _}), do: true
  [%{city: "São Paulo", state: "SP"}]
end
```

# View and Templates

# Brackets
```elixir
# Brackets with output in the view
<%= %>

# Brackets with to execute some condition, without output in the view
<% %>
```

# Ecto

Ecto is a wrapper to relational database, allowing developers to read and persist data to underlying
storage such as Postgresql.

Similiar to Active Record from Rails, but with some differences.

Incapsulated query language that we can use to build layered queries.

## Changeset
Changeset is a feature on Ecto, that holds all changes we want to perform on database,
Encapsulates the whole process of receiving external data, casting and validating before writing it to
the database.
