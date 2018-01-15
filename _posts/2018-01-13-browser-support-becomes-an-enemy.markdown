---
layout: blogpost
title:  "B(r)owser Koopa: Browser support becomes an enemy"
author: "Ian Caio"
date:   2018-01-13 01:15:34 -0200
header-image: "/imgs/posts/1/notebook.png"
category: "Front-End"
---
I want to start writing on this blog talking about an issue that probably
haunts every single front-end web-developer out there: _browser support_.

It's a very controversal discussion, where you can find all different sets of
opinions. So many that adding to this discussion is a difficult task. One thing
that is common ground between all those divergent lines of thought, is
that you should support what your client **needs** (slash requires) you to support.

Sometimes those needs are not clear and show up in surprising ways: I once read
a comment from a web-developer about how he made a website for an hospital and
received complaints about it displaying wrong. For his surprise their computers still
had IE6 installed, and they couldn't upgrade because their intranet software
would break with a browser update.

But what about those moments where the decision is up to you? How do you decide
where to draw the line?

The problem with being overly supportive lies on having a "ball and chain" keeping
you from using new and useful technology because older browsers don't support it.
[_Flexbox_](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) is an example.
It's great for making fluid responsive designs, but if you decide to use it you'll
give up support for IE9 and older. You'll also have to be sure not to step on any
[bugs](https://caniuse.com/#search=flexbox) when displaying it on IE11.
Bootstrap 4, the newest version of the popular HTML/CSS and JS framework, decided
to drop support for IE9, so it can be fully designed over _Flexbox_.

There are other CSS features that are even less supported, like _Grid_ layout. It tries
to fill the gaps left by other layout methods by being very straight forward when dealing
with two dimensional positioning of elements. But it's still only supported on the most
recent versions of some browsers. Not only that, but some implementations were based on
it's older specification. We'll still have to wait a while to use _Grid_ on production
websites...

If CSS support wasn't enough, you'll have to worry about writing or using scripts that
are compatible with the browsers you need to support. Javascript ECMAScript 5 specification
adds some useful features, like the _Function.prototype.bind()_ method, is not supported by
IE 6 and 7. Arrow functions are not supported at all on IE. Polyfills and workarounds
will be often required when trying to make scripts work on older browsers. Did I mention
HTML5 support and elements like `<input type="range">`?

That was barely scratching the surface, browser compatibility will be a constant concern that
can be relieved with some good practices, but will never vanish. If you are free to make a choice
though, drawing your line below IE9 might be removing a better experience on the
users of recent browsers, increasing a lot of work time and not winning much in return.
