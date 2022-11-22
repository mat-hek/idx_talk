---
marp: true
theme: default
style: |
  section {
    background-color: black;
  }
  p, li, :is(h1, h2, h3, h4, h5, h6) {
    color: white
  }
  h1 {
    font-size: 60px;
  }
  h2 {
    position: absolute;
    padding-right: 100px;
    top: 100px;
    font-size: 50px;
  }
  marp-pre {
    background: #ebebe9;
  }
---

# A (more) declarative approach <br> to accessing collections in Elixir

---

```elixir
users_list = [
  %User{id: 1, nick: "Bob", age: 20},
  %User{id: 2, nick: "Eve", age: 30},
  ...
]
```

---

```elixir
users = Map.new(users_list, fn user -> {user.id, user} end)

%{
  1 => %User{id: 1, nick: "Bob", age: 20},
  2 => %User{id: 2, nick: "Eve", age: 30},
  ...
}
```

```elixir
Map.get(users, 1)

%{id: 1, nick: "Bob", age: 20}
```

---

## Will this work?

```elixir
bob = Enum.find(users, fn user -> user.nick == "Bob" end)

```

---

## Nope. This will:

```elixir
{_id, bob} = Enum.find(users, fn {_id, user} -> user.nick == "Bob" end)

```

---

## Speeding it up

```elixir
user_nick_to_id = Map.new(users, fn {id, user} -> {user.nick, id} end)

bob_id = Map.get(user_nick_to_id, "Bob")

bob = Map.get(users, bob_id)
```

---

## Speeding it up

```elixir
user_nick_to_id = Map.new(users, fn {id, user} -> {user.nick, id} end)

bob_id = Map.get(user_nick_to_id, "Bob")

bob = Map.get(users, bob_id)

Map.delete(users, bob_id)

Map.delete(user_nick_to_id, "Bob")
```

---

## Idx

```elixir
Mix.install(idx: "~> 0.1.0")

users = Idx.new(users_list, fn user -> user.id end)

Idx.get(users, 1)

%{id: 1, nick: "Bob", age: 20}
```

---

## This works

```elixir
bob = Enum.find(users, fn user -> user.nick == "Bob" end)

%{id: 1, nick: "Bob", age: 20}
```

---

## ...and can be shorter

```elixir
users = Idx.new(users_list, & &1.id)

bob = Enum.find(users, & &1.nick == "Bob")
```

---

## Speeding it up

```elixir
users = Idx.new(users_list, & &1.id)

users = Idx.create_index(users, :nick, & &1.nick)

bob = Idx.get(users, 1)

%{id: 1, nick: "Bob", age: 20}

bob = Idx.get(users, Idx.key(:nick, "Bob"))

%{id: 1, nick: "Bob", age: 20}
```

---

## Which one is faster?

```elixir
bob = Enum.find(users, fn user -> user.nick == "Bob" end)
```

vs

```elixir
users = Idx.create_index(users, :nick, & &1.nick)

bob = Idx.get(users, Idx.key(:nick, "Bob"))
```

---

## Which one is faster?

```elixir
bob = Enum.find(users, fn user -> user.nick == "Bob" end)
```

vs

```elixir
users = Idx.create_index(users, :nick, & &1.nick, lazy?: true)

bob = Idx.get(users, Idx.key(:nick, "Bob"))
```

---

# Decouple what from how

---
