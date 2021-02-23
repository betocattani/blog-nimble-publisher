%{
  title: "Kernel Elixir",
  author: "Luiz Cattani",
  tags: ~w(elixir kernel),
  description: "Using Kernel module in Elixir"
}
---

# Read the file

Using the module File from Elixir to read a file
```elixir
# without the trailing bang to read the file returns the a tuple with the escaped json data
file = File.read("filename.json")
{:ok,
 "[\n  {\n    \"Player\":\"Joe Banyard\",\n    \"Team\":\"JAX\",\n    \"Pos\":\"RB\",\n    \"Att\":2,\n    \"Att/G\":2,\n    \"Yds\":7,\n    \"Avg\":3.5,\n    \"Yds/G\":7,\n    \"TD\":0,\n    \"Lng\":\"7\",\n    \"1st\":0,\n    \"1st%\":0,\n    \"20+\":0,\n    \"40+\":0,\n    \"FUM\":0\n  },\n  {\n    \"Player\":\"Shaun Hill\",\n    \"Team\":\"MIN\",\n    \"Pos\":\"QB\",\n    \"Att\":5,\n    \"Att/G\":1.7,\n    \"Yds\":5,\n    \"Avg\":1,\n    \"Yds/G\":1.7,\n    \"TD\":0,\n    \"Lng\":\"9\",\n    \"1st\":0,\n...}"


# using the trailing bang to read the file returns the escaped json data
file = File.read!("filename.json")
"[\n  {\n    \"Player\":\"Joe Banyard\",\n    \"Team\":\"JAX\",\n    \"Pos\":\"RB\",\n    \"Att\":2,\n    \"Att/G\":2,\n    \"Yds\":7,\n    \"Avg\":3.5,\n    \"Yds/G\":7,\n    \"TD\":0,\n    \"Lng\":\"7\",\n    \"1st\":0,\n    \"1st%\":0,\n    \"20+\":0,\n    \"40+\":0,\n    \"FUM\":0\n  },\n  {\n    \"Player\":\"Shaun Hill\",\n    \"Team\":\"MIN\",\n    \"Pos\":\"QB\",\n    \"Att\":5,\n    \"Att/G\":1.7,\n    \"Yds\":5,\n    \"Avg\":1,\n    \"Yds/G\":1.7,\n    \"TD\":0,\n    \"Lng\":\"9\",\n    \"1st\":0,\n...}"
```


```elixir
file = File.read!("rushing.json")

players = Poison.decode!(file)

last_player = List.last(players)

Map.get(player_map, "Player")

decoded_file |> List.first |> Map.get("Player")

Enum.each decoded_file, fn file ->
  IO.inspect file
end

Enum.each players, fn player ->
  formated_player = %Player{average: player["Avg"], name: player["Player"], team: player["Team"]}

  IO.puts "Player: #{formated_player.name}, Average: #{formated_player.average}, Team: #{formated_player.team}"
end
```


# File

## Read file
With this approach we are reading the entire file and we need to wait the elixir arrives in the end o file
```elixir
# returns a tuple
iex> File.read("filename.txt")
{:ok, "rixilE evol I\nnrael ot ysae os si tI\nedoc lanoitcnuf taerG\n"}

# To create a multi line pipe we need add in the end of each line the `\`
iex> fixed_contents = contents \
iex>  |> String.split("\n", trim: true) \
iex>  |> Enum.map(&String.reverse/1) \
iex>  |> Enum.join("\n")

# inline version
iex> context |> String.split("\n", trim: true) |> Enum.map(&String.reverse/1) |> Enum.join("\n")
```

## Stream file
Stream a file we can read line by line of the file

Usually for big files is better use the `File.stream!` instead use `File.read!`
```elixir
# iex multiline
iex> File.stream!("filename.txt") \
iex> Enum.map(&String.trim/1) \
iex> Enum.map(&String.reverse/1)
iex> Enum.join("\n")

# iex inline
iex> File.stream!("haiku.txt") |> Enum.map(&String.trim/1) |> Enum.map(&String.reverse/1) |> Enum.join("\n")

=> "I love Elixir\nIt is so easy to learn\nGreat functional code"
```

# Write Files
```elixir
iex> File.write('path_name.txt', contents)
```
