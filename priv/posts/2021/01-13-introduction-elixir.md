%{
  title: "Introduction Elixir",
  author: "Luiz Cattani",
  tags: ~w(elixir),
  description: "Introduction Elixir"
}
---
# Functional Programming

# Install Elixir

```bash
$ brew update
$ brew install elixir
```

## Imperative Languages

Limitations of imperative languages:
  - shared mutating values
  - mutating values can be dangerous to concurrency

Multiple parts of an application running in parallel and having access to this value at the same time
If have access to modify, what happen if in the middle of some operation the values changes because
of another process?

## Functional Programming Paradigm
 - Functions are the basic building blocks
 - All values are immutable
 - Code is declarative

## Elixir
  - Dynamic
  - Functional language
  - Lives in the Erlang ecosystem, existed for 30 years, delivering software with nine 9s reliability

## Functional Paradigm in a functional language
  - Functions
  - Immutability
  - Declarative code


## Immutable Data

### Ruby example freeze objects
```rb
irb> Food = Struct.new(:name)
irb> foods = [Food.new("barbecue"), Food.new("lasagna")].freeze
irb> puts foods.inspect
irb> "[#<struct Food name="barbecue">, #<struct Food name="lasagna">]"

# In Ruby when we freeze some object, we can't add or remove items,
irb> foods.push("Pasta")
irb> "FrozenError (can't modify frozen Array: [#<struct Food name="barbecue">, ...)"

# but we can modify the the stored value
irb> foods.first.name = "piza"
irb> puts foods.inspect
irb> "[#<struct Food name="pizza">, #<struct Food name="lasagna">]"
```


## Functions
Functions are the primary tool in the functional programming for building a program.
We combine multiple little functions to create a large program

### Pure functions:
  - These values are immutable
  - The function's result is affected only by the function's arguments
  - The function doens't generate effects beyond the value it returns

```elixir
add2 = fn(n) -> n + 2 end
add2.(2)
=> 4
```

### Inpure Functions
  - The results are unpredictable


## Using Values Explicitly

### Object Oriented Paradigm
The conventionals object oriented languages use objects to store a state, providing methods
for operating on that state.
```rb
class MySet
  attr_reader :items

  def initialize
    @items = []
  end

  def push(item)
    if items.include?(item)
      return "item '#{item}' already included"
    else
      items.push(item)
    end
  end
end

# set = MySet.new
# set.push("apple")

# The operations must be called from a method that belongs to an object that contains data
```

### Functional Paradigm
Functional programming always passes the values explicitly between the functions, making
clear to the developer what the inputs and outputs are.

```elixir
defmodule MySet do
  defstruct items: []

  def push(set = %{items: items}, item) do
    if Enum.member?(items, item) do
      set
    else
      %{set | items: items ++ [item]}
    end
  end
end

# set = %MySet{}
# set = MySet.push(set, "apple")

# The data must be explicitly sent to the MySet.push
# Every time we call the function, it generates a new data structure with updated values.
# Then we updated the set variable to store the updated value and print it
```

## Using function in arguments
Functions can be used in the arguments and results of Functions

```elixir
iex> Enum.map(["dogs", "cats", "birds"], &String.upcase/1)

# The Enum.map function knows how to apply String.upcase to each item in the list.
# The result is a new list with all words uppercased
```

## Transforming Values
Elixir focus is on the data transformation flow, it has a special operator called pipe "|>",
to combine multiple functions calls and result.

Using the pipe operator the result of each expression will be passed to the next function

```elixir
def capitalize_words(title) do
  title
    |> String.split
    |> capitalize_all
    |> join_with_whitespace
end
```

# Declaring code

## Imperative Programming
Focus on how to solve a problem, describing each step as actions

When we use imperative mindset we'll need control flow structures like 'for' to navigate through
each element of the list, incrementing the variable 'i' one by one, then we need push the new uppercased
string  in the newList variable
```js
var list = ['dogs', 'cats', 'birds']

function upcase(list) {
  var newList = [];

  for(var i = 0; i < list.length; i++) {
    newList.push(list[i].toUpperCase());
  }

  return newList;
}

// upcase(list)
// => ['DOGS', 'CATS', 'BIRDS']
```

## Declarative Programming
Focus on what is necessary to solve a problem, describing the data flow.

- The upcase result of an empty list is an empty list
- When the list has items, the result is a new list where the first string is uppercased
  and the rest of the items are passed to the upcase function.
- We describe how the data must be, not the actions to generate the result

```elixir
defmodule StringList do
  def upcase([]), do: []
  def upcase([first | rest]), do: [String.upcase(first) | upcase(rest)]
end

# StringList.upcase(["dogs", "cats", "birds"])
# ['DOGS', 'CATS', 'BIRDS']
```

# Elixir File Compilation and Scripted Mode

## Creating Elixir files
We can define a elixir file using two extension .ex or .exs.
```elixir
# my_file.ex
# my_file.exs
```

### .ex
Most of the times is convenient to write modules into the files so they can be compiled and reused.
Each time that we compiled the file, will generates a beam file containing the bytecode for the defined module
Files used by our application business logic

```elixir
# my_elixir_file.ex
defmodule MyModule do
  def sum(a, b) do
    a + b
  end
end

$ iex my_elixir_file.ex
iex> MyModule.sum(1, 1)
iex> 2
```

### .exs
Also called as Scripted mode

Used to write elixir scripts that don't need to generates a compiled version to be executed.

```elixir
# another_elixir_file.exs
defmodule MyModule do
  def sum(a, b) do
    a + b
  end
end

$ iex another_elixir_file.exs
iex> 2
```
