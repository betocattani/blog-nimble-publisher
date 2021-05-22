%{
  title: "Tests in Elixir using ExUnit",
  author: "Luiz Cattani",
  tags: ~w(elixir tests phoenix),
  description: "This is my notes studing tests in Elixir"
}
---

# ExUnit

Anatomy of a test

```Setup/ Exercise / Verify / Teardown```

Setup - Preparing data to pass in as parameters to the code under test or ig can be a staging data.

Exercise - The call to run the code under test. Without it, there would be nothing to do.

Verify - make the assertions about the behaviour of you code, sometimes can be inline with the exercise step, verify if the code behaves a certain way. Without it your tests would be pointless

Teardown - If our setup phase involved any impact on shared state, we need to return things to how the were beforehand during the teardown phase, there are tools to take care of this step automatically, but always happen.

The `order` of the stages isn't guaranteed to be setup, exercise, verify and teardown. Sometimes test code has to be written in a different order than it is executed. The commom case for it when is a `test double`, if test double utilizes assertions, the code defining it will have to be written before the exercise step.

The secondary pourpose to write tests, if written well, it becomes a source of documentation of our code, with text description of how we expect the code to work and working examples of how to exercise the code.

### Describing The Tests

`ExUnit` provides a good tool for grouping tests together with a file, `describe`.
Using `describe` allow us to pass a description for the groups that share a common setup.

ExUnit allow only one level of grouping, `we can not nest describe` blocks, brings healthy balance between organizing our tests and avoiding nesting messy for setup.

```elixir
defmodule YourApp.YourModuleTest do
  use ExUnit.Case

  describe "thing_to_do/1" do
    test "it returns :ok, call the function if the key is correct"
    
    test "it does not call the function if the key is wrong"
  end
end
```

# Setup Block

# Life cycle
When we run `mix test`, Mix essentialy compiling the project, loading all the files ending in  `_test.exs` from the `test` directory of the project, and then runnint 
`test/test_helpers.exs`. Mix is taking care of starting the ExUnit suite for us.

Start ExUnit and automqtically runs tests right before the VM terminates.

`ExUnit.start()` relies on `System.at_exit/1` to actually run the whole test suit right
before shutting down the virtual machine. This give us a chance to load everything we need and compile every test case module before running any test.
```elixir
ExUnit.start()
```

# Test Cases
Every module whose name ends with a trailing `Test` and contain a call to use ExUnit is
considered a test case.

A test case is essentialy a collection of setup callbacks na tests.

`Test cases` can be executed either concurrenclty or sequentially in a relation to other test cases.

All the `Test Cases` that use the `async: true` option