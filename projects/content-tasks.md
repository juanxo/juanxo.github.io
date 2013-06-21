---
  layout: project
  title: Content Tasks
---

Content Tasks
============

<div class="project-links">
  <a class="project-link" href="https://github.com/Juanxo/content-tasks">GitHub Project</a>
</div>

Content Tasks is a set of small ruby scripts to automate some tedious common tasks on web development.
Currently there are 4 tasks:

- Compress jpg files
- Compress png files
- Convert unanimated gifs to png
- Process assets, creating concatenated and minified bundles (alas small subset of Sprockets)

Usage
=====

Just call the desired script with the folder to process as a parameter:

    $ ruby compress_png.rb myimagedir

And it will process the files while keeping you updated about the progress.

![Output](/images/content-tasks-output.png "Output")

Be aware that processed objects will be stored only **IF** their size is smaller than the original
object
