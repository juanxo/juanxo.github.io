---
  layout: post
  title: Chicisimo Frontend Review
---

Chicisimo Frontend Review
===============

<span class="post-date">{{ page.date | date_to_string }}</span>

A couple of weeks ago, I was at SpainJS where I met a lot of great developers. One of those
developers was [@pedrotgimenez](https://twitter.com/pedrotgimenez), a young aspiring craftsman, a
little bit too much GitHub obsessed ;). He's currently working on [Chicisimo](http://chicisimo.com),
as a backend developer.

I like to see what projects or techs are other people into, so, although he is on the backend and
most of his work is nearly invisible to the user, I checked out Chicisimo. It took a little bit to
load, and once loaded, I felt like the scroll was slow sometimes(not just while loading more user
images dinamically).

This, combined with my passion for Web Performance Optimization, as [I have shown in the
past](/posts/2013-05-01-mashmetv-frontend-review/), and frontend systems made me jump into Chrome's
devtools and start disecting the site. This is what I've found:

As there are a lot of things, I'm going to separate them in sections:

1. Resources
2. Javascript
3. CSS
4. Server

## Resources ##

### Minification ###
Minification is the process of reducing file size by removing all unnecesary characters and perform
some optimizations. It's mostly applied to JS & CSS, but sometimes is also applied to HTML.

In this case, it's a little bit strange, as it seems that the HTML is minified, but both JS and CSS
files aren't, except some external dependencies, like jQuery.

Looking at the [WebPageTest report for this
page](http://www.webpagetest.org/breakdown.php?test=130724_Z2_9RN&run=1&cached=0), it says that
currently there are 1.5MB of JS and 600KB of CSS. To put this into context, a page like Amazon has
300KB of JS and less than 100KB of CSS 0__o, and I don't think that Chicisimo has to have 5 times
more JS and CSS than Amazon.

So, how to solve this? The answer depends on the tech Chicisimo is using, but even if you don't have
anything for this, you can hack some small gruntjs scripts to do the job automatically.

In my tests, I have found that just minifying the files you reduce the size of the JS by 30%
(1014KB) and the size of the CSS by another 30% (406KB). A **30% overall** improvement just by removing
unnecesary characters. This is still a lot, but I'll adress the reason later.

### Bundle Resources ###

Any HTTP request has some overload, be it SSL negotiation, waiting for the server to create a
response, parsing the response, etc.

To make things worse, there are a couple of limitations about how many requests browsers do make
in parallel:

- There are a maximum number of requests that a browsers do in parallel for the same host. This
  number depends of each browser.
- Javascript resources block browser while it downloads them, parses them and execute them. This is
  because a javascript file can change the DOM, potentially trashing any work that would be done in
  the meantime. There are a couple of ways to work around this, like browser prefetching, async or
  defer scripts.

Taking all this into consideration, you have a hell lot of requests. **204 MOTHERFUCKING. REQUESTS**. A
similar site to Chicisimo in style, Pinterest, has only 86 requests, and even that number could be
too big. Just think about this: if every request in Chicisimo has an overload of 0.2
seconds([source](http://www.webpagetest.org/result/130724_Z2_9RN/1/details/), check green, orange
and purple bars) and there wouldn't be parallel downloads, with 233 requests, I could write this
article even before the page loaded (well, actually I couldn't cause I wouldn't take a look at it
yet, but you get the point)

What to do to decrease that number? There are a couple of things that come to my mind:

- Bundle all CSS and JS in one or two files: Depending on some factors, a number or another could be
  more beneficial( 2 scripts, one for external deps that doesn't change that often and another one for local scripts; 3 css links,
  one global, one per section and one per page specific content, reducing the size of the files too)
- Remove unused dependencies or files (more about this later)
- Create image sprites for small similar icons, like flags, badges, etc. Also, you could even try to
  sprite some user images together (this can cause some problems on mobile about file size)

### Gzipping ###

Gzipping is a technique that consists on serving compressed version of the resources, reducing the
number of bytes to send over the wire. This process usually reduces size by ~70%
Most webservers have an option to activate this. In this case, it's a litte bit strange because, as
it happened with the minification, only HTML files have gzip activated.

This technique is only useful for HTML, CSS and JS, as images for the web are usually compressed by
default and plays really well along the _Bundle Assets_ technique, allowing for better compression
rates.

Based on the tests, compressed CSS would be around 74KB and compressed JS would be at 346KB

Think about it: By just applying 3 simple techniques, CSS & JS size has gone from 2.1MB to 420KB.
This means an 80% reduction in CSS & JS size and almost a 50% in overall size (from 4.1MB to 2.4MB)

### Resource Distribution Across the Page ###

As I have said previously, JS scripts block the browser, so it's a good practice to put all scripts
at the end of the body.
You can see a visual representation of this in the [network cascade of
webpagetest](http://www.webpagetest.org/result/130724_Z2_9RN/1/details/). CSS files download almost
in parallel until the browser hits the scripts (request #18)

### Images ###

Currently Chicisimo is serving 138 images for a total of 1.6MB. There are several ways to reduce
both the number of requests and the total size

#### Avoid Resizing on Client ####
One of the worst thing that we can do to the browser is force it to scale images unnecessarily. This
both means increasing the cpu usage to transform the images and reducing the final quality of the
images. Letting retina aside (as it seems you aren't actively supporting it), the closest (read: exact) the
size we can serve an image to the size specified, the better. Check out the following screenshot of
the timeline to see how image resizing affects performance:

![Timeline](/images/chicisimo-timeline.png)

Checking the frame bars at top, you can see that many of the frames go over 60fps and many of them
are full of green(paint events). If you dig around those frames, you will see that many of those
events are about decoding or resizing images(Left panel).

These events come from the dynamic image loading that you do with the users images, forcing them to
a different size from the original one (from 290x425 to 245x359 on the main images and 50x50 to
30x30 on the user avatar, for example)

#### Compress Images ####
Images are compressed by default. But sometimes these compressions include a lot of unnecessary
information (metadata, color palette, comments) that increase the file size without improving the
image at all.

I have used [a project of mine](http://juanxo.github.io/projects/content-tasks.html) to automate this
compression, but you can choose whatever you prefer.

After both optimizations, the size is reduced about 600KB.

### Unnecessary Resources ###

The best resource is the one you don't have to use. So removing all the bits that you aren't using
can get you a lot of performance improvement.
For example, looking at the list of js files that Chicisimo references, we can spot a couple of guys
that shouldn't be there:

* hyphenator.js (**268KB**): I haven't spotted any place in all your scrips where you use it.
* log4javascript.js (**128KB**): It's only used a couple of times inside _chicisimo.fb.js_, but
  logging debug traces in production shouldn't be that neccessary

Just by removing those 2 files you get rid of 400KB (that would be less after minifying and
compressing, but anyway, it't a lot)

Another case of unnecessary resources (in the form of resource duplication) is related to using
different versions (that have totally the same content) of your images. For example:

* all-sprites.png(**16KB**) is included three times
* new-all-sprites.png(**96KB**) is included twice

## Javascript ##

Chicisimo has a lot of interactions with the client, relying in JS for a great part of the site. But
there are a couple of places where the way you use JS decreases the site's performance.
The main offenders are all the $().scroll functions.

Chicisimo has a lot of big images and interactions (zoom cursor and overlay on user images, comment
count fading, etc). All these interactions are active while scrolling, meaning that scrolling can
trigger a lot of animations. There are solutions for these (mainly triggering the animations with
javascript, where you can disable them while scrolling) but it's not a performance sink.

Searching for _scroll_ jquery function in your site gives back 7 results (7 functions attached to
scroll event) and most of those functions do a hell load of work. Mainly, asking for things like
offests, dimensions, animations cause a lof of layout trashing and restylings that decrease, a lot,
the performance. Take into consideration that scroll events trigger many times during a user scroll.

Another place where you are overloading the site is on dynamic page loading. You are asking for a
final html version of the new page, to append it directly. That response weights 59KB.
To put it into context, if you load all 10 default pages, you are downloading another 600KB of data.

I would go with JSON + Client Side rendering (as you are using Backbone already). That would mean a
1-2KB download + templates and logic (and these would be reused between requests)

## CSS ##

CSS plays a great part on a site's performance, especially if it's a site with a lot of animations,
images and little details, as is the case of Chicisimo.

Some things to take into consideration are:

* Don't overspecify selectors
* On the other hand, don't use loose selectors (tag selectors, universal selector)
* Try to minimize unused rules

To do this part, I have used [another tool I
made](https://juanxo.github.io/projects/css-rule-checker.html), Chrome's CSS analyzer and DevTool's
Timeline

The output of this CSSRuleChecker for this site looks like:

```
Stylesheet 0: http://chicisimo.com/wp-content/plugins/wp-postratings/postratings-css.css?ver=1.50
  appliedSelectorCount: 0, totalSelectorCount: 6
Stylesheet 1: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/combobox.css?ver=21092012
  appliedSelectorCount: 1, totalSelectorCount: 41
Stylesheet 2: http://chicisimo.com/wp-content/plugins/on-post-outfit/assets/css/onimage/jquery.onimage.css?ver=15032012
  appliedSelectorCount: 0, totalSelectorCount: 22
Stylesheet 3: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/chicisimo.brands.css?ver=10102012
  appliedSelectorCount: 0, totalSelectorCount: 162
Stylesheet 4: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/buttons/sexybuttons.css?ver=3.2.1
  appliedSelectorCount: 14, totalSelectorCount: 30
Stylesheet 5: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/js/colorbox/colorbox.css?ver=3.2.1
  appliedSelectorCount: 27, totalSelectorCount: 39
Stylesheet 6: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/960.css?ver=3.2.1
  appliedSelectorCount: 4, totalSelectorCount: 123
Stylesheet 7: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/wrapper.css?ver=3.2.1
  appliedSelectorCount: 7, totalSelectorCount: 11
Stylesheet 8: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/jquery-ui.custom/jquery-ui.custom.css?ver=3.2.1
  appliedSelectorCount: 0, totalSelectorCount: 63
Stylesheet 9: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/jquery.selectBox.css?ver=3.2.1
  appliedSelectorCount: 0, totalSelectorCount: 18
Stylesheet 10: http://chicisimo.com/wp-content/plugins/on-post-outfit/assets/css/jquery.autocomplete.css?ver=1.50
  appliedSelectorCount: 0, totalSelectorCount: 6
Stylesheet 11: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/chicisimo.sizes.css?ver=09072013
  appliedSelectorCount: 1, totalSelectorCount: 298
Stylesheet 12: http://chicisimo.com/wp-content/themes/chicisimo-social/style.css?ver=19072013A
  appliedSelectorCount: 195, totalSelectorCount: 2236
Stylesheet 13: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/chicisimo.past.css?ver=19072013
  appliedSelectorCount: 90, totalSelectorCount: 922
Stylesheet 14: http://chicisimo.com/wp-content/themes/chicisimo-social/_inc/css/chicisimo.style.css?ver=19072013
  appliedSelectorCount: 85, totalSelectorCount: 455
Stylesheet 15: Inline Style
  appliedSelectorCount: 0, totalSelectorCount: 10
Stylesheet 16: Inline Style
  appliedSelectorCount: 2, totalSelectorCount: 74
Stylesheet 17: Inline Style
  appliedSelectorCount: 0, totalSelectorCount: 61

There are 4577 rules
Applied: 426, Not applied: 4151
Median selectorsInRule: 1, partsInSelector: 3
95th percentil selectorsInRule: 2, partsInSelector: 7
Average selectorInRules: 1.1972908018352633, partsInSelector: 3.396897810218978
Max

  partsInSelector:
    count: 12
    rule: ".popup-content .look_content .image:hover #popup-action-buttons > div.chics.chicing.chiched a span"
  selectorInRule:
    count: 62
    rule: "html, body, div, span, applet, object, iframe, h1, h2, h3, h4, h5, h6, p, blockquote, pre, a,
      abbr, acronym, address, big, cite, code, del, dfn, em, font, img, ins, kbd, q, s, samp, small,
      strike, strong, sub, sup, tt, var, b, u, i, center, dl, dt, dd, ol, ul, li, fieldset, form, label,
      legend, table, caption, tbody, tfoot, thead, tr, th, td"

Frequencies
  partsInSelector:
    1: 725
    2: 1385
    3: 1120
    4: 934
    5: 627
    6: 313
    7: 169
    8: 105
    9: 53
    10: 29
    11: 19
    12: 1
  selectorsInRule:
    1: 3984
    2: 458
    3: 80
    4: 33
    5: 9
    6: 5
    7: 1
    9: 4
    12: 1
    16: 1
    62: 1

```
_Note: PartsInSelector refers to hoy many different elements has a selector (div.class[disabled] has
3 parts). SelectorsInRule refers to hoy many different selectors a rule has (h1,h2,h3,h4 has 3
selectors)_

So, looking at these results we can draw some conclusions:

* Less than 10% of the rules are used. Sites usually have a lot of unused rules but using only an 8%
  its way too low.
* Some rules are overspecified: Take the selector with the most parts (*.popup-content .look_content .image:hover #popup-action-buttons > div.chics.chicing.chiched a span*). It has an id at the middle, so all things before that id are worthless. The way that browsers work, this selector isn't efficient, as it will take ALL spans, rendering the id useless. Also, .chics.chicing.chiced? Come with a better name please, this is redundant. In general, using more than 3 parts for a selector is a sign of poor specificity or layout.

On the other hand, there are some places where you can improve performance (especially on painting times).
You can take advantage of latests browsers for this. For example, take a look at these 2 different
timelines. Both are recorded scrolling on a page with only your header.

![Shot1](/images/chicisimo-header-without-transform.png)

![Shot2](/images/chicisimo-header-with-transform.png)

The only difference between them is that the second one has a `-webkit-transform: translateZ(0)`
applied to it

## Server ##

After all, Pedro weren't going to save so easily ;)
Being the black box that is the server side, the only issue I spot is the Time to First Byte. On the
WebPageTest page, you can see that it's at 0.7s. These means that the server takes almost a second
to generate the response. WPT reports a desired time of 0.2s. The ways to do this are:

- Obviously, reduce the steps required to generate the responses (optimizing queries, etc)
- Chunked responses: After having generated the head element (with all the external resources),
  flush the content. This means that the client gets something earlier, and also it can start
  prefetching resources. It's a win-win.


So this is all for now. I hope you like it.
