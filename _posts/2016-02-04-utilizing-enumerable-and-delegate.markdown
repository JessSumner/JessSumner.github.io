---
layout: post
title: Utilizing enumerable and delegate
date: 2016-02-04 15:47:00
---

This week I created a feature for my Any Rat Rescue project to display their
flyers and newsletters. My initial work up of this feature created a logic
heavy controller. Several new ideas were suggested to me and in implementing
these changes I learnt so much. My resulting solution is a tiny controller and
a small, flexible object which contains most of the logic originally defined in
the controller. This refactor incorporates the single responsability principle,
object dependencies and the open-closed principle. I should also add that I am
currently reading through Practical Object Oriented Designs in Ruby by Sandi
Metz. Reading chapter 3 I immediately saw problems in my code.

{% highlight ruby %}
class NewslettersController < ApplicationController
  def index
    @flyers = NewsletterFinder.new.flyers
    @news = NewsletterFinder.new.news
  end
end

class NewsletterFinder
  def news
    newsletters_with_file_extension("news").map do |newsletter|
      newsletter.gsub(/\.pdf/, "")
    end
  end

  def flyers
    newsletters_with_file_extension("monthly_flyers").map do |newsletter|
      newsletter.gsub(/\.pdf/, "")
    end
  end

  def newsletters_with_file_extension(file_name)
    unsorted_newsletters(file_name).sort do |date_1, date_2|
      Date.parse("1-#{date_2}") <=> Date.parse("1-#{date_1}")
    end
  end

  def unsorted_newsletters(file_name)
    file_contents(file_name).select { |newsletter| newsletter =~ /.*\.pdf/i }
  end

  def file_contents(file_name)
    Dir.entries("app/assets/newsletters/#{file_name}")
  end
end

{% endhighlight %}

Here are my two objects a `NewsletterController` and a `NewsletterFinder`.
Between these objects there are a lot of dependencies. `NewsletterController`
knows that there is a `NewsletterFinder` object and it knows two methods that
can be called on the `NewsletterFinder` object. The `NewsletterFinder` is also
dependent on the folder names. I feel there is little to be done regarding the
first dependency however we can improve the code to reduce the other
dependencies.

Step 1 - passing an argument not calling a method

In the `NewsletterController` request a new instance of NewsletterFinder and
pass an argument. The argument represents the folder name which holds the
documents desired by the view.

This requires the `NewsletterFinder` object to have an initialize method which
accepts an argument. 

This change results in the `NewsletterController` having less knowledge about
the `NewsletterFinder` class which reduces dependencies. It also removes all
responsability of the `NewsletterFinder` class to know what the folder name is.
This means changes can be made to the folder name without needing to change
anything within the `NewsletterFinder` class.

{% highlight ruby %}
class NewslettersController < ApplicationController
  def index
    @flyers = NewsletterFinder.new("monthly_flyers")
    @news = NewsletterFinder.new("news_documents")
  end
end

class NewsletterFinder
  def initialize(type)
    @type = type
  end

  def file_contents
    Dir.entries("app/assets/newsletters/#{@type}")
  end
end

{% endhighlight %}

Step 2 - reducing responsabilities down to one

The NewsletterFinder has two methods `flyers` and `news`. These methods have
two responsabilities, they define a variable which gets passed through the
method chain and they remove the .pdf from the file name. 

Fortunately Step 1 means the variable assignment can be cut out as that is now
being passed in from the controller. All that is left is two copies of the same
logic which can be made into one method called `newsletter_list`.

{% highlight ruby %}
def newsletters_list
  newsletters_with_file_extension.map do |newsletter|
    newsletter.gsub(/\.pdf/, "")
  end
end
{% endhighlight %}

Step 3 - undefined method error each

Making these changes results in an error when I refresh the view. Undefined
method each for `@flyers.each`. This led me into learning about the
[`Enumerable Module`](http://ruby-doc.org/core-2.3.0/Enumerable.html). The
`Enumerable Module` is a mixin which is included in the Array class. It is
responsible for many of the methods that we take advantage of such as find,
map, select and reduce. The `Enumerable Module` takes advantage of a definition
of each to build these other methods. The array class has a
definition of `each` and can therefore take advantage of the `Enumerable
Module`. The `NewsletterFinder` object is not an array. `@flyers` is an
instance of `NewsletterFinder` and hence it does not know how to respond to
`each`.

Step 4 - including the Enumerable module

The `Enumerable Module` can be included into any class.

{% highlight ruby %}
class NewsletterFinder
  require Enumerable
end
{% endhighlight %}

However we still have to define `each`. Up until receiving the error in Step 3
I had taken `each` for granted. My first thought was "How do you define each?".
After some reading and thinking `each` (at least in my example) simply returns
each value in a collection. In my example the collections I am interested in
are the contents of the folders I am passing in.

This was an incredibly frustrating realization for me. In my mind defining each
would require me writing out all the file names which is precisily what I was
avoiding by writing the logic in `NewsletterFinder`.

{% highlight ruby %}
  def each
    yield "jul-15".pdf
    yield "dec-15".pdf
    ..
  end
{% endhighlight %}

Step 5 - delegate to the rescue

In my example `newsletter_list` is an array. Therefore `newsletter_list` has
each already defined. So we can delegate the responsability of defining each to
`newsletter_list`.

{% highlight ruby %}
delegate :each, to: :newsletter_list
{% endhighlight %}

This neatly avoids having to yield each file as I originally dreaded. However
it can still be improved. Having set this delegation to `newsletter_list`,
everytime `each` is called on an instance of `NewsletterFinder`
(i.e. `@flyers`) the whole method stack will run. This seems unecessary.

{% highlight ruby %}
def newsletter_list  
  @newsletter_list ||= newsletters_with_file_extension.map do |newsletter|
    ...
  end
end
{% endhighlight %}

The operator ||= will evaluate the left first and if the response is nil then
it will evaluate the right. The first time `each` and therefore
`newsletter_list` is called it will run through the method stack and save a
value to `@newsletter_list`. Then if `each` is called again `@newsletter_list`
is returned.

Step 6 - making the methods private

These methods should not be called by other objects. They should only be called
by instances of `NewsletterFinder`. Therefore other than the initialize method
they should be private and only visible to their class.

The final classes!

{% highlight ruby %}
class NewslettersController < ApplicationController
  def index
    @flyers = NewsletterFinder.new("monthly_flyers")
    @news = NewsletterFinder.new("news_documents")
  end
end
class NewsletterFinder
  include Enumerable
  delegate :each, to: :newsletters_list

  def initialize(type)
    @type = type
  end

  private

  def newsletters_list
    @newsletters_list ||= newsletters_with_file_extension.map do |newsletter|
      newsletter.gsub(/\.pdf/, "")
    end
  end

  def newsletters_with_file_extension
    unsorted_newsletters.sort do |date_1, date_2|
      Date.parse("1-#{date_2}") <=> Date.parse("1-#{date_1}")
    end
  end

  def unsorted_newsletters
    file_contents.select { |newsletter| newsletter =~ /.*\.pdf/i }
  end

  def file_contents
    Dir.entries("app/assets/newsletters/#{@type}")
  end
end


{% endhighlight %}

The final classes are simpler and I feel they align better with the open/closed
prinicple, single responsability principle and have fewer dependencies on each
other.

I was able to test this conclusion when I decided to change the name of one of
my folders. To do so I did not need to modify anything within the
`NewsletterFinder` class. I think that demonstrates removing dependencies
effectively!
