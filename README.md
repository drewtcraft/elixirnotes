# Elixir Notes

### Agent
```elixir
# start agent with empty kwlist state
pid = Agent.start(fn -> [] end)

# get some piece of agent state
Agent.get(pid, fn s -> Keyword.get(s, :registrations, []) end)

# update agent state and retrieve value
Agent.get_and_update(pid, fn s ->
  new_state = [] # calc new state
  {:return_value, new_state}
end)

# see also: Agent.update
```

### Comprehensions
```elixir
def get_combinations(tops, bottoms, opt \\ []) do
  for t <- tops,
      b <- bottoms, # zip tops and bottoms
      # filters below
      t[:base_color] != b[:base_color], 
      b[:price] + t[:price] <= Keyword.get(opt, :maximum_price, 100) do
    {t, b}
  end
end
```

### Files
```elixir
def read_emails(path) do
  # open file, second argument has options for how to interact
  case File.open(path, [:read, :utf8]) do
    {:ok, pid} ->
      case IO.read(pid, :eof) do
        :eof -> [] # case when end of file is reached (or file is empty)

        data -> # split file output on newlines
          data
          |> String.split("\n")
          |> Enum.filter(fn s -> s != "" end)
      end

    {:error, _} -> []
  end
end

def open_log(path) do
  case File.open(path, [:write]) do
    {:ok, pid} -> pid
  end
end

def log_sent_email(pid, email) do
  # use IO module to interact with process that is managing file
  IO.puts(pid, email)
end

def close_log(pid) do
  File.close(pid)
end
```

### Processes
```elixir
defmodule TakeANumber do
  @spec start(initial_state :: integer()) :: pid()
  def start(initial_state \\ 0) do
    spawn(fn -> loop(initial_state) end)
  end

  @spec loop(state:: integer()) :: nil
  def loop(state) do
    receive do
      {:report_state, pid} ->
        send(pid, state)
        loop(state)

      {:take_a_number, pid} ->
        send(pid, state + 1)
        loop(state + 1)

      :stop -> :ok # returning :ok stops execution??

      _ -> loop state # always call loop to keep process alive
    end
    
  end
end
```

### Protocols
```elixir
defmodule RPG do
  defmodule Character do
    defstruct health: 100, mana: 0
  end

  defmodule LoafOfBread do
    defstruct []
  end

  defmodule ManaPotion do
    defstruct strength: 10
  end

  defmodule EmptyBottle do
    defstruct []
  end

  defprotocol Edible do
    def eat(item, character)
  end

  defimpl Edible, for: RPG.LoafOfBread do
    def eat(_, character) do
      {nil, %RPG.Character{character | health: character.health + 5}}
    end
  end    

  defimpl Edible, for: RPG.ManaPotion do
    def eat(%RPG.ManaPotion{strength: strength}, character) do
      {%RPG.EmptyBottle{}, %RPG.Character{character | mana: character.mana + strength}}
    end
  end
end
```

### Spec (typing, types)
```elixir
@type address_map :: %{street: String.t(), postal_code: String.t(), city: String.t()}
@type address_tuple:: {street :: String.t(), postal_code :: String.t(), city :: String.t()}

# union type
@type address :: address_map() | address_tuple()

# spec a method
@spec blanks(n :: non_neg_integer()) :: String.t()
def blanks(n) do
  String.duplicate("X", n)
end

```

### Tuple
Extract via pipe
```elixir
{:heyo, "wahoo", 42}
|> then(&elem(&1, 2)) # &1 refers to pipe argument
```

### Error Handling
Catch and return tuples.
```elixir
def do_something(stack, operation) do
  try do
    {:ok, operation.(stack)}
  rescue
    e -> {:error, Exception.message(e)}
  end
end
```
