%{
  title: "Using Distillery to deploy Phoenix app on Gigalixir",
  author: "Luiz Cattani",
  tags: ~w(gigalixir deploy elixir phoenix),
  description: "This is my notes about how to deploy a Phoenix application using Distillery"
}
---

# Distillery Elixir

Add do mix file

```elixir
defp deps do
  [
    {:distillery, "~> 2.1"}
  ]
end
```

Install dependencies
```bash
$ mix deps.get
```

Initialize and Create Distillery configuration file
```bash
$ mix distillery.init
rel/config.exs
```

Change the file `prod.secret`
```elixir
config :store, Store.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "",
  # ssl: true,
  url: database_url,
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10")
```

Change the file config/prod.exs
```elixir
config :store, StoreWeb.Endpoint,
  load_from_system_env: true,
  http: [port: {:system, "PORT"}],
  server: true,
  check_origin: ["http://localhost:4000/", "https://localhost:4000", "https://store.gigalixirapp.com"],
  secret_key_base: "${SECRET_KEY_BASE}",
  url: [host: "${APP_NAME}.gigalixirapp.com", port: 443],
  cache_static_manifest: "priv/static/cache_manifest.json"
  ```

```bash
$ cd assets 
$ npm run deploy
```  

Compress static files
```bash
$ mix phx.digest
```

Export environment variables
```bash
export SECRET_KEY_BASE="$(mix phx.gen.secret)"
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/store_dev"
export APP_NAME=store
export MY_HOSTNAME=example.com
export MY_COOKIE=secret
export REPLACE_OS_VARS=true
```

Build the release locally
```bash
$ MIX_ENV=prod mix distillery.release --env=prod
```

Start the release on foreground
```bash
$ MIX_ENV=prod MY_NODE_NAME=store@127.0.0.1 PORT=4000 _build/prod/rel/store/bin/store foreground
```

Possible to access the release from the localhost:4000


# Deploy to production Enviroment
Uncomment the ssl config on Repo

```elixir
config :store, Store.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "",
  ssl: true,
  url: database_url,
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10")
```

Create a Git repository
```bash
$ git init
$ git add
$ git commit -m 'Deploy to gigalixir using distillery'
```

```bash
$ git push gigalixir main
```

# Install Gigalixir

Install Gigalixir using homebrew
```bash
$ brew install gigalixir
```

Login in the Gigalixir account
```bash
$ gigalixir login
$ type email
$ type password
Would you like us to save your api key to your ~/.netrc file? [Y/n]:
$ Y
```
Create a new app in gigalixir / autommaticaly set the remote to the repository
```elixir
$ gigalixir create
```

List information from app existent
```bash
$ gigalixir apps
[
  {
    "cloud": "gcp",
    "region": "v2018-us-central1",
    "replicas": 1,
    "size": 0.3,
    "stack": "gigalixir-20",
    "unique_name": "store",
    "version": 5
  }
```

Creatint Postgres database on Gigalixir
```bash
$ gigalixir ps:create --free
[
  {
    "app_name": "store",
    "database": "1ba0618a-c105-4ed6-9def-271bcaa54e78",
    "host": "postgres-free-tier-v2020.gigalixir.com",
    "id": "1ba0618a-c105-4ed6-9def-271bcaa54e78",
    "limited_at": null,
    "password": "pw-9fa2c493-4998-406e-93c1-524e4e7e0940",
    "port": 5432,
    "state": "AVAILABLE",
    "tier": "FREE",
    "url": "postgresql://1ba0618a-c105-4ed6-9def-271bcaa54e78-user:pw-9fa2c493-4998-406e-93c1-524e4e7e0940@postgres-free-tier-v2020.gigalixir.com:5432/1ba0618a-c105-4ed6-9def-271bcaa54e78",
    "username": "1ba0618a-c105-4ed6-9def-271bcaa54e78-user"
  }
]
```

Set POOLS
```bash
$ gigalixir config:set POOL_SIZE=2
```

Creating buildpack files to set elixir, erlang and node versions and commit the new files
```elixir
# /elixir_buildpack.config
elixir_version=1.11.4
erlang_version=23.3
always_rebuild=true

# /phoenix_static_buildpack.config
node_version=14.4.0
npm_version=6.14.5
clean_cache=true
```

Push the code to Gigalixir and trigger the deploy
```bash
$ git push gigalixir main
```

# Database
```bash
$ gigalixir ps:migrate
```

# SSH
```bash
gigalixir account::ssh_keys::add %"{}"
```

# Add ssh key in Gigalixir
```bash
$ gigalixir account:ssh_keys:add "$(cat, ~/.ssh/id_rsa.pub)"
```

# Access the remote console in Gigalixir service
```bash
$ gigalixir ps:remote_console
$ iex(store@10.56.14.197)1>
```

# Run Seeds in Gigalixir service

Access the remote console
```elixir
$ gigalixir remote.console

iex> seed_script = Path.join(["#{:code.priv_dir(:myapp)}", "repo", "seeds.exs"])
iex> Code.eval_file(seed_script)
```

# Logs

See all logs from a specific machine
```bash
$ gigalixir logs -a store
```

# Status of an application

See the current status of the application
```bash
$ gigalixir ps store
```