---
  layout: project
  title: Pagos Movistar
---

# Pagos Movistar #

<div class="project-links">
  <a href="https://github.com/Juanxo/pagosMovistar">GitHub Project</a>
</div>

Pagos Movistar is a little internal project I developed to help testers at my job. Testing process
includes checking the purchases on external systems. But this process is painly slow and error
prone, as you have to use a different login for each product and click through a couple of pages
before you get to the results.

My solution was to develop a small RoR application that showed the user a list of all products and
let him/her select what products he/she want to check.

Once the products are checked and user has started the process, the application calls the external system asynchronously, once for
each product, and do some magic (as login filling automatically the parameters and getting the
results). All this without changing the current page, showing results in real-time. If you have
selected a lot of products, you can even go grab a coffee, Pagos Movistar will happily work for you.

_Note: I have removed all the sensible information of this application, meaning that it won't even
compile. This is just for portfolio purposes._

# Screenshots #

![Screenshot1](/images/pagos-movistar-header.jpg)

![Screenshot2](/images/pagos-movistar-product.png)

![Screenshot2](/images/pagos-movistar-result.png)
