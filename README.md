# Turbine

<p align="center" style="text-align:center;">
  <img alt="turbine logo" src="docs/turbine-outline.svg" width="500" />
</p>

Turbine is a data application framework for building server-side applications that are event-driven, respond to data in real-time, and scale using cloud-native best practices.

The benefits of using Turbine include:

* **Native Developer Tooling:** Turbine doesn't come with any bespoke DSL or patterns. Write software like you normally would!

* **Fits into Existing DevOps Workflows:** Build, test, and deploy. Turbine encourages best practices from the start. Don't test your data app in production ever again.

* **Local Development mirrors Production:** When running locally, you'll immediately see how your app reacts to data. What you get there will be exactly what happens in production but with _scale_ and _speed_.

* **Available in many different programming langauages:** Turbine started out in Go but is available in other languages too:
    * [Go](https://github.com/meroxa/turbine-go)
    * [Javascript](https://github.com/meroxa/turbine-js)
    * [Python](https://github.com/meroxa/turbine-py)
    * [Ruby](https://github.com/meroxa/turbine-rb) 
-----------

## Requirements 

- *Recommended:* Ruby version management tool of your choice. We recommend either the following:
    - [rbenv](https://github.com/rbenv/rbenv)
    - [RVM](https://rvm.io/)
    - [asdf](https://github.com/asdf-vm/asdf-ruby)
- [Latest Ruby version](https://www.ruby-lang.org/en/downloads/). If you have a Ruby version management tool installed, you can install ruby through your version management tool and specify which version you would like installed for your development use case.
- You’ll also need to [download the Meroxa CLI](https://github.com/meroxa/cli#installation-guide).

## Getting Started

To get started, [download the Meroxa CLI](https://github.com/meroxa/cli#installation-guide). Once downloaded and installed, go to your terminal and initialize a new project:

```ruby
$ meroxa apps init my-ruby-app-name --lang ruby
```

The CLI will create a new folder called my-ruby-app located in the directory where the command was issued. If you want to initialize the app somewhere else, you can append the `--path` flag to the command (`meroxa apps init my-ruby-app-name --lang ruby --path ~/anotherdir`). Once you enter the my-ruby-app-name directory, the contents will look like this:

```bash
my-ruby-app-name
├── Gemfile
├── app.json
├── app.rb
└── fixtures
   └── demo.json
```
   
This will be a full-fledged Turbine app that can run. You can even run the tests using the command meroxa apps run in the root of the app directory. It provides just enough to show you what you need to get started.

Install the necessary dependencies with: bundle install.

### **`Gemfile`**

This is a file that is created to describe the gem dependencies required to run a Ruby program.

### **`app.json`**

This file contains all of the options for configuring a Turbine app. 

`{
  "name": "ruby-example",
  "language": "ruby",
  "environment": "common",
  "resources": {
    "demopg": "fixtures/demo.json"
  }
}`

- `name` - The name of your application.
- `language` - Tells Meroxa what language the app is upon deployment.
- `environment` - "common" is the only available environment. Meroxa does have the ability to create isolated environments but this feature is currently in beta.
- `resources` - These are the named integrations that you'll use in your application. The `demo_pg` is currently in place of the source_name. You can replace this with your resource. This needs to match the name of the resource that you'll set up in Meroxa using the `meroxa resources create` command or via the Dashboard. You can point to the path in the fixtures that'll be used to mock the resource when you run `meroxa apps run`.

### **`App.rb`**

This configuration file is where you begin your Turbine journey. Any time a Turbine app runs, this is the entry point for the entire application. When the project is created, the file will look like this:

```ruby
# frozen_string_literal: true

require "rubygems"
require "bundler/setup"
require "turbine_rb"

class MyApp
  def call(app)
    # To configure resources for your production datastores
    # on Meroxa, use the Dashboard, CLI, or Terraform Provider
    # For more details refer to: http://docs.meroxa.com/
    #
    # Identify the upstream datastore with the `resource` function
    # Replace `demopg` with the resource name configured on Meroxa
    database = app.resource(name: "demopg")

    # Specify which upstream records to pull
    # with the `records` function
    # Replace `collection_name` with a table, collection,
    # or bucket name in your data store.
    # If a configuration is needed for your source,
    # you can pass it as a second argument to the `records` function. For example:
    # database.records(collection: "collection_name", configs: {"incrementing.column.name" => "id"})
    records = database.records(collection: "collection_name")

    # Register secrets to be available in the function:
    # app.register_secrets("MY_ENV_TEST")

    # Register several secrets at once:
    # app.register_secrets(["MY_ENV_TEST", "MY_OTHER_ENV_TEST"])

    # Specify the code to execute against `records` with the `process` function.
    # Replace `Passthrough` with your desired function.
    # Ensure desired function matches `Passthrough`'s' function signature.
    processed_records = app.process(records: records, process: Passthrough.new)

    # Specify where to write records using the `write` function.
    # Replace `collection_archive` with whatever data organisation method
    # is relevant to the datastore (e.g., table, bucket, collection, etc.)
    # If additional connector configs are needed, provided another argument. For example:
    # database.write(
    #   records: processed_records,
    #   collection: "collection_archive",
    #   configs: {"behavior.on.null.values": "ignore"})
    database.write(records: processed_records, collection: "collection_archive")
  end
end

class Passthrough < TurbineRb::Process
  def call(records:)
    puts "got records: #{records}"
    # To get the value of unformatted records, use record .value getter method
    # records.map { |r| puts r.value }
    #
    # To transform unformatted records, use record .value setter method
    # records.map { |r| r.value = "newdata" }
    #
    # To get the value of json formatted records, use record .get method
    # records.map { |r| puts r.get("message") }
    #
    # To transform json formatted records, use record .set methods
    # records.map { |r| r.set('message', 'goodbye') }
    records
  end
end

TurbineRb.register(MyApp.new)
```


### Fixtures

Fixtures are JSON-formatted samples of data records you can use while locally developing your Turbine app. Whether CDC or non-CDC-formatted data records, fixtures adhere to the following structure:

```json
{
  "collection_name": [
    {
      "key": "1",
      "value": {
		  "schema": {
			  //...
		  },
		  "payload": {
			  //...
		  }
		}
	}
  ]
```
* `collection_name` — Identifies the name of the records or events you are streaming to your data app.
* `key` — Denotes one or more sample records within a fixture file. `key` is always a string.
* `value` — Holds the `schema` and `payload` of the sample data record.
* `schema` — Comes as part of your sample data record. `schema` describes the record or event structure.
* `payload` — Comes as part of your sample data record. `payload` describes what about the record or event changed.

Your newly created data app should have a `demo-cdc.json` and `demo-non-cdc.json` in the `/fixtures` directory as examples to follow.

### Unit Testing

Those familiar with software development are likely already in the habit of writing unit tests. We encourage you to follow this convention throughout the development process to ensure each unit of code works as expected and as a way to identify any issues or bugs early on.

Unit testing is language-specific, you may use any testing framework of your choice.

## Documentation

The most comprehensive documentation for Turbine and how to work with Turbine apps is on the Meroxa site: [https://docs.meroxa.com/](https://docs.meroxa.com)
