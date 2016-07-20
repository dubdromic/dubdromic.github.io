---
layout: post
title: 'Notes: Rails Autoloading + Superclasses In Modules'
date: 2015-04-13
---

Rails' autloading can cause a subtle inheritance issue when inheriting
classes defined both inside and outside a module.

{% highlight ruby %}
class AwesomeSuperclass
  def name
    'Base::AwesomeSuperclass'
  end
end
{% endhighlight %}

{% highlight ruby %}
module AwesomeModule
  class AwesomeSuperclass
    def name
      'AwesomeModule::AwesomeSuperclass'
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
module AwesomeModule
  class AwesomeClass < AwesomeSuperclass
    def name
      "AwesomeClass inherited from #{super()}"
    end
  end
end

puts AwesomeModule::AwesomeClass.new.name # => ?
{% endhighlight %}

The output of `AwesomeClass#name` depends entirely on what file the
Rails autoloader loaded first. Rails won't include a file if the class
is already defined, even if it's defined in a different module context.

The fix: specify the module when inheriting from the superclass.

{% highlight ruby %}
module AwesomeModule
  class AwesomeClass < AwesomeModule::AwesomeSuperclass
    def name
      "AwesomeClass inherited from #{super()}"
    end
  end
end

puts AwesomeModule::AwesomeClass.new.name
# => "AwesomeClass inherited from AwesomeModule::AwesomeSuperclass"
{% endhighlight %}