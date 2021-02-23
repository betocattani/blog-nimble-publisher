%{
  title: "Tests in Elixir",
  author: "Luiz Cattani",
  tags: ~w(tests elixir),
  description: "Testing in Elixir"
}
---

# Elixir Test

## ExUnit
Tool default in Elixir language to test our code

Elixir supports the ability to run tests concurrently within the same Elixir instance.

## Running
Run all specs
```bash
$ mix test
```

Run a specific spec
```bash
$ mix test test/hello_world_test.exs
```

## Tags
We can create tags for our specs

```elixir
@tag :simple_test
test "does not greets the world" do
  assert HelloWorld.hello() == :world
end
```
## only
Run the specs with a specific tag
```bash
$ mix test --only tag_name
```
## exclude
Run all tests excluding a specific tag
```bash
$ mix test --exclude tag_name
```


## Curiosity:
if a duplicated describe of a spec exists in the same file Elixir raise an error, awesome \o/
```elixir
test "does not greets the world" do
  refute HelloWorld.hello() == :wolrd
end

test "does not greets the world" do
  assert HelloWorld.hello() == :world
end

$ mix test
(ExUnit.DuplicateTestError) "test does not greets the world" is already defined in HelloWorldTest
```
