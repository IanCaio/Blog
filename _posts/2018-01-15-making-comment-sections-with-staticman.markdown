---
layout: blogpost
title:  "Making comment sections with Staticman"
author: "Ian Caio"
date:   2018-01-15 23:29:34 -0200
header-image: "/imgs/posts/2/staticman.png"
category: "Front-End"
---
While researching about ways to implement comment sections in static webpages,
I noticed many blogs out there use third-party commenting systems. I'd say
a considerable number of the ones I visited used Disqus or WordPress Jetpack,
but there are many options out there, like Facebook Comments and Google+ Comments.
Even when choosing a third-party commenting system there are some advantadges and
disvantadges about each one you'll have to pounder before choosing.
But generally speaking, as tempting as it was to use a commenting system without
worrying too much about any coding myself (except what I'd need to embbed it),
I had a few reasons that made me reconsider it:

### Using third-party code makes your website only as secure as the third-party website

Don't take me wrong, I'm not saying I'm better than the Disqus, Jetpack, or any
other commenting system security staff, but I'm the type of person that dislikes
paying for my mistakes, but dislikes even more to be paying for somebody else's
mistakes. So if some day my website is compromised, I'd rather know it was my
fault (and learn from it) than to know it was due to some company not taking the
necessary security measures.

But not only that: Unless you have a very popular website, it's likely those
comment systems are much more valued targets to hackers than your website itself.
And who knows if one of their attempts might succeed. It has
[happened](https://www.csoonline.com/article/3076225/security/flaw-in-popular-wordpress-plug-in-jetpack-puts-over-a-million-websites-at-risk.html)
[before](https://thehackernews.com/2017/10/disqus-comment-system-hacked.html)...

### Having less control on changes you'd like to do in the comment system

Since you're using third-party code, you will be restrained to their
implementation of the comment system and adding new features will be really
hard, sometimes not possible at all. Even layout customizations will be more tricky.

### You don't really "own" your comments

The comments are stored in the third-party comment system database and most of
them don't allow exports. So you don't really "own" the comments from your website. One of
the consequences is that if the comment system you use suddenly vanishes (unlikely for big
companies but possible for some not-so-big ones), so will all your comments.

This also makes difficult to make use of the comments information for anything other
than what the system you're using allows you to. For example, say you want to make some
statistics about the commentaries, or maybe some section to search for a comment by content
or date. Unless the system already has this built-in capability you won't be able to
do it, since you don't have direct access to the comments database.

### Privacy policies most third-party commenting systems adopt

Most of the third-party commenting system companies require you to create an
account. And they usually track your activity when using it. Disqus used to
[create a profile](https://www.baekdal.com/insights/the-first-rule-of-privacy)
with all the comments you ever made with their system and also the blogs you own
that use it. They now allow you to
[disable](https://help.disqus.com/customer/portal/articles/1197204-making-your-activity-private)
the public profile but still track your activity to build a profile of preferences,
similar to Facebook. Some people might not be comfortable with those privacy concerns.

### I'm a stubborn "reinvent the wheel" person

Guilty as charged.
{: .extra-margin}

## The alternative
{: .center}

As I was doing some research I ran into a possible alternative:
[Staticman App](https://github.com/eduardoboucas/staticman). That opensource app opens the possibility of
posting information directly to a Github page through the Github API. You add the app's
github account as a collaborator, HTTP/POST the forms to an url defined by the app and
the content will be commited to your page repository in the path specified in the
configuration file in one of the supported formats (`yml`, `yaml`, `json` or `frontmatter`).
You can use the [public instance of the app](https://staticman.net/) or set up your own.

Sounded very good, but there was still one thing I still wanted to circunvent, which was having
to open the repository of the website for edition completely. It raises security concerns
in the case of a breach in the App. You can still keep a regular backup of your website
locally and easily remedy it in the case of a compromise, but I also had another idea:

_Compartmentalization_.

> In matters concerning information security, whether public or private sector,
> compartmentalization is the limiting of access to information to persons or other entities
> who need to know it in order to perform certain tasks.
>
> <span class="author">Wikipedia - Compartmentalization (information security)</span>

Instead of opening the whole website repository, we could create one dedicated only to
store and organize the comments. The website would then use AJAX (_Asynchronous Javascript
and XML_) calls to fetch the comments and display them in the article page. That way, any
compromises would be limited to the comment "database" instead of the whole webpage.

On the next articles I'll go into details on the issues I faced while working in the comment
system and how I solved them.
