---
layout: post
title:  "Programming in Elixir with the Phoenix Framework - Building a basic CRUD app"
categories: jekyll update
---
Elixir is **hip**. I want to be hip. So I've been giving elixir a try over the last week. So far I'm very pleased with it overall and look forward to learning it more in-depth.

What finally caused me to take the jump was a framework called [Phoenix](https://github.com/phoenixframework/phoenix).  It looks similar in many ways to [Rails](http://rubyonrails.org/).

__TL;DR:__ I _really_ like Phoenix and I plan to use it for more side projects.


<br />


## Installing Phoenix
Installing Phoenix is actually pretty easy, and you can find the full docs [here](https://github.com/phoenixframework/phoenix#setup).

```bash
mix phoenix.new phoenix_crud ~/code/phoenix_crud
mix do deps.get, compile
mix phoenix.start
```
This will give you a full project structure to get you started.


<br/>


#### Project folder
---
![](/images/phoenix_folder_structure.png)

Most of these are pretty self explanatory, but I thought I'd point out a few things of note.

- `deps` is quite similar to `vendor` in rails apps.
- `mix.exs` is the application config file / `Gemfile` where you can specify dependencies.
- `web` is the primary place that we will be working with to for this tutorial.


<br/>


#### Web folder
---
![](/images/phoenix_web_folder_structure.png)

The `web` folder has things that might already be familiar if you've done any previous web development.

- MVC: `models`, `views` (view objects / presentation layer - which is awesome), `controllers` , `templates`
- `router.ex` which is similar to the `routes.rb` file in rails
- `channels` which we won't be covering here, but [here is an example of them](https://github.com/chrismccord/phoenix_chat_example)


<br />


## Setting up the Database

First thing we need to do is to add the dependencies for [`postgrex` and `ecto`](https://github.com/elixir-lang/ecto) so we can interact with Postgres in our app.

Add this to the `mix.exs` file.

__file__: `mix.exs`

```ex
  def application do
    [
      mod: { PhoenixCrud, [] },
      applications: [:phoenix, :cowboy, :postgrex, :ecto]
    ]
  end

  defp deps do
    [
      {:phoenix, github: "phoenixframework/phoenix"},
      {:cowboy, "~> 1.0.0"},
      {:postgrex, ">= 0.0.0"},
      {:ecto, "~> 0.2.0"}
    ]
  end
```

Then we need to run `mix deps.get` to install the dependencies.


Next we need to add a Repo object to configure `ecto`

__file__: `lib/phoenix_crud/repo.ex`

```ex
  defmodule Repo do
    use Ecto.Repo, adapter: Ecto.Adapters.Postgres

    def conf do
      parse_url "ecto://postgres:postgres@localhost/phoenix_crud"
    end

    def priv do
      app_dir(:phoenix_crud, "priv/repo")
    end
  end
```


Here we are setting the database config for postgres. We're telling it to use a database called `phoenix_crud`. We are also specifying that all the migration files should go into `priv/repo`.


Next we'll change the `supervisor` file to watch this file.

__file__: `lib/phoenix_crud/supervisor.ex`

```ex
  defmodule PhoenixCrud.Supervisor do
    use Supervisor

    def start_link do
      :supervisor.start_link(__MODULE__, [])
    end

    def init([]) do
      # Adding repo to be sent into supervise
      tree = [worker(Repo, [])]
      supervise(tree, strategy: :one_for_one)
    end
  end
```


After we have it setup we can add a migration to create the tables we want.
Ecto comes with a handle generator to create a migration file.

`mix ecto.gen.migration Repo create_users`

This will create a migration file (very similar to how rails would) that we can use to create our tables.

__file__: `lib/phoenix_crud/repo/migrations/20140815053139_create_users.exs`

```ex
  defmodule PingPong.Repo.Migrations.CreateUsers do
    use Ecto.Migration

    def up do
      "CREATE TABLE users(id serial primary key, content varchar(140))"
    end

    def down do
      "DROP TABLE users"
    end
  end
```

Similar to rails, we have an up and down method that get called when we're migrating or rolling back.  Here we're creating a simple `users` table that we will interact with.

At last we finally have our database setup, hurray!

![](/images/excited.gif)


<br />


## Adding a simple Welcome Page

Phoenix comes with a `PageController` and splash page by default, but I think it's a good experiement to go through and do it yourself so you can get a feel for how things tie together.

I like to start creating a route, so let's get to it.

__file__: `web/router.ex`

```ex
  defmodule PhoenixCrud.Router do
    use Phoenix.Router

    plug Plug.Static, at: "/static", from: :phoenix_crud

    scope alias: PhoenixCrud do
      get "/", WelcomeController, :index, as: :root
    end
  end
```

Phoenix gives us a very nice (familiar) DSL to define our routes. This gives us a `root_path` that will trigger the `WelcomeController` `index` action.

Let's go ahead and take a look at that controller.

```ex
  defmodule PhoenixCrud.WelcomeController do
    use Phoenix.Controller

    def index(conn, _params) do
      render conn, "index"
    end
  end
```

The code is quite simple.  I'm really

- controller
- view
- template

## Adding a Page controller (and 404 page)
- hook up normal show route
- hooking up show route to handle unauthorised params

## Adding in a User Resource

## Creating a User Model
- play in console

## Adding in a JSON package


What I really love about Phoenix, or maybe just even Elixir, is even though I have only been using the language for a week with no prior experience with a functional language - I found it extremely easy to dig into the source code of the project and work out most of the things that came up. I think clarity in a framework can lend to a very high userbase and I'm happy to see for at least the moment, that is the case.
