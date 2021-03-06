---
layout: post
title: Hit or miss dependent fields
date: 2015-12-17 18:53:00
---

My adoption application form took an exciting turn when I realised that the benefit of the form being online was that it could be dynamic. I could create a dependency on certain questions so they would only appear if the answer to the previous question was "yes". Therefore people who have never had rats before don't get asked how many they have had. In order to do this I found a gem dependent-fields-rails which kindly incorporates the javascript needed to create this dependency. I learnt a lot of details installing this gem and even more after I thought I was finished with it. This post will guide you through my process from start to finish using this gem.

Searching the question "How to make fields dependent Ruby on Rails?" yielded the result of the gem and having found several sources mentioning it I thought let's give this a go. So in goes the gem to my gemfile and bundle.

The next step is to add a line of code to the Javascript manifest file. Here I made two mistakes. Firstly I added the `//= require dependent-fields` below the `//= tree .` and then on correcting that mistake I deleted the `.` from the `//= tree .`.

The next step in the README is to add the jQuery. To me it makes sense to put this in its own sub folder, so for my app it is in app/assets/javascripts/adoption_applications.js. Therefore just by looking at the folder name it is clear what the code applies to. 

After restarting the server it is time to add the dependencies to the form. I decided to use radio buttons in order to answer the question on which other questions were dependent.

{% highlight erb %}
  <div class="radio"> 
    <%= radio_button_tag(:current_rats, "yes") %>
    <%= label_tag(:current_rats_yes, "I currently own one or more rats") %></br>

    <%= radio_button_tag(:current_rats, "no", true) %>
    <%= label_tag(:current_rats_no, "I do not currently own any rats") %>
  </div>
  
  <div class="js-dependent-fields" data-radio-name='current_rats' data-radio-value='yes'>
    <%= f.label :current_rat_count %>
    <%= f.text_field :current_rat_count %>

    <%= f.label :current_rat_gender %>
    <%= f.text_field :current_rat_gender %>

    <%= f.label :current_rat_age %>
    <%= f.text_field :current_rat_age %>
  </div>
{% endhighlight %}

Here is the code for one of my questions and its dependent questions.

Ctrl-s and Cmd-r localhost:3000 and "Ta-da". If I answer "yes", I get to see more questions otherwise I continue on.

I use [hound](https://github.com/thoughtbot/hound) by thougtbot as a style guide and when I create a pull request for my projects I immediately get an onslaught of violations. I find hound particularly useful as it keeps my code style consistent. It has also taught me lots of good practices - however that is for another blog post! Back to dependent fields.... On pushing my code to GitHub, hound informed me that I was missing a semicolon in my jQuery. I immediately corrected this violation and moved on. A little while later I decided to show off my new form so I followed the route from the homepage and much to my horror I could see all my questions. 

Being new to javascript and very skeptical I put all blame on the semicolon I had added as this was the only change I had made. I promptly removed the semicolon and refreshed and "Ta-da!" the dependent fields were working again! 

Chuffed at fixing the problem I moved on. However this phenomenon kept happening. After multiple attempts with and without the semicolon I realised that the form was loading correctly on a page refresh but not when following the route from the homepage. 

The problem: Turbolinks. Turbolinks is a gem which is included when a new Rails app is generated. When page links are followed Turbolinks allows the page to be loaded without all the requests being made to the server. Turbolinks gains all of its information from the page which is currently loaded, therefore not making the same requests over and over and reducing the load speed. Cmd-r is a hard reload, Turbolinks is not involved in this request, therefore all files are requested from the server and the page loads correctly.

{% highlight javascript %}

$(document).ready(function() {
  DependentFields.bind();
});

$(document).on('ready page:load', function() {
  DependentFields.bind();
});

{% endhighlight %}

The first example here is the original code which I modified to write the second example. The second example reads, for the document on every page load peform this function. In the original it was not specifying every page load and therefore the form was only loading correctly on a page refresh.

An alternative solution is to remove Turbolinks from the application. Without Turbolinks the first code example would have worked.

Problem fixed, I am thrilled with how the form turned out. I learnt a lot more than I expected to by creating this dependency and finding this to be a useful change I have submitted a pull request to the author of the gem.

All that is left to say is "Sorry semicolon for all the blame I sent your way!".
