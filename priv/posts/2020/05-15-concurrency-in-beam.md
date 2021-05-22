%{
  title: "Concurrency in BEAM",
  author: "Luiz Cattani",
  tags: ~w(elixir phoenix process beam otp concurrency),
  description: "This is my notes explaning what is concurrency in BEAM"
}
---

# Concurrency in BEAM

Erlang is all about writing highly available systems, systems that run forever and are
always able to meaningfullly respond to client requests.
To make your system highly available, you have to tackle the following challenges.

- **Fault Tolerance** - Minimize, isolate, and recover from the effects of runtime errors.
- **Scalability** - Handle a load increase by adding more hardware resources without changing
or redeploying the code.
- **Distribution** - Run your system on multiple machines so that others can take over if one
machine crashes.

If you addres these challenges, your system can constantly provide service with minimal downtime and failures.

Concurrency plays an important role in achieving high availability.

In BEAM the unity of concurrency is a `process`, a basic building block that makes it possible to build
scalable, fault tolerance, distributed systems.

A `BEAM process` shouldn't be confused with an OS process. BEAM process are much lighter and cheaper than OS
process.

In production, a typical server system must handle many simultaneous requests from many different clients,
maintain a shared state (for example, caches, user session data and server-wide data), and run some additional
backround jobs. For the server to work normally, all of theses tasks should run reasonably quickly and be reliable.

Because many tasks are pending simultaneously, it's imperative to execute them in parallel as much as possible, thus
taking advantage of all available CPU resources.
It's extremely bad if the lengthy processing of one request blocks all other pending requests and background jobs.
Such behaviour can lead to a constant increase in the request queue, and the system can become unresponsive.

Moreover tasks should be as isolated from each other as possible. You don't want an unhandled exception in on request
handler to crash another unrelated request handler, a background job or, especially, the entire server.
You also don't want a crashing task to leave behind an inconsistent memory state, which might later compromise another task.

That's exacltly what the `BEAM` concurrency model does for us. Process help us run things in parallel, allowing us to 
achieve `scalability`, the ability to localize exceptions and recover from them, you can implement a system that truly
never stops, even when unexpected errors occur.

Process also ensure isolation, which in turn gives us `fault-tolerance` the ability to localize exceptions and recover
from them, you can implement a system that truly never stops, even when unexpected errors occur.

In BEAM, a `process is a concurrent thread of execution`. Two process `run concurrenlty` and `may therefore run in parallel`, 
assuming at `least two CPU cores are available`.

Unlike OS process or threads, BEAM process are lightweight concurrent entities handled by the VM, which uses its own 
scheduler to manage their concurrent execution.


**BEAM as a single OS process, using a few threads to schedule a large number of processes.**
CPU --------------- CPU -------------- CPU ------------- CPU -------------
                          OS Process

                             BEAM

  OS thread        OS thread          OS thread        OS thread
     |                 |                  |                |
  Scheduler        Scheduler          Scheduler        Scheduler
     |                 |                  |                |
   Process          Process             Process          Process
   Process          Process             Process          Process
   Process          Process             Process          Process
   Process          Process             Process          Process

By default BEAM use as many schedulers as there are CPU cores available, for example, on a `quad-core machine`,
four schedulers are used.

Each scheduler run in its own thread, and the entire VM runs in a single OS process.

A scheduler is in charge of the interchangeable execution of process. Each process gets an execution time slot,
after the time is up, the running process is preempeted and the next one takes over.

Process are light, it takes only a couple of microseconds to create a single process, and its initial memory
footprint is a few kilobytes.
By comparison, OS Threads usually use a couple of megabyts just for the stack. Therefore you can create a large number
of process, the theoretical limit imposed by the VM is roughly 134 milion.

This feature `can be exploited in server side systems` to manage `various tasks that should run simultaneously`. Using 
`dedicated process for each task`, you can `take advantage of all available CPU cores and parallelize the work` as much as
possible.

Moreover, running `tasks in different process` improves the `server's reliability` and `fault-tolerance`. BEAM process are
`completly isolated`, they `share no memory`, and a `crash of one process won't take down other processes`.

In addition, `BEAM provides a means to detect process crash` and do something about it, such `as restarting the crashed process`.

All this makes it easier to `create systems that are more stable and can gracefully recover from unexpected errors`, which inevitably occur in production.

Finally, `each process can manage some state` and `can receive messages from other processes to manipulate or retrieve that state`.

As you know `data in Elixir is immutable`, to keep it alive you `have to hold on to it`, constantly `passing the result of one function to another`. A process` can be considered a container of this data`. A `place where an immutable structure is stored and kept alive` for a longer time, possibly forever

There's more `concurrency than parallezation` of the work

# Working with process

The `benefits of process` are most obvious when you want to `run something conrrently and paralleize` wht work as much as possible.

## Concurrency vs. Parallelism
`Concurrency doesn't necessarily imply parallelism`. Two concurrent things have independent execution contexts, but this
doesn't mean they will run in parallel. If you run two CPU-bound concurrent tasks and you only have on CPU core, parallel
execution can't happen. `You can achieve parallelism by adding more CPU cores` and relying on an efficient concurrent framework.
But you should be aware that concurrency itself doesn't necessarily speed things up.

Simulation of a long-running database query
```elixir
run_query = 
  fn query_def ->
    Process.sleep(2000) # simulate a long-running operation
    "#{query_def} result"
  end
```
When you call the `run_query` lambda, the shell is blocked until the lambda is done.

```elixir
iex> run_query.("query 1")
"query 1 result" # Two seconds later
```

If you run `five queries`, it `will take 10 seconds` to get all results
```elixir
iex> Enum.map(1..5, &run_query.("query #{&1}"))
["query 1 result", "query 2 result", "query 3 result", "query 4 result", "query 5 result"]  # Ten seconds later
```

Obviously, `this is neither performant nor scalable`, assuming that the queries are already optimized, the only thing
`you can do to try to make things faster is to run the queries concurrenlty`. `This won't speed up the running time of a single query`, but the total time required `to run all the queries should be much less`. In the BEAM world, to
`run something concurrently, you have to create a separate process`.

### Creating a process

To create a process, you can use the auto imported `spawn/1` function:
```elixir
iex> spawn(fn ->
  expression_1
  ...
  expression_2
  ...
end)
```

The function `spawn/1` takes a zero-arity lambda that `will run in the new process`.
After the process is created, `spawn immediately returns`, and the `caller process's execution continues`.
The provided lambda is executed in the new process and therefore runs concurrently.
After the lambda is done, the spawned process exists, and its memory is released.


```elixir
iex> spawn(fn -> IO.puts(run_query.("query 1")) end)
#PID<0.48.0>  Immediately returned

result of query 1 # Printed after 2 seconds
```
The call to `spawn/1` returns immediately, and `you can do something else in the shell` while the query runs. This is
happen because you call the `IO.puts` from a separate process.

The returns of spawn `#PID <0.48.0>` is the identifier of the created process, often called a PID. Useful to communicate
with the process.

```elixir
iex> async_query =
      fn query_def ->
        spawn(fn -> IO.puts(run_query.(query_def)) end)
      end

iex> async_query.("query 1")
```

This code demostrates an important technique: `passing data to the created process`.
Notice that `async_query` takes one argument and `binds` it to the `query_def variable`.
This `data is then passed to` the newly created process `via the closure mechanism`.
The `inner lambda the one that runs in a separate process`, references the variable `query_def` from the
outer scope. `This results in cross-process data passing`, the `contents` of `query_def` are passed from
the main process to the newly created one.
When it's passed to another process, the data is `deep-copied`, because `two processes can't share any memory`.

NOTE: In BEAM everything runs in a process. This also holds for the interactive shell. All expressions in you enter
in `iex` are executed in a single shell-specific process

```elixir
Enum.each(1..5, &async_query.("query #{&1}"))
:ok # returns immediately

result of query 5
result of query 4
result of query 3    # returns after two seconds
result of query 2
result of query 1
```

As expected, the call Enum.each/2 now returns immediately and all the results are printed at practically the same time,
two seconds later, which `is a five-fold improvement over the sequential version`, this is happen because you `run each  computation concurrently`.

Note that because processes run `concurrently`, the `order of execution isn't guaranteed`.

The processes run concurrently, each one printing the result to the screen. At the same time, the `caller process` run
`independently and has no access to any data from the spawned processes`, `processes are completely independent and isolated`.

Often, a simple `fire and forget` concurrent execution, where the `caller process doesn't receive any notification` from the
spawned ones.

### Messaging Passing
In complex systems you often need concurrent tasks to cooperate in some way. You may have
a main process that spawns multiple concurrent calculations, and then you want to handle all
the results in the main process.

Being completely isolated, `processes can't use shared data structures` to exchange knowledegment
Instead, processes communicate via messages.

When process A wants process B to do something, it `sends an asynchronous message` to B. The 
content of the message is an `Elixir Term`.

Sending a message amounts to `storing into` the receiver's `mailbox`. The caller then continues
with its own execution, and the `receiver can pull the message` in at any time and process it in 
some way. Because processes can't share memory, a message is deep-copied when it's sent.


### Mailbox
The process `mailbox is a FIFO queue` limited only by the available memory. The `receiver consumes messages in the order received`, and a message can be `removed from the queue only if it's consumed`.

To send a message to a process, you need to have access to its process identifier (PID).

You can obtain the pid of the current process by calling the auto-imported self/0 function.
```elixir
iex> self()
```

Once you have a receiver's pid, you can send it messages using Kernel.send/2 function:
```elixir
send(pid, {:an, :arbitrary, :term})
```

The consequence of send is that a message is placed in the mailbox of the receiver.

On the receiver side, to pull a message from the mailbox, you have to use the receive expression:
```elixir
receive do
  expresion_1 -> do_something
  expresion_2 -> do_something_else
end
```

The receiver expression works similarly to the case expresion.
It tries to pull one message from the process mailbox, match it agains any of the provided patterns,
and run the corresponding code.

```elixir
iex> send(self(), "a message") # Sends the message

iex> receive do # receives the message
       message -> IO.inspect(message)
     end

"a message"
```

To handle a specific message, you can rely on pattern matching:
```elixir
iex> send(self(), {:message, 1}) # Sends the message

iex> receive do
      {:message, id} ->  # Pattern-matches the message
        IO.puts("received message #{}")
     end

received message 1
```

If there are no message in the mailbox, receive waits indefinitely for a new message to arrive,
you need to manually terminate it.
```elixir
 iex> receive do
      {:message, id} ->
        IO.puts("received message #{}")
     end  # The shell is blocked because the process mailbox is empty
```

The same thing happens if a message can't be matched agains provided pattern clauses:
```elixir
iex> send(self(), {:message, 1})

iex> receive do
      {_, _, _} ->  # This doesn't match the send messagae
        IO.puts("received")
     end  # so the shell is blocked
```

To make the receive not block you can specify the `after clause`, which is executed if a message
isn't received in a given time frame(in milliseconds).
```elixir
iex> receive do
       message -> IO.puts("received")
     after
       5000 -> IO.puts("message not received")
     end
message not received  # Five seconds later
```

The `receive expression` works as follows:

1 - Take the first message from the mailbox
2 - Try to match it against any of the provided patterns, going from the top to bottom.
3 - If a pattern matches the message, run the corresponding code
4 - If no pattern matches, put the message back into the mailbox at the samae position it originally
occupied. Then try the mext message.
5 - If there are no more messages in the queue, wait for a new one to arrive. When a new message arrives, start from step 1, inspecting the first message in the mailbox.
6 - If the after clause is specified and no message is mateched in the given amout of time, run the code after block.

The result of receive is the `result of the last expression` in the appropriate cluase
```elixir
iex> send(self(), {:message, 1}) # Sends the message

iex> receive_result = 
      receive do
        {:message, x} ->
          x + 2 # the result of receive
      end

iex> IO.inspect(receive_result)
3
```

# Synchronous Sending
The basic message-passing mechanism is the asynchronous "fire and forget" kind. A process
sends a message and then continues to run.

Sometimes a callers needs some kind of response from the receiver. There's no special language
construct for doing this, instead `you must program both parties to cooperate using the basic asynchronous messaging` facility.

The` caller must include its own pid in the message` contents and then wait for a response from 
the receiver. The `receiver uses the embedded pid to send the response to the caller`.

Synchronous send and receive implemented on top of the asynchronous messages.
```
SENDER ----{sender_pid}-------> RECEIVER 
RECEIVER --Response to sender_pid -----> SENDER
```

```elixir
run_query = 
  fn query_def ->
    Process.sleep(2000)
    "#{query_def} result"
  end

async_query =
  fn query_def ->
    caller = self()
    spawn(fn ->
      send(caller, {:query_result, run_query.(query_def)})
    end)
  end

Enum.each(1..5, &async_query.("query #{&1}"))

get_result =
  fn ->
    receive do
      {:query_result, result} -> result
    end
  end

results = Enum.map(1..5, fn _ -> get_result.() end)
```

In this code you first store the pid of the calling process to a distinct caller variable.
This is necessary so the worker process (the one doing the calculation) can know the pid of the process that should receive the response.

This run all the queries concurrently, and the result is stored in the mailbox of the caller proces.
```elixir
Enum.each(1..5, &async_query.("query #{&1}"))
```

The caller process is neither blocker nor interrupeted while receiving messages. Sending a message doesn't distrub the receiving process in any way. The only thing affected is the content of receiving process's mailbox.

It's also worth pointing out that `results arrive in a nonderterministic order`, because all `computations run concurrently`, `it's not certain` in which order they'll finish.

Parallel map technique that can be used to process a larger amount of work in parallel and the collect the results into a list.
```elixir
iex> 1..5 |>
iex> Enum.map(&async_query.("query #{&1}")) |>
iex> Enum.map(fn _ -> get_result.() end)
```

## Stateful Server Processes
Spawning processes to perform one-off tasks isen't the only use case for concurrency. In Elixir, it's common to create `long-running processes that can respond to various messages`. Such processes
can `keep their internal state`, which `other processes can query or even manipulate`.

In this sense, `stateful server processes resemble objects`, they `maintain state and can interact with other processes via messages`. But a process is concurrent, so multiple server processes can run in parallel.

## Server Processes
A `Server Process` is as informal name for a process that `runs for a long time or forever` and can handle various (messages).

To make a process run forever you have to use `endless tail recursion`. If the last thing a function does is call another function (or itself), a simple jump takes place instead of a stack push.

```elixir
defmodule DatabaseServer do
  def start do
    spawn(&loop/0) # Starts the loop concurrently
  end

  defp loop do
    receive do
      #...       Handles one message
    end

    loop() # Keeps the loop running
  end
end
```
The function `start/0` is called `interface function` that's used by clients to start the server process. When start/0 is called, it creates the long-running process that runs forever. 
This is ensured in the `loop/0` function, which waits for a message, handles it, and finally calls itself, thus ensuring that the process never stops.

This loop isn't CPU-instensive, waiting for a message puts the process in a suspendend state and 
doesn't waste CPU cycles.

The function in this module run in differente processes. The function start/0 is callend by clients and runs in a client process.
The private function loop/0 runs in the server process. It's normal to have different functions frmo the same module running in different processes.

When implementing a server process, it's make sens to pull all of its code in a single module.
The functions of this module generally fall into two categories: Interface and Implementation.

`Interface functions`: are public and are executed in the called process. They hide details of process creation and the communication protocol.

`Implementation functions`: are usually private and run in the server processes

```elixir
defmodule DatabaseServer do
  def start do
    spawn(&loop/0) # Starts the loop concurrently
  end

  defp loop do
    receive do
      {:run_query, caller, query_def} ->
        send(caller, {:query_result, run_query(query_def)}) # runs the query and sends the response to the caller
    end

    loop()
  end

  def run_query(query_def) do
    Process.sleep(2000)        # query execution
    "#{query_def} result"
  end

  def run_async(server_pid, query_def) do
    send(server_pid, {:run_query, self(), query_def})
  end

  def get_result do
    receive do
      {:query_result, result} -> result
    after
      5000 -> {:error, :timeout}
    end
  end
end
```

Exercising the DatabaseServer Process
```elixir
server_pid = DatabaseServer.start()

DatabaseServer.run_async(server_pid, "query 1")
DatabaseServer.get_result()
"query 1 result"

DatabaseServer.run_async(server_pid, "query 2")
DatabaseServer.get_result()
"query 2 result"
```

Executing multiple queries in the same process, the server process continues running after a message is received.

Because communnication details are wrapped in functions, the client isnt't aware of them, intead it communicates with the process with plain functions.

### Server Process are sequential
<!-- todo -->

### Keeping a process state
<!-- todo -->

### Process mutable state
<!-- todo -->

### Building a Generic server process

No matter what kind of server process you run, you'll always need to do theses tasks:

- Spawn a separate process
- Run an inifinite loop in the process
- Maintain the process state
- React to messages
- Send a response back to the caller

So it's worth moving this code to a single `place`, `concrete implementations` can then `reuse` this code and focus on their specific needs.

#### Plugging in with modules

The `generic code` will perfom various tasks to server process, for example the generic code will spawn a process,
but the `concrete implementation` must determine the initial state.

Or the generic code will recieve a message and optionally send the response, but the concrete implementaiton must 
decide how the message in handled and what the response is.

In other words the `generic code drives the entire process`, and `specific implementation must fill in the missing pieces`.
Therefore, you `need a plug-in mechanism` that lets the `generic code call into the concrete implementation` when a specific
decision needs to be made.

The simplest way to do this is to use `Modules`,
Modules name is an `atom`, you can store that atom in a variable and use the variable later to invoke functions on the module.

```elixir
iex> some_module = IO   # Store a module atom in a variable

iex> some_module.puts("Hello")   # Dynamic invocation
Hello
```

You can use this feature to provide `callback hooks` from the generic code
1. Make the generic code accept a plug-in module as the argument. That module is called a `callback module`.
2. Maintain the module atom as part of the process state.
3. Invoke `callback-module` functions when needed.

For this to work, a callback module must `implement` and `export a well-defined set of functions`.

Generic code
```elixir
defmodule ServerProcess do
  def start(callback_module) do
    spawn(fn -> 
      initial_state = callback_module.init()
      loop(callback_module, initial_state)
    end)
  end
end
```