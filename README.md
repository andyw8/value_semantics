# ValueType

Immutable struct-like value classes, with light-weight validation and coercion.

These are intended for internal use, as opposed to validating user input like ActiveRecord.
Invalid or missing attributes cause an exception intended for developers,
not an error message intended for the user.

## Basic Usage

```ruby
require 'value_type'

class Person < ValueType
  def_attr :name
  def_attr :age, default: 31
end

tom = Person.new(name: 'Tom')


#
# Read-only attributes
#
tom.name  #=> "Tom"
tom.age  #=> 31


#
# Convert to Hash
#
tom.to_h  #=> { :name => "Tom", :age => 31 }


#
# Non-destructive updates
#
old_tom = tom.with(age: 99)

old_tom  #=> #<Person name="Tom" age=99>
tom      #=> #<Person name="Tom" age=31> (unchanged)


#
# Equality
#
other_tom = Person.new(name: 'Tom', age: 31)

tom == other_tom  #=> true
tom.eql?(other_tom)  #=> true
tom.hash == other_tom.hash  #=> true
```


## Validation (Types)

Validators are objects that implement the `===` method,
which means you can use `Class` objects (like `String`) and also `Regexp` objects:

```ruby
class Person < ValueType
  def_attr :name, String
  def_attr :birthday, /\d\d\d\d-\d\d-\d\d/
end

Person.new(name: 'Tom', ...)  # works
Person.new(name: 5, ...)
#=> ArgumentError:
#=>     Value for attribute 'name' is not valid: 5

Person.new(birthday: "1970-01-01", ...)  # works
Person.new(birthday: "hello", ...)
#=> ArgumentError:
#=>     Value for attribute 'birthday' is not valid: "hello"
```

A custom validator might look something like this:

```ruby
module Odd
  def self.===(value)
    value.odd?
  end
end

class Person < ValueType
  def_attr :age, Odd
end

Person.new(age: 9)  # works
Person.new(age: 8)
#=> ArgumentError:
#=>     Value for attribute 'age' is not valid: 8
```

Default attribute values also pass through validation.


## Coercion

Coercion blocks can convert invalid values into valid ones, where possible.

```ruby
class Person < ValueType
  def_attr :birthday do |value|
    case value
    when Date then value
    when String then Date.parse(value)
    when Array then Date.new(*value)
    else fail("#{value} is not a valid birthday")
    end
  end
end

Person.new(birthday: Date.today)    # works
Person.new(birthday: '2018-08-19')  # works
Person.new(birthday: [2018, 8, 19]) # works
Person.new(birthday: 42)
#=> ArgumentError:
#=>     Value for attribute 'birthday' is not valid: 42
```

Coercion happens before validation.
If coercion is not possible, one option is to return the value unchanged,
allowing the validator to fail instead of raising an error within the block.

Default attribute values also pass through coercion.

## All Together

```ruby
class Coordinate < ValueType
  def_attr(:latitude, Float, default: 0) { |value| value.to_f }
  def_attr(:longitude, Float, default: 0) { |value| value.to_f }
end

Coordinate.new(longitude: "123")
#=> <Coordinate latitude=0.0 longitude=123.0>
```


## Installation

Add this line to your application's Gemfile:

```ruby
gem 'value_type'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install value_type


## Contributing

Bug reports and pull requests are welcome on GitHub at:
https://github.com/tomdalling/value_type


## License

The gem is available as open source under the terms of the [MIT
License](http://opensource.org/licenses/MIT).
