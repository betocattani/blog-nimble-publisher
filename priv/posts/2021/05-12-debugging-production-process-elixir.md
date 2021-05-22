%{
  title: "Debbuging production process in Eliir",
  author: "Luiz Cattani",
  tags: ~w(elixir debugging phoenix process),
  description: "This is my notes of the talk from Saša Jurić explaning how to debug a process in production"
}
---

## Debugging a production consumming of memory

```elixir
 
iex> :observer.start 
# open load charts tab
```

```elixir
# List all pids process with the size of memory consuming
Process.list 
|> Enum.map(&{ Process.info(&1, :reductions), &1})
|> Enum.sort(&>=/2)
|> Enum.take(5) 
|> Enum.each(&IO.inpect/1)

# Find a specify Proces using PID 
Process.find(0, PID, 0)

# Get info about the current_stacktrace form the process
Process.info(pid, :current_stacktrace)

# start tracer
dbg.tracer

# looking only in the call function can be garbage collector, messages been sent etc.
dbg.p(pid, [:call]) 

# Set a pattern, that we are interested in local calls of every single functions of module
# This is going to generate quite a lot ouput, so we make a trick to sleep 1 second and stop the thing.
# have one second trace of the activity of that particular process and we can tell that is 
# just tail recursiving and now we pretty much know what's the problem
# passing three arguments the start, the finish and the accumulated
dbg.tpl(:_, []); :time.sleep(1000); :dbg.stop()

# Stop de Process
# :kill - exit signal, special because it can be trapped, it can be intercepted it's kind of like
# kill -9, but we are bruttaly killing the activity
Process.exit(pid, :kill)
```

```elixir
# Start the observer
:observer.start()
```

```elixir
# Stronger Guarantees / Monitoring during of the activity to kill the process

# Ask to the runtime let me know if that thing dies, no matter how
# Set up a motnior to the process and than we ask to the process to stop politely and now it has a 
# chance to flash its work save some final westwards and then we awainting for the corresponding
# message and now not matter how process dies even if it crashes before my request reachest it 
# we going to get this message, we're going to know that it can stimulate it
# Separete process where we can wait for this down message to arrived  for some time if it doesn't 
# then the other thing refuses to stuff maybe too busy maybe just ignoring our request and we going
# to brutally kill it, we going to send it to kill exit signal and then we waitint again for the down
# because the exit signal itself is asynchronous so we need to wait for the down message, but now we # pretty sure that it's going to happen because it's a brutal termination

# Sketch of how Supervisors terminated children, so polite please start followed by a brutally die
# if you refuse the only difference is it doesn't use a special message it uses an exit signal which is not kill, kind of the core idea.

mref = Process.monitor(pid)

send(pid, :please_stop)

after :time.seconds(5) ->
  Process.exit(pid, :kill)

  receive do
    {:DOWN, ^mrefm, :process, ^pid, _reason} ->
      :ok
  end
end
```

```elixir
 task = Task.async(fn -> ... end)

 case Task.yield(task, :timer.seconds(5000)) do
  {:ok, result} -> do_something(result)
  nil -> Task.shutdown(task, :brutal_kill)
 end
```


# Supervision tree

Supervisor foundations
"Frequent context switching activities as runtimes citizen observable process termiantion
stoppable processes shared-nothing concurrency"


"You should ask not only what your language and your library of frameworks can do for you
but you should first and foremost ask what your runtime can do for you because it can make a really huge difference".

References: 
- https://www.youtube.com/watch?v=5SbWapbXhKo
