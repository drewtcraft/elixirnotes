# Elixir Notes

### Error Handling
```elixir
def do_something(stack, operation) do
  try do
    {:ok, operation.(stack)}
  rescue
    e -> {:error, Exception.message(e)}
  end
end
```
