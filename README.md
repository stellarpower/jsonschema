[![GitHub release](https://img.shields.io/github/release/aarongodin/json-schema-cr.svg)](https://github.com/aarongodin/json-schema-cr/releases)
[![CI](https://github.com/aarongodin/json-schema-cr/actions/workflows/crystal.yml/badge.svg)](https://github.com/aarongodin/json-schema-cr/actions/workflows/crystal.yml)
![GitHub](https://img.shields.io/github/license/aarongodin/json-schema-cr?style=plastic)

# json-schema-cr

A compile-time generator of [JSON Schema](https://json-schema.org/) validation for Crystal.

## Installation

1. Add the dependency to your `shard.yml`:

   ```yaml
   dependencies:
     json-schema-cr:
       github: aarongodin/json-schema-cr
   ```

2. Run `shards install`

## Overview

This library reads JSON files from your file system **at compile time** and generates Crystal code to convert a JSON Schema document to a `JSONSchema::Validator` object.

## Usage

### Generating Validators

Generate code using the provided macros which output a reference to a `JSONSchema::Validator` object. You can assign the value to a variable or use it any place an expression can be used.

```crystal
require "json-schema-cr"
validator = JSONSchema.create_validator "my_schema.json"
```

You can use `create_validator_method` to define a method that returns the reference to the `JSONSchema::Validator`.

```crystal
class RequestBody
  JSONSchema.create_validator_method "my_schema.json"
end
```

This is syntactically equivalent to:

```crystal
class RequestBody
  def validator : JSONSchema::Validator
    JSONSchema.create_validator "my_schema.json"
  end
end
```

The `create_validator_method` macro allows you to also customize the method name that is generated by passing a second argument:

```crystal
class ExampleRoute
  JSONSchema.create_validator_method "request_schema.json", "request_body_validator"
  JSONSchema.create_validator_method "response_schema.json", "response_body_validator"
end

r = ExampleRoute.new

r.request_body_validator # => #<JSONSchema::ObjectValidator:...
r.response_body_validator # => #<JSONSchema::ObjectValidator:...
```

> **Omitting extension**: You can omit the file extension in any of the macros, in which case a file with `.json` as the extension will be loaded.
> ex:
> ```crystal
> JSONSchema.create_validator "my_schema" # => Loads "my_schema.json"
> ```

### Validating JSON

Use the `#validate` method on any generated validator to receive a `JSONSchema::ValidationResult`:

```crystal
require "json"
require "json-schema-cr"

class RequestBody
  JSONSchema.create_validator_method "my_schema.json"
end

request_body = RequestBody.new
input_json = JSON.parse({ "example" => "test" })

request_body.validator.validate(input_json) # => JSONSchema::ValidationResult(@status=:success, @errors=[])
```

The `JSONSchema::ValidationResult` will contain either `:success` or `:error` as the `status`. On `:error`, you can check the `@errors` array for a list of `JSONSchema::ValidationError` and respond in your code accordingly.

## Features

### Core Types

All JSON Schema types _are_ supported!

* `string`
* `number` and `integer`
* `array`
* `object`
* `null`
* `boolean`

### Composite Schema

Composite schemas using `not`, `anyOf`, `allOf`, or `oneOf` _are_ supported!

## Unsupported

These features of JSON Schema are not yet supported, but will be supported in a future release (at least before `1.0.0`).

* [References](https://json-schema.org/understanding-json-schema/structuring.html#ref), `$id`, `$anchor`, and any other relationship-driven schema definition.
* [Media Types](https://json-schema.org/understanding-json-schema/reference/non_json_data.html)

### Dialects

The latest revision of this shard only supports the latest revision of JSON Schema (2020-12). There is not yet support for using a different dialect.

## Roadmap

I would like to focus on these features beyond JSON schema that will make this library more useful in a variety of implementations:

1. **Message customization/i18n**: This module has a list of error messages that should be made customizable.
2. **Runtime generation**: Some implementations may want to create `JSONSchema::Validator` instances at runtime, without having to work with the underlying classes themselves.

## Acknowledgements

The source for this shard is inspired by the `ECR` and `JSON` implementations from the std lib. Thanks to the Crystal team for creating an amazing standard library!