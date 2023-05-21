---
title: Enhancing Test Expressiveness with Custom Matchers in RSpec
date: 2023-05-20 14:00:00
tags: [Ruby, RSpec, Testing]
description: |
             Discover the power of custom matchers in RSpec, a popular Ruby testing framework.
             Learn how custom matchers can improve the expressiveness and readability of your tests,
             enabling more effective code validation
categories: [Ruby]
---

## Introduction

RSpec, a widely-used testing framework for Ruby, empowers developers
to create comprehensive tests for their codebase.
Among its impressive features, RSpec offers the flexibility to build
custom matchers, allowing for more expressive and readable tests.
Custom matchers allow you to define your own assertions that can be
used in your tests.
This can be especially useful when testing complex objects or behaviors
that don't have built-in matchers.

In this article, we will explore the benefits of leveraging custom
matchers in RSpec and how they can enhance your testing experience.

## Scenario

Imagine a scenario in Ruby where we have a module that includes important
logic and instance variables in an included class.
Our application relies heavily on these variables, making it crucial to
ensure their presence within the included class.

```ruby
module Identifiable
  attr_reader :identifier
  attr_accessor :last_change, :content

  ...
end

class DataFile
  include Identifiable
end
```

Now, in order to check if class has attribute reader `identifier`,
we can write following test case.

```ruby
# data_file_spec.rb

RSpec.describe DataFile do
  describe 'Identifiable' do
    it 'includes required methods' do
      expect(described_class.new).to respond_to(:identifier)
      expect(described_class.new).not_to respond_to(:identifier=)
    end
  end
end
```

As you can see, with this approach, we would have to write multiple lines
of code just to test a few variables and methods.
This is where we can introduce our custom matcher to simplify our task
and enable us to write clean and readable test cases.

## Writing Very First Custom Matcher

RSpec matcher consists of three major parts, as follows

1. `match`, which accepts the block with comparison logic.
2. `failure_message`, block to write custom logic when test case fails.
3. `failure_message_when_negated`, as name suggests, when negated test
   case fails, this block will be executed.

With this knowledge, let's write our first matcher.

```ruby
# spec/matchers/attr_reader.rb

require 'rspec/expectations'

RSpec::Matchers.define :have_attr_readers do |attrs|
  match do |klass_instance|
    attrs.map do |attr|
      !klass_instance.respond_to?("#{attr}=") &&
        klass_instance.respond_to?(attr)
    end.all?
  end

  failure_message do |klass_instance|
    "#{klass_instance} expected to have all #{attrs} as an `attr_reader`."
  end
end
```

> Do not forget to require above file in the `spec_helper.rb`

With our code in place and configured, we can now update our test cases as

```ruby
# data_file_spec.rb

RSpec.describe DataFile do
  describe 'Identifiable' do
    it 'includes required functionality to the class' do
      expect(described_class.new).to have_attr_readers([:identifier])
    end
  end
end
```

## Best Practices When Using Custom Matchers

1. Even though you can directly write matchers in the `spec_helper.rb`,
   prefer to save all matchers under `spec/support/matchers/` and import
   them in `spec_helper.rb`.
   This approach makes it easier to maintain multiple matchers.
2. Maintain a separate file for each matcher.
3. While providing failure messages is optional, it's good to have
   custom-tailored messages.
4. Ensure that your match block returns a boolean value.

## Summary

Custom matchers in RSpec offer developers a powerful tool to enhance
the expressiveness and readability of their tests.
By creating tailored assertions, developers can validate code functionality more
effectively, leading to improved code comprehensionand maintainability.
Embrace the potential of custom matchers in RSpec to elevate your
testing practices and drive confidence in your Ruby projects.

RSpec matcher consists of following blocks,

1. `match`(required), comparison logic between actual and expected data,
    and must return boolean value.
2. `failure_message`(optional), provides custom message when test case fails.
3. `failure_message_when_negated`(optional), custom message when negated test
   case fails.
