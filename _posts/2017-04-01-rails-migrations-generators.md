---
layout: post
tags: [ruby, migrations, generators]
title: "Understanding and writing Rails migration generators"
header-img: img/posts/rails-migrations-generators.jpeg
card-img: img/cards/rails-migrations-generators.png
---

Have you ever used the notorious [Devise](https://github.com/plataformatec/devise) gem?
Of course you have! Just in case you haven't - it's a very well known Rails gem
that adds a complete authentication mechanism to any Rails app. It's so simple
to set up and easy to use, in my opinion, Devise it's a classroom example of
what a great gem should work and be documented like.

`</advertisement>`

One of the perks of using Devise is how easy it is to set up. It's only a matter
of adding the gem to the `Gemfile` and then running two `rails generate`
commands that will set up the gam and create the model and the required
migrations for that model. The commands usually look like:

{% highlight bash %}
rails generate devise:install
{% endhighlight %}

{% highlight bash %}
rails generate devise User
{% endhighlight %}

Let's ignore the first one and see what the output usally looks like from the
second command:

{% highlight bash %}
rails generate devise User

Running via Spring preloader in process 52335
    invoke  active_record
    create    db/migrate/20170327212055_devise_create_users.rb
    create    app/models/user.rb
    invoke    test_unit
    create      test/models/user_test.rb
    create      test/fixtures/users.yml
    insert    app/models/user.rb
     route  devise_for :users
{% endhighlight %}

As you can see, Devise by using Rails' generators created a model, a migration,
some tests and routes for our new `User` model. Let's look at the magic behind
creating that migration and understand (a little bit better) how generators work
in Rails?

## What are Rails Generators?

Generators are an integral part of Rails - it's very hard to use the framework
without any use of generators. They allow us to bootstrap our application with
various files and configurations in any stage of the development of the
application.

For example, the `controller` generator generates controllers
(and corresponding views) for it's actions. The `model` generator creates models
in our application. Also, gems (like Devise) have generators - for example this
one:

{% highlight bash %}
rails generate devise User
{% endhighlight %}

In fact our Rails apps have a plethora of generators. If you want to see how many
are there actually, run this from you application's root path:

```
$ rails generate --help
```

You will see something like this:

```
Usage: rails generate GENERATOR [args] [options]

General options:
  -h, [--help]     # Print generator's options and usage
  -p, [--pretend]  # Run but do not make any changes
  -f, [--force]    # Overwrite files that already exist
  -s, [--skip]     # Skip files that already exist
  -q, [--quiet]    # Suppress status output

Please choose a generator below.

Expected string default value for '--helper'; got true (boolean)
Expected string default value for '--assets'; got true (boolean)
Rails:
  assets
  controller
  generator
  helper
  integration_test
  job
  mailer
  migration
  model
  resource
  scaffold
  scaffold_controller
  task

ActiveRecord:
  active_record:migration
  active_record:model

FactoryGirl:
  factory_girl:model

Haml:
  haml:application_layout
  haml:controller
  haml:mailer
  haml:scaffold

Js:
  js:assets

Kaminari:
  kaminari:config
  kaminari:views

Rspec:
  rspec:controller
  rspec:feature
  rspec:helper
  rspec:install
  rspec:integration
  rspec:job
  rspec:mailer
  rspec:model
  rspec:observer
  rspec:request
  rspec:scaffold
  rspec:view

Scss:
  scss:assets
  scss:scaffold

TestUnit:
  test_unit:controller
  test_unit:generator
  test_unit:helper
  test_unit:integration
  test_unit:job
  test_unit:mailer
  test_unit:model
  test_unit:plugin
  test_unit:scaffold

WebpackRails:
  webpack_rails:install
```

As you can see, Rails provides us with bunch of generators that we can use.


But, how do generators actually work?

Well, Rails' generators are built on top of [Thor](https://github.com/erikhuda/thor),
which is a toolkit for building command line tools. This means that Thor enables
awesome methods like `create_file`, `create_link`, `remove_file` and so on to be
used in a generator. In fact, if we look at some of the generators in Rails'
[source code](https://github.com/rails/rails/tree/master/railties/lib/rails/generators),
we can see how these methods are utilized.

