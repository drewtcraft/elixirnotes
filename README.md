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
