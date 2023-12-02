---
title: initialize_clone and initialize_dup in Ruby
date: 2023-10-11 10:00:00
categories: [Ruby]
tags: [Rails, Ruby]
---

## Introduction

In the realm of Ruby, the duplication of objects can be achieved through
the utilization of the `clone` or `dup` methods offered by the Object class.
While the `dup` method generates a new instance based on the class of the
descendant object, the `clone` method, in contrast, produces an object
encompassing an internal state.
The ensuing example offers a swift insight into the functioning of the `clone`
and `dup` methods.

```ruby
class Person
  def initialize
    @name = 'Sam Jose'
  end
end

module NameHelper
  def print_name
    puts @name
  end
end

person = Person.new
person.extend(NameHelper)
person.print_name           #=> Sam Jose

clonned_person = person.clone
clonned_person.print_name   #=> Sam Jose

dupped_person = person.dup
dupped_person.print_name    #=> NoMethodError: undefined method `print_name` for #<Person:0x006>
```

When invoking the `clone` or `dup` methods on an object,
they internally trigger either `initialize_clone` or `initialize_dup`.
These methods take the old object as an argument, allowing
us to customize the new object being generated based on the
characteristics of the old object.
This enables a flexible and dynamic approach to object duplication in Ruby.

## TL;DR

This becomes valuable when there is a need to update the
internal state of an object during duplication or cloning.
The subsequent example provides a straightforward implementation
to illustrate how these methods can be employed for customizing
object duplication according to specific requirements.

```ruby
class Counter
  attr_reader :duplicate_count, :clone_count

  def initialize
    @duplicate_count = 0
    @clone_count = 0
  end

  def initialize_dup(old)
    self.instance_variable_set('@duplicate_count', old.duplicate_count + 1)
    initialize_copy(old)
  end

  def initialize_clone(dup)
    self.instance_variable_set('@clone_count', old.clone_count + 1)
    initialize_copy(old)
  end
end

counter = Counter.new

counter2 = counter.dup    #=> #<Counter:0x0938 @clone_count=0, @duplicate_count=1>
counter3 = counter.clone  #=> #<Counter:0x0b00 @clone_count=1, @duplicate_count=0>
```

## Internal implementation

There is not clear documentation on how these methods behave internally,
but more or less following is the psuedo code.

```ruby
class Object
  def dup
    dup = self.class.allocate
    dup.copy_instance_variables(self)
    dup.initialize_dup(self)
    dup
  end

  def clone
    clone = self.class.allocate
    clone.copy_instance_variables(self)
    clone.copy_singleton_class(self)
    clone.initialize_clone(self)
    clone.freeze if frozen?
    clone
  end
end
```

## Summary

- Ruby implements `initialize_dup` and `initialize_clone` methods which
  allows user to update internal state of object being duplicated.
- These functionalities are integrated into subsequent releases of Ruby 1.8.
- Both of these methods invoke `initialize_copy`, and it is considered best
  practice to include a call to `initialize_copy` when overriding these methods.
