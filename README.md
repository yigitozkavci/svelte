# Svelte

Svelte is a Swagger-to-Ruby object mapper.

It reads a Swagger specification file in JSON, and automatically generates Resource Classes with static methods to represent the various HTTP endpoints.

## Installation

Add this line to your application's Gemfile:

```ruby
gem "svelte"
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install svelte

## Usage

Point a service at an actual API spec.

You may pass in a URL pointing to a Swagger spec or the JSON directly:

```ruby
service = Svelte::Service.create(url: "http://path/to/swagger/spec/resource.json", module_name: 'PetStore')
service = Svelte::Service.create(json: "{ <JSON here> }", module_name: 'PetStore')
```

This will build a dynamically generated client on top of `Svelte::Service::PetStore`.

The structure of the new module will be based on the API paths and their respective operations.
Let's look at an example. Using the complete PetStore spec, we can find the following path:

```json
"/pet/findByStatus": {
  "get": {
    "tags": [
      "pet"
    ],
    "summary": "Finds Pets by status",
    "description": "Multiple status values can be provided with comma separated strings",
    "operationId": "findPetsByStatus",
    "produces": [
      "application/xml",
      "application/json"
    ],
    "parameters": [
      {
        "name": "status",
        "in": "query",
        "description": "Status values that need to be considered for filter",
        "required": true,
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "available",
            "pending",
            "sold"
          ],
          "default": "available"
        },
        "collectionFormat": "multi"
      }
    ],
    "responses": {
      "200": {
        "description": "successful operation",
        "schema": {
          "type": "array",
          "items": {
            "$ref": "#/definitions/Pet"
          }
        }
      },
      "400": {
        "description": "Invalid status value"
      }
    },
    "security": [
      {
        "petstore_auth": [
          "write:pets",
          "read:pets"
        ]
      }
    ]
  }
}
```

The path contains two parts: `pet` and `findByStatus`. This will generate
the following hierarchy in the new module:

```ruby
Svelte::Service::PetStore::Pet::FindByStatus
```

We can see the path has one `get` operation. A method will be generated in the
`FindByStatus` module based on the `operationId` Swagger attribute, which will
have the following signature:

```ruby
Svelte::Service::PetStore::Pet::FindByStatus.find_pets_by_status(
  request_payload,
  request_options = {}
)
```

Where `request_payload` is a `Hash` representing the parameters of the operation
and `request_options`, defaulting to an empty `Hash`, will be a `Hash` of
options to pass to the request.

In our case, the parameters would look like this:

```ruby
request_parameters = {
  status: ['available', 'pending']
}
```

### Responses

Svelte will return a [`Faraday::Request`](http://www.rubydoc.info/gems/faraday/0.9.1/Faraday/Response) object as a response to a call.

### Models

Svelte also provides generators for Swagger models. They allow an easy way
to programmatically create and validate requests.
They also provide an `as_json` that will generate a valid json body for
 a given request.

Consider the definitions key of this Swagger model:

```json
{
  "definitions": {
    "MoneyView": {
      "id": "MoneyView",
      "description": "",
      "required": [
        "amount",
        "currencyCode"
      ],
      "extends": "",
      "properties": {
        "amount": {
          "type": "number",
          "format": "double",
          "description": "Decimal amount"
        },
        "currencyCode": {
          "type": "string",
          "description": "ISO 3 letter currency code"
        }
      }
    }
  }
}
```

In Svelte you can generate the ruby mapper like this:

```ruby
class MoneyRequest
  extend Svelte::ModelFactory
  define_models_from_file(path_to_models_json_file)
end

view = MoneyRequest::MoneyView.new
view.valid? # false
view.validate # {"currencyCode"=>"Invalid parameter: Missing required parameter", "amount"=>"Invalid parameter: Missing required parameter"}
view.currencyCode = "GBP"
view.amount = 40.00
view.valid? # true
view.as_json # {:currencyCode=>"GBP", :amount=>40.0}
```

### Options

You can specify a timeout option on a per request basis. If the request times out a `Svelte::TimeoutError` exception
will be raised.

```ruby
begin
  Svelte::Service::PetStore::Pet::FindByStatus.find_pets_by_status(request.as_json, { timeout: 10 })
rescue Svelte::TimeoutError => e
  handle_timeout_error(e)
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release` to create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

1. Fork it ( https://github.com/notonthehighstreet/svelte/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
