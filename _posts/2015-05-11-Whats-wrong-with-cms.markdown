---
title: What's wrong with CMS systems
layout: post
category: commentary
---

## What's wrong with CMS systems now

I'd briefly like to brainstorm what's fundamentally wrong with major
systems like wordpress in deployment now. In fairness, my interaction with
these systems is mainly informed by support operations at a cpanel webhosting
company, so the ways I see the cms system is often tied to the places
where it does introduce problems large enough to elicit concerns.

# Security implications of editable pages

Write access from the internet is a really bad idea.
The promise is that "non-technical" content authors can
manage the system in a way that doesn't require typing any commands.
This is a great idea. The pitfalls have always been when you
try to elevate the notion of content into commands, or full-blown
dashboards that allow editing the core of the system, creating new
scripts, editing the styles, and installing extensions or plugins. The
possibly misguided premise here was that non-technical content authors
can become their own developers. This allows a publicly exposed interface
that makes a webserver into a globally editable turing machine,
you can put any code anywhere with a wordpress admin account. 

# Plugins as a developer substitute

Plugins in wordpress allow a means to add new functionality to a site,
by installing a project through a simple marketplace style interface.
This can vary from code that changes or disables core functions, or that
adds new capabilities (like a lightbox widget). These plugins are developed by
individuals or small companies, and these come and go. Updates may cease
for free plugins, and paid plugins still have serious problems. Many plugins
introduce dependencies or suggestions that other components be installed, and each
plugin runs on each page, registers its configuration components, and potentially
add database interactions on each step (more on this later).

As the plugin architecture is designed to make this the easiest way for users to
become developers, it's expected that a lot of amateur or first-attempt plugins will
be made, and there might be multiple options of varying quality for any particular need.

# Themes as designer substitutes

The smallest theme contains the css and basic menu/navigation means. This normally
boils down to the stylesheet, any supporting scripts (think bootstrap), and some
minimal server code generating a header and footer. These are planned to make the site
uniform without adding boilerplate to every page ([DRY](http://www.c2.com/cgi/wiki?DontRepeatYourself).)

This is laudable, and allows, like plugins, a site design marketplace. The first problem is that the
headers and footers are first class citizens on the machine. There's not any limit on the kinds
of code that can be run in the php used to generate the footers compared to the code used to access
the database or add new functionality. This means any code in a header or footer, will run with full
access to the system and on every page load. That's a huge performance impact, and a critical inject malware.

# Friendly error pages load the entire site

Two conspiring factors are at play here. One is the replacement of actual paths with friendly urls using
mod_rewrite or the equivalent, sending all content that's not static to the index.php entry point,
and letting this dispatch (so http://mysite.com/kitties/ actually loads /index.php, which  sees the url
was /kitties/ and gets a page, category, or post from the database). The second is the desire to again
give uniform appearance to the site, so the index.php entry point, after deciding there are no posts, pages,
or categories that relate  to /puppies/, you get a "page not found" 404 page that looks like the main site,
and possibly suggests the visitor see other pages. The cost of this is that you then run the code for the site
to do this. Since a 404 not found page is almost always going to be a cache-miss, any caching you have will
not be effective, and the 404 error is now more expensive than any activity a real visitor might exercise.
This makes a scripted scanner (like [wpscan](https://github.com/wpscanteam/wpscan), for example) effective
at a small scale Denial of Service on the server. 