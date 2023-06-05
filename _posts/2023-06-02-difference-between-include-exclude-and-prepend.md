---
title: Difference between extend, include and prepend in Ruby
date: 2023-06-02 10:00:00
categories: [Ruby]
tags: [Rails, Ruby]
---

## Introduction

In Ruby, one can implement multiple inheritance using a concept called Mixins.
Mixins allow you to include methods and behaviors from multiple modules into a class,
enabling code reuse and enhancing the flexibility of your program's structure.

In Ruby, classes are allowed to have only one superclass, meaning they can inherit from a single class.
However, this limitation can be overcome with the concept of Mixins.
Mixins are modules that contain a set of methods, constants, and other behaviors that can be included
in a class using the `include`, `extend`, and `prepend` keywords.

In this blog post, we will explore how these keywords work.
For simplicity we will consider following code snippent throughout this blog post.

```ruby
# mixins.rb

module Bar
  def bar
    "#{foo} from Bar#bar"
  end
end

class Foo
  def self.foo
    'Foo.foo called'
  end

  def foo
    'Foo#foo called'
  end
end
```

## extend

The `extend` keyword is used to add methods from module to specific object or a class itself.
`extend` adds the module's methods directly to the class or object itself,
making them accessible as class or singleton methods.
Let's see how `extend` works with following code sample.

```ruby
# mixins_test.rb

require 'mixins'

foo1 = Foo.new
foo2 = Foo.new

foo1.extend Bar

puts foo1.bar      # => Foo#foo called from Bar#bar
puts foo2.bar      # => NoMethodError
```

As you can see, on line#13 we are adding the method from `Bar` module
as a singleton method in Foo class instance.
But we can add all methods from module as a class methods by calling `extend` on class or adding a line
in class definition as follows.

```ruby
Foo.extend Bar

# or

class Foo
  extend Bar
  ...
  ...
end
```

## include

When a module is included in a class, its methods become instance methods of that class and
same method will be available in all active instances of class.
This allows the class instances to access and use those methods.
Following example shows how you can include the module in the class.

```ruby
# mixin_test.rb

foo1 = Foo.new
foo2 = Foo.new

puts foo1.bar # => NoMethodError
puts foo2.bar # => NoMethodError

Foo.include Bar

puts foo1.bar # =>  Foo#foo called from Bar#bar
puts foo2.bar # =>  Foo#foo called from Bar#bar
```

## prepend

With `prepend`, we can add a method from a module as an instance method in a class,
but `prepend` works a bit differently.
When we `prepend` a module to a class, Ruby first searches for method definitions within
that module before searching the class itself.
This means that methods defined in the prepended module take precedence over methods
defined within the class.
This behavior is also known as method overriding or method aliasing.
The prepend method is used to add a module at the beginning of the class's ancestor chain,
which alters the order in which methods are looked up.

In order to make this concept clear, we will use `ancestor` method on class to list
all the ancestors and change our method name in `Bar` module from `bar` to `foo` as follows.

```ruby
# mixin.rb

module Bar
  def foo
    "#{super} called from Bar#foo"
  end
end

# mixin_test.rb

foo = Foo.new

puts "#{foo.foo}\n"         # => Foo#foo called
print "#{Foo.ancestors}\n"  # => [Foo, Object, Kernel, BasicObject]

Foo.prepend Bar

puts "#{foo.foo}\n"         # => Foo#foo called from Bar#bar
print Foo.ancestors         # => [Bar, Foo, Object, Kernel, BasicObject]
```

As you can see on line 18, calling foo on a class instance executes the method from
the module instead of the class.
The reason for this is that, as shown on line 19, the module Bar is added at the
beginning of the ancestor chain.
Therefore, the method from the module is given preference over the method from the class.

## Summary

1. In Ruby multiple inheritance can be achieved by concept called mixins module and,
   calling `include`, `extend` and `prepend` on class.
2. With `extend`, methods from module are added into class as class methods or
   can be added as singleton methods on instance.
3. With `include`, methods from module are added into class as instance methods and
   available to instances of the class.
4. `prepend` adds module in front of the ancestoral chain of class, thus aliasing the
   methods from class.
