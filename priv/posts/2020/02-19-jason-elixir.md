%{
  title: "Jason Elixir",
  author: "Luiz Cattani",
  tags: ~w(elixir jason),
  description: "Using Jason Elixir"
}
---

# JASON Elixir

Why use Jason instead Poison?
- The primary reason is performance.
- Jason is about twice as fast and uses half the memory
- while being almost 100% functionally compatible with Poison

Add dependencies to mix file
```elixir
# mix.exs

defp deps do
  [
    {:jason, "~> 1.2"}
  ]
end
```

# Decode json into a list of map data
```elixir
iex> file = File.read("filename.json")
iex> Jason.decode!(file)
```
