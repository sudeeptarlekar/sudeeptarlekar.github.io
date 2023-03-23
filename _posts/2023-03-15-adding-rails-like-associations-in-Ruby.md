---
title: How to add Rails like associations in simple Ruby
date: 2023-03-15 18:25:00
categories: [Ruby]
tags: [Ruby, Rails, Metaprogramming]
---

![Image by StockSnap from Pixabay](/assets/article_images/2023_03_15_adding_rails_like_associations_in_ruby.jpg)

Rails associations are a way to define relationships between entities in a Ruby on Rails application.
They allow you to specify how different models are connected and how data can be accessed across
those relationships, making it easier to build complex applications with interconnected data.
Some examples of Rails associations include has_many, belongs_to, and has_and_belongs_to_many.

Association in Rails are powered by ActiveRecord gem for working with databases in the Ruby on Rails
applications. It also allows you to interact with a database through Ruby code,
without having to write any SQL statements. Associations include `has_one`, `has_many`, `belongs_to` etc.
> Take a quick look at [ActiveRecord](https://guides.rubyonrails.org/active_record_basics.html).

With these functionality we can add nice and layman readable associations between entities in our application.

Recently I was working on developing Ruby library and wanted to have something similar in my library, so that
I can associate the entities. But why I am trying to reinvent the wheel? Why not add available gem or library to solve
this problem? The reasons for that,
- For each dependency in the library, we have to qualify and prove the security tests which is a painful task. So more
  dependencies mean more qualification efforts we have to put in.
- Entities here have a unique identifier, so it will be nice if I reduce the effort of iteration, every time I am searching
  for an entity by identifier and have something similar to Hash datatype.
- High chance of facing license issues in the future.

So here we can do this with just a few lines of code and magic to Ruby's metaprogramming and blocks.

Ruby has a metaprogramming feature using which we can define methods at runtime. We can do this using ***define_method***
and ***define_singleton_method***.
> Find out more about `define_method` and `define_singleton_method`
[here](https://sudeeptarlekar.com/posts/define-method-and-define-singleton-method/)

So let's add a module called `Associable` and define the methods there.

```ruby
module Associable
  def belongs_to(method_name, **options)
    methods = []

    methods << <<~RUBY
      define_method("#{method_name}") { }
    RUBY

    methods << <<~RUBY
      define_method("#{method_name}=") do |value|
        define_singleton_method("#{method_name}") { value }
      end
    RUBY

    add_methods methods
  end

  def has_many(method_name, **options)
    methods = []

    methods << <<~RUBY
      define_method("#{method_name}") { {} }
    RUBY

    methods << <<~RUBY
      define_method("assign_#{method_name}=") do |value|
        new_val = self.send("#{method_name}").merge({ value.identifier => value })
        define_singleton_method("#{method_name}") { new_val }
      end
    RUBY

    add_methods methods
  end

  private

  def add_methods(methods)
    methods.each { |met| class_eval met }
  end
end
```

As you can see, we are defining two methods here `belongs_to` and `has_many`. Now, whenever you want
to add associations to an entity you just have to add a module to this class.


Let's test our code quickly and see if it is working according to our expectations.

```ruby
class TransportCompany
  extend Associable

  has_many :carriers
end

class Carrier
  extend Associable

  attr_accessor :identifier

  belongs_to :company

  def initialize
    @identifier = "M1234"
  end
end

company = TransportCompany.new
company.carriers
# => {}

carrier = Carrier.new
carrier.company
# => nil

company.assign_carrier = carrier
company.carriers
# => { 'M1234' => #<Carrier:0x00007fe2aa871420 @identifier="M1234"> }

carrier.company = company
carrier.company
# => #<TransportCompany:0x00007fe2ab130cc8>
```

> The above code is just for demonstration purposes and a few things can be improved such as good method
names for adding associated objects, adding circular dependency between objects being associated, and mainly,
memory optimization. You can also pass other options if you want to check for the type.

This is a glimpse of what you can do with Ruby's metaprogramming and how you can create rails-like associations with
just a few lines in Ruby.
