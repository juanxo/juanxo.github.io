---
  layout: project
  title: Backbone Teambox Tasks
---
# Backbone Teambox Tasks #

<div class="project-links">
  <a class="project-link" href="https://github.com/Juanxo/backbone-teambox-tasks">GitHub Project</a>
  <a class="project-link" href="https://teambox-tasks.herokuapp.com">Project Webpage</a>
</div>

This is your daily dose of todo applications made to learn Backbone.js, but with the twist of using
an external API (teambox). You will need a Teambox account in order to try it.

_Disclaimer: This isn't by any means at the same level of quality and polish that Teambox. If
you are thinking about using a great project management tool, check [them](https://teambox.com)_

![Screenshot](https://raw.github.com/Juanxo/backbone-teambox-tasks/master/screenshot.png)


# Why #

In a conversation over email (sooo retro) with @masylum(their CTO), he told me that they moved to a
client-side solution, using Backbone.js.

I've never tried any client-side framework before (I used to think that they weren't really
neccessary, as you could squeeze enough performance from a conventional rails application and it
would be far easier to maintain)

# TODO #

I just started to get my feet wet with client side frameworks, and there are some things I would
like to do in this application:

* Expandable tasks: Click on task header to expand them (nested views, etc)
* Better user feedback: Warn about auth popup, spinner on tasks loading.
* Add task lists and filters (Backbone.Rel, events et al.)
* Server side templates (trimmer perhaps, once I get my head around it)
* Better way to oauth (not having to give authorization everytime you connect and get rid of the
  popup window)
