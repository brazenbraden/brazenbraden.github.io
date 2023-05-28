---
layout: post
title: Better Sidekiq Classes
crawlertitle: Sidekiq but testable
summary: Better Sidekiq classes
date: 2023-05-24
categories: posts
tags: ['sidekiq']
bg: "post-sidekiq.jpg"
---

[Sidekiq](https://github.com/sidekiq/sidekiq) is pretty much the go-to solution for enqueuing jobs for background processing when working on a Ruby-based project. It's simple to implement, has a clear DSL, and is well-supported by common testing frameworks like RSpec. That being said, and I may be nitpicking, but the standard implementation examples do add a little extra overhead if you wish to be more flexible with your job business logic and a little extra test setup is required.

## The "normal" way

Given we want to have a little background job that updates a user's name and some score and any other business logic needed, we could create a standard Sidekiq job with that logic and call it from our code as such:
```ruby
class UpdateUserJob
  include Sidekiq::Job

  def perform(user_id, name, score)
    user = User.find(user_id)
    user.update!(name: name, score: user.score + score)
    # and maybe do some more stuff?
  end
end

UpdateUserJob.perform_async(12, "bob", 5) # perform the job as soon as possible
UpdateUserJob.perform_in(5.minutes, 12, "bob", 5) # delay the processing
```
This is simple enough. But what if we want to test the logic being performed inside this job? Firstly, we'd need to update our `spec_helper` to let it know what to do with a job. If you're using RSpec, you might even need to add the [rspec-sidekiq](https://github.com/philostler/rspec-sidekiq) gem for additional configuration and matcher options. A classic-looking spec for this job might look something like this:
```ruby
it "updates the user" do
  UpdateUserJob.perform_async(1, "bob", 5)
  expect(UpdateUserJob.jobs.size).to eq(1)
  UpdateUserJob.drain
  expect(User.find(1).name).to eq("bob")
end
```
We need to `drain` the job before we can test the result of its work. Alternatively, we could wrap the test inside one of the Sidekiq testing helper methods:
```ruby
it "updates the user" do
  Sidekiq::Testing.inline! do
    UpdateUserJob.perform_async(1, "bob", 5)
    expect(User.find(1).name).to eq("bob")
  end
end
```
This does work as expected however it does leave a little to be desired. We have to remember to do this whenever testing job logic (the number of times I've had to search through previous specs to remind myself of the syntax), it adds a few extra lines to the tests which are not necessary and there is a spec performance hit as well.

## The "use case" way

So, what options do we have? Well, the first option would be to extract the logic performed by the job into a class which we would then pass the job attributes to for processing. That way we can independently test the business logic without having to mess with background workers and the like. I have used various names for these classes, **Use Case**, **Resource**, or **Library** to name a few (use whatever best suits you and your conventions). Our new job would look something like this:
```ruby
class UpdateUserJob
  include Sidekiq::Job

  def perform(user_id, name, score)
    UpdateUserUseCase.new(user_id, name, score).call
  end
end

class UpdateUserUseCase
  def initialize(user_id, name, score)
    @user_id = user_id
    @name = name
    @score = score
  end

  def call
    user = User.find(@user_id)
    user.update!(name: @name, score: user.score + @score)
    # ...
  end
end
```
Now, when it comes to testing, we could just forgo testing the actual job class as it does nothing except pass on some arguments to a plain old Ruby class. We can just test the class directly. Sidekiq is also already extensively tested so there is little point in testing whether the jobs on the queue have increased or not unless you are expecting new jobs to be created as a knock-on effect on your business logic.
```ruby
it "updates the user" do
  UpdateUserUseCase.new(1, "bob", 5).call
  expect(User.find(1).name).to eq("bob")
end
```
The added benefit to doing it this way is that you now also have a plain old Ruby class that you can call independently and inline in your code elsewhere when the need arises. For instance, you may want to defer the processing using the job when you are handling a mass import of data, however, when being performed as a single HTTP request to your API, you wish to do it inline (although you probably do not want to be sending emails inline given our current example).

The downside to this is that we now, however, have had to create an additional class to provide this flexibility. That means, if we wish to keep this convention going, for each Job class, there would be a corresponding UseCase class. What if we could take this idea one step further? What if we could turn our Job class into a plain old Ruby object right from the start? Let's rewrite our `UpdateUserJob` class.

## The "runner" way (with POROs)

Let's pretend you had never heard of Sidekiq or background queues, and you want to write some code to perform a specific task. For the sake of convention, I have ended the following class name with "Job" to match previous examples however by syntax definition, it is no longer what we would call a classic "job". It is now just a simple Ruby class (identical to our previous `UpdateUserUseCase`).
```ruby
class UpdateUserJob
  def initialize(user_id, name, score)
    @user_id = user_id
    @name = name
    @score = score
  end

  def call
    user = User.find(@user_id)
    user.update!(name: @name, score: @user.score + @score)
    # ...
  end
end
```
With that in place, we need to set up some sort of mechanism to have this code be run by Sidekiq in our background queue. For that, we can introduce a simple class to wrap the sidekiq calls, allowing us to inject our custom job class with its arguments. To not conflict with the Sidekiq namespace, I have created the `Runners` module which will run this code for me (side note: no matter what project I've worked on in the past, I always seem to use these "runner" style classes for various things).
```ruby
module Runners
  class Sidekiq
    include ::Sidekiq::Job

    sidekiq_options(retry: 3, backtrace: true, queue: "default")

    def perform(job_class_name, *args)
      Kernel.const_get(job_class_name).new(*args).call
    end

    def run(job_class, *args)
      self.class.perform_async(job_class.to_s, *args)
    end

    def run_in(delay_in_seconds, job_class, *args)
      self.class.perform_in(delay_in_seconds, job_class.to_s, *args)
    end
  end
end
``````
And with that, we have everything we need to be able to throw plain old Ruby classes into the background for processing. If we wish to run our business logic inline, we simply run
```ruby
UpdateUserJob.new(12, "bob", 5).call
```
but if we want to delegate the processing to the background queue, we can send it off immediately or delay it like this
```ruby
Runners::Sidekiq.new.run(UpdateUserJob, 12, "bob", 5) # asap
Runners::Sidekiq.new.run_in(5.minutes, UpdateUserJob, 12, "bob", 5) # delayed
```
And that's pretty much it. You can now define your background job classes as plain old Ruby files, run them inline, or pass them over to Sidekiq to run in the background, and have the classes be easily testable at the same time. One caveat here is that all the job classes will need to follow the same convention of the `initialize` method accepting all the arguments (remember, with background jobs, do not pass in complex objects as arguments) and a `call` method (in this example; you could use `execute` or `process` or whatever default you prefer) to perform the business logic.
