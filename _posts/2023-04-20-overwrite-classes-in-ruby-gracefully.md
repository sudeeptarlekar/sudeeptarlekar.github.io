---
title: Overwrite classes in Ruby gracefully
date: 2023-04-20 15:30:00
categories: [Ruby]
tags: [Rails, Ruby]
---

![Image](/assets/article_images/2023_04_20_overwrite_gracefully.jpg)

## Introduction
In the Ruby world, everything is an object and is represented by classes.
This makes it easy to tweak the functionality of certain classes based on your needs.
In real life, this is called monkey patching. However, when you add a monkey patch to a class,
it may cause disruption in other places where the new functionality is not required,
because the modification that you add is applied globally.

Therefore, it is always a good practice to overwrite certain functionality only when you need it.
In Ruby, you can do this using `refine`, which provides a way to apply modifications locally.

Here is how it looks like

```ruby
module MyWorld
  refine String do
    def length
      puts "My length is: #{super}"
    end
  end
end

using MyWorld

'Hello World!'.length
# => My length is: 12
```

## Scoping in Refinements
Refinements are lexical in scope, meaning they will only be activated within the scope
in which they are called using the `using` keyword. Once control is transferred to an outside scope,
the refinements are deactivated. Therefore, if you call or require a refined method from outside
of the current scope, the refinement will be deactivated.

```ruby
class Foo
end

module Refinement
  refine Foo do
    def call
      puts "Hello from Foo#call"
    end
  end
end

def main(obj)
  obj.call
end

using Refinement

obj = Foo.new
obj.call
# => Hello from Foo#call

main(obj)
# => undefined method `call' for #<Foo:0x00007faa340ef980> (NoMethodError)
```
As you can see, we activate the refinement on line #16 and call a method on an object of `Foo`,
which prints a statement for us. However, when we call `main`, the scope changes,
and the refinement is not activated inside `main`, so we get a **NoMethodError**.

Note that if you activate a refinement in a file and then require that same file in other modules,
the refinements will not be available there. Let's quickly test this with the following code.

```ruby
# refinement.rb
module Refinement
  refine String do
    def greet
      puts "Hello #{self}!"
    end
  end
end

using Refinement

class Test
  def call(obj, method)
    obj.send method
  end
end

# main.rb
require './refinement'

Test.new.call('John', :greet)
# => Hello John!

'John'.greet
# => undefined method `greet' for "John":String (NoMethodError)
```

As you can see that the refinement that we activated in **refinement.rb** is not
available in **main.rb**

> When a module includes multiple refinements, and they are activated using the `using` keyword,
  all the refinements from the module become active.

When you add refinements it is added to the ancestors
To demonstrate this take a look at the following code sample.

```ruby
module Foo
  def self.call
    refine String do
      def count
        length
      end
    end
  end
end

sample_test = Foo.call

sample_test.class
# => Module

sample_test.ancestors
# => [#<refinement:String@Foo>, String, Comparable, Object, Kernel, BasicObject]
```

## Method Lookups

When you call the refinement, Ruby searches for method from class in reverse order, as follows:
1. Modules which are prepended to the class.
2. Refinements of the class
3. Included modules from refinements of the class

Note that when you call a refined method, such as `x` in a class,
which is referred to by another method, such as `y` in the same class,
when you call `y` on an object of the class,
`y` will refer to the original method `x` and not the refined method.
Check the code sample below to understand this.

```ruby
class Foo
  def call
    post
  end

  def post
    puts 'Original: Hello from Foo#post'
  end
end

module Refinement
  refine Foo do
    def post
      puts 'Overwritten: Hello from Foo#post'
      super
    end
  end
end

using Refinement

Foo.new.call
# => Original: Hello from Foo#post

Foo.new.post
# => Overwritten: Hello from Foo#post
# => Original: Hello from Foo#post
```

## Changes in Ruby3.0

1. 3.1.0

    Ruby 3.1.0 add many more features in refinements, as you can combine multiple methods from other
    modules using `import_methods`.

    ```ruby
    module Utilities
      def greet
        "Hello #{self}!"
      end
    end

    module Bar
      refine String do
        import_methods Utilities
      end
    end

    using Bar
    'John'.greet
    ```

2. 3.2.0

   Ruby 3.2.0 adds a great way to check if any refinements are being used by calling `used_refinements` on `Module`.

   ```ruby
   module Utilities
     refine(String) { def count = length }
   end

   Module.used_refinements # => []
   using Utilities
   Module.used_refinements # => [#<refinement:String@Utilities>]
   ```


## Summary
1. Refinements are a great way to add monkey patching to Ruby code without causing any side effects.
2. The argument to the `refine` should be class, and refinements are activated by calling `using`.
3. Refinements follows lexical scoping and are activated only in the current scope.
   They are deactivated outside the current scope.
4. A file with an active refinement will not activate the refinement when imported.
5. Methods are searched in a reversed order when called.

## Further Reading

1. [Refinements in Ruby](https://docs.ruby-lang.org/en/2.4.0/syntax/refinements_rdoc.html)
2. [Ruby-Doc](https://ruby-doc.org/core-3.1.0/Refinement.html)
