#

A Taste of Elixir

# Elixir

* Functional
* José Valim (Rails Core)
* Erlang VM
* Elixir/Erlang both compile to BEAM

# Installing

* Install Erlang from erlang-solutions.com
* Install Elixir from source (Homebrew or Github)

#

```elixir
def sum(a, b) do
  a + b
end
```

#

```elixir
def sum(a, b) do
  a + b
end

sum(1, 2) # => 3
```

#

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end

Math.sum(1, 2) # => 3
```

# Data Types

```elixir
1          # integer
0x1F       # integer
1.0        # float
:atom      # atom (symbol)
{1,2,3}    # tuple
[1,2,3]    # list (array)
<<1,2,3>>  # binary (string)
```

# Commands

* elixir  (*.exs)
* elixirc (*.ex)
* mix     (mix.exs)
* iex

# Code!

* iex>
* Pattern Matching
* Modules & Functions
* DocTests

# Recursion

```elixir
defmodule Math do
  def sum(list) when is_list(list) do
    do_sum list, 0
  end

  defp do_sum([head|tail], total) do
    total = total + head
    do_sum tail, total
  end

  defp do_sum([], total), do: total
end
```

# Recursion

```elixir
sum [1,2,3]
do_sum [1|[2,3]], 0
do_sum [2|[3]], 1
do_sum [3|[]], 3
do_sum [], 6
```

# Enumerator

```elixir
Enum.reduce [1,2,3], fn(a, b) -> a + b end
```

# Enumerator

```elixir
Enum.reduce [1,2,3], fn(a, b) -> a + b end

Enum.reduce [1,2,3], &1 + &2
```

# Pipeline Operator

```elixir
parse(read(config_file))
```

# Pipeline Operator

```elixir
parse(read(config_file))

config_file |> read |> parse
```

# Macros

```elixir
if true do
  "yay"
end

if true, do: "yay"
```

# Macros

```elixir
if true, do: "yay"

defmacro if(condition, clauses) do
  do_clause = Keyword.get(clauses, :do, nil)
  else_clause = Keyword.get(clauses, :else, nil)

  quote do
    case unquote(condition) do
      _ in [false, nil] -> unquote(else_clause)
      _                 -> unquote(do_clause)
    end
  end
end
```

# Pattern-Matching

```elixir
case File.open(path) do
  { :ok, pid } -> parse_file pid
  { :error, error } -> IO.puts error
end
```

# Records

```elixir
defrecord User, name: nil

user = User.new name: "Justin"
user = User[name: "Justin"]

user.name # => "Justin"
```

# Protocols & Implementations

# Protocols & Implementations

¯\_(ツ)_/¯

# Actors

```elixir
def counter(total) do
  receive do
    n -> total = total + n
  end

  counter total
end

pid = spawn MyModule, :counter, [0]
pid <- 123
```

# OTP

* Super-Actors!
* GenServer
* Supervision Trees
* Call/Cast
* State

# Mix

```sh
mix new hello_world
```

# Dynamo

github.com/elixir-lang/dynamo

# Dynamo

```elixir
get "/" do
  conn = conn.assign(:title, "Welcome!")
  render conn, "index.html"
end

get "/hello/world" do
  conn.resp 200, "Hello world"
end
```
# Resources

* elixir-lang.org
* PeepCode: Meet Elixir with José Valim
* Ruby Rogues #114 with José Valim
* Programming Elixir by Dave Thomas
* #elixir-lang

# github.com/justincampbell

* elixir-examples
* url-shortener-elixir

# Thanks!

@justincampbell
