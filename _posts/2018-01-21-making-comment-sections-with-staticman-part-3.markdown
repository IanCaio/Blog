---
layout: blogpost
title:  "Making comment sections with Staticman - Part 3"
author: "Ian Caio"
date:   2018-01-21 12:55:31 -0200
header-image: "/imgs/posts/2/staticman.png"
comment-section: "https://iancaio.github.io/blog-comments/get/post4comments.json"
comment-post-link: "https://api.staticman.net/v2/entry/IanCaio/blog-comments/master/post4comments"
category: "Front-End"
---
On the final part of this articles series I discuss the client-side part
of the comment system. Making the layout and using Javascript to safely display
the comments on the page.

[Click here]({{ site.baseurl }}{{ page.previous.previous.url }})
if you want to start reading from the first article!

*DISCLAIMER: This article was made for educational purposes only. The comment system was just
recently deployed and is still under testing. It seems functional and bug free, but I'm not
responsable for the misuse of the code or any consequences that might arise from using it
on production websites.*
{: .small-text .extra-margin }

## Working on the CSS template for the comment section

I'm not going to describe in details how I designed the CSS template for the comment section,
since this is outside the scope of these articles. But I wanted to show how I styled the
nesting:

```css
.comment-div {
	padding: 0.5rem 0 0.5rem 5%;
}
.comment-div .comment-div .comment-div .comment-div .comment-div {
	padding: 0.5rem 0;
}
```

What the snippet above is doing is adding a left padding of 5% to the comments. That way, when
you nest a comment inside another, the padding will move the inside comment a little to the
right, resulting in that nested look.

However, when you have 5 or more nested comments there will be no more left padding. This
is to prevent the inner comments to run out of space when you nest several levels of comments.
When you have 8 comments nested, they will move to the right up to the 5th, and then just be
aligned vertically up to the 8th.

```
|_1st comment
  |_2nd comment
    |_3rd comment
      |_4th comment
        | 5th comment
        | 6th comment
        | 7th comment
        | 8th comment
```

The comment section styling is up to your creativity. You have the information in your hands,
and can decide how to display it yourself. You could follow the traditional comment section
layout, or you can be bold and try something completely new. Maybe a carousel with the comments
and their answers below them? In terms of design you're not limited as you'd be if you were using
a third-party comment system.

## Writing the Javascript code to build the comment section

Before going to the code let's go through what it is supposed to do:

First, it should make a HTTP Request for the comment database, asking for the dynamic JSON
comment tree file (described in the [previous article]({{ site.baseurl }}{{ page.previous.url }})).
The file will be fetched and parsed to a Javascript object.

If the Javascript object is empty, it means we have no comments, so we display a message stating it.
If it's not empty, we will use the object to create all the comment HTML markup and add it to our
comment section. It should be done respecting the nesting of the comment tree.

While creating the comment HTML markup we should be very careful to sanitize the content we will
add to the page, to avoid potential script injection. We also will have to convert some special
character so they are displayed properly. New lines for example, should be transformed to
`<br />`{: .language-html} tags.

Finally, we can set a Interval to be constantly updating the comment section without requiring the user
to refresh the page.

### AJAX request

```javascript
function loadComments() {
  xhttp = new XMLHttpRequest();
  xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200){
      var obj = JSON.parse(this.responseText);
      if(obj.length == 0){
        document.getElementById("comment-section").innerHTML = "<p class=\"empty-list\">There are no comments yet. Be the first to comment!</p>";
      } else {
        document.getElementById("comment-section").innerHTML = buildComments(obj);
      }
    }
  };
  xhttp.open("GET", "https://username.github.io/comment-database/commentTree.json", true);
  xhttp.send();
}
```

Here we simply request our comment database for the comment tree JSON file
(`https://username.github.io/comment-database/commentTree.json`) and parse it into a Javascript object
with `var obj = JSON.parse(this.responseText);`{: .language-javascript}. Note that the object we have now
is an array of objects, because our JSON file starts with a bracket followed by the comment objects. So
if the length of the array is 0 (`if(obj.length == 0){`{: .language-javascript}) it means we don't have
any comments and we display a message stating it.

Otherwise we send the object to another function that is going to build the comment section with the
object information and assign the value returned to the inner HTML of the comment section `<div>`{: .language-html}.

### Building the comment section

To build the comment section, we first create a variable that will hold **all** the HTML we will have inside
the comment section `<div>`{: .language-html}. Then we will iterate through the comment tree recursively and
append the HTML markup with the information to this variable and return it. Once all the tree was checked, we
return the final HTML markup to be included in the comment section.

```javascript
function buildComments(JSONArr){
  var currentText = "";

  for(var i = 0; i < JSONArr.length; i++){
    currentText += "<div class=\"comment\">\n";
    currentText += "\t<div class=\"comment-avatar\">\n";
    if(JSONArr[i].github){
      currentText += "\t\t<img src=\"https://avatars.githubusercontent.com/"+encodeURI(formatStr(JSONArr[i].github))+"\">\n";
    } else {
      currentText += "\t\t<img src=\"https://avatars.githubusercontent.com/staticmanapp\">\n";
    }
    currentText += "\t</div>\n";
    currentText += "\t<div class=\"comment-body\">\n";
    currentText += "\t\t<p><strong>"+formatStr(JSONArr[i].name)+"</strong> <em>"+formatStr(JSONArr[i].date)+"</em></p>\n";
    currentText += "\t\t<p>"+formatStr(JSONArr[i].comment)+"</p>\n";
    currentText += "\t</div>\n";
    currentText += "\t<div class=\"comment-controls\">\n";
    currentText += "\t\t<h6><a href=\"#comment-form-element\" onclick=\"replyTo('"+formatStr(JSONArr[i].id)+"');\">Reply</a></h6>\n";
    currentText += "\t</div>\n";
    if(JSONArr[i].reply){
      currentText += buildComments(JSONArr[i].reply);
    }
      currentText += "</div>\n";
    }

    return currentText;
}
```

Some points on the code above that are worth mentioning:

```javascript
for(var i = 0; i < JSONArr.length; i++){
```
{: .small-margin}

This is the loop that is going to iterate through all the messages on the current level. This could be the
root level messages, or one of the replies depending if the function was called from the AJAX state change
scope or from inside the `buildComments();`{: .language-javascript} function recursively.
{: .small-text}

```javascript
currentText += "\t\t<img src=\"https://avatars.githubusercontent.com/"+encodeURI(formatStr(JSONArr[i].github))+"\">\n";
```
{: .small-margin}

This is used in the case the user provides a Github account when commenting. It will display the user's Github avatar instead of the default one (staticmanapp's avatar). We sanitize the string with `formatStr();`{: .language-javascript} and encode it to a URL format with
`encodeURL();`{: .language-javascript}.
{: .small-text}

```javascript
currentText += "\t\t<p><strong>"+formatStr(JSONArr[i].name)+"</strong> <em>"+formatStr(JSONArr[i].date)+"</em></p>\n";
```
{: .small-margin}

This fills the body comment with the message, after sanitizing it with `formatStr();`{: .language-javascript}.
{: .small-text}

```javascript
currentText += "\t\t<h6><a href=\"#comment-form-element\" onclick=\"replyTo('"+formatStr(JSONArr[i].id)+"');\">Reply</a></h6>\n";
```
{: .small-margin}

This line is used to set up the reply link. I won't describe the reply feature, because it's also out of the scope of that article, but essentially it fills a readonly field on the commenting form with the ID of the message the user is replying to. That field will fill the `parent` value of the comment posted.
**Note:** Right now the `formatStr();`{: .language-javscript} is being used to sanitize the ID field, but since it's being used inside
a Javascript code **it's not enough!**. The only reason I didn't update it **yet** is the fact this field is automatically filled by
Staticman. If Staticman is compromised however, Javascript code could be injected through this variable.
{: .small-text}

```javascript
if(JSONArr[i].reply){
  currentText += buildComments(JSONArr[i].reply);
}
```
{: .small-margin}

This snippet calls the same function recursively to add the replying comments markup before closing the current comment `<div>`{: .language-html}.
{: .small-text}

### Sanitizing the values

To avoid Cross-Site Scripting attacks, we must first sanitize the values we have in the comments before adding them to the
page markup.

The function we use to sanitize the values basically creates a map that will be used to change certain special characters to
their equivalent HTML escaped codes.

```javascript
var escapeMap = {
  '&': '&amp;',
  '<': '&lt;',
  '>': '&gt;',
  '"': '&quot;',
  "'": '&#39;',
  '/': '&#x2F;',
  '`': '&#x60;',
  '=': '&#x3D;'
};
```
{: .h-small-box .center .no-margin}

The map used to escape special HTML characters
{: .center .small-text}

#### [Go back to part 2]({{ site.baseurl }}{{ page.previous.url }})
