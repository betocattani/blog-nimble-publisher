%{
  title: "Comprehension Elixir",
  author: "Luiz Cattani",
  tags: ~w(comprehension elixir database),
  description: "This is my notes about Comprehenison in Elixir"
}
---

# Elixir Comprehension

Comprehension is a sugar syntax for loop over a Enumerable, filtering some result
and mapping the result for a new list.

Mapping an integer list and multiplying the each integer on the list by itself
```elixir
iex> for n <- [5, 6, 7, 8], do: n * n
=> [25, 36, 49, 64]
```

A Comprehension is made of three parts: **Generators, filters and collectable**

## Generators
In the example below the chunk of the code `n <- [5, 6, 7, 8]` it's the `generators`
We can think as "Generating values to be used in the comprehension"
Any enumerable can be passed on the right-hand side of the generator expression
```elixir
iex> for n <- [5, 6, 7, 8], do: n * n
=> [25, 36, 49, 64]
```

## Multiple Generators
```elixir
iex> for n <- [a:, :b, :c], j <- ["luiz", "cattani"], do: {n, j}
=> [a: "luiz", a: "cattani", b: "luiz", b: "cattani", c: "luiz", c: "cattani"]
```

## Filters
### Pattern Matching
Generators also supports Pattern Matching on their left-hand side, all non-matching patterns
are ignored.

```elixir
iex> classifications = [upvote: 1, downvote: 0, upvote: 1, upvote: 1, downvote: 0]

iex> for {:upvote, n} <- classifications, do: n * 5  
=> [5, 5, 5]

iex> for {:downvote, n} <- classifications, do: n * 5
=> [0, 0]

iex> for {:when_atom_does_not_matching, n} <- classifications, do: n * 5
=> []
```

we can use filter to select some particular elements, in the example all number multiple of 3,
and discard all that does not are multiple of 3, returns false or nil.
```elixir
iex> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
iex> for n <- 0..10, multiple_of_3?.(n), do: n * 10
```

## Collectable

The variables assignments inside the comprehension, be it in generators, filters,
or inside a block, are not reflected outside of the comprehension.
