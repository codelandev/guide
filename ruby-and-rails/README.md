# Back-end Rails Project files organization

This is an example of how you should organize your files inside the respective folders for each kind of use.
To not make this readme so extense I'll move each file example to another readme file and link on respective titles. I put just the folders/files that aren't default from new projects.
Let's say you are using the default CODELAND project generator, you will have something like this:

### [Controllers](https://github.com/codelandev/guide/blob/master/ruby-and-rails/controller_example.md)

```
/app
..../controllers
......../api
............/v1
................/examples_controller.rb
........other_examples_controller.rb
```

### [Decorators]() (and use [Draper](https://github.com/drapergem/draper) for this )

```
..../decorators
......../example_decorator.rb
```

### [Models]()

```
..../models
......../example.rb
```

### [Policies]() (and use [Pundit](https://github.com/elabs/pundit) for this )

```
..../policies
......../example_policy.rb

```

### [Serializers]() ( and use [Active Model Serializers](https://github.com/rails-api/active_model_serializers) for this )

```
..../serializers
......../api
............/v1
................/example_serializer.rb # for single example)
................/examples_serializer.rb # for more than one example, index method for example
```

### [Uploaders]() ( if you use [CarrierWave](https://github.com/carrierwaveuploader/carrierwave) for this )

```
..../uploaders
......../example_uploader.rb
```

### [Views]()

```
..../views
......../layouts
............/application.html.slim
............/mailer.html.slim
......../examples
............/index.html.slim
............/_form.html.slim
............/new.html.slim
............/edit.html.slim
```

### [Config]()

```
/config
..../locales
......../example.pt-BR.yml # always put different languages on different files
......../example.en.yml
....database.yml.sample
```

### [Specs]() ( I recommend use [rspec-rails](https://github.com/rspec/rspec-rails) )

```
/spec
..../controllers
......../api
............/v1
................/examples_controller_spec.rb
..../models
......../example_spec.rb
..../policies
......../example_policy_spec.rb
..../support
........blueprints.rb
........helpers.rb
....rails_helper.rb
....spec_helper.rb
```

### [Procfile](https://devcenter.heroku.com/articles/procfile)

```
Procfile # always have a Procfile on your project
```

### [README.md](https://github.com/codelandev/snippets/blob/master/Documents/Example%20of%20README.md)

```
README.md # Try to put informations of how to install the application or deploy
```

