---
layout: post
title: Ensuring that a ruby method is called from a specific location
category: posts
---

If you, like me, get interested in tricky programming questions found randomly
on twitter, here's a nice one for you:

Suppose you have a Ruby class with a method and another with other methods that
call it:

```ruby
class Foo
  def bar
  end
end

class Work
  def execute
    # calling Foo.new.bar here should work
  end

  def other_method
    # calling Foo.new.bar here should raise error
  end
end
```

The requirement is simple - enforce that `Foo#bar` is ONLY called from inside
`Work#execute`, not from elsewhere.

First thought that comes to mind is to use magic "caller" array in Ruby to
make sure that a given location is in the callstack:

```ruby
# test.rb

module Guardian
  def self.ensure_method_called!(location)
    unless caller.any? {|line| line.include?(location)}
      raise ArgumentError.new("nice try lol")
    end
  end
end

class Foo
  def bar
    Guardian.ensure_method_called!("test.rb:21")
    "bar"
  end
end

class Work
  def execute
    # calling Foo.new.bar here should work
    Foo.new.bar + " from execute"
  end

  def other_method
    # calling Foo.new.bar here should raise error
    Foo.new.bar
  end
end

puts Work.new.execute
puts Work.new.other_method
```

Executing it, we get the following output:

```
$ ruby test.rb
bar from execute
test.rb:6:in `ensure_method_called!': nice try lol (ArgumentError)
        from test.rb:13:in `bar'
        from test.rb:26:in `other_method'
        from test.rb:31:in `<main>'

shell returned 1
```

When can see that the approach works, BUT there are several issues with the implementation:

- if you rename the file, the checks will not work (filename is hardcoded)
- if you change the source, the change will also break (line number is hardcoded)

To _really_ solve the challenge, we need to have a flexible solution
that will not be coupled with any hardcoded values. Ideally, the API should look
like this:

```ruby
class Foo
  def bar
    Guardian.ensure_instance_method_called!(Work, :execute)
    "bar"
  end
end
```

We can use the same `caller` array check, but this time we need to somehow figure
out lines where a method is defined, and then check if one of the lines of the
method is included in the `caller` array. The code would look line this:

```ruby
module Guardian
  def self.ensure_instance_method_called!(klass, method_name)
    file_name, source_lines = instance_method_info(klass, method_name)

    is_method_called = caller.any? do |frame|
      # frame looks like this:
      #   /Users/andrei/Play/alisnic.github.com/test.rb:14:in `work'
      file, line = frame.split(":")[0..1]
      file == file_name && source_lines.include?(line.to_i)
    end

    raise ArgumentError.new("nice try lol") unless is_method_called
  end
end
```

All that is left is to implement the `instance_method_info`, which should return
the file name and a range with line numbers where an instance method is defined.

Ruby has "instance_method" method, which can give us some information about an
instance method:

```ruby
irb(main):003:0> Work.instance_method(:execute)
=> #<UnboundMethod: Work#execute>
irb(main):004:0> Work.instance_method(:execute).methods - Object.methods
=> [:arity, :original_name, :owner, :bind, :source_location, :parameters, :super_method]
```

We can notice that among the defined methods there is an interesting one -
`source_location`. Let's see what it returns:

```ruby
irb(main):005:0> Work.instance_method(:execute).source_location
=> ["/Users/andrei/Play/alisnic.github.com/test.rb", 19]
```

We are close! We have the file name and the line where the method _starts_. All
that is left is to figure out where the method ends.

As you saw above, in Ruby we can ask a class all of its methods. We can also ask
for all instance methods:

```ruby
irb(main):007:0> Work.instance_methods - Object.methods
=> [:execute, :other_method]
```

**NOTE**: we are subtracting `Object.methods` from the result because Ruby defines
a bunch of methods for us. We are interested only in those that we wrote.

So how to figure out where a method ends? Well, it certainly ends before the next
one begins! We can get source_location of all methods, sort it, and make a guess
where each method ends:

```ruby
module Guardian
  def self.instance_method_info(klass, method_name)
    methods = klass.instance_methods - Object.methods

    # We start by building an array with all the instance methods of the class.
    # Each entry would look like this:
    #
    # {
    #   name:       :other_method,
    #   file:       "/Users/andrei/Play/alisnic.github.com/test.rb",
    #   start_line: 24
    # }
    candidates = methods.map do |name|
      location = klass.instance_method(name).source_location

      {
        name:       name,
        file:       location[0],
        start_line: location[1]
      }
    end

    # Next, we sort those entries by start line, in reverse order. We will start
    # guessing method locations from bottom to up
    reversed = candidates.sort_by {|entry| -entry[:start_line] }

    # We don't know where is the end of the last method of the class, hence we
    # assume it is 99999. Starting from there, each method's last line is the
    # line before the previous method start line
    with_lines = reversed.reduce([]) do |result, entry|
      previous_method = result.last
      previous_line   = previous_method ? previous_method[:start_line] : 99999

      result << entry.merge(lines: entry[:start_line]..previous_line)
    end

    # Next, we find the method we are interested in
    method = with_lines.find {|entry| entry[:name] == method_name }

    # And return only the information we need
    [method[:file], method[:lines]]
  end
end
```

When we try to execute the defined method, we get the following result:

```ruby
irb(main):114:0> Guardian.instance_method_info(Work, :execute)
=> ["/Users/andrei/Play/alisnic.github.com/test.rb", 19..24]
```

Awesome! We have the range of lines where the method is defined, let's write the
final test script:

```ruby
# test.rb

module Guardian
  def self.ensure_method_called!(location)
    unless caller.any? {|line| line.include?(location)}
      raise ArgumentError.new("nice try lol")
    end
  end

  def self.ensure_instance_method_called!(klass, method_name)
    file_name, source_lines = instance_method_info(klass, method_name)

    is_method_called = caller.any? do |frame|
      file, line = frame.split(":")[0..1]
      file == file_name && source_lines.include?(line.to_i)
    end

    raise ArgumentError.new("nice try lol") unless is_method_called
  end

  def self.instance_method_info(klass, method_name)
    # same code as above
  end
end

class Foo
  def bar
    Guardian.ensure_instance_method_called!(Work, :execute)
    "bar"
  end
end

class Work
  def execute
    # calling Foo.new.bar here should work
    Foo.new.bar + " from execute"
  end

  def other_method
    # calling Foo.new.bar here should raise error
    Foo.new.bar
  end
end

puts Work.new.execute
puts Work.new.other_method
```

When we will execute it, it will work as expected:

```
$ ruby test.rb
bar from execute
test.rb:18:in `ensure_instance_method_called!': nice try lol (ArgumentError)
        from test.rb:66:in `bar'
        from test.rb:79:in `other_method'
        from test.rb:84:in `<main>'

        shell returned 1
```

Here's a link to a gist with the whole source in case you want to play with it:
[https://gist.github.com/alisnic/1ccc6e1357e0365ee4ed8655f2e1d7dd](https://gist.github.com/alisnic/1ccc6e1357e0365ee4ed8655f2e1d7dd)

