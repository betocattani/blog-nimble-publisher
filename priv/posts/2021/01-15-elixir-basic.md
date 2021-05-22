%{
  title: "Basic Concepts Elixir",
  author: "Luiz Cattani",
  tags: ~w(elixir),
  description: "Basic concepts in Elixir"
}
---

# Elixir

## Simple expressions
*to do*

## Values
*to do*

## Variables
*to do*

## Operators
*to do*

## Functions first class citzen
*to do*

## Anonymous Functions
*to do*

## Scope of functions
*to do*

## Imutability
*to do*

## Closures
*to do*

## Data Types

### Tuples
Tuples are collections that are stored in contiguously in memory, allowing fast access to their elements
by index.

They are often used to pass a signal with values
```elixir
# The first item indicates a success with an atom
# The second item indicates a computed value
iex> {:ok, 42}
```

```elixir
iex> {a, b ,c} = {1, 2, 3} # we are using the destructuring action
# we unpacking values from the tuple and binding them to variables

iex> a
=> 1

iex> c
=> 3
```

```elixir
iex> say_hello = fn -> {:ok, "hello friend"} end
iex> {:ok, msg} = say_hello.()
iex> IO.puts "hey, #{msg}"
```

A good practice in elixir and libraries community is to return
```elixir
{:ok, value} # to success process
{:error, error_type} # to failure process
```

### Strings
Strings are binaries in elixir

## Command History
By default the history of iex is disabled when we installed elixir.

Enable iex to maintain the history of the commands and make the narrows up and down available to navigate in the commands already executed.

```bash
# This command passing to Erlang configuration option through IEx using --erl
$ iex --erl "-kernel shell_history enabled"

# we can set a system environment variable to load all the times that we access the iex shell
$ export ERL_AFLAGS="-kernel shell_history enabled"

# It's important to close and open a new bash session to the system env variable it will be available.
# We can check if the env variable is set
$ iex
iex> System.get_env("ERL_AFLAGS")
=> "-kernel shell_history enabled"
```

# Import Modules
To reduce the necessity of writing `FileModule` every time we call `.write` and `.read` functions by importing the module `File`, with the `[only:]` option to be clear from where
functions are comming, is a good practice be explicit about it.
```elixir
defmodule TaskList do
  import File, only: [write: 3, read: 1]

  @file_name = "task_list.md"

  def add(task_name) do
    task = "[ ] " <> task_name <> "\n"
    write(@file_name, task, [:append]) # <FileModule>.write/3
  end

  def show_list do
    read(@file_name) # <FileModule>.read/1
  end
end
```

### Function Arity
The number after the function name in the import directives is called function arity
Function arity is the number of arguments a function receives name_function/arity

#Using Named Function as Values

Wrap the function in an anonymous function.
```elixir
iex> upcase = fn word -> String.upcase(word) end

iex> upcase.("snake")
iex> "SNAKE"
```

## Handy function capturing operator
`&` capture the reference to the function String.upcase/1, short way of binding
named functions to variables or functions' arguments:
```elixir
iex> upcase = &String.upcase/1

iex> upcase.("Snake")
iex> "SNAKE"
```

Using `&` operator to create anonymous function, & capture the beginning of function,
and its body is inside of the parentheses:
```elixir
iex> divide = &(&1, &2)

iex> divide.(100, 5)
iex> 20.0
```

Parentheses is optional:
```elixir
iex> divide = & &1 / &2`
```

We can't use the capture syntax for creating anonymous function with `zero arity`:
```elixir
iex> truthy = &(true)
=> (CompileError) iex:13: invalid args for &, expected an expression in the format of &Mod.fun/arity, &local/arity or a capture containing at least one argument as &1, got: true
```

We should use the explicit `fn` form
```elixir
iex> truthy = fn -> true end
iex> truthy.()
iex> true
```
## Using the `&` operator with caution because of the lack of arguments names

## Exercises

1 - Expression
```elixir
iex> (10 * 0.1) + (3 * 2) + (1 * 15)
iex> 22.0
```

2 - Calculate the average of velocity
```elixir
iex> distance = 200
iex> hours = 4
iex> velocity = distance / hours
iex> IO.puts "Travel distance - #{distance}, Time: #{hours}, Average Velocity: #{velocity}"
```

3 - Anonymous function to apply tax in a price
```elixir
iex> apply_tax = fn price -> tax = price * 0.12; IO.puts "Price: #{price + tax}, Tax: #{tax}" end

```

4 - Create a module to divide the matchsticks inside of boxes with diferent sizes

```elixir
defmodule MatchstickFactory do
  @size_big 50
  @size_medium 20
  @size_small 5

  def boxes(matchsticks) do
    big_boxes = div(matchsticks, @size_big)
    remaining = rem(matchsticks, @size_big)

    medium_boxes = div(remaining, @size_medium)
    remaining = rem(remaining, @size_medium)

    small_boxes = div(remaining, @size_small)
    remaining = rem(remaining, @size_small)

    %{
       big: big_boxes,
       medium: medium_boxes,
       small: small_boxes,
       remaining_matchsticks: remaining
     }
  end
end
```

# Pattern Matching
Elixir pattern matching shapes everything you program, tries to match both side, left and right

Pattern matching allows developers to easily destructure data types such as tuples and lists

Algebra analogy - "if x = 1 then 1 = x"

## Equal operator
```elixir
iex> x = 1
iex> 1

iex> 1 = x
iex> 1

iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

## Rebinding
We can bind new values of a expression on the right side to the variable already defined
```elixir
iex> x = 2
iex> 2 = x
iex> 2

iex> x = 1
iex> 1 = x
iex> 1
```

```elixir
iex> {a, b, c} = {:hello, "world", 42}
iex> {:hello, "world", 42}
iex> a
iex> :hello
iex> b
iex> "world"

iex> {:ok, result} = {:ok, 13}
iex> {:ok, 13}
iex> result
iex> 13

iex> [a, b, c] = [1, 2, 3]
iex> [1, 2, 3]

iex> a
iex> 1
```

## Pin operator
To avoid a Rebinding in a variable we can use the `pin ^` operator:

- With the `pin` operator elixir use the value of the variable to match.
- we pinned the value to the variable
- When they don't match the program stops the execution with a matching error

```elixir
iex> x = 2
iex> 2

iex> ^x = 2
iex> 2

iex> ^x = 1
** (MatchError) no match of right hand side value: 1
```

```elixir
iex> x = 1
iex> 1

iex> [^x, 2, 3]  = [1, 2, 3]
iex> [1, 2, 3]

iex> {y, ^x} = {2, 1}
iex> {2, 1}

iex> y
iex> 2

iex> {y, ^x} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
```

```elixir
iex> {x, x} = {1, 1}
iex> {1, 1}

iex> {x, x} = {1, 2}
iex> ** (MatchError) no match of right hand side value: {1, 2}
```

## Prepending the items to a list using pattern matching
```elixir
iex> [head | tails] = [1, 2, 3]
iex> [1, 2, 3]

iex> head
iex> 1

iex> tail
iex> [2, 3]

iex> list = [1, 2, 3]
iex> [1, 2, 3]

iex> [0 | list]
iex> [0, 1, 2, 3]
```

# Unpacking values from various Data Types
Pattern matching is also useful to extracting parts of values to variables in a process called destructuring.

## Destructuring
We use destructuring with pattern match when we are making two things match.

### Matching parts of a String
Using the `<>` operator to check the beginning of a string, useful to checking text
organized in `key/value` pairs.

```elixir
iex> "Authentication: " <> credentials = "Authentication: Basic sdvkSDAsa"
iex> "Authentication: Basic sdvkSDAsa"

iex> credentials
iex> "Basic sdvkSDAsa"
```

### Restriction in pattern matching
we can't use a variable on the left side of the <> operator, strings are binaries, and <> is a binary operator

We can't start our expression with a variable without providing its binary size:
```elixir
iex> first_name <> " Doe" = "John Doe"
** (CompileError) a binary field without size is only allowed at the end of a binary pattern and never allowed in binary generators
```
We can easily avoid this by reversing the string values:
```elixir
iex> "eoD" <> first_name = String.reverse("John Doe")
iex> String.reverse(first_name)
iex> "John "
```

## Matching Tuples
Tuples are collections that are stored in memory, allowing fast access to their elements by index.
{ok, 42}

index: 0
:ok

index: 1
42

### Useful for signaling successes and failures in a function's return

```elixir
iex> process_life_the_universe_and_everything = fn -> {:ok, 42} end
iex> {:ok, answer} = process_life_the_universe_and_everything.()
iex> IO.puts "The answer is #{answer}"
iex> The answer is 42.
```

The .exs extension is for Elixir scripts that don't need to generate a compiled version
useful for simple scripts.

the underline(_) character inside of tuple {ability_score, _}, used to ignore some parts of
the pattern matching.
```elixir
iex> user_input = IO.gets "Write your ability score:\n"
iex> {ability_score, _} = Integer.parse(user_input)
iex> ability_modifier = (ability_score - 10) / 2
iex> IO.puts "Your ability modifier is: #{ability_modifier}"
```

To execute this script, just run in the terminal
```bash
$ elixir <name_of_file>.exs
```

# Equals operator =

`=` is used for pattern matching
`==` returns true when the elements are equal and when Integers and floats are equivalent numbers
`===` returns true when arguments are equivalent and have the same type

```elixir
1 = 1 # returns 1
2 = 1 # match error!
1 == 1.0 # returns true
2 == 1 # returns false
1.0 === 1.0 # returns true
1.0 === 1 returns false
```

# Matching list
```elixir
# ex-1
iex> [a, b, c] = [1, 2, 3]
=> [1, 2, 3]

# ex-2
iex> [a, b, b] = [1, 2, 2]
=> [1, 2, 2]

# ex-3
iex> [ head | tail ] = [1, 2, 3, 4, 5]

iex> head
=> 1

iex> tail
=> [2, 3, 4, 5]

# ex-4
iex> [a, b | rest] = ["apple", "orange", "grape", "coconut"]

iex> a
=> "apple"

iex> b
=> "orange"

iex> rest
["grape", "coconut"]

# ex-5
iex> [a, b, a] = ["apple", "grape", "banana"]
=> (MatchError) no match of right hand side value: ["apple", "grape", "banana"]

# ex-6
iex> [_, b, _] = ["banana", "apple", "coconut"]
iex> b
=> "apple"

# ex-7
iex> [a, b, _] = ["banana", "apple", "coconut"]
iex> a
=> "banana"

iex> b
=> "apple"
```

# Matching tuples
```elixir
# ex-1
iex> {a, b, c} = {1, 2 ,3}
=> {1, 2 ,3}

# ex-2
iex> {:ok, answer} = {:ok, "hello world"}
iex> IO.puts answer
iex> "hello world"

# ex-3
iex> {:ok, _} = {:ok, "wherever I want to put here"}
=> {:ok, "wherever I want to put here"}
```

# Matching Maps
Maps are data type structured in key/value pairs

```elixir
# ex-1
iex> user_data = %{email: "cattani@mail.com", password: "elixir"}
iex> %{email: "cattani@mail.com"} = user_data
=> %{email: "cattani@mail.com", password: "1234"}

# ex-2
iex> date = %{"2020/01" => "January"}
iex> %{"2020/01" => "January"} = date
=> %{"2020/01" => "January"}

# ex-3
iex> date = %{"2020/01" => "January"}
=> %{"2020/01" => "January"}
iex> %{"2020/01" => date_month} = date
=> %{"2020/01" => "January"}
iex> date_month
=> "January"
```

# Matching Structs

# Matching Dates
```elixir
~D[2020-01-01]

Date.new(2020, 1, 1)
```

# Control flow with functions
```elixir
defmodule NumberCompare do
  def greater(number, other_number) do
    check(number >= other_number, number, other_number)
  end

  defp check(true, number, _), do: number
  defp check(false, _, other_number), do: other_number
end
```

# Private functions
```elixir
defp private_function do

end
```

# Applying default values for functions
```elixir
defmodule Checkout do
  def total_cost(price, quantity \\ 10), do: price * quantity
end
```
