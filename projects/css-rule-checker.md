---
  layout: project
  title: CSS Rule Checker
---
Css Rule Checker
================

<div class="project-links">
  <a class="project-link" href="https://github.com/Juanxo/css-rule-checker">GitHub Project</a>
</div>

CSS Rule Checker (CRC from now on) is a javascript bookmarklet that analyzes your webpage's CSS and gives you an
overview of its current state through some statistics.

By default, CRC gives you 7 results, printing them to the browser console:

1. Total selector count
2. Summary of how many rules have been applied to the current page
3. Median
4. 95th percentil
5. Average
6. Max (showing the offender)
7. Frequencies

In rules 3 to 7 there is also a distinction between the number of selectors in a rule and the number
of elements in a selector

_Warning: Many webpages and browsers block javascript coming from non-secure domains. If that's your
case, try CRC with your dev version_

To use it, create a new bookmark in your browser and use the following as link:

```javascript
javascript:(function(){var%20script=document.createElement('script');script.src='http://juanxo.github.io/scripts/css-rule-checker/rulechecker.js';document.body.appendChild(script);})()
```

Then, browse to the URL that you want to test and click the bookmarklet. It should show something
along the lines of this (this results are from [MashMeTV](http://mashme.tv)):

```javascript
There are 541 rules
Applied: 93, Not applied: 448
median {
  selectorsInRule: 1,
  partsInSelector: 2
}
95th percentil {
  selectorsInRule: 2,
  partsInSelector: 6
}
average {
  selectorInRules: 1.2643253234750462,
  partsInSelector: 2.902046783625731
}
max {
  selectorInRule: {
    count: 68,
    rule: "a, abbr, address, article, aside, audio, b, blockquote, body, caption, cite, code, dd, del, dfn, dialog, div, dl, dt, em, fieldset, figure, footer, form, h1, h2, h3, h4, h5, h6, header, hgroup, hr, html, i, iframe, img, ins, kbd, label, legend, li, mark, menu, nav, object, ol, p, pre, q, samp, section, small, span, strong, sub, sup, table, tbody, td, tfoot, th, thead, time, tr, ul, var, video"
  }
  partsInSelector: {
    count: 11
    rule: "html#landing body article .columns .left_column .step .content #coin-slider #slider_buttons .slider_btn:hover"
  }
}

frequencies {
  partsInSelector: {
    1: 182
    2: 173
    3: 86
    4: 120
    5: 77
    6: 18
    7: 12
    8: 9
    9: 3
    10: 1
    11: 3
  }
  selectorsInRule: {
    1: 486
    2: 40
    3: 9
    4: 3
    5: 1
    6: 1
    68: 1
  }
}
```

![refactor all the css](http://cdn.memegenerator.net/instances/400x/26947966.jpg)
