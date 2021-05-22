%{
  title: "What is Elixir Process by Elixir in Action Book",
  author: "Luiz Cattani",
  tags: ~w(elixir phoenix process),
  description: "This is my notes explaning what is elixir process reading the Elixir in Action book"
}
---

# Concurrence in BEAM
In BEAM, a process is a concurrent thread of execution. Two processes run concur- rently and may therefore run in parallel, assuming at least two CPU cores are available.

Processes help us run things in `parallel`, allowing us to achieve `scalability` — the ability to address a load increase by adding more hardware power that the system automatically takes advantage of.

Processes also ensure `isolation`, which in turn gives us `fault-tolerance` — the ability to localize and limit the impact of unexpected runtime errors that inevitably occur. If you can localize exceptions and recover from them, you can implement a system that truly never stops, even when unexpected errors occur.
In BEAM, a process is a concurrent thread of execution. 

Two processes run concurrently and may therefore run in parallel, assuming `at least two CPU cores` are available. Unlike OS processes or threads, BEAM processes are lightweight concurrent entities han- dled by the VM, which uses its own scheduler to manage their concurrent execution.

# Process is the 
# Running elixir process concurrently
```elixir
run_query =
  fn query_def ->
    Process.sleep(2000)
    "#{query_def} result"
  end

Enum.map(1..5, &run_query.("query #{&1}"))
```

# Creating process
Using the auto imported function `spawn`

```elixir
spawn(fn ->
  expression_1   # (runs in the new process)
  expression_2   # (runs in the new process)
end)
```

The function `spawn/1` takes a zero arity lambda that will run in the new process, after the process is created `spawn` immediately returns, and the caller process's exectuon continues.

The provided lambda is executed in the new process and therefore run concurrently. After the lambda is done the `spawned` process exists, and its memory is released.

```elixir
spawn(fn -> IO.puts(run_query.("#{query 1}")) end)
#PID<0.48.0>   returns immediately

result of query 1   # Printed after two seconds
```

We can do something else in the shell while the query runs.

# Running elixir process concurrently and paralalized
Important technique applied below `Passing data to the created process`

assync_query takes one argument and binds it to the query_def variable. This data is then passed to the newly created process via the `Closure Mechanism`

The inner lambda, run in a separated process, references the variable query_def from the outer scope.

This result in `cross-process data passing`, the content of `query_def` are passed from the main process to the newly created one.
When it's passed to another process, the data is `deep-copied`, because processes can't share any memory.

```elixir
async_query =
  fn query_def ->
    spawn(fn -> IO.puts(run_query.(query_def)) end)
  end

Enum.each(1..5, &async_query.("query #{&1}"))
:ok         # returns immediately

result of query 5
result of query 4
result of query 3        # after two seconds
result of query 2
result of query 1
```

Five time faster thant the first sequential version that we had to wait 10 seconds, all results printed at practically the same time, two seconds later.

This is happen because we run our code concurrently, with this the order of exection is not guaranteed.

In BEAM, everything runs in a process


# Messaging Passing

### Inter-process communication via messages
When process A need to process B make some calculation,
`sends a assynchronous message` to B

`Send a message` is the same of to `storing it` into the `receiver's mailbox`.

The caller continue with it's own exectuon, and the receiver pull the message in at any time and process it
in some way, because `process can't share memory`, a `message is deep-copied` when it's sent.

The process `mailbox` is a `FIFO queue` limited only by available memory

To send a message to a process we need to have acces to the `process identifier PID`

The content of the message in an Elixir term, anything we can store in a variable

The receiver consumes message in the order received, and a message can be removed form the queue only if it's consumed. 

The return of spawn function is the PID of the new process

To obtain the PID of the current process we can use the function `self/0`

Sending a message to a process using the PID and the function `send/2`.
```elixir
send(pid, {:an, :arbitrary, :term})
```

Send placed in the mailbox of the receiver, the caller continues doing something else.

On the Receiver side, to `pull message` from the `mailbox`, we need to use the `receive` expression.
```elixir
receive do
  pattern_1 -> do_something
  pattern_2 -> do_something_else
end
```

The `receive expression` works similar to the `case expression`.

It tries to `pull` on message from the `process mailbox`, match it against any of the provided patterns, and run the corresponding code.

Sending a message to the process itself

```elixir
iex> send(self(), "a message")
"a message"

receive do
  message -> IO.inspect(message)
end
"a message"
```

handle specific mesage using pattern matching
```elixir
send(self(), {:message, 1})

receive do
  {:message, id} -> 
    IO.puts("received message #{id}")
end

"received message 1"
```

If theare are `no messages` in the `mailbox`, receive waits indefinitely for a new message to arrive, we need terminate it mannualy.
```elixir
receive do
  message -> IO.inspect(message) # The shell is blocked because the process mailbox is empty.
end
```

The same things happens if a message can't be matched against provided pattern clauses:
```elixir
send(self(), {:message, 1})

receive do
  {_, _, _} -> 
    IO.puts("received")
end

# The shell is blocked
```

To the receiver not block the shell, we can speficy the
`after` clause, which is executed if a message isn't received in a give time frame (in mileseconds)

```elixir
send(self(), {:message, 1})

receive do
  {_, _, _} -> 
    IO.puts("received")
  after
    5000 -> IO.puts("message not received")
  end
end

"message not received"
```

```elixir
send(self(), {:message, 1})

receive do
  message -> IO.inspect(message)
after
  5000 -> IO.puts("message not received")
end

{:message, 1}
{:message, 1}
```

```elixir
receive do
  message -> IO.inspect(message)
after
  5000 -> IO.puts("message not received")
end

"Message not received"
```


Receive Expression works as follows:
1 - Take the first message from the `mailbox`.
2 - Try to `match` it against any of the `provided patterns` going from the top to bottom.
3 - If a `pattern matches` the message, `run` the corresponding `code`.
4 - If `no pattern matches`, put the message `back into the mailbox` at the same position it originally occupied. Then try the `next message`.
5 - If there are `no more messages in the queue`, `wait for a new one to arrive. When a new messages arrives, start from step 1`, inspecting the first message in the mailbox
6 - If the `after` clause is specified and no messages is matched in the given amount of time, run the code from the `after` block.


The result of `receive` block is the result of the `last expression` in the appropriate clause
```elixir
iex> send(self(), {:message, 1})
iex> receive_result =
      receive do
        {:message, x} ->
          x + 2        # The result of receive
      end

iex> IO.inspect(receive_result)
```

### Process functions

```elixir
# Returns the info about one process
Process.info(self())

# Registers the given pid_or_port under the given name.
Process.register(pid_or_port, :my_process_name)
# Returns nil if the PID or PORT not exist locally and alive
# The name is already registered
# The pid or port is already registered under a different name

# Return a list os process registered
Process.registered
[:my_process_name, :logger_sup.. ]

# Find a process by module name, returns PID of the process
Process.whereis(:logger_sup)
Process.whereis(IEx.Pry)

# list all pids from alive process
Enum.map(Process.list, fn process -> Process.whereis(process))

# return PID from group leader process
Process.group_leader

# sleep process by 4000 miliseconds
Process.sleep(4000)

# Tells whether the given process is alive on the local node.
Process.alive?(pid)
```

## Synchronous passing messaging on top asynchronous
```elixir
# execution of the query
run_query = 
  fn query_def ->
    Process.sleep(2000)
    "#{query_def} result"
  end

async_query = 
  fn query_def ->
    caller = self() # store the pid of the calling process
    spawn(fn ->
      send(caller, {:query_result, run_query.(query_def)})
    end)
  end

# run all queries concurrently and store in the mailbox of the caller process
Enum.each(1..5, &async_query.("query #{&1}"))


# Get result from mailbox process, in this case the shell process
get_result = 
  fn ->
    receive do
     {:query_result, result} -> result
    end
  end

results = Enum.map(1..5, fn _ -> get_result.() end)
```

```elixir
1..5
  |> Enum.map(&async_query.("query #{&1}"))    
  |11> Enum.map(fn _ -> get_result.() end)
```


Exercise DatabaseServer in Iex
```elixir
Enum.map(1..100, fn _ -> DatabaseServer.start() end)

Enum.each(
  1..50,
  fn query_def ->
    server_pid = Enum.at(pool, :rand.uniform(100) - 1)
    DatabaseServer.run_async(server_pid, query_def)
  end
)

Enum.map(1..50, fn _ -> DatabaseServer.get_result() end)
```
