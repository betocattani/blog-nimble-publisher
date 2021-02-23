%{
  title: "Ecto Elixir",
  author: "Luiz Cattani",
  tags: ~w(ecto elixir database),
  description: "This is my notes about Ecto the persistence Frameworks for Elixir"
}
---

# ECTO

Database wrapper and query generator for Elixir.
Ecto Provides a standarised API and a set of abstractions for talking to all different kinds of database.

Ecto is split in 4 main components

# Ecto.Repo
# Ecto.Schema
# Ecto.Query
# Ecto.Changeset

# Creating a new Elixir project with supervisor
```bash
$ mix new MyApp --sup
```

## Config Ecto in a Elixir Mix Project

### Add dependencies
ecto_sql - Common query API
postgrex - Postgresql driver to elixir talk with database

```elixir
# /mix.ex

defp deps do
  [
    {:postgrex, "~> 0.15.8"},
    {:ecto_sql, "~> 3.5"}
  ]
end
```

### Install dependencies
```bash
$ mix deps.get
```

### Creating a Repository Pattern to the application

Configuration file to  connect with the database
```bash
$ mix ecto.gen.repo -r MyApp.Repo
```
### Generate configuration file
This configures how Ecto will connect with the database
```elixir
# config/config.exs
config :my_app, MyApp.Repo,
  database: "my_app_repo",
  username: "postgres",
  passowrd: "postgres",
  hostname: "localhost"
```

The Friends.Repo module is defined in `lib/friends/repo.ex` by our `mix.ecto.gen.repo` command.

This module is what we'll be using to query our database, It use the `Ecto.Repo` module, and
the `otp_app` tells Ecto which Elixir application it can look for database configuration, in this case `:my_app` application where Ecto find that configuration and so Ecto will use the configuration that was setup in
`config/config.exs` and finally we configure the database `:adapter` to Postgres.
```elixir
defmodule MyApp.Repo do
  user Ecto.Repo,
    otp_app: :my_app,
    adapter: Ecto.Adapters.Postgres
end
```

### Setup the MyApp.Repo as a Supervisor within the application's Supervisor Tree.

This piece of configuration will start the Ecto process which receives and executes our application's queries.
Without it we wouldn't be able to query the database at all.
```Elixir
# lib/my_app/application.ex

def start(_type, _args) do
  children = [
    Friends.Repo
  ]

  opts = [strategy: :one_for_one, name: Friends.Supervisor]
  Supervisor.start_link(children, opts)
end
```

### Config the Ecto repository
Tell to the application what repo to look
```elixir
# config/config.exs

config :my_app, ecto_repos: [MyApp.Repo]
```

### Create the database
```bash
$ mix ecto.create
=> The database for Friends.Repo has been created.
```

### Generate a new migration
```bash
$ mix ecto.gen.migration create_people
```

### Create data to populate the database
```elixir
people = [
  %Friends.Person{first_name: "luiz", last_name: "cattani", age: 34 },
  %Friends.Person{first_name: "jose" , last_name: "valim", age: 38 },
  %Friends.Person{first_name: "Chris", last_name: "Mccord", age:  31 }
]
```

### Insert a new register using changeset and insert method from Ecto
```elixir
# Create the struct to a Person
person = %Friends.Person{first_name: "Chris", last_name: "Mccord", age:  31 }

# create a changeset to validate the data
changeset_valid = Friends.Person.changeset(person, %{age: 1})
changeset_invalid = Friends.Person.changeset(person, %{first_name: ""})

# case condition to return a specific output depends from pattern matching
case Friends.Repo.insert(changeset) do
  {:ok, person} ->
    person
  {:error, changeset} ->
    changeset.errors
end

Friends.Repo.insert! Friends.Person.changeset(%Friends.Person{}, %{first_name: "Ryan"})
```

### Update a register using changeset and update method from Ecto
```elixir
# get the first Person on database using Ecto Query
person = Friends.Person |> Ecto.Query.first |> Friends.Repo.one

# create a changeset to validate the data
changeset_valid = Friends.Person.changeset(person, %{age: 10})
changeset_invalid = Friends.Person.changeset(person, %{first_name: ""})

# create a case condition to return a specific output depends from the pattern matching
case Friends.Repo.update(changeset) do
  {:ok, person} ->
    IO.inspect person
  {:error, changeset} ->
    IO.inspect changeset.errors
end
```

### insert_all
```elixir
Friends.Repo.insert_all(Friends.Person,
  [[first_name: "pedro", last_name: "barros", age: 30],
  [first_name: "bob", last_name: "Burnquist", age: 37]])

# INSERT INTO "people" ("age","first_name","last_name") VALUES ($1,$2,$3),($4,$5,$6) [28, "pedro", "barros", 45, "bob", "burnquist"]
```


# Reference Links
Eric Meadows Jöhnson - Ecto - database library for Elixir - Code Beam STO -
https://www.youtube.com/watch?v=RT4p_g0SLUU
