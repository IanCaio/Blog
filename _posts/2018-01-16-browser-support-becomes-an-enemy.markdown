---
layout: blogpost
title:  "B(r)owser Koopa: Browser support becomes an enemy"
author: "Ian Caio"
date:   2018-01-16 19:14:00 -0200
header-image: "/imgs/posts/1/notebook.png"
comment-section: "https://iancaio.github.io/blog-comments/get/post1comments.json"
comment-post-link: "https://api.staticman.net/v2/entry/IanCaio/blog-comments/master/post1comments"
category: "Front-End"
---
I want to start writing on this blog talking about an issue that probably haunts every single front-end web-developer out there: _browser support_.

It's a very controversal discussion, where you can find all different sets of opinions. So many that adding to this discussion is a difficult task. One thing that is common ground between all those divergent lines of thought, is that you should support what your client **needs** (slash requires) you to support. Sounds obvious, but sometimes those needs are not clear and show up in surprising ways.

<figure class="center medium">
<img src="{{site.baseurl}}/imgs/posts/1/oldcomputer.jpeg" alt="old computer"/>
<figcaption>What if that's the kind of computer your client is using?</figcaption>
</figure>

I once read a comment from a web-developer about how he made a website for an hospital and received complaints about it displaying wrong. For his surprise their computers still had IE6 installed, and they weren't willing to upgrade because their intranet software would break with a browser update. Apparently that is not an uncommon thing on the health system. In 2015, IE 7 and 8 accounted for 61% of England's and Scotland's [National Health Systems traffic](https://www.linkedin.com/pulse/nhs-browser-statistics-mark-reynolds), and we are talking about developed countries. Thankfully things improved since 2015, specially in the regular personal user statistics, and old IE browsers have been increasingly losing space. Some websites build statistics according to the user-agent header and display those freely: [W3Schools](https://www.w3schools.com/browsers/default.asp), [Data Gov](https://data.gov.uk/data/site-usage#browsers_versions) and [StatCounter](http://gs.statcounter.com/) are some examples.

Some very sound advice:

> Any IT professional who is still allowing IE6 to be used in a corporate setting is guilty of malpractice.
>
> <span class="author">Ed Bott - American technology journalist</span>

Unfortunately there are still many unprepared IT employees out there ignoring that advice, and convincing them to change might turn into a arduous quest. So in those cases, you don't have much choice left besides making the website supportive or decline the job.

But what about those moments where the decision is up to you? How do you decide where to draw the line?

The problem with being overly supportive lies on having a "ball and chain" keeping you from using new and useful technology because older browsers don't support it. [_Flexbox_](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) is an example. It's great for making fluid responsive designs, but if you decide to use it you'll give up support for IE9 and older. You'll also have to be sure not to step on any [bugs](https://caniuse.com/#search=flexbox) when displaying it on IE11. Bootstrap 4, the newest version of the popular HTML/CSS and JS framework, decided to drop support for IE9, so it can be fully designed over _Flexbox_. With the large increase in the number of mobile devices, relying on the old positioning paradigms is completely counterintuitive. They simply weren't designed to deal with screen size adaptation and you will have to work much harder circunvent that.

There are other CSS features that are even less supported, like _Grid_ layout. It tries to fill the gaps left by other layout methods by being very straight forward when dealing with two dimensional positioning of elements. But it's still only supported on the most recent versions of some browsers. Not only that, but some implementations were based on it's older specification. We'll still have to wait a while to use _Grid_ on production websites...

If CSS support wasn't enough, you'll have to worry about writing or using scripts that are compatible with the browsers you need to support. Javascript ECMAScript 5 specification adds some useful features, like the `Function.prototype.bind()`{:.language-javascript} method, is not supported by IE 6 and 7. Arrow functions are not supported at all on IE. Javascript evolved with time, which is completely natural since it was [designed in only 10 days](https://www.jwz.org/blog/2010/10/every-day-i-learn-something-new-and-stupid/#comment-1021). The older the browser, the more likely it will be not comforming to the latest standards, making polyfills and workarounds often required when trying to write scripts for them.

The 5th version of the HTML standard (a.k.a. HTML5) was released in 2014, so older browsers will not support the new semantics (tags like `<video>`{:.language-html}, `<article>`{:.language-html}, `<header>`{:.language-html} or `<figure>`{:.language-html}, `datasets`{:.language-html}, new attributes like `required`{:.language-html}, `<input type="range">`{:.language-html} and many others). IE9 provides very [partial support](https://html5test.com/compare/browser/ie-9.html) for HTML5. It also gives partial [support to CSS3](https://www.impressivewebs.com/css3-support-ie9/) properties, but some like `text-shadow`{:.language-css}, `gradient`{:.language-css} and `transitions`{:.language-css} are left out (IE versions below 9 don't support CSS3 at all).

That is barely scratching the surface, browser compatibility will be a constant concern that can be relieved with some good practices, but will never vanish. If you are free to make a choice though, drawing your line below IE9 might be removing a better experience on the users of recent browsers, increasing a lot of work time and likely not winning much in return. As much as we need to try to be as inclusive as possible and code in a way that stimulate a graceful degradation, at some points we will need to leave ghosts behind and embrace enhancements. After you're done, if you think it's worth it you can pull up the sleeves and work on some renderable fallback, focusing on the most important elements instead of the whole website.
