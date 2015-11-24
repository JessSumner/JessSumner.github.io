---
layout: post
title: A Challenge Of Data
date: 2015-11-23 20:46:00
---

A project that I am working on is for Any Rat Rescue, located in Phoenix, AZ. I would like to build them a web app with more functionality that would save trees and time. So that is what I am doing. This blog post covers how I refactored and DRYed up a data heavy section on my code. Whilst not a hard challenge I enjoyed the process and reveled in the satisfaction at the end.

It is important to the rescue to be able to share information of vets who are proficient in caring for rats. This led me to the creation of a resources page. 

My first attempt at the page included all the information in the view. As my view grew larger and larger I realised that this was not a good idea. The resources had geographic divisions so my solution was to divide the data and create separate partials. I now had six small partials. This was better but my code was extremely repetative. 

To deal with the repetition I created each loops within the partials. I moved the each data group into an array of hashes and set the array to a variable. My partials were now better but my controller was now enormous.

My final refactor was to extract the arrays into YAML files. This had two benefits: the controller and view files were small and clear and it would be easy and to make changes to the data in the future.

Here is an extract of my controller

{% highlight ruby %}

def index
  @vets_phoenix = vets_phoenix
end

private

def vets_phoenix
  YAML.load_file("config/resources/vets_phoenix.yml")
end

{% endhighlight %}

and here is the corresponding partial.

{% highlight erb %}

<% @vets_phoenix.each do |vet| %>
  <%= vet["name"] %>
  <%= simple_format vet["description"]
  <a href="<%= vet["website"] %>"> website </a>
  <%= vet["phone"] %>
<% end %>

{% endhighlight %}

Whilst the process of displaying this information is longer and spread over four separate files the code is much cleaner. Ignoring the YAML files which are of course larger, the largest file is 37 lines, shorter than this blog post!

I am still enjoying working through this process and am so proud of the result. 
