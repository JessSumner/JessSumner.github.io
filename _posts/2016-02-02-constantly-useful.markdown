---
layout: post
title: CONSTANTly useful
date: 2016-02-02 17:16:00
---

I recently built a mailer feature for my animal rescue site and it turned out that using a constant cleaned up my code significantly. I was very excited by this refactor and I learnt a lot from it so I wanted to document the process.

At this point I have already built an adoption application form which a visitor could fill out if they were interested in adopting a rat. The information that they submitted is saved to the database but that was all. I wanted the information to be emailed directly to someone at the rescue who could take their application further hopefully resulting in an adoption. 

Rails uses mailers so this seems the simplest implementation. A mailer behaves like a controller, it contains methods and communicates with the mailer templates. Just as a controller communicates with a view. You can also generate them using `rails g mailer MyMailer` just as you would a controller.

So for this purpose I envisioned sending an email to the rescue with a simple table with two columns. The first column containing the application form question and the second containing the applicant's response. All this information is stored within the `@adoption_application` object that was created when they clicked on the link to the adoption application form. However @adoption_application is not an array or a hash so there was no immediate way to iterate through the object to render each row of the table - at least in my mind!

Playing in the terminal using `example = AdoptionApplication.first` as my example of `@adoption_application` can give us an idea of how to retrieve the information needed. The form questions are all known and defined in the controller as they are the params that a visitor is allowed to change. 

{% highlight bash %}

> AdoptionApplication.first
> example =_
> example.name
  => Jessica

{% endhighlight %}

Realising that the question name could be called on the object and return the value gives us a way of retrieving the data. So I quickly started creating my table.

{% highlight erb %}

<table>
  <tr>
    <td> Name: </td>
    <td> @adoption_application.name </td>
  </tr>
</table>

{% endhighlight %}

No sooner as I had written this first line did I realise this was going to be a chore and a lot of repetition. There must be a better solution to this!

There is, and fortunately during the PR for this feature a solution was suggested.

 
