---
layout: post
title:  "Programming in Elixir with the Phoenix Framework - Building a basic CRUD app"
categories: elixir phoenix learning
---
Elixir is **hip**. I want to be hip. So I've been giving elixir a try over the last week. So far I'm very pleased with it overall and look forward to learning it more in-depth.

What finally caused me to take the jump was a framework called [Phoenix](https://github.com/phoenixframework/phoenix).  It looks similar in many ways to [Rails](http://rubyonrails.org/).

__TL;DR:__ I _really_ like Phoenix and I plan to use it for more side projects.

You can view the [github repo here](https://github.com/gogogarrett/phoenix_crud).

- [Installing Phoenix](#installing_phoenix)
- [Setting up the Database](#setting_up_the_database)
- [Adding a simple Welcome Page](#adding_a_simple_welcome_page)
- [Adding a Page controller (and 404 page)](#adding_404_page)
- [Adding a User Resource](#adding_a_user_resource)
- [Summary](#summary)

<br />


## Installing Phoenix<a name="installing_phoenix"></a>
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

- `deps` is managed by mix - and a nice place to find all the 3rd party dependencies of your app. (I found it helpful to dig through some source in here)
- `mix.exs` is the application config file / `Gemfile` where you can specify dependencies.
- `web` is the primary place that we will be working with for this tutorial.


<br/>


#### Web folder
---
![](/images/phoenix_web_folder_structure.png)

The `web` folder has things that might already be familiar if you've done any previous web development.

- MVC: `models`, `views` (view "objects" / presentation layer - which is awesome), `controllers` , `templates`
- `router.ex` which is similar to the `routes.rb` file in rails
- `channels` which we won't be covering here, but [here is an example of them](https://github.com/chrismccord/phoenix_chat_example)


<br />


## Setting up the Database<a name="setting_up_the_database"></a>

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
Ecto comes with a handy generator to create a migration file.

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

Similar to rails, we have an `up` and `down` method that get called when we're migrating or rolling back.  Here we're creating a simple `users` table that we will interact with.

Let's run that migration with `mix ecto.migrate Repo`.

At last we finally have our database setup, hurray!

![](/images/excited.gif)


<br />


## Adding a simple Welcome Page<a name="adding_a_simple_welcome_page"></a>

Phoenix comes with a `PageController` and splash page by default, but I think it's a good experiment to go through and do it yourself so you can get a feel for how things tie together.

I like to start by creating a route, so let's get to it.

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

__file__: `web/controllers/welcome_controller.ex`

```ex
  defmodule PhoenixCrud.WelcomeController do
    use Phoenix.Controller

    def index(conn, _params) do
      render conn, "index"
    end
  end
```

The code is quite simple.  I'm really pleased with the readability of the code. It makes the transfer of knowledge coming from Rails much easier.

Here we're telling it to render the "index" view/template, so let's have a look at those.

__file__: `web/views/welcome_view.ex`

```ex
  defmodule PhoenixCrud.WelcomeView do
    use PhoenixCrud.Views

  end
```

This is all the boilerplate we need in order to get things working.

First, I should say this is a pretty different concept if you're coming straight from the _Rails Way_&#8482;. These views (in my opinion) are similar to the Presenter pattern.  Some might think they just reflect helpers in rails, but I think it's actually a better approach overall.

In Phoenix, the views are responsible for rendering the templates.  You can also create nice helpers here if you wish.

```ex
  def capitalize(string) do
    String.capitalize("abcd")
  end
```

Which could then be used in the templates with `<%= capitalize(@user.content) %>`.


Let's get back to our Welcome page and create a simple template, which uses the [EEx templating framework](http://elixir-lang.org/docs/stable/eex/).

__file__: `web/templates/index.eex`

```html
  <div class="jumbotron">
    <h2>Simple CRUD app using Phoenix and Elixir!</h2>
    <p class="lead">First try at getting a basic crud app up and working with Phoenix.</p>
  </div>
```

This template will be injected into the `templates/layouts/application.html.eex` file where we specifiy using the `<%= @inner %>` tag.  Very similar to `<%= yield %>` in rails applications.

If we browse to our server, we should see the new changes reflected.

![](/images/reflected.gif)


<br />


## Adding a Page controller (and 404 page)<a name="adding_404_page"></a>

Now that we have a better understanding of how to go through creating a basic route I'll be skimming through this part, but I still want to show how to setup a general 404 page.

First let's add the route to our routes file.

__file__: `web/router.ex`

```ex
  defmodule PhoenixCrud.Router do
    use Phoenix.Router

    plug Plug.Static, at: "/static", from: :phoenix_crud

    scope alias: PhoenixCrud do
      get "/", WelcomeController, :index, as: :root
      get "/pages/:page", PageController, :show, as: :page
    end
  end
```

After we add the new route in, let's check out the controller.

__file__: `web/controllers/page_controller.ex`

```ex
  defmodule PhoenixCrud.PageController do
    use Phoenix.Controller

    # pattern match againsts unauthorized params and redirect to 404 page
    def show(conn, %{"page" => "unauthorized"}) do
      conn
      |> assign_layout(:none)
      |> render "unauthorized"
    end

    def show(conn, %{"page" => page}) do
      render conn, "show", page: page
    end
  end
```

This is a very basic example of how powerful Elixir's pattern matching can be harnessed to make your code easier to write, and as a result be easier to read.

Here we are catching when the `page` parameter equals `unauthorized` and assigning a specific layout then rendering an unauthorized (404) page.

The second show action catches the page param and sets it to a local variable `page` to be used inside the action.  We then render a normal `show` view/template and pass `page` as variable to be used in the templates through `@page` (similar to Rails instance variables.)


<br />


## Adding a User Resource<a name="adding_a_user_resource"></a>

I think we're finally familiar enough with Phoenix to a point where we can get started adding in the User CRUD.

Like before, let's start with the routes.

__file__: `web/router.ex`

```ex
  defmodule PhoenixCrud.Router do
    use Phoenix.Router

    plug Plug.Static, at: "/static", from: :phoenix_crud

    scope alias: PhoenixCrud do
      get "/", WelcomeController, :index, as: :root
      get "/pages/:page", PageController, :show, as: :page
      resources "users", UserController
    end
  end
```

This `resource` method gives us a very rails like REST resource of 7 actions.

![](/images/phoenix_routes.png)

__NOTE__: you can run `mix phoenix.routes` to easily see all the routes of your application.


<br />


#### Creating a User Model

Before we get right into the controller, we need to setup the model.

__file__: `web/models/user.ex`

```ex
  defmodule PhoenixCrud.User do
    use Ecto.Model

    validate user,
       content: present()

    schema "users" do
      field :content, :string
    end
  end
```

Here we are including the `Ecto.Model` which will give us all the magic we need in order to get this working. Under the hood it has `Ecto.Model.Schema`, `Ecto.Model.Validations` and will eventually have `Ecto.Model.Callbacks`.

The `validate user` creates a `validate(user)` method that can be used to ensure the model passes these specific validations.

The `schema` matches the database columns so that it can be used for querying.

Let's have a play in the console to see how we can use these objects.

```bash
user = %User{content: "Hello"}
user.content #=> "Hello"

Repo.insert(user) # Saves to the database

user = Repo.get(User, 1) #=> %User{content: "Hello}

user = %{user | content: "Goodbye"}
Repo.update(user) #=> %User{content: "Goodbye"}

Repo.delete(user) # Deleted the object
```

<br />


#### Let's create this CRUD controller! One action at a time.

__file__: `web/controllers/user_controller.ex`

```ex
  defmodule PhoenixCrud.UserController do
    use Phoenix.Controller
    use Jazz
    alias PhoenixCrud.Router
    alias PhoenixCrud.User

    def index(conn, _params) do
      render conn, "index", users: Repo.all(User)
    end
  end
```

Here we start with the `index` action.  This is pretty straight forward.  We render the `index` template and assign `@users` to `Repo.all(User)` which simply just returns all the users in the database.

```ex
  def show(conn, %{"id" => id}) do
    case Repo.get(User, id) do
      user when is_map(user) ->
        render conn, "show", user: user
      _ ->
        redirect conn, Router.page_path(page: "unauthorized")
    end
  end
```

Next we bring in the fantastic `show` action. Here we do a case statement on `Repo.get(User, id)`. This will return a map object from the database if it finds the record.  Otherwise, it will return an errors array, but here we just catch everything else and redirect to the unauthorized (404) page.


```ex
  def new(conn, _params) do
    render conn, "new"
  end
```

This new action is very straightfoward so instead of explaining this I'll cover what the template looks like.

__file__: `web/templates/users/new.eex`

```html
  <h1>New User</h1>

  <form action="/users" method="post">
    <div class="form-group">
      <label for="user[content]">Content</label>
      <input type="text" name="user[content]" class="form-control" />
    </div>
    <button type="submit" class="btn btn-primary">Save</button>
  </form>
```

Here we create a basic html form that `post` to `/users`.  We simply pass back content under a user hash (simply because this is what I'm used to with rails, but could easily be done anyway you please).

This leads us into the controllers `create` action so let's get back to it.

__file__: `web/controllers/user_controller.ex`

```ex
  def create(conn, %{"user" => %{"content" => content}}) do
    user = %User{content: content}

    case User.validate(user) do
      [] ->
        user = Repo.insert(user)
        render conn, "show", user: user
      errors ->
        render conn, "new", user: user, errors: errors
    end
  end
```

Here we create a user instance, validate it and persist it if it passes validations. If it fails, we render the new action again, but this time passing an `@errors` value that can be shown to the user.

```ex
  def edit(conn, %{"id" => id}) do
    case Repo.get(User, id) do
      user when is_map(user) ->
        render conn, "edit", user: user
      _ ->
        redirect conn, Router.page_path(page: "unauthorized")
    end
  end
```

This resembles very closly to the show action, so I don't go into any details.

```ex
  def update(conn, %{"id" => id, "user" => params}) do
    user = Repo.get(User, id)
    user = %{user | content: params["content"]}

    case User.validate(user) do
      [] ->
        Repo.update(user)
        # [g] really hacky way to redirect in the client.. (is there a better way?)
        json conn, 201, JSON.encode!(%{location: Router.user_path(id: user.id)})
      errors ->
        json conn, errors: errors
    end
  end
```

We're almost there! Here we are finding the User from the database.  Then we merge with the new params, validate and update if it passes.

The only downside here is I have not found a nice way to handle the redirection from a PUT or DELETE request.  Rails, and other frameworks, seem to send a GET request and just hi-jack the `_method` param to redirect to the correct action, but I haven't found anything quite like that in Phoenix. So I had to manually send a PUT/PATCH and DELETE request with jQuery, and redirect to the location I pass back via javascript in the client.

__file__: `web/templates/user/edit.ex`

```html
<div class="col-md-12">
  <h3>Edit: <%= @user.content %></h3>

  <div class="actions">
    <form action="/users/<%= @user.id %>" method="post">
      <div class="form-group">
        <input type="text" name="user[content]" value="<%= @user.content %>" class="form-control" />
      </div>
      <button type="submit" class="btn btn-primary">Update</button>
    </form>
  </div>
</div>

<script>
  $("form").on("submit", function(event) {
    event.preventDefault();

    $that = this;

    $.ajax({
      url: $that.getAttribute('action'),
      type: "PUT",
      data: $('form').serialize(),
      success: function(data) {
        window.location = data.location;
      }
    });
  });
</script>
```

On the jQuery ajax `success` I redirect to the location sent back from the controller.

__file__: `web/controllers/user_controller.ex`

```ex
  def destroy(conn, %{"id" => id}) do
    user = Repo.get(User, id)
    case user do
      user when is_map(user) ->
        Repo.delete(user)
        json conn, 200, JSON.encode!(%{location: Router.users_path})
      _ ->
        redirect conn, Router.page_path(page: "unauthorized")
    end
  end
```

The last action!  By now, this should all seem pretty straightforward, but it's just here for completeness.

Just as a recap, the entire controller should look like this now:

```ex
  defmodule PhoenixCrud.UserController do
    use Phoenix.Controller
    use Jazz
    alias PhoenixCrud.Router
    alias PhoenixCrud.User

    def index(conn, _params) do
      render conn, "index", users: Repo.all(User)
    end

    def show(conn, %{"id" => id}) do
      case Repo.get(User, id) do
        user when is_map(user) ->
          render conn, "show", user: user
        _ ->
          redirect conn, Router.page_path(page: "unauthorized")
      end
    end

    def new(conn, _params) do
      render conn, "new"
    end

    def create(conn, %{"user" => %{"content" => content}}) do
      user = %User{content: content}

      case User.validate(user) do
        [] ->
          user = Repo.insert(user)
          render conn, "show", user: user
        errors ->
          render conn, "new", user: user, errors: errors
      end
    end

    def edit(conn, %{"id" => id}) do
      case Repo.get(User, id) do
        user when is_map(user) ->
          render conn, "edit", user: user
        _ ->
          redirect conn, Router.page_path(page: "unauthorized")
      end
    end

    def update(conn, %{"id" => id, "user" => params}) do
      user = Repo.get(User, id)
      user = %{user | content: params["content"]}

      case User.validate(user) do
        [] ->
          Repo.update(user)
          # [g] really hacky way to redirect in the client.. (is there a better way?)
          json conn, 201, JSON.encode!(%{location: Router.user_path(id: user.id)})
        errors ->
          json conn, errors: errors
      end
    end

    def destroy(conn, %{"id" => id}) do
      user = Repo.get(User, id)
      case user do
        user when is_map(user) ->
          Repo.delete(user)
          json conn, 200, JSON.encode!(%{location: Router.users_path})
        _ ->
          redirect conn, Router.page_path(page: "unauthorized")
      end
    end
  end
```

## Summary<a name="summary"></a>

What I really love about Phoenix, or maybe just even Elixir, is even though I have only been using the language for a week with no prior experience with a functional language - I found it extremely easy to dig into the source code of the project and work out most of the things that came up. I think clarity in a framework can lend to a very high userbase and I'm happy to see for at least the moment, that is the case with Phoenix.


[github repo here](https://github.com/gogogarrett/phoenix_crud).

