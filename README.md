# ReactPhoenix

[![Build Status](https://travis-ci.org/geolessel/react-phoenix.svg?branch=master)](https://travis-ci.org/geolessel/react-phoenix)
[![Hex.pm](https://img.shields.io/hexpm/v/react_phoenix.svg)](https://hex.pm/packages/react_phoenix)

Functions to make rendering React.js components easy in Phoenix.

Combined with the javascript also included in this package, rendering React
components in your Phoenix views is now much easier. The module was built
with Brunch in mind (vs Webpack). Since Phoenix uses Brunch by default, this
package can make getting React into your application much faster than
switching over to a different system.

Beyond just rendering your React.js components on the client-side, you
can also optionally enable server-side rendering. This allows you to
render all the React components in your controller and output the
resulting html on initial page load (which can help with things like
search engine optimization).

## Note regarding Phoenix 1.3

Although this package works just fine with Phoenix 1.3, the installation
instructions vary slightly from below. If you are using Phoenix 1.2 and its
default directory structure, carry on. If you are using Phoenix 1.3 and its
default directory strucutre, some chanages are required.

While I'm working on updating this README to reflect the different installation
instructions, I have created an example Phoenix 1.3 application for you to
take as an example. I have broken out in each commit the steps required to get
from `mix phx.new` to rendering components both client- and server-side.

You can see that [example Phoenix 1.3 app here](https://github.com/geolessel/react-phoenix-example-1.3).


## Installation in 3 (or 4 [or 5]) EASY STEPS!

This package is meant to be quick and painless to install into your Phoenix
application. It is assumed that you have already brought React into your
application through `npm`.

> Would you rather watch a video? Watch me set it all up from `mix phoenix.new` to rendering
> a React component in 4 minutes [here](https://youtu.be/icwjAbck8yk).

### 1. Declare the dependency

The package can be installed by adding `react_phoenix` to your list of
dependencies in `mix.exs`:

```elixir
def deps do
  [{:react_phoenix, "~> 0.4.3"}]
end
```

After adding to your mix file, run:

```
> mix deps.get
```

### 2. Add the javascript dependency to package.json

In order to correctly render a React component in your view templates, a
provided javascript file must be included in your `package.json` file in
the dependencies section. It might look like this:

```
{
  ...
  "dependencies": {
    "babel-preset-react": "^6.23.0",
    "minions.css": "^0.3.1",
    "phoenix": "file:deps/phoenix",
    "phoenix_html": "file:deps/phoenix_html",
    "react": "^15.4.2",
    "react-dom": "^15.4.2",

    "react-phoenix": "file:deps/react_phoenix" <-- ADD THIS!
  },
  ...
}
```

Then run
```
> npm install
```

### 3. Import and initialize the javascript helper

In your main application javascript file (usually `web/static/js/app.js`), add the
following line:

```javascript
import "react-phoenix"
```

### 4. (optional) Import the module into your views for less typing

If you'd like to just call `react_component(...)` in your views instead of the full
`ReactPhoenix.ClientSide.react_component(...)`, you can import `ReactPhoenix.ClientSide`
into your `web/web.ex` views section. It might look like this:

```elixir
def view do
  quote do
    use Phoenix.View, root: "web/templates"

    import Phoenix.Controller, only: [get_csrf_token: 0, get_flash: 2, view_module: 1]

    use Phoenix.HTML

    import MyPhoenixApp.Router.Helpers
    import MyPhoenixApp.ErrorHelpers
    import MyPhoenixApp.Gettext

    import ReactPhoenix.ClientSide # <-- ADD THIS!
  end
end
```

### 5. (optional) Enable server-side rendering

**As of v0.4, Server-side rendering is currently not working as intended. While I work on returning
this functionality, client-side rendering is working fine. The issue is making sure that each component
that needs to be rendered server-side has ALL the needed javascript libraries included inside the
component file. I'm attempting to find a way to do this with brunch and then make it easy to use
with this library. If you have any good ideas, please let me know.**

If you'd like to enable server-side rendering, there are a small handful of extra
steps you'll need to take to configure your Phoenix app. The documentation on getting
everything set up for that is extensively covered in the
[moduledoc for ReactPhoenix.ServerSide](https://hexdocs.pm/react_phoenix/ReactPhoenix.ServerSide.html)
so I won't restate everything here.

Once fully set up and configured, you can do this in your controllers:

```elixir
def index(conn, _params) do
  people = ["Jack", "John", "Sayid", "Sawyer"]
  html = ReactPhoenix.ServerSide.react_component(
    "characters",
    %{people: people}
  )
  render(conn, "index.html", react_html: html, people: people)
end
```


## Usage

Once installed, you can use `react_component` in your views by:

1. Making sure that the component you'd like rendered is in the global namespace.
   You can do that in `app.js` like this (for example):
   
   ```javascript
   import MyComponent from "./components/my_component"
   window.Components = {
     MyComponent
   }
   ```

2. In your view template, you can then render it like this:

   ```elixir
   # with no props
   <%= ReactPhoenix.ClientSide.react_component("Components.MyComponent") %>

   # with props
   <%= ReactPhoenix.ClientSide.react_component("Components.MyComponent", %{language: "elixir", awesome: true}) %>

   # with props and a target html element id option
   # this can be used for server-side rendering (continuing with example from that section above)
   <span id="my-react-span"><%= @react_html %></span>
   <%= ReactPhoenix.ClientSide.react_component("Components.Characters", %{people: people}, target_id: "my-react-span") %>
   ```
   
   This will render a special `div` element in your html output that will then be recognized by the
   javascript helper as a div that should be turned into a React component. It will then render the
   named component in that `div` (or a different element specified by ID via the `target_id` option).


## Troubleshooting

* **I keep getting a compilation error like this**

  ```
    19 Apr 20:52:40 - error: Compiling of web/static/js/component.js failed. SyntaxError: web/static/js/component.js: Unexpected token (10:6)
     8 |   render() {
     9 |     return (
  > 10 |       <h1>You rendered React!</h1>
       |       ^
    11 |     )
    12 |   }
    13 | } ^G
  ```

  Most likely, you haven't set up your brunch config to know how to handle JSX files. I recommend
  the following:

  1. Run `npm install babel-preset-react babel-preset-env --save`
  2. Modify your `brunch-config.js` file to include those presets

      ```js
      // ...
      // Configure your plugins
      plugins: {
        babel: {
          presets: ["env", "react"], // <-- ADD THIS!
          // Do not use ES6 compiler in vendor code
          ignore: [/web\/static\/vendor/]
        }
      },
      // ...
      ```


## Documentation and other stuff

This package is heavily inspired by the [react-rails](https://github.com/reactjs/react-rails) project.

For more detailed documentation, check out the hex docs at 
[https://hexdocs.pm/react_phoenix](https://hexdocs.pm/react_phoenix)
