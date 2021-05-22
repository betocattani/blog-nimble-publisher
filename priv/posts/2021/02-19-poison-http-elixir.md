%{
  title: "Poison HTTP Framework Elixir",
  author: "Luiz Cattani",
  tags: ~w(poison elixir),
  description: "Using Poison Elixir"
}
---

# Poison Elixir

# Install
Add the dependencies to the `mix.exs` file

```elixir
def deps do
  [
    {:poison, "~> 3.1"}
  ]
end

iex> mix deps.get
```

List all functions available on the Poison module
```elixir
iex> Poison.module_info(:functions)
[
  __info__: 1,
  decode: 1,
  decode: 2,
  decode!: 1,
  decode!: 2,
  encode: 1,
  encode: 2,
  encode!: 1,
  encode!: 2,
  encode_to_iodata: 1,
  encode_to_iodata: 2,
  encode_to_iodata!: 1,
  encode_to_iodata!: 2,
  module_info: 0,
  module_info: 1
]
```

Information about a specific function from Poison module
```elixir
iex> h Poison.encode!

# def encode!(value, options \\ [])
#
# @spec encode!(Poison.Encoder.t(), Keyword.t()) :: iodata() | no_return()
#
# Encode a value to JSON, raises an exception on error.
#
# iex> Poison.encode!([1, 2, 3])
# "[1,2,3]"
```

# Encode Data

Encode some data to json
```elixir
# with trailing bang the return is the encoded json
iex> Poison.encode!(%{"player" => "Michael", "team" => "Chicago Bulls"})
=> "{\"team\":\"Chicago Bulls\",\"player\":\"Michael\"}"

# without trailing bang the returns is a tuple
iex> Poison.encode(%{"player" => "Michael", "team" => "Chicago Bulls"})
{:ok, "{\"team\":\"Chicago Bulls\",\"player\":\"Michael\"}"}

# Elixir also accepts %{key: value} instead use the hash rocket `=>`
iex> Poison.encode!(%{player: "Michael", team: "Chigago Bulls"})
=> "{\"team\":\"Chicago Bulls\",\"player\":\"Michael\"}"
```

# Decode File

Decode json into a list of map data
```elixir
json_data = "{\"team\":\"Chicago Bulls\",\"player\":\"Michael\"}"

# without the trailing bang returns a tuple
Poison.decode(json_data)
=> {:ok, %{"player" => "Michael", "team" => "Chicago Bulls"}}

# using the trailing bang returns the decoded json data
Poison.decode!(json_data)
%{"player" => "Michael", "team" => "Chicago Bulls"}
```
