## New mix projects and mix_templates

#### Donald Plummer
#### dplummer@avvo.com

^ Welcome to the July Seattle elixir meetup. I'm Donald Plummer, a software developer here at Avvo.

---

# Creating a new elixir project

^ I'm going to talk tonight about creating new elixir projects. I'll start by going over the built in project generators, and then talk about a library to build your own generators.

---

^ When we say “project” you should think about Mix. Mix is the tool that manages your project. It knows how to compile your project, test your project and more.

^ So a basic project in Elixir is created by running `mix new`.

# Creating a new elixir project

* `mix new foo_bar`

---

^  This creates a simple directory structure: config, lib, and test. And the associated files, like mix.exs and config files. From here, you can add dependencies to the mix.exs file, and write code in the lib directory. This is sufficient for libraries and simple applications that don't use processes.

# Creating a new elixir project

* `mix new foo_bar`


```
foo_bar
├── README.md
├── config
│   └── config.exs
├── lib
│   └── foo_bar.ex
├── mix.exs
└── test
    ├── foo_bar_test.exs
    └── test_helper.exs
```

---

^ The next step up is adding a supervision tree. Mix again makes this easy, with `mix new --sup`.

# Supervised project

* `mix new foo_sup --sup`

---

^ This adds an application and the basic supervisor. Now we can store state and supervise our processes. Any long-running application can start from this template.

# Supervised project

* `mix new foo_sup --sup`

```
foo_sup
├── README.md
├── config
│   └── config.exs
├── lib
│   ├── foo_sup
│   │   └── application.ex
│   └── foo_sup.ex
├── mix.exs
└── test
    ├── foo_sup_test.exs
    └── test_helper.exs
```

^ This gets you really far. Any application you want to build can start from just a simple `mix new` project.


---

^ The easy way to build a web app in Elixir is with Phoenix. And it has a nice generator to get you started. When you create a new Phoenix project with `mix phx.new`, it adds some web files (controllers, views, templates, etc).

![inline](phoenix_framework.png)

`mix phx.new my_web_app`

```
my_web_app
├── README.md
├── assets
├── config
├── lib
├── mix.exs
├── priv
└── test
```

---

^ As your applications grow in size and complexity, you may want to group components of your project together. Say you have an ecommerce app, you could split up the app into smaller projects for orders, inventory management, frontend, etc. But if you split up your application into different git repositories, this gets to be a hassle when making updates that span projects. Elixir has a solution: umbrella apps.

![inline](monolithic_vs_microservices.jpg)

---

^ An umbrella app is a collection of other applications that get built into a single release.

# Umbrella app

---

^ When you run `mix new --umbrella` you get a sparse mix.exs file, and an apps directory. Any app you create in the apps directory, will share the dependencies of the other apps, and you can call the other apps public APIs.

# Umbrella app

`mix new foo_brella --umbrella`

```
foo_brella
├── README.md
├── apps
├── config
│   └── config.exs
└── mix.exs
```

---

^ Phoenix 1.3 has an umbrella generator to get you started too. When you use `mix phx.new --umbrella` you get an apps directory with one application for your business logic and database schema, and a second application for your web logic: controllers, views, templates, etc. This provides a nice seperation of concerns. If you later want to have a different interface for your project, like a CLI, a background job, or responding to events from a message bus (like Kafka or RabbitMQ), then those can be separate applications that depend on your business logic app, but don't need to concern themselves with the web.

![inline](phoenix_framework.png)

`mix phx.new my_web_app --umbrella`

```
my_web_app_umbrella
├── README.md
├── apps
│   ├── my_web_app
│   └── my_web_app_web
├── config
│   ├── config.exs
│   ├── dev.exs
│   ├── prod.exs
│   └── test.exs
└── mix.exs
```

---

# Umbrellas aren't a silver bullet

> Overall, umbrella applications do not magically improve the design of your code. They can, however, help enforce boundaries when the code is well designed.
- `https://elixir-lang.org/getting-started/mix-otp/dependencies-and-umbrella-apps.html`

^ Overall, umbrella applications do not magically improve the design of your code. They can, however, help enforce boundaries when the code is well designed.

---

# Standardized project features

^ Once you've built a few projects, you find there are certain pieces of infrastructure that are pretty similar across each. Here at Avvo, we have a checklist of features each app that goes into production needs to have, and a preferred implementation.

---

^ We deploy our apps with docker, so we build our release with mix_docker. Our SQL database server is MySQL instead of the postgres default. We configure our environments with environment variables, instead of a prod.secret.exs file. We do continuous integration with CircleCI, and much more.

# Standardized project features

* Docker container, using mix_docker
* Backend database using mysql
* Configuration done with environment variables
* Continuous integration with CircleCI
* etc

---

^ To facilitate a standardized project setup and speed up getting started, we used a new set of packages from Pragmatic Dave Thomas called mix_generator and mix_templates.

# from Dave Thomas:
## `mix_generator`
## `mix_templates`

---

Live demo!!

^ To get started we'll install the mix task globally with `mix archive.install hex mix_templates`,
that's the templating system.

^ Then we'll install the generator `mix archive.install hex mix_generator`, that's the mix task that generates brand new projects.

^ A brief overview of `mix template`: You can run `mix template` to get a list of your locally installed templates. And you can run `mix template.hex` to get a list of the available templates on hex. So if we wanted to create a new project using Dave's template, we can run `mix template.install hex gen_template_project`, then generate a new project with that template with `mix gen project my_app`.

^ That's handy, but the real power is creating and using your own templates.

`mix archive.install hex mix_templates`
`mix archive.install hex mix_generator`
`mix template`
`mix template.hex`
`mix template.install hex gen_template_project`
`mix gen project my_app`

^ Luckily there's a template for new templates.

^ Let's say all your libraries need a CHANGELOG.md file. Let's install the template template: `mix template.install hex gen_template_template`. Then we'll generate our new template `mix gen template library_project`.

^ We can have our library project depend on the default project template, by editing our library_project.ex and adding `based_on: :project`.

^ We can test out our new template with `mix gen ./library_project foobar`.

^ Now let's add our basic changelog file, and we'll add our initial version with today's date.

^ We can test out our new template with `mix gen ./library_project foobar`.

---

![inline](yo-dawg-templates.jpg)

---

![inline](cory-schmitt.png)

^ A more powerful new template would be for a phoenix project. There's a really handy blog post from Cory Schmitt that goes into exactly that. Basically, you generate a new template, then generate a new phoenix app in that template, and do some string replacements.

---

^ A few handy tricks are shown there, like using vim to edit all files in a project:

# Vim global replace

```
:args `find . -type f`
:argdo %s/ReplaceMe/<%= @project_name_camel_case %>/g | update
:argdo %s/replace_me/<%= @project_name %>/g | update
```

---

^ Since mix_templates uses Eex to evaluate the template files, any actual eex templates you need to edit and use two percents instead of one.

# Replace % with %%

`<%= 1 + 1 %>`

`<%%= 1 + 1 %>`

---

^ Additionally, mix_templates does not support non-UTF-8 encoded files, so the binaries added to your project by phx.new like phoenix.png and the favicon cause the generator process to explode, since it tries to eval them as eex files.

# Binary files not supported

`rm phoenix.png`
`rm favicon.ico`

---

^ So in summary, if you need a repeatable way to create new projects with your own file structure, use mix_templates.


https://github.com/dplummer/slides-mix-templates

https://pragdave.me/blog/2017/04/18/elixir-project-generator.html

https://github.com/pragdave/mix_generator

https://github.com/pragdave/mix_templates

http://www.schmitty.me/phoenix-1-3-and-webpack-2-0/

## Donald Plummer |  dplummer@avvo.com | @dplummer

---

![inline](avvo.png)

# We're hiring!

^ Come Code With US! We're Hiring!

^ Elixir and Ruby/Rails Full-Stack Developers with an eye for front-end

^ Back-End your Jam? No problem, we're looking for skilled Back-End Developers to join our team! Elixir + Ruby + Opensource!

* Frontend
* Backend
* Ruby (205 repos)
* Elixir (48 repos)
* Last hackathon 5 of 13 projects used Elixir
