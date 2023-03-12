---
title: What is the difference between define_method and define_singleton_method?
date: 2023-03-04 14:00:00
categories: [Ruby]
tags: [Ruby, Rails, Metaprogramming]
---

# What is the difference between Ruby's define_method and define_singleton_method?
![What is difference](/assets/img/carbon.png)

Ruby's metaprogramming is bliss. It is also the most important factor of the Ruby programming language
which empower strong web framework like Rails. Metaprogramming in Ruby gives the programmer the ability
to create the methods at runtime.
Today we will discover how you can achieve this by **define_method** and **define_singleton_method**.
Both these methods accept the method name as an argument, but there is
a slight difference between them. 
Let's see how these two methods work differently

## define_method
You can define a method in Ruby using ***define_method*** as follows
```ruby
define_method 'my_method' do
  puts "Executing 'my_method'"
end
```

Let's create a dummy class and define a method at the runtime, as follows
```ruby
class Car
  def self.build(color)
    define_method 'start_building' do
      puts "Starting a new car build with a color #{color}"
    end
  end
end

obj = Car.new
obj.class.build('Red')
obj.start_building
```

When you execute the code above you will see our message from a method printed on the screen
> Starting a new car build with a color Red

If you observe closely our `build` is a class method, why is that so? This is because ***define_method*** is only available
as a class method and not as an instance method, if you convert the `build` as an instance method, you will end up
getting a NoMethodError.
Another thing you may find weird is, why I created an object first when a method is available to a class. This is where I would like to make
a point at the key difference between these two methods.

```ruby
red_car = Car.new
blue_car = Car.new

red_car.class.build('Red')
red_car.start_building
# => Starting a new car build with a color Red

blue_car.class.build('Blue')
blue_car.start_building
# => Starting a new car build with  acolor Blue
```
So far everything is looking good, but when I call `start_building` on `red_car` again I see some issues with output.
```ruby
red_car.start_building
# => Starting a new car build with a color Blue
```
How color for `red_car` got replaced from **red** to **blue**? As I stated earlier **define_method** is common for all instances of
the class. So let's fix this by passing a parameter to the method
```ruby
class Car
  def self.build
    define_method 'start_building' do |color|
      puts "Starting a new car build with a color #{color}"
    end
  end
end

red_car = Car.new
blue_car = Car.new

red_car.class.build

red_car.start_building('Red')
# => Staring a new build with a color Red
blue_car.start_building('Blue')
# => Starting a new build with a color Blue
```

## define_singleton_method
With this method as well we can create methods at the runtime, but this function works a bit differently, more on this later but first, let us see how we can create a method using ***define_singleton_method***.
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
- We have to explicitly call `build` on each object, without this we will get an error when calling the dynamic method.
- `define_singleton_method` is available as an instance as well as a class method, however, the level of the parent class defines the
level of the dynamic method, compare the following code snippets to understand this

```ruby
class Car
  def self.build(color)
    define_singleton_method 'start_building' do
      puts "Starting a new car build with color #{color}"
    end
  end
end

red_car = Car.new
blue_car = Car.new

red_car.class.build('Red')
red_car.class.start_building
# => Starting a new build with a color Red

blue_car.class.start_building
# => Starting a new build with a color Red
```
As you can see above that, `start_building` is a class method.

```ruby
class Car
  def build(color)
    define_singleton_method 'start_building' do
      puts "Starting a new build with a color #{color}"
    end
  end
end

red_car = Car.new
blue_car = Car.new

red_car.build('Red')
red_car.start_building
# => Starting a new build with a color Red

blue_car.build('Blue')
blue_car.start_building
# => Starting a new car build with a color Blue
```

## Summary
- We can create methods in Ruby at runtime using ***define_method*** and ***define_singleton_method***
- ***default_method*** is available as a class method while ***define_singleton_method*** is available as a class as well as an instance method, and if the parent method is a class method then a dynamic method will be a class method and vice versa for an instance method.
- ***define_method*** adds a dynamic method to all instances of the class defining it, while ***define_singleton_method*** as the name suggests adds a method to only a single instance of a class.
