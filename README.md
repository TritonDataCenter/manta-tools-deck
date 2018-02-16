# Manta Tools Overview

This deck is a brief overview of tools that are useful for Manta operators.
It's inspired by Trent Mick's [Some Tools I
Use](https://github.com/trentm/talk-some-tools-i-use) and [Advanced DTrace:
Tips, Tricks, and Gotchas](http://dtrace.org/resources/bmc/dtrace_tips.pdf).


## Format and "build"

This repository contains both source files and generated files.  To view the
deck, just look at the generated files in the "docs" directory.  Editing the
deck requires more work.

The "manta-tools.md" Markdown file contains the source for the talk's content.
It's intended to be used with "reveal.js".  Unfortunately, it's not clear how to
effectively package a Markdown-based "reveal.js" talk.  Here's what's been done
to generate the sources here today.  From the root of a clone of this repo:

    $ git clone https://github.com/hakimel/reveal.js deps-reveal.js
    ...
    $ cd deps-reveal.js/
    $ git checkout 3.6.0
    $ npm install
    $ cp ../manta-tools.htm ../manta-tools.md .
    $ npm start

Then:

* Load in Chrome: `http://localhost:8000/manta-tools.htm`
* Right-click the page and click "Save As..."
* Navigate to the root of the clone of *this* repository and save the file as
  type "Webpage, Complete" called "docs/index.html".
* `git add` and commit the newly-added files.

Improvements to this process would be most welcome, including possibly
converting the source of the deck into a format more suitable for packaged
presentations (or even a format that works for `mdpress`, which interprets
nested code blocks differently than reveal.js does here, even though it uses
reveal.js).
