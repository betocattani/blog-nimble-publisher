%{
  title: "Debugging Elixir Code",
  author: "Luiz Cattani",
  tags: ~w(debugger elixir),
  description: "This is my notes about how to debugger elixir code"
}
---

# Elixir debugger

### IEX.pry

We can use `IEx.pry` to debugger the code

```elixir
defmodule Calculator do
  def sum(x, y) do
    require IEx; IEx.pry
    
    make_sum(x, y)
  end

  def make_sum(x, y) do
    x + y
  end
end
```

### IO.inspect

We can use the `IO.inspet` to print the data
```elixir
defmodule Calculator do
  def sum(x, y) do
    IO.inspect(x)
    IO.inspect(y)
    
    make_sum(x, y)
  end

  def make_sum(x, y) do
    x + y
  end
end
```

### Breakpoints

For those who enjoy breakpoints but are rather interested in a visual debugger, Erlang/OTP ships with a graphical debugger conveniently named :debugger. Let’s define a module in a file named example.ex

```elixir
iex> :debugger.start()
{:ok, #PID<0.87.0>}

iex> :int.ni(Calculator)
{:module, Calculator}

iex> int.break(Calculator, 3)
:ok

iex> Calculator.double_sum(1, 2)
```

### Observer

For debugging complex systems, jumping at the code is not enough. It is necessary to have an understanding of the whole virtual machine, processes, applications, as well as set up tracing mechanisms. Luckily this can be achieved in Erlang with :observer. In your application:

```elixir
iex> :observer.start()
```