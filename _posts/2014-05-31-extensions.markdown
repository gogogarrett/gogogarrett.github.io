---
layout: post
title:  "Extensions"
categories: rails ruby learning
---

> Extensions add methods at runtime to classes.

Extensions can be easily compared to decorators, presenters, concerns, and whatever other jargon you possess. We've chosen the name Extensions because it makes sense in the context in which we are using them, but feel free to label them as you please.



## History
I got hired as a Contractor at Blake Education to build a greenfield project __hurray__.  We are building an app to help teach students their year's curriculum through fun html5 canvas games.  Teachers will give their students homework via the app and it will all be self-marking.



## The problem
From the start, we knew there would be two entirely different ways to interact with the app - as a teacher, and as a student.  We could have easily made a simple, *project  killing*, monolithic app with `Teacher::` and `Student::` namespacing, but we decided against that approach.

We instead tackled this problem by splitting these up into small, isolated, separate apps with an intermediate gem for sharing common models between them.

> `wf-student`, `wf-teacher`, `wf-models`.

`wf-models` - holder of all things shared

`wf-teacher` - controller of all things student

`wf-student` - receiver of fun, travel, and knowledge



## Where do extensions come into play?
We use extensions to keep the `wf-models` very thin and very dumb.  We don't want these shared models to know anything about the specific business logic of either app. Instead we extend these classes when the `wf-teacher` or `wf-student` app is started and inject methods into these classes.



## Setup
`config/application.rb`
```ruby
config.to_prepare do
  Dir.entries("app/extensions")
    .select{ |f| !File.directory? f}
    .each do |file_name|

    array = file_name.split("_")
    array.pop
    klass_name = array.map(&:capitalize).join
    klass = klass_name.constantize

    klass.class_eval do
      include "#{klass}Extensions".constantize
    end
  end
end
```

Here we find the model class and include the respected extension module.



## How we use them

`app/extensions/school_extensions.rb`
### `wf-teacher` School Extension
```ruby
module SchoolExtensions
  extend ActiveSupport::Concern

  included do
  end

  def address
    [street, suburb, state, postcode, country].join(', ')
  end

  def all_students_except_from(obj)
    students.active.where.not(id: obj.student_ids)
  end

  def all_students_except(student_id)
    students.active.where("users.id <> ?", student_id)
  end

  module ClassMethods
  end
end
```

### `wf-model` School Class
```ruby
class School < ActiveRecord::Base
  has_many :students
  has_many :teachers

  def to_param
    "#{id}-#{name.parameterize}"
  end

  def to_s
    name
  end
end
```

Here we add methods that are specific to the teacher app that the student app will never need to use or know about.  Thus, we moved this logic into the teacher app by extending the real class at runtime and the student will never be any wiser.  This restricts the `wf-student`  school class to a very limited amount of public api methods and makes it easier to see what you have available by only showing what is relevant to the app.


## Benefits
- `wf-models` stays thin!
- methods only exist where they belong
- keeps files small!



## Things to be weary of
Don't just throw every method in the world into an extension and hide the fact that your shared model has 100 methods.  Design and forethought is still a very important part about designing software and not every *design pattern* will apply to every project.



## Summary
- Keep shared models dumb
- Keep extensions small
- Keep business logic where it belongs
