---
layout: post
title: CONSTANTly useful
date: 2016-02-07 17:16:00
---

I recently built a mailer feature for my animal rescue site and it turned out
that using a constant cleaned up my code significantly. I was very excited by
this refactor and I learnt a lot from it so I wanted to document the process.

At this point I have already built an adoption application form which a visitor
can fill out if they are interested in adopting a rat. The information they
submit is saved to the database but that is all. I want the information to be
emailed directly to the rescue so their application can move forward hopefully
resulting in an adoption. 

Rails uses mailers so this seems the simplest option here. A mailer behaves
like a controller, it contains methods and communicates with the mailer
templates. Just as a controller communicates with a view. You can also generate
them using `rails g mailer MyMailer` just as you would a controller.

So for this purpose I envisioned sending an email to the rescue with a simple
table containing two columns. The first column will have the application form
question and the second will have the applicant's response. All this
information is stored within the `@adoption_application` object created when
they clicked on the link to the adoption application form. However
@adoption_application is not an array or a hash so there was no immediate way
to iterate through the object to render each row of the table - at least in my
mind!

Playing in the terminal using `AdoptionApplication.first` as my example of
`@adoption_application` can give us an idea of how to retrieve the information
needed. The form questions are all known and defined in the controller as they
are the params that a visitor is allowed to change. 

{% highlight bash %}

> example = AdoptionApplication.first
> example.name
  => Jessica

{% endhighlight %}

Realising that the question name could be called on the object and return the
value gives us a way of retrieving the data. So I quickly started creating my
table.

{% highlight erb %}

<table>
  <tr>
    <td> Name: </td>
    <td> @adoption_application.name </td>
  </tr>
</table>

{% endhighlight %}

No sooner as I had written this first line did I realise this was going to be a
chore and a lot of repetition. There must be a better solution to this!

There is, and fortunately during the PR for this feature one was suggested.

In this example the left column states the adoption application attribute and
the right column contains the value saved for this attribute. So why not make
an object that contains all the attributes. If the object is an array it will
be possible to iterate through the array and display the attribute on the left
and call the attribute on @adoption_application on the right.

Creating a constant which equals an array of attributes seems to be a great
idea. A benefit of a constant is they are clearly defined. Their names are
conventionally written in all capitals and they are defined at the very top of
the file. Another benefit is they can be called throughout the application.
They are not limited to the method or class that they are defined in.

{% highlight ruby %}

class AdoptionApplication < ActiveRecord::Base
  FORM_FIELDS = %w(
    name
    address
    ...
    preferred_characteristics
  ).freeze
end

{% endhighlight %}

Here the `%w` splits the following list on whitespace and forms an array of strings. Giving us the desired array of attributes.

In the view this constant can be called as so.

{% highlight erb %}
<table>
  <% AdoptionApplication::FORM_FIELDS.each do |field| %>
  <tr>
    <td><%= field %></td>
    <td><%= @adoption_application.field %></td>
  </tr>
</table>
{% endhighlight %}

In order to access the constant from a file other than where it is defined, the constant name is prefixed by its class. 

Using a constant gave me the solution I hoped would be possible. The view is small and simple. I am not repeating myself. Also should the form change in the future the mailer itself will not need to change.
