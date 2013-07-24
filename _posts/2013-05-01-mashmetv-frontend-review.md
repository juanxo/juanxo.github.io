---
  layout: post
  title: MashMeTV Frontend Review
---

MashMeTV Front End review
===============

I couldn't sleep the night of the International Worker's Day, and I had an interview a couple of
days later with MashMeTV that I hadn't confirmed yet, so I wrote this confirmation tweet:

<blockquote class="twitter-tweet"><p>An informal FrontEnd review of <a
href="https://twitter.com/MashMeTV">@MashMeTV</a> to celebrate Worker&#39;s Day in Spain: <a
href="https://t.co/8daxjfu8qR">https://t.co/8daxjfu8qR</a> /cc <a
href="https://twitter.com/VictorSanchez">@VictorSanchez</a></p>&mdash; Juan Guerrero (@jguerrero_)
<a href="https://twitter.com/jguerrero_/statuses/329481447469613056">May 1, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

You can read the review below:

## Why? ##

As one of my passions is Web Performance, something I do whenever I apply for an open position is to
review how well they are doing on the front end.

This is a review of the landing page only, I keep other secrets for the Friday xD

## Review ##

MashMeTV front end is really pretty.

**THE END**

Wait, what? You wanted a real review? Maaan, you are so exigent, but ok, keep reading

## Parts ##

1. Slider
2. Image Size
3. Too many scripts
4. Many dns to resolve

### 1. Slider ###

Currently, the worst offender are the slider and the images it contains.
The logic that uses the slider is as follow:

```
From javascript, load all images contained in the slider
Check, every half second, if they have been all loaded
When they have been all loaded:
  Start the slider, which shows the first slide
  From now on, every 3 seconds, change to the next slide, which means:
    Remove current image from the DOM
    Create a new image with the new path
    Do the animation
    Change subtitle at the bottom right corner
```

The pros of this approach are:

  - It's simple
  - It's working now
  - Depending on the serverside, it could be easier to mantain the logic of
image loading on the client side, instead of serving the images right from
the html
  - Also, I can sneaky peek images not showed in the slider (muahahaha). This
could be removed easily by minifying files, so it isn't too much important

The cons of this approach are:

  - Loading the images from JS means that we are removing optimizations
  current browsers do while loading pages, like prefetching or parrallel
  downloads.
  - You are checking every half second if all the images have been loaded.
  It's better to invert the dependency and let the browser notify you when it
  has finished loading the images (the real win would be serving all
  from server, of course):

```javascript
var onloadImage = function(){
  console.log("image loaded: "+ this.src);
  unloadedImages--;
  if (unloadedImages === 0) {
    startSlider();
  }
};
```

  - Changing slides mean messing with the dom always, and that's not
good. As a side effect, you can't click on the imgs in the chrome's
element panel cause they get deselected when they are removed (damn
    you chrome)


The approach I would take for this would be:

  - Serve images and subtitles from the server. Something along the
lines of:

```html
<img class="slide" src="/static/images/landing/1.png" />
<img class="slide" src="/static/images/landing/2.png" />
<img class="slide" src="/static/images/landing/3.png" />
```

  - You still could wait until all images get loaded to start, using
the approach i described above
  - Instead of removing and adding a single image every slide
iteration, just have all of them hidden and changing slides
would be along the lines of:

```
Pick the selected image (just add slide-active whenever you change your slide, so it can be easily picked)
Get the next img sibling
Remove slide-active from the current image (after doing the fadeout effect)
Add slide-active to the new image (after doing the fadein effect)
```

### 2. Image Size ###

As you can see on the [WebPageTest
graphic](http://www.webpagetest.org/result/130501_0X_46W/1/details/), most components of the page
are loaded around 6 seconds, but page takes another 6 seconds to load just waiting for the slider
images.
One thing to take in consideration when looking at that graph is that as you have the image loading
code into a ```$(document).ready()``` block, they don't start loading until 2.5 secs, when the
onready event fires (vertical purple bar in the graph), delaying even more the page load.

I know the webpage is responsive, so images can grow in bigger pages, but all images are like
900x500, when on most user screens they wont take more than ~400px height (being that dimension the
    limited one on the css).
This means that they will be downloaded at full size, but only showed at 80% of its size in the best
case.

Also, all those images have transparent horizontal padding. I suppose it's to make easier to align
them on the laptop graphic, but I don't know if they are necessary.

### 3. Too Many Scripts ###

MashMeTV is serving 15 different js files currently. This means 15 from the 44 requests are just js
requests, or a 30%.
I know most of them are external dependencies, like google analytics or social buttons, but the less
http requests, the better.
So combining all possible files into one and minifying that file could improve the speed a lot.

Also, browsers block the downloading process each time they find a script until they have parsed and
executed it, so putting them at ```<head>``` is a bad choice. As I haven't seen any issue, I would
recommend moving them to the end of the body.

### 4. Many DNS To Resolve ###

An special issue with this page, not present in many other pages, is that it references resources
from many different domains:

```
www.mashme.tv
plusone.google.com
api.mixpanel.com
apis.google.com
platform.twitter.com
ssl.google-analytics.com
fbstatic-a.akamaihd.net
ssl.gstatic.com
ajax.cloudflare.com
r.twimg.com
s-static.ak.facebook.com
www.facebook.com
mixpanel.com
p.twitter.com
ajax.googleapis.com
connect.facebook.net
```

This means that for almost every domain in that list, the browser has to resolve the name for the
first resource with that domains, an operation that takes from 0.5 to 1 sec (The orange part in the
    graph linked previously)
To make it worse, most of that domains only have 1 request, so it's a huge payload upfront

## Conclusion ##

There are other minor issues that could be applied (minifying all external dependencies, combining
    files, creating sprite images,etc), but overall, MashMeTV front end is in great shape.

Applying just the 4 concepts explained here, it could go from _great_ to _OMG! Take all my money and
shut up!_

I hope you enjoyed the reading. See you on Friday.
