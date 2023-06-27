---
layout: post
title: Memory Store and the Repository Pattern
crawlertitle: Repository pattern with memory store
summary: Development simplified with the repository pattern and memory store
date:   2023-05-23
categories: posts
tags: ['design patterns']
bg: "post-repository-pattern.jpg"
---

I was recently tasked with implementing a new "greenfield" feature, unreliant for the most part on the rest of the application. During the planning and prototyping phase of development, it was unclear as to the direction the data would take, the focus being the business logic required to handle our inputs, perform the necessary processing, and the required outputs. This meant I was unable to preselect the database technology that would be most suitable but was that even a decision needed at this point?

## The Data Source is but a Detail

A wise man once told me that the "database is just a detail." By "wise man," I am referring to Uncle Bob, and by "once told me," I am referring to a talk he gave, available on YouTube(1). We all too often prioritise what database to use even before a solid idea of what to build is understood, which can lead one to shoehorn their application to later fit within the limits of the data source solution initially chosen. There is a clear divide between how data is stored and how data is processed, and we should focus on the application's business logic before any concrete decisions on how we store that data is made. This is where the implementation of the Repository Pattern comes in useful.

## The Repository Pattern

The main idea behind the Repository Pattern is to introduce a layer between your business logic and database that allows you to easily swap out your data layer for something else. A classic use case for this would be, in production, to use your persisted MySQL / MongoDB / whatever database (obviously), but when running tests during development where data is transient, there is no need to truly persist it, so therefore no need to leverage an entire database solution which is inherently slow. We could just store our data in memory instead. To fully take advantage of the Repository Pattern in your application, it would need to be designed and built with it in mind. Very often, one gets involved in a large mature application that has been built "the Rails way," so fully implementing the repository pattern is not practical or viable. However, certain aspects and concepts of it can be.

## Memory Models and the Memory Store

All data at the end of the day can be represented by a key-value pair, also known as a hash or dictionary. A database model can have a field (e.g. "name") and an attribute (e.g. "Bobby"). With this basic knowledge in mind, we can set up the base of our memory "database" by initialising, in this case, a constant, which will store and manage all our data.
```ruby
MEMORY_STORE = Hash.new { |hash, key| hash[key] = [] }
```
An alternative approach would be to make use of a singleton object as our store using the Singleton Pattern (although the case for singletons is hotly debated and not one I wish to deal with here). There are, of course, many ways to skin a cat. Once we have our empty data store persisted in memory, we can implement the common interface methods one would use when talking to a data source, in our case, through ActiveRecord, but the same can be done for any ORM you wish to use;

```ruby
module Mem
  module MemoryStore
    extend ActiveSupport::Concern

    included do
      @entity = self::Entity

      @entity.define_method(:attributes) do
        instance_values.with_indifferent_access
      end

      @entity.define_method(:update) do |**attributes|
        attributes.each do |key, value|
          key = key.to_s + "="
          self.public_send(key.to_sym, value)
        end
      end
    end

    class_methods do
      def all
        instance
      end

      def create(**attributes)
        index = instance.size + 1
        obj = @entity.new(attributes)
        obj.id = index
        instance << obj
        obj
      end

      def where(**attributes)
        instance.select do |item|
          attributes.map do |key, value|
            item.send(key) == value
          rescue
            false
          end.uniq == [true]
        end
      end

      def destroy(id)
        instance.delete_if { |entity| entity.id == id }
      end

      private

      def instance
        MEMORY_STORE[@entity.to_s]
      end
    end
  end
end
   ```
The Memory Store above is a simplified version of the complete code. It can easily be extended to include the common `find,` `find_by,` `count,` `destroy_all,` etc., methods that are provided by your ORM, as you need them. With reference to the Repository Pattern, this is essentially the InMemory Repository object.
We can now create our model to use this memory repository. The following example is an extremely simplified version of our Project model, which stores all sorts of information on projects. Still, for our purposes and the functionality we are writing, we do not need to replicate the entire ActiveRecord model with all of its attributes.

```ruby
module Mem
  class Project
    class Entity
      attr_accessor :uid, :name

      def initialize(**attributes)
        @uid = attributes[:uid] || SecureRandom.hex(3)
        @name = attributes[:name] || Faker::Name.name
      end
    end

    include MemoryStore
  end
end
   ```
With our model now implemented and including the MemoryStore logic, we can now speak to our project model as we would with the ActiveRecord version.

```ruby
# ActiveRecord
Project.all
Project.create(name: "SomeName")
Project.where(name: "SomeName")

# MemoryModel
Mem::Project.all
Mem::Project.create(name: "SomeName")
Mem::Project.where(name: "SomeName")
```````
## Putting it into Action

Given that we now have our memory model version, how could we structure our code to allow us the flexibility of choosing between it or the production model? It's quite common for coders to implicitly use their data models inside of their business logic use case classes but that couples your code with the database layer. Instead, by passing in the model we wish to act on (a.k.a Dependency Injection), we allow ourselves the flexibility of specifying what model we want to use at run time (pseudocode below).

```ruby
module ProjectBusinessLogic
  class Create
    def initialize(model: Project)
      @model = model
    end

    def call(attributes = {})
      @model.create(attributes)
    end
  end
end
```
By defaulting the initialisation of our class to use our production ORM model, we can call this class as we would expect to.

```ruby
ProjectBusinessLogic::Create.new.call(attributes)
   ```
Alternatively, when performing tests, we would not want to leverage the entire database, as reading and writing to the database is slow going. Instead we could can inject in our memory model at runtime.

```ruby
ProjectBusinessLogic::Create.new(model: Mem::Project).call(attributes)
``````
## But

But doesn't maintaining two different models (one ActiveRecord and one Mem for instance) lead to extra maintenance overhead? What if our ActiveRecord models are complicated with many associations, uploaders, integrations with external services, etc? What if we need to execute raw SQL code?

These are all valid questions. The memory model concept utilised here were focused on getting business logic working on a greenfield project with little interaction with the mature system. It is not something that is sustainable at this point; however, when presented with an opportunity to develop a new application from the ground up, the ideas explored here could help altering the design of our models and classes to better support this repository pattern with dependency injection workflow, resulting in cleaner, DRY-er and more maintainable code, not forgetting speedier tests.

If you would like to further your reading and experiment with various implementations of the Repository Pattern, using DRY methodologies, I recommend taking a glance at dry-rb and rom-rb. If you're interested in a Rails-esque framework designed with these patterns in mind, take a look at Hanami.

## References

1. [Uncle Bob's 2011 Keynote Speach](https://youtu.be/WpkDN78P884?t=2502)
2. [8th Light Blog](https://8thlight.com/blog/mike-ebert/2013/03/23/the-repository-pattern.html)
3. [Ruby Singleton Design Pattern](https://www.rubyguides.com/2018/05/singleton-pattern-in-ruby/)
4. [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection)
5. [dry-rb Ruby Libaries](https://dry-rb.org/)
6. [Ruby Object Mapper](https://rom-rb.org/)
7. [Hanami Web Framework](https://hanamirb.org/)
