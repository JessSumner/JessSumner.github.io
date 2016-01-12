---
layout: post
title: Five stars
date: 2016-01-09 14:48:00
---

Since last week I have spent some time learning about the jQuery library using [Introduction to jQuery](https://www.edx.org/course/introduction-jquery-microsoft-dev208x-0). The culmination of this has been creating a five star review widget. Please feel free to have a play with the [widget](http://blog.unlikely-rubyist.com/review-widget/). In this post I thought I could go through the steps that led to the building of this widget.

Making this widget is the assessment for the first module so the basic HTML template is provided. The text around the widget was some context of rating a conference which I decided to update to be more personal - hopefully if I can get it to work I should be able to give myself 5 stars! At least that was what I was thinking. 

<h3>Disclaimer</h3>
This post includes spoilers for the first module. If you are thinking of running through the course I recommend doing so before reading any further.

<h3>The goal:</h3>
On hover the stars should fill up from the left to where the mouse is with yellow. On click, setting a rating, the stars should fill up from the left with green. However even if a rating is saved on hover all the stars from the left to the mouse should fill with yellow. But when the mouse is moved away the saved rating should return.

<h3>Step 1 - Let's get the hover to work!</h3>
The five stars are siblings all within a parent `<div>`. The star that is being hovered over (I will refer to as the target) can be accessed within the function by `$(this)`. The stars to the left of the target are the previous siblings. Therefore they can be accessed by using `prevAll()`.

I want to highlight a silly error I made here. When I first read `prevAll()` I interpretted it as `prevALL()` and this did not work. I even tried `nextALL()` which would in this example access the stars to the right of the target. I eventually remembered the camelcase conventions in javascript and realised that it was weird that the "L"s were capitalized too. I went back to look at my sources and indeed I had misread the text. Anyway I just wanted to share this as a little error in my reading stumped me for quite a while.

Being able to access the target star and the ones to the left the next step is to add the yellow color. In this example two classes are defined `'rating-hover'` and `'rating-chosen'`. So in order to add the color we can simply `addClass('rating-hover')`.

The event handler that we want the yellow to appear on is `mouseenter`. So as soon as the user's mouse enters the star we want it to turn yellow along with the stars to the left.

On `mouseleave` we want the yellow to go away and therefore we can use `removeClass('rating-hover')` on both the target and `prevAll()`.

This is all the code we need if we just cared about mousing over and not actually fixing a rating. But that would be too easy!

<h3>Step two - Click and set a rating!</h3>

Here we will follow similar steps in the logic. Again we are going to have a target which has been clicked on. Again we can access it using `$(this)` and the stars to the left using prevAll. Then we can simply `addClass('rating-chosen')`. However the stars will already have the class `'rating-hover'` due to the previous logic. Therefore before we add the rating-chosen class we should `removeClass('rating-hover')`.

There is one more important step here. We want to be able to save the rating somehow. To do this we can set `var rating = $(this)`. We will want to access `rating` outside of this function on click so it should be defined outside of the function, as shown below.

{% highlight javascript %}
var rating;      

$('.star-five').click(function(){
  rating = $(this);
  rating.removeClass('rating-hover').addClass('rating-chosen');
  rating.prevAll().removeClass('rating-hover').addClass('rating-chosen');
});
{% endhighlight %}

<h3>Step three - Things aren't quite right!</h3>

At this point if we hover over the stars after saving a rating the stars do not appear how we want them to. The green remains and any that weren't green but are being hovered over will turn to yellow.

Looking back at the code for mouseenter we are adding the 'rating-hover' class but that does not override 'rating-chosen'. Therefore on mouseenter we need to `removeClass('rating-chosen')` before addClass('rating-hover').

<h3>Step four - On mouseleave everything is gone!</h3>

This is where setting var rating is so important. On mouseleave, the rating-hover class is removed so the stars are left empty. However if rating has been saved it can be called and given the 'rating-chosen' class.

{% highlight javascript %}
$('.star-five').mouseleave(function(){
  $(this).removeClass('rating-hover');
  $(this).prevAll().removeClass('rating-hover');
  if (rating) {
    rating.addClass('rating-chosen');
    rating.prevAll().addClass('rating-chosen');
  }
});
{% endhighlight %}

These steps are the process I went through to make this widget. Making this widget was a fun application on what I learnt throughout the module. I am really enjoying this course and I highly recommend it for anyone who is new to jQuery and javascript. It helps with the syntax and obviously the methods that are available. The content is a mixture of text and video and there is a quiz and lab at the end of each module. The course is self paced which is great for fitting into busy schedules.
