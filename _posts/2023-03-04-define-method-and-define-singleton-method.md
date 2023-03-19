---
title: What is the difference between define_method and define_singleton_method?
date: 2023-03-04 14:00:00
categories: [Ruby]
tags: [Ruby, Rails, Metaprogramming]
---

# What is the difference between Ruby's define_method and define_singleton_method?
![Image by StockSnap from Pixabay](/assets/article_images/2023-03-04-define-method-and-define-singleton-method/blog_image.jpg)

Ruby is a popular object-oriented programming language known for its expressiveness and flexibility.
One of the unique features of Ruby is its support for metaprogramming,
which allows developers to write code that can modify and extend the behavior of a program at runtime.
Metaprogramming in Ruby involves using techniques such as defining methods dynamically,
modifying classes and objects, and leveraging the reflective capabilities of the language.
In this blog post, we will explore the ***define_method*** and ***define_singleton_method***
Both these methods accept the method name as an argument and block to define the functionality or return
value, but there is a slight difference between them.

Let's see how these two methods work differently

## define_method
With ***define_method*** you can add the method dynamically in a class instance.
The ***define_method*** method takes two arguments: the name of the new method to be defined as a symbol or a string,
and a block that contains the code to be executed when the new method is called.
Let's see an example to understand this

```ruby
class Dummy
  define_method 'my_method' do
    puts "Executing 'my_method'"
  end
end

dummy = Dummy.new
dummy.my_method
# => Executing 'my_method'
```

The key point here I would like to make is that ***define_method*** is only available at the class
level and if you try to create a method from the instance you will end up getting an error.

However, it is important to use ***define_method*** with caution, as it can also lead to
unexpected behavior if not used properly. Let's understand this with following example.

```ruby
class Car
  def self.build(color)
    define_method 'start_building' do
      puts "Starting a new car build with a color #{color}"
    end
  end
end

red_car = Car.new
blue_car = Car.new

red_car.class.build('Red')
red_car.start_building
# => Starting a new car build with a color Red

blue_car.class.build('Blue')
blue_car.start_building
# => Starting a new car build with a color Blue
```

If you observe closely, our `build` is a class method, why is that so? This is because ***define_method*** is only available
as a class method and not as an instance method, if you convert the `build` as an instance method, you will end up
getting a **NoMethodError**.

So far, everything is looking good, but when you call `start_building` on `red_car` again you will see some issues with output.

```ruby
red_car.start_building
# => Starting a new car build with a color Blue
```

You can fix by moving color parameter as follows

```ruby
define_method 'start_building' do |color|
  puts "Starting a new car build with a color #{color}"
end
```

## define_singleton_method
With ***define_singleton_method*** as well we can create methods at the runtime, but this function works a bit differently,
more on this later but first, let us see how we can create a method using ***define_singleton_method***.

```ruby
class Car
  def build(color)
    define_singleton_method 'start_building' do
      puts "Starting a new car build with color #{color}"
    end
  end
end

red_car = Car.new
blue_car = Car.new

red_car.build('Red')
red_car.start_building
# => Starting a new car build with a color Red

blue_car.start_building
# => NoMethodError: undefined method 'start_building' for Car:Class

blue_car.build('Blue')
blue_car.start_building
# => Starting a new car build with a color Blue
```
Two things I would like to point out here are
- We have to explicitly call ***build*** on each object, without this we will get an error when calling the dynamic method.
- ***define_singleton_method*** is available as an instance, as well as a class method, however,
  the level of the parent class, defines the
  level of the dynamic method, compare the following dummy code snippets to understand this

```ruby
class DummyKlass
  # Method will be added as a class method
  define_singleton_method 'my_method' do
    "Executing 'my_method'"
  end
end

class DummyInstance
  def build
    define_singleton_method 'my_method' do
      "Executing 'my_method'"
    end
  end
end

klass_test = DummyKlass.new
klass_test.my_method
# => undefined method `my_method' for #<DummyKlass:0x0000000113202e48> (NoMethodError)

klass_test.class.my_method
# => "Executing 'my_method'"

instance_test = DummyInstance.new
instance_test.my_method
# => "Executing 'my_method'"

instance_test.class.my_method
# => undefined method `my_method' for DummyInstance:Class (NoMethodError)
```

- `DummyKlass` is executing ***define_singleton_method*** at a class level, which adds **my_method** as a
class method.
- `DummyInstance` is executing ***define_singleton_method*** from an instance method, which
adds **my_method** as an instance method and not as a class method.

## Summary
- We can create methods in Ruby at runtime using ***define_method*** and ***define_singleton_method***
- ***define_method*** has to be executed by class and it creates the instance method.
- ***define_singleton_method*** can be called from class or from class instance. If it is called from class then
  new method will be added as a class method and if called from an instance, then new method will be added as an
  instance method, to only an instance executing it.
- Be cautious while adding defining methods as this may lead to the unintended behaviour.
