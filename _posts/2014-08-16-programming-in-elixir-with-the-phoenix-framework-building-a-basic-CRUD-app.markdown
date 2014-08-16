---
layout: post
title:  "Programming in Elixir with the Phoenix Framework - Building a basic CRUD app"
categories: jekyll update
---
Elixir is **hip**. I want to be hip. So I've been giving elixir a try over the last week. So far I'm very pleased with it overall and look forward to learning it more in-depth.

What finally caused me to take the jump was a framework called [Phoenix](https://github.com/phoenixframework/phoenix).  It looks similar in many ways to [Rails](http://rubyonrails.org/).

## Installing Phoenix
Installing Phoenix is actually pretty easy, and you can find the full docs [here](https://github.com/phoenixframework/phoenix#setup)

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

The `web` folder has things that might already be familiar if you've done any web development before.

- MCV: `controllers`, `models`, `views` (view objects / presentation layer - which is awesome), `templates`
- `router.ex` which is similar to the `routes.rb` file in rails
- `channels` which we won't be covering here, but [here is an example of them](https://github.com/chrismccord/phoenix_chat_example)


<br />


## Setting up the Database

First thing we need to do is to add the dependencies for [`postgrex` and `ecto`](https://github.com/elixir-lang/ecto) so we can interact with Postgres in our app.

Add this to the `mix.exs` file

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

- repo object
- changing supervisor file
- adding in a migration

## Adding a simple Welcome Page
- route
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
