---
layout: post
title: 'Notes: The &&= Operator'
date: 2015-03-27
---

`&&=` is a Ruby assignment operator. It allows the assignment of a
variable if the variable is already a *non-falsy* value.

Like `||=`, it doesn't act quite like you'd expect with hash assignments.
Avoid using it in that case.

`||=` operator:

{% highlight ruby %}
def awesome_method(input = nil)
  input ||= :default
end

awesome_method(false) #=> :default
awesome_method(nil) # => :default
awesome_method(:hello) # => :hello
{% endhighlight %}

`&&=` operator:

{% highlight ruby %}
def awesome_method(input = nil)
  input &&= :default
end

awesome_method(false) #=> nil
awesome_method(nil) # => nil
awesome_method(:hello) # => :default
{% endhighlight %}