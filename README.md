Slides: https://gitpitch.com/dplummer/slides-mix-templates

# Mix new and templates

Two parts:
* Mix new
* mix_generator and mix_templates from pragdave

## Mix new
* library of just functions
* library of functions plus own supervision tree
* application
* application with umbrella app
* subset of application: phoenix apps, with/without umbrella

Commands:
* Lib: `mix new app_name`
* Lib with supervisor: `mix new app_name --sup`
* Application:
  * `mix new app_name --sup`
  * plus distillery to create a release
* Application with umbrella:
  * `mix new app_name --umbrella`
  * `cd apps && mix new foo_bar --sup`
  * plus distillery to create a release
* Phoenix
  * `mix phx.new todo`
  * `mix phx.new hello --umbrella`
  ```
  Would generate the following directory structure and modules:

    hello_umbrella/   Hello.Umbrella
      apps/
        hello/        Hello
        hello_web/    Hello.Web
  ```

### Why umbrella?
* Better structure for more monolithic apps
* Able to split apps into separate projects later when Conway's law comes up
* Run two phoenix apps on different ports within the same binary
> Overall, umbrella applications do not magically improve the design of your code. They can,
> however, help enforce boundaries when the code is well designed.

> Using umbrella projects you can get the benefits of monoliths and microservices.

## mix_generator and mix_templates from pragdave

https://medium.com/@a4word/building-small-elixir-services-using-ecto-without-phoenix-1aba00b53e54
https://pragdave.me/blog/2017/04/18/elixir-project-generator.html
https://github.com/pragdave/mix_generator
https://github.com/pragdave/mix_templates
http://www.schmitty.me/phoenix-1-3-and-webpack-2-0/

mix_templates manages the local installation and uninstallation of templates that get used by
mix_generator

mix_generator uses installed templates to build a new project

* Docker container, using mix_docker
* Backend database using mysql
* Configuration done with environment variables
* Continuous integration with CircleCI
* A configuration file for orchestration and routing
* A standardized status endpoint for monitoring
* A logging format that is machine-parseable
* And much more


---

I'm going to talk tonight about creating new elixir projects. I'll start by going over the built in
project generators, and then talk about a library to build your own generators.

> When we say “project” you should think about Mix. Mix is the tool that manages your project. It
> knows how to compile your project, test your project and more. It also knows how to compile and
> start the application relevant to your project.

So a basic project in Elixir is created by running `mix new`. This creates a simple directory
structure: config, lib, and test. And the associated files, like mix.exs and config files. From
here, you can add dependencies to the mix.exs file, and write code in the lib directory. This is
sufficient for libraries and simple applications that don't use processes.

The next step up is adding a supervision tree. Mix again makes this easy, with `mix new --sup`.
This adds an application and the basic supervisor. Now we can store state and supervise our
processes. Any long-running application can start from this template.

This gets you really far. Any application you want to build can start from just a simple `mix new`
project.

The easy way to build a web app in Elixir is with Phoenix. And it has a nice generator to get your
started. When you create a new Phoenix project with `mix phx.new`, it adds some web files
(controllers, views, templates, etc), but starts up just like any other process: in your
Application file, it adds `supervisor(MyApp.Web.Endpoint, [])`.

As your applications grow in size and complexity, you may want to group components of your project
together. Say you have an ecommerce app, you could split up the app into smaller projects for
orders, inventory management, frontend, etc. But if you split up your application into different git
repositories, this gets to be a hassle when making updates that span projects. Elixir has a
solution: umbrella apps.

An umbrella app is a collection of other applications that get built into a single release. When
you run `mix new --umbrella` you get a sparse mix.exs file, and an apps directory. Any app you
create in the apps directory, will share the dependencies of the other apps, and you can call the
other apps public APIs.

Phoenix 1.3 has an umbrella generator to get you started too. When you use `mix phx.new
--umbrella` you get an apps directory with one application for your business logic and database
schema, and a second application for your web logic: controllers, views, templates, etc. This
provides a nice seperation of concerns. If you later want to have a different interface for your
project, like a CLI, a background job, or responding to events from a message bus (like Kafka or
RabbitMQ), then those can be separate applications that depend on your business logic app, but
don't need to concern themselves with the web.

> Overall, umbrella applications do not magically improve the design of your code. They can,
> however, help enforce boundaries when the code is well designed.
- https://elixir-lang.org/getting-started/mix-otp/dependencies-and-umbrella-apps.html

Once you've built a few projects, you find there are certain pieces of infrastructure that are
pretty similar across each. Here at Avvo, we have a checklist of features each app that goes into
production needs to have, and a preferred implementation.

We deploy our apps with docker, so we build our release with mix_docker. Our SQL database server
is MySQL instead of the postgres default. We configure our environements with environement
variables, instead of a prod.secret.exs file. We do continuous integration with CircleCI, and much
more.

To facilitate a standardized project setup and speed up getting started, we used a new set of
packages from Pragmatic Dave Thomas called mix_generator and mix_templates.

To get started we'll install the mix task globally with `mix archive.install hex mix_templates`,
that's the templating system. Then we'll install the generator
`mix archive.install hex mix_generator`, that's the mix task that generates brand new projects.

A brief overview of `mix template`: You can run `mix template` to get a list of your locally
installed templates. And you can run `mix template.hex` to get a list of the available templates
on hex. So if we wanted to create a new project using Dave's template, we can
`mix template.install hex gen_template_project`, then generate a new project with that template
with `mix gen project my_app`.

That's handy, but the real power is creating and using your own templates. Luckily there's a
template for new templates.

Let's say all your libraries need a CHANGELOG.md file. Let's install the
template template: `mix template.install hex gen_template_template`. Then we'll generate our new
template `mix gen template library_project`.

We can have our library project depend on the default project template, by editing our
library_project.ex and adding `based_on: :project`.

We can test out our new template with `mix gen ./library_project foobar`.

Now let's add our basic changelog file, and we'll add our initial version with today's date.

A more powerful new template would be for a phoenix project. There's a really handy blog post from
Cory Schmitt that goes into exactly that. Basically, you generate a new template, then generate a
new phoenix app in that template, and do some string replacements.

A few handy tricks are shown there, like using vim to edit all files in a project:

```
:args `find . -type f`
:argdo %s/ReplaceMe/<%= @project_name_camel_case %>/g | update
:argdo %s/replace_me/<%= @project_name %>/g | update
```

Since mix_templates uses Eex to evaluate the template files, any actual eex templates you need to
edit and use two percents instead of one.

Additionally, mix_templates does not support non-UTF-8 encoded files, so the binaries added to
your project by phx.new like phoenix.png and favicon.ico cause the generator process to explode,
since it tries to eval them as eex files.

So in summary, if you need a repeatable way to create new projects with your own file structure,
use mix_templates.
