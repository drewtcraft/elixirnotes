# Elixir Notes

### Agent
TODO: why use agents instead of bare processes or genservers?
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

### Case
```elixir
def from_milliliter({_, amt}, unit) do
  case unit do
    :milliliter -> {unit, amt}
    :cup -> {unit, amt / 240}
    :fluid_ounce -> {unit, amt / 30}
    :teaspoon -> {unit, amt / 5}
    :tablespoon -> {unit, amt / 15}
    _ -> {unit, amt}
  end
end

# TODO can pattern match in here
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

### Custom Exceptions
```elixir
defmodule RPNCalculator.Exception do
  defmodule DivisionByZeroError do
    defexception message: "division by zero occurred"
  end

  defmodule StackUnderflowError do
    defexception message: "stack underflow occurred"

    # handle different values passed into exception
    @impl true
    def exception(value) do
      cond do
        is_binary(value) ->
          %StackUnderflowError{}
          |> Map.update!(:message, fn m ->
            m <> ", context: " <> value
          end)

        true ->
          %StackUnderflowError{}
      end
    end
  end
end

# usage
raise RPNCalculator.Exception.StackUnderflowError
raise RPNCalculator.Exception.StackUnderflowError, "added exception context"
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

### Guards
```elixir
defmodule GuessingGame do
  def compare(_) , do: "Make a guess"
  def compare(_, guess) when guess == :no_guess , do: "Make a guess"
  def compare(secret_number, guess) when secret_number == guess, do: "Correct"
  def compare(secret_number, guess) when abs(secret_number - guess) == 1, do: "So close"
  def compare(secret_number, guess) when guess > secret_number, do: "Too high"
  def compare(secret_number, guess) when guess < secret_number, do: "Too low"
end
```

### Pattern Matching
```elixir
def to_milliliter({:milliliter, amt}), do: {:milliliter, amt}
def to_milliliter({:cup, amt}), do: {:milliliter, amt * 240}
def to_milliliter({:fluid_ounce, amt}), do: {:milliliter, amt * 30}
def to_milliliter({:teaspoon, amt}), do: {:milliliter, amt * 5}
def to_milliliter({:tablespoon, amt}), do: {:milliliter, amt * 15}
```

### Processes
unlinked
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

linked
```elixir
def start_reliability_check(calculator, input) do
  pid = spawn_link(fn -> calculator.(input) end)
  %{input: input, pid: pid}
end

def await_reliability_check_result(%{pid: pid, input: input}, results) do
  # receive will only match messages that fit these patterns
  receive do
    {:EXIT, ^pid, :normal} ->
      Map.put(results, input, :ok)

    {:EXIT, ^pid, {e, _}} when is_exception(e) ->
      Map.put(results, input, :error)

    {:EXIT, ^pid, _} ->
      Map.put(results, input, :error)
  after # timeout 100ms
    100 -> Map.put(results, input, :timeout)
  end
end

def reliability_check(calculator, inputs) do
  # required to capture the {:EXIT, ...} tuples in the receive block
  old_value = Process.flag(:trap_exit, true)

  res =
    inputs
    |> Enum.map(&start_reliability_check(calculator, &1))
    |> Enum.reduce(%{}, &await_reliability_check_result/2)

  Process.flag(:trap_exit, old_value)
  res
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

### With
```elixir
def get_new_passport(now, birthday, form) do
  with {:ok, ts} <- enter_building(now),
       {:ok, manual_fn} <- find_counter_information(now),
       counter <- manual_fn.(birthday),
       {:ok, checksum} <- stamp_form(ts, counter, form) do
    {:ok, get_new_passport_number(ts, counter, checksum)}
  else
    {:coffee_break, _} -> {:retry, NaiveDateTime.add(now, 15, :minute)}
    err -> err
  end
end
```
