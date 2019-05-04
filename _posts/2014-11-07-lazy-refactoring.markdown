---
layout: post
title:  "Lazy Refactoring"
date:   2014-11-07 10:24:13
categories: web ruby good-code
excerpt: Look over our shoulder as we refactor and optimize a set of queries.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/lazy-refactoring)*

On the first Tuesday of each month our Boston office hosts Boston.rb's [project
night]. This is an opportunity for the community to get together and hack away on
each other's projects. At a recent project night we encountered an interesting
problem. We went through several iterations, before we found a solution that
satisfied us.

## Initial problem

We're implementing the ability to search events. Search terms should be matched
against a set of columns. If no matches are found in the first column, search
the next column. If no results are found in any column, fall back to a fuzzy
search.

Here is the initial implementation:

```ruby
def self.search(query)
  matching_column = [:name, :city, :title].detect do |column|
    Event.where(column => query).presence
  end

  if matching_column
    Event.where(matching_column => query)
  else
    Event.fuzzy_search(query)
  end
end
```

First we query all the columns and find the first column that contains results.
Then query the column again and return the results. If no column contains any
results, perform a fuzzy search.

The `presence` method returns the object it is called on or `nil` if it is empty.

This solution performs a duplicate query after finding which column to search.
In addition the implementation is a little awkward.

## First try

Our first attempt was to refactor the original implementation to avoid the extra
queries.

```ruby
def self.search(query)
  result = nil
  [:name, :city, :title].detect do |column|
    result = Event.where(column => query).presence
  end

  if result
    result
  else
    Event.fuzzy_search(query)
  end
end
```

We've added the local variable `result` to cache the query results. This takes
care of the extra query. However, by initializing an empty variable and mutating
it over the course of a loop we've introduced an [anti-pattern].

This code is still tough to read.

## Booleans

We decided to go back to the drawing board and think about the problem in
abstract terms. This led to:

```ruby
def self.search(query)
  exact_search(:name, query).presence ||
    exact_search(:city, query).presence ||
    exact_search(:title, query).presence ||
    fuzzy_search(query)
end

def self.exact_search(field, query)
  Event.where(field => query)
end
```

It would be tempting here to see all these `||`s and attempt to
construct a <abbr title="Structured Query Language">SQL</abbr> `OR` statement.

```ruby
def self.search(query)
  Event.where("name = :query OR city = :query OR title = :query", query: query)
end
```

However, <abbr title="Structured Query Language">SQL</abbr> `OR` is _not_
equivalent to logical `||`. `||` will _only_ return the first non-`nil` set.
`OR` will return all results that match _any_ of the criteria. The <abbr
title="Structured Query Language">SQL</abbr> query above is equivalent to:

```ruby
def self.search(query)
  exact_search(:name, query) +
    exact_search(:city, query) +
    exact_search(:title, query)
end

def self.exact_search(field, query)
  Event.where(field => query)
end
```

This would return a mix of name, city, and title results rather than results for
the first field that returned any values.

The boolean approach was a clean solution to our problem. However, a new
requirement emerged that the exact search should occur on an arbitrary array of
columns.

## Procedural approach

We could have iterated on the previous solution by throwing in some meta
programming but decided to explore some other approaches. Here we pull out some
old-style procedural programming:

```ruby
def self.search(query, columns)
  index = 0
  results = Event.none

  while index < columns.length
    results = Event.where(columns[index] => query)
    break if results.exists?
    index += 1
  end

  results.presence || Event.fuzzy_search(query)
end
```

This solution is very un-Ruby like but does meet our new requirement. This uses
a `while` loop to iterate over each element in the `columns` array and breaks if
results are found.

This reintroduces the temporary variable + loop anti-pattern
mentioned above.

## Custom enumerator

We discovered that we really wanted was an array of queries that would only be
evaluated as they were accessed. This led to building a custom Enumerator:

```ruby
def self.search(query, column_names)
  results(column_names).detect(&:exists?)
end

def self.results(column_names)
  columns = column_names.each

  Enumerator.new do |yielder|
    loop do
      yielder << Event.where(columns.next => query)
    end

    yielder << Event.fuzzy_search(query)
  end
end
```

## Looking at Enumerator

To understand the code above, let's dig into Ruby's `Enumerator`. `Enumerator`
implements the [internal and external iterator patterns].

```ruby
[1] pry(main)> results = Enumerator.new do |yielder|
[1] pry(main)*   yielder << "first"
[1] pry(main)*   yielder << "second"
[1] pry(main)* end
=> #<Enumerator: ...>
[2] pry(main)> results.next
=> "first"
[3] pry(main)> results.next
=> "second"
[4] pry(main)> results.next
StopIteration: iteration reached an end
from (pry):7:in `next'
```

Calling `next` on an enumerator will run the code inside its block until a value
is sent to the `yielder`. The code block is then suspended. Subsequent
invocations of `next` will continue running the block from the previous
location. When the block reaches its end, a `StopIteration` error is raised.

Some collections are not just a set of hard-coded values.

```ruby
[1] pry(main)> even_numbers = Enumerator.new do |yielder|
[1] pry(main)*   n = 1
[1] pry(main)*   loop do
[1] pry(main)*     yielder << n if n.even?
[1] pry(main)*     n += 1
[1] pry(main)*   end
[1] pry(main)* end
=> #<Enumerator: ...>
[2] pry(main)> even_numbers.next
=> 2
[3] pry(main)> even_numbers.next
=> 4
[4] pry(main)> even_numbers.next
=> 6
[5] pry(main)> even_numbers.take(10)
=> [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

The code inside the `loop` block will be run until a value is sent to the
yielder. The code inside `loop` could be executed multiple times before a value
gets sent to `yielder`. In this case, it only yields on every other run. We've
created an infinite series of even numbers. `take` will calculate the first `n`
numbers of the series.

```ruby
[1] pry(main)> evens = Enumerator.new do |yielder|
[1] pry(main)*   yielder << 'first'
[1] pry(main)*   n = 1
[1] pry(main)*   loop do
[1] pry(main)*     if n < 7
[1] pry(main)*       yielder << n if n.even?
[1] pry(main)*     else
[1] pry(main)*       raise StopIteration
[1] pry(main)*     end
[1] pry(main)*     n += 1
[1] pry(main)*   end
[1] pry(main)*   yielder << 'last'
[1] pry(main)* end
=> #<Enumerator: ...>
[2] pry(main)> evens.to_a
=> ["first", 2, 4, 6, "last"]
```

This combines the two examples above and introduces the `StopIteration` error.
This allows use to break our `loop` giving us a finite enumerator.

The final piece needed to understand our solution is to realize that calling
`each` without a block returns an `Enumerator`.

```ruby
[1] pry(main)> enum = [1,2].each
=> #<Enumerator: ...>
[2] pry(main)> enum.next
=> 1
[3] pry(main)> enum.next
=> 2
[4] pry(main)> enum.next
StopIteration: iteration reached an end
from (pry):4:in `next'
```

## Back to the custom enumerator solution

Now that we know about enumerators, let's take a closer look at the proposed
solution:

```ruby
def self.search(query, column_names)
  results(column_names).detect(&:exists?)
end

def self.results(column_names)
  columns = column_names.each

  Enumerator.new do |yielder|
    loop do
      yielder << Event.where(columns.next => query)
    end

    yielder << Event.fuzzy_search(query)
  end
end
```

By calling `each` without a block on `column_names` we get an enumerator. Inside
the `loop` we call `next` on that enumerator to retrieve a column name. Once
there are no more column names `columns.next` will raise a `StopIteration`
error, breaking the `loop`. We add our default fuzzy search as the last call to
`yielder`.

`results` now behaves like an array of queries that only get evaluated if they
are accessed. Just what we set out to build.

We can now use `detect` to find the first result that exists. `detect` evaluates
each value until one matches the given expression. The remaining values are not
evaluted.

## Lazy enumeration

It turns out the solution above is quite similar to lazy enumerators introduced
by Ruby 2.0:

```ruby
def self.search(query, column_names)
  queries = column_names.lazy.map do |column|
    Event.where(column => query)
  end

  queries.detect(&:exists?) || Event.fuzzy_search(query)
end
```

Lazy enumerators will pass a value through the entire chain of methods
before evaluating the next value.

```ruby
[1] pry(main)> def query(q)
[1] pry(main)*   puts "querying #{q}"
[1] pry(main)*   q
[1] pry(main)* end
=> :query
[2] pry(main)> ['title', 'name', 'city'].map { |n| query(n) }.detect { true }
querying title
querying name
querying city
=> "title"
[3] pry(main)> ['title', 'name', 'city'].lazy.map { |n| query(n) }.detect { true }
querying title
=> "title"
```

As we can see above, `lazy.map.detect` only calls `query` with 'title' while
`map.detect` calls `query` for every element in the array.

![lazy vs non-lazy](https://images.thoughtbot.com/lazy-refactoring/lazy-vs-non-lazy.gif)

## What's next

* Dig deeper with [Enumerator: Ruby's Versatile Iterator]
* Learn more about enumerators and `each` in this [blog post]
* Pat Shaughnessy has an excellent post on [lazy enumerators]

[project night]: http://bostonrb.org/project_night
[anti-pattern]: https://thoughtbot.com/blog/iteration-as-an-anti-pattern
[internal and external iterator patterns]: https://chickenriceplatter.github.io/blog/2013/04/07/internal-vs-external-iterators/
[blog post]: http://blog.arkency.com/2014/01/ruby-to-enum-for-enumerator/
[Enumerator: Ruby's Versatile Iterator]: http://blog.carbonfive.com/2012/10/02/enumerator-rubys-versatile-iterator/
[lazy enumerators]: http://patshaughnessy.net/2013/4/3/ruby-2-0-works-hard-so-you-can-be-lazy
