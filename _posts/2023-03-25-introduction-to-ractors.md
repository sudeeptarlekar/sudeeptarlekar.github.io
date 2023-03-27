---
title: Introduction to Ractors in Ruby
date: 2023-03-25 14:05:00
categories: [Ruby]
tags: [Ruby, Rails]
---

![Image](/assets/article_images/2023-03-25-introduction-to-ractors.jpg)

Ruby is commonly considered slow in terms of performance compared to other programming languages,
and the Ruby community is actively working to improve its speed.
Have you heard of [Ruby3x3](https://blog.heroku.com/ruby-3-by-3)?
According to [Yukihiro Matsumoto](https://twitter.com/yukihiro_matz), also known as Matz,
Ruby was primarily created for productivity and fun learning of programming, which resulted in its slower performance.
However, the community is now working to improve the speed of Ruby threefold with Ruby3.
This new version includes many experimental features, such as Ractors.

## Introduction

Ractor is an experimental feature introduced in Ruby3,
based on the Actor model - a concurrent computation model in computer science that treats actors as basic building blocks.
Despite still being an experimental feature, you can still try it out and use it in real-life applications.

Creating Ractor is simple. You can also pass a name to a Ractor

```ruby
# Creating ractor without name

ractor = Ractor.new { puts 'Hello world!' }

# <internal:ractor>:267: warning: Ractor is experimental, and the behavior may change
# in future versions of Ruby! Also there are many implementation issues.
#
# Hello World!
#
# => #<Ractor:#2 (irb):1 terminated>

# Creating ractors with the name

test_ractor = Ractor.new(name: 'Test') { puts 'Hello World!' }

# <internal:ractor>:267: warning: Ractor is experimental, and the behavior may change
# in future versions of Ruby! Also there are many implementation issues.
#
# Hello World!
#
# => #<Ractor:#3 (irb):1 terminated>

test_ractor.name

# => 'Test'
```

## Thread Safety

Main feature of Ractor is thread safety. Ractor implements this thread safety by not sharing objects.
Like threads Ractors do not share objects between Ractords or any other object/variables from outer
scope, which helps in implementing the thread safety.

```ruby
var = 'World!'

r = Ractor.new { puts "Hello #{var}" }

# <internal:ractor>:267:in `new': can not isolate a Proc because it accesses outer variables (var). (ArgumentError)
```

## Passing value to Ractor

As you can see, when trying to access an object from an outer scope, we are getting an error.
However, we can still pass values from the outer world by using ***receive*** and ***send***.

```ruby
str = 'World!'

ractor = Ractor.new do
  val = receive
  puts "Hello #{val}!"
end

ractor.send str

# Hello World!
# => #<Ractor:#2 (irb):3 blocking>
```

> Ractor also provides `<<` as shorthand for `send` method.
  Calling `ractor << str` will produce the same result as above.

You also pass the object to `new` which will add it as a parameter to the block

```ruby
data = ['Hello', 'World!']

ractor = Ractor.new(data) { |d| puts d.join(' ') }

# Hello World!
# => #<Ractor:#3 (irb):12 blocking>
```

Whenever we pass a value to a Ractor from the outside world, it will either be copied or moved,
with the default action being a full copy of the object being passed through deep cloning of non-shareable parts.
The following code demonstrates this behavior. Only `str` is copied over as it is non-shareable,
while the rest of the values are passed to the Ractor without cloning.

```ruby
data = [1, 'str', 4.0, 'frozen_string'.freeze]
data.map { |val| val.object_id }.join(' ')
# => "3 32200 36028797018963970 32220"

ractor = Ractor.new(data) { |s| s.map(&:object_id).join(' ') }
ractor.take
# => "3 54720 36028797018963970 32220"
```

Cloning an object is slow and sometimes it is impossible to clone an object itself.
In such cases, `move: true` can be used when sending data to a Ractor.
When an object is moved, it will not be available to the outside world.

```ruby
data = ['bar', 'baz']

ractor = Ractor.new { puts receive.join(' ') }
ractor.send(data, move: true)
# bar baz

data.inspect
# => Raises Ractor::MovedError('can not send any methods to a moved object')
```

## Sharing values between Ractors
There are two ways we can share the objects between ractors
1. `send` and `yield`
2. `receive` and `take`

Let's see how we can share the objects between two Ractors.

```ruby
sender = Ractor.new do
  puts "Sender #{self.inspect} initializing the call"
  puts "Sender is sending: 'Message'"
  Ractor.yield 'Message'

  response = receive

  puts "Received response from caller: #{response}"
end

# Sender #<Ractor:#2 (irb):1 running> initializing the call
# Sender is sending: 'Message'

caller = Ractor.new(sender) do |s|
  puts "Caller #{self.inspect} is ready to receive message"

  data = s.take
  puts "Sender sent: #{data}"
  puts "Acknowledging client with: '10-4'"

  s.send('10-4')
end

# Sender sent: Message
# Acknowledging client with: '10-4'
# Received response from caller: 10-4
```

As you can see here, we are creating two Ractors: one is the *sender* and the other is the *caller*.
As soon as we create the *sender* Ractor, it prints out a message on the console and holds
the execution as it encounters the `yield` on line 4.

On line 14, we define a new Ractor(*caller*) with the *sender* Ractor as an argument.
Then, on line 17, we receive the value that the *sender* is yielding on line 4 (i.e., **Message**),
and on line 21, we `send` data to the *sender* Ractor as usual.

> A point to note here is that Ractor objects are shareable and will not be copied over.
  Try calling `object_id` on Ractors and see if you get different object IDs or the same one.

## Passing a class and constants

Classes and modules in Ruby are shareable, but not their instances and their attributes.
The following example demonstrates that when we pass the class instance to the Ractor,
they are copied over along with their attributes.

```ruby
class Foo
  attr_accessor :val

  def initialize(val) = @val = val # You can also write single line function in Ruby3
end

foo = Foo.new('val')
puts "Foo's object_id: #{foo.object_id}; Attribute object_id: #{foo.val.object_id}"

# Foo's object_id: 22280; Attribute object_id: 22300

ractor = Ractor.new do
  resp = receive
  puts "Instance's object_id: #{resp.object_id}; Attribute object_id: #{resp.val.object_id}"
end
ractor.send(foo)

# Instance's object_id: 31980; Attribute object_id: 32000
```

While classes and module are shareable, there attribuets are not.
If we try to access the class attributes from the Ractors, we will end up getting an error.

```ruby
class Foo
  class << self
    attr_accessor :val
  end
end
Foo.val = 'val'

ractor = Ractor.new do
  resp = receive

  puts "Ractor received klass: #{resp}"
  puts "Ractor inspecting class attributes: #{resp.val}"
end

ractor << Foo
# Ractor received klass: Foo
# => Ractor::IsolationError, can not get unshareable values from instance variables of
#    classes/modules from non-main Ractors
```

When it comes to contants, Ractors can only access shareable contants from the outside scope.

```ruby
FROZEN = 'Frozen'.freeze
IMMUTABLE = 'Immutable'

Ractor.new { puts "Can access #{FROZEN}, but"; puts "cannot access #{IMMUTABLE}" }

# Can access Frozen, but
# => can not get unshareable values from instance variables of classes/modules from non-main Ractors
```

To check if an object is shareable or not, Ractor provides a method called `shareable?`,
which returns true if the object is shareable, otherwise false.
Using `make_shareable`, you can convert any object to a shareable form.

```ruby
Ractor.shareable?(1.0)                # => true
Ractor.shareable?('String')           # => false
Ractor.shareable?('String'.freeze)    # => true

data = ['foo', 1]
puts "#{data.frozen?}; #{data[0].frozen?}; #{data[1].frozen?}"
# false; false; true

Ractor.make_shareable(data)

puts "#{data.frozen?}; #{data[0].frozen?}; #{data[1].frozen?}"
# true; true; true
```

## Summary

1. Ractor was introduced in Ruby 3, and although it is still an experimental feature,
   it is a great alternative to threads to implement parallel programming in Ruby.
2. Ractors do not share objects from outside scope, thus implementing the Thread safety.
3. You can pass values to and from Ractor using `send`, `receive`, `yield` and `take`.
4. Whenever you pass data to a Ractor and if the data is non-shareable, it will be deep cloned by default,
   i.e., the data will be copied over. Ractor also provides a `move: true` option to move data if you do
   not wish to copy it. For example, `ractor.send(data, move: true)`.
   Once the data is moved, it will not be available in the outer scope.
5. Classes, modules, and other Ractors are shareable, but their instances are non-shareable.
   Class or instance attributes, depending on their datatype, may or may not be shareable.
6. You can check if an object is shareable or not using the `Ractor.shareable?` method and also make an
   object shareable using `Ractor.make_shareable`, which will freeze the object.
7. You can check how many Ractors are running by using `Ractor.count`.
8. You can add your custom method by calling `define_method` on Ractor. If you wish to remove your method, you can do it by
   calling `undef_method`.
   ```ruby
      Ractor.define_method('my_method') { puts 'Custom method called on Ractor' }
      ractor = Ractor.new { '' }
      ractor.my_method
      # Custom method called on Ractor
   ```
