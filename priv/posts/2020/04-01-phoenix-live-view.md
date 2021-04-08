%{
  title: "Phoenix Live View",
  author: "Luiz Cattani",
  tags: ~w(phoenix liveview elixir),
  description: "This is my notes about Phoenix LiveView"
}
---

# Live View

### LiveView Loop
- LiveView will receive events, like link clicks, key presses, or page submits
- Based on those events, we will change your state
- After change the state, LiveView will re-render only the portions of the page 
that are affected by changed state
- Afte rendering, LiveView again waits for events, and we go back to the top.

## Live View LifeCycle
LiveView manage state in structs called `socket`.
The module Phoenix.LiveView.Socket creates these structs.

What lives and you see on these socket structs as a variable or argument whithin a LiveView, recognized
it as the data that constitutes the live view's state.

Socket struct how LiveView represents state
The Socket struct contains all the data that Phoenix needs to manage a LiveView connection.
```elixir
iex> Phoenix.LiveView.Socket__struct__
#Phoenix.LiveView.Socket<
  assigns: %{},
  changed: %{},
  endpoint: nil,
  id: nil,
  parent_pid: nil,
  root_pid: nil,
  router: nil,
  view: nil,
  ...
>
```

## Render the LiveView

The lifecyle begins in the Phoenix Router `live route`
A `live route` maps a incoming request to a specified live view so that request to that endpoint will start up the live view process. The Process initialize the `live view's state` by setting up the `socket` in a function called `mount/3`, then the LiveView will render that state in some markup for the client.

```
ROUTER ----> Index.mount ----> Index.render
```

The `mount` function is responsible for establishing the initial state for the LiveView by 
populating the socket assigns.
```elixir
def mount(_params, _session, socket) do
  {:ok, assign(socket, query: "", results: %{})}
end
```

The `socket` contains the data representing the state of the LiveView and the `:assigns` key, 
referred to the `socket assigns`, holds custom data.

LiveView helper function `assign/2` simply adds `key/value` pairs to a given socket assigns `query: ""` and `results: %{}`.

Initial socket struct
```elixir
%Socket{
  assigns: %{
    query: "",
    results: %{}
  }
}
```

The mount function returns a result tuple, the first element can be `:ok` or `:error`, the
second element has the initial contents of the socket.

After the initial `mount` finishes, LiveView then passses the value of the `socket.assigns` key to the `render` function.
If there's no render function, LiveView looks for a `template` to render based on the name of the view.


```html
<input
  type="text"
  name="q"
  value="<%= @query %>" placeholder="Live dependency search" list="results"
  autocomplete="off" />
```

LiveView will populate the `<%= @query %>` expression with the value of `socket.assigns.query`, which we set in mount.

When liveView finishes calling `mount` and then `render`, it returns the initial web page to the browser.

For tradittionals connections the flow ends here, but for LiveView just getting started, after the initial web page is rendered in the browser, LiveView establishes a `persistent WebSocket` connection and awaits events over the connection.

## Run the LiveView Loop



## Defining a new LiveView
Add a new route to the web scope
```elixir
live "/guess", GuessLive
```

Create a new module to Guess Live
```elixir
defmodule MyAppWeb.GuessLive end
  use MyAppWeb, :live_view
end
```

the macro `use MyAppWeb, :live_view` makes available a special sigil, we'll use whitin the `render` function


Create a new module to Guess Live
```elixir
defmodule MyAppWeb.GuessLive end
  use MyAppWeb, :live_view

  # a mount function assign a socket and add the key/pairs value to socket struct
  def mount(_params, _session, socket) do
    {:ok, assign(socket, score: "", message: %{})}
  end
end
```

Defining a explicit `render/1` function that takes an argument from `socket.assigns`
```elixir
defmodule MyAppWeb.GuessLive end
  use MyAppWeb, :live_view

  # a mount function assign a socket and add the key/pairs value to socket struct
  def mount(_params, _session, socket) do
    {:ok, assign(socket, score: "", message: %{})}
  end

  def render(assigns) do
    ~L"""
    <h1>Your score: <%= @score%></h1>
    <h2>
      <%= @message %>
    </h2>

    <h2>
      <%= for n <- 1..10  do %>
        <a href="#" phx-click="guess" phx-value-number="<%= n %>"><%= n %></a>
      <% end %>
    </h2>
    """
  end
end
```
## Mount
## Render
## Handle Event

access the page http request
call mount to create ea session and establish a connetion with websocket
render

```html
<!-- event_name guess -->
<a href="#" phx-click="guess">

<!-- name of attribute last word  -->
<a href="#" phx-click="guess" phx-value-number="<%= n %>"><%= n %></a>
```

## Heartbeat
After the first request to the server phoenix established a websocket connection with the client using the status code `101 (Switching protocols)`, and in a time interval the client send a heartbeat to the server to verify if exist diff to change on the frontend.
```bash
[null,"1","phoenix","heartbeat",{}]
[null,"1","phoenix","phx_reply",{"response":{},"status":"ok"}]	

Diff with response from backend
["4","2","lv:phx-FnKuV7ABMYNOMQ-j","event",{"type":"click","event":"guess","value":{"number":"10"}}]	
["4","2","lv:phx-FnKuV7ABMYNOMQ-j","phx_reply",{"response":{"diff":{"2":{"0":"-1","1":"Your guess: 10. Wrong. Guess again. ","2":"2021-04-04 14:46:08.122785Z"}}},"status":"ok"}]	
```



## CRC Constructor / Reducers / Converters

The main sections of Phoenix Pipeline are:

connection_from_request
|> endpoint
|> router
|> custom_application