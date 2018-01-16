---
layout: blogpost
title:  "Making comment sections with Staticman"
author: "Ian Caio"
date:   2018-01-15 23:29:34 -0200
header-image: "/imgs/posts/1/notebook.png"
category: "Front-End"
---
While researching about ways to implement comment sections in static pages, I noticed many blogs out there use third-party commenting systems. I'd say a considerable number of the ones I visited used Disqus. As tempting as it was to use a ready-to-go commenting system and not have to worry about any coding myself, I had a few reasons that made me reconsider it:

### Using third-party code makes your website only as secure as the third-party website. Any security holes there could implicate in bad content in your website.

Don't take me wrong, I'm not saying I'm better than the Disqus or WP Jetpack security staff (or any other commenting system company), but I'm the type of person that dislikes paying for my mistakes, but dislikes even more to be paying for somebody else's mistake. So if some day my website is taken down, I'd rather know it was my fault (and learn from it) than to know it was due to some company not taking the necessary security measures.

But not only that: Unless you have a very popular website, it's likely those comment systems are much more targeted to hacker attacks than your website itself. And who knows if one of those attempts might succeed, it has [happened](https://www.csoonline.com/article/3076225/security/flaw-in-popular-wordpress-plug-in-jetpack-puts-over-a-million-websites-at-risk.html) [before](https://thehackernews.com/2017/10/disqus-comment-system-hacked.html)...

### Having less control on changes you'd like to do in the comment system.

Since you're using third-party code, you will be restrained to their implementation of the comment system and adding new features will be really hard, some times not possible at all. Even layout customizations will be more tricky.

### Privacy policies most third-party commenting systems adopt.

Most of the third-party commenting system companies require you to create an account. And they usually track your activity when using it. Disqus used to [create a profile](https://www.baekdal.com/insights/the-first-rule-of-privacy) with all the comments you ever made with their system and also the blogs you have using their system. They now allow you to [disable](https://help.disqus.com/customer/portal/articles/1197204-making-your-activity-private) the public profile but still tracks your activity to build a profile of preferences, similar to Facebook. Some people might not be comfortable with those privacy concerns.

## The alternative

While doing some research I ran into a cool alternative: Staticman App. That opensource app simply creates the possibility of posting information to a Github page through the Github API. You add the app account as a collaborator, POST the forms to an url defined by the App and the content posted will be commited to your page repository in the path specified in a configuration file. The only thing I still wanted to workaround was having to open the repository of the website for edition completely, which raises security concerns in the case of a breach in the App.

So I thought for a while and decided I'd try to implement the comments having a repository for the blog itself and another repository for the commentaries only, which would be the one shared with Staticman App. The blog would then fetch the comments from that repository through AJAX calls and fill the comment section of each post. But the world ain't all sunshine and rainbows, and this approach would increase the complexity A LOT.

On the next posts I'll go into details on the issues I faced while working in the comment system and how I figured my way out of them.

...

Note that this method can only work if your blog repository and comments repository are in the same domain. If you are using the same github account to host both and didn't change the domain (or changed both to the same domain) you can still make this work. If you have different domains for each of them, you'll run into issues because of the Cross-Domain requests policies. It basically states that **normally** a script shouldn't be able to request data from another domain.

As you can see in the screenshots below, it does work if you have the same domain set up:

(screenshots)

References:

http://perltricks.com/article/104/2014/7/29/Your-users-deserve-better-than-Disqus/
