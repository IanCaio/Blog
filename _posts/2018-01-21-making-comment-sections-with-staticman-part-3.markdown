---
layout: blogpost
title:  "Making comment sections with Staticman - Part 3"
author: "Ian Caio"
date:   2018-01-21 21:44:12 -0200
header-image: "/imgs/posts/2/staticman.png"
comment-section: "https://iancaio.github.io/blog-comments/get/post4comments.json"
comment-post-link: "https://api.staticman.net/v2/entry/IanCaio/blog-comments/master/post4comments"
category: "Front-End"
---
On the final part of this articles series I discuss the client-side part
of the comment system. Making the layout and using Javascript to safely display
the comments on the page.

[Click here]({{ site.baseurl }}{{ page.previous.previous.url }})
if you want to start reading from the beginning!

*DISCLAIMER: This article was made for educational purposes only. The comment system was just
recently deployed and is still under testing. It seems functional and bug free, but I'm not
responsable for the misuse of the code or any consequences that might arise from using it
on production websites.*
{: .small-text .extra-margin }

## Working on the CSS template for the comment section

I'm not going to describe in details how I designed the CSS template for the comment section,
since it's outside the scope of these articles. But I wanted to show how I styled the
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
you add a comment inside another, the padding will move the inside comment a little to the
right, resulting in that nested look.

However, when you have 5 or more nested comments there will be no more left padding. This
is to prevent the inner comments to run out of space when you nest several levels of comments.
For example, when you have 8 comments nested, they will move to the right up to the 5th comment,
and then just be aligned vertically from there up to the 8th.

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
{: .h-small-box .center .no-margin}

Limiting the nesting to 5 levels
{: .small-text .center}

The comment section styling is up to your creativity. You have the information in your hands
and get to decide how to display it yourself. You could follow the traditional comment section
layout, or you could be bold and try something completely new. Maybe a carousel with the comments
and their answers below them? In terms of design you're not limited as you'd be if you were using
a third-party comment system.

## Writing the Javascript code to build the comment section

Before going through the code itself let's review what it is supposed to do:

First, it should make a HTTP Request for the comment database, asking for the dynamic JSON
comment tree file (described in the [previous article]({{ site.baseurl }}{{ page.previous.url }}#writing-the-dynamic-comment-tree)).
The file will be fetched and parsed to become a Javascript object.

If the Javascript object is empty, it means we have no comments, so we display a message stating it.
If it's not empty, we will use the object to create all the comment HTML markup and add it to our
comment section. It should be done respecting the nesting of the comment tree.

While creating the HTML markup we should be very careful to sanitize the content we will
add to the page, to avoid potential script injections. We also will have to convert some special
character so they are displayed properly. New lines for example, should be replaced with
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
with `var obj = JSON.parse(this.responseText);`{: .language-javascript}.

Note that the object we have now is an array of objects. That's because our JSON file
[starts with a square bracket](https://stackoverflow.com/a/5034547/6676042), making it an array containing our
comment objects. So if the length of the array is 0 (`if(obj.length == 0){`{: .language-javascript}) it means we don't have
any comments and we display a message stating it.

Otherwise we send the object to `buildComments();`{: .language-javascript}, which is going to build the comment section with the
object information, and assign the returned value to the inner HTML of the comment section `<div>`{: .language-html}.

If we want the comment section to update itself automatically (without requiring the user to reload the page) we
can set a Interval that will call this function every number of seconds:

```javascript
setInterval(loadComments, 5000);
```

### Building the comment section

To build the comment section, we first create a variable that will hold all the HTML markup we will have inside
the comment section `<div>`{: .language-html}. Then we will iterate through the comment tree recursively and
append the HTML markup to this variable and return it. Once all the tree was checked, we
return the final HTML markup to be included in the comment section.

```javascript
function buildComments(JSONArr){
  var currentText = "";

  for(var i = 0; i < JSONArr.length; i++){
    currentText += "<div class=\"comment\">\n";
    currentText += "\t<div class=\"comment-avatar\">\n";
    if(JSONArr[i].github){
      currentText += "\t\t<img src=\"https://avatars.githubusercontent.com/"+encodeURI(formatHTMLStr(JSONArr[i].github))+"\">\n";
    } else {
      currentText += "\t\t<img src=\"https://avatars.githubusercontent.com/staticmanapp\">\n";
    }
    currentText += "\t</div>\n";
    currentText += "\t<div class=\"comment-body\">\n";
    currentText += "\t\t<p><strong>"+formatHTMLStr(JSONArr[i].name)+"</strong> <em>"+formatHTMLStr(JSONArr[i].date)+"</em></p>\n";
    currentText += "\t\t<p>"+formatHTMLStr(JSONArr[i].comment)+"</p>\n";
    currentText += "\t</div>\n";
    currentText += "\t<div class=\"comment-controls\">\n";
    currentText += "\t\t<h6><a href=\"#comment-form-element\" onclick=\"replyTo('"+formatIDStr(JSONArr[i].id)+"');\">Reply</a></h6>\n";
    currentText += "\t</div>\n";
    if(JSONArr[i].reply){
      currentText += buildComments(JSONArr[i].reply);
    }
      currentText += "</div>\n";
    }

    return currentText;
}
```

### Some code lines explained
{: .center}

```javascript
for(var i = 0; i < JSONArr.length; i++){
```
{: .small-margin}

This is the loop that is going to iterate through all the messages on the current level. This could be the
root level messages or one of the replies level, depending whether the function was called from the AJAX callback
or from inside the `buildComments();`{: .language-javascript} function recursively.

```javascript
currentText += "\t\t<img src=\"https://avatars.githubusercontent.com/"+encodeURI(formatHTMLStr(JSONArr[i].github))+"\">\n";
```
{: .small-margin}

This is used in the case the user provides a Github account when commenting. It will display the user's Github
avatar instead of the default one (currently staticmanapp's avatar). We sanitize the string with
`formatHTMLStr();`{: .language-javascript} and encode it to a URL format with `encodeURL();`{: .language-javascript}.

```javascript
currentText += "\t\t<p><strong>"+formatHTMLStr(JSONArr[i].name)+"</strong> <em>"+formatHTMLStr(JSONArr[i].date)+"</em></p>\n";
currentText += "\t\t<p>"+formatHTMLStr(JSONArr[i].comment)+"</p>\n";
```
{: .small-margin}

This fills the comment with its content, after sanitizing the values it with `formatHTMLStr();`{: .language-javascript}.

```javascript
currentText += "\t\t<h6><a href=\"#comment-form-element\" onclick=\"replyTo('"+formatIDStr(JSONArr[i].id)+"');\">Reply</a></h6>\n";
```
{: .small-margin #replyTo-html-markup}

This line is used to set up the [reply link](#replying-comments). It creates an event listener on the link that will
fill the `<input>`{: .language-html} of the `parent` field with the comment ID of the comment we want to reply to.

```javascript
if(JSONArr[i].reply){
  currentText += buildComments(JSONArr[i].reply);
}
```
{: .small-margin}

This snippet calls the same function recursively to add the nested replying comments
markup before closing the current comment `<div>`{: .language-html}.

### Sanitizing the values

To avoid [Cross-Site Scripting](https://en.wikipedia.org/wiki/Cross-site_scripting)
attacks, we must first sanitize the values we have in the comments before adding them to the
page markup. Be careful to dedicate some time to this task and always review it: It's easy to forget the context where
the values are being added and unwillingly allow some bad characters to bypass the filtering.

For example, when we add content to the inner HTML of some element, we will escape a list of HTML special characters,
but will probably allow parenthesis, since they are valid in a comment. But if we are adding a variable inside a Javascript function
arguments list, parenthesis could be a concern. Always think about what characters **can't** be included where you're appending the
value and clear anything that shouldn't be there. Or, when it's an option, take the opposite approach and be very
conservative of what characters you're allowing.

Right now I'm using 2 functions to sanitize the values: `formatHTMLStr();`{: .language-javascript}
and `formatIDStr();`{: .language-javascript}. The former escapes a list
of special HTML characters and converts newlines to `<br />`. The latter takes the conservative approach and retrieve
**only** the alphanumeric characters or hifens of the ID, starting from the beginning of the string.  
It does that using a Javascript regex: `sanitizedString = rawString.match(/^([0-9]|[a-zA-Z]|-)*/g);`{: .language-javascript}

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

Be sure to give this issue some consideration, since a bad sanitized comment section compromises your website
and is an open invitation to XSS attacks.

## Problem with optional fields

If you have optional fields on your comment system (in my case the `parent` and `github` fields) and include a
`<input>`{: .language-html} element for them inside your comment submission `<form>`{: .language-html}, even if
they are empty the field will be sent to Staticman with an empty value, which will create an object that looks
something like this:

```json
{ "name":"Mike", "parent":"", "comment":"Hello!" }
```

Now check the following line we have on the dynamic comment tree Jekyll code from the
[previous article]({{ site.baseurl }}{{ page.previous.url }}#writing-the-dynamic-comment-tree):

{% raw %}
```liquid
{%- for level1_file in folder_files1 -%}
{%- assign level1 = level1_file[1] -%}
{%- unless level1.parent -%}
...
{%- endunless -%}
{%- endfor -%}
```
{% endraw %}

Because of the way the dynamic comment tree is currently written, it's required that root comments **don't have any parent
fields**, even if they are empty. That means that comments with `"parent":""`{: .language-json} would not show in the comment
tree. We can either work on the Jekyll code to allow empty `parent` fields or work on the client-side ensuring empty fields are not sent:

```javascript
//Avoid empty github or parent input to be sent
var commentForm = document.getElementById("comment-form-element");
  commentForm.addEventListener("submit", function(){
    var inputField = commentForm.getElementsByTagName("input");

    for(var i = 0; i < inputField.length; i++){
      //If the github field has no value but has a name (active) deactivate it (name="")
      if ((inputField[i].name == "fields[github]") && !inputField[i].value){
        inputField[i].name = "";
      }
      //If the parent field has no value but has a name (active) deactivate it (name="")
      if ((inputField[i].name == "fields[parent]") && !inputField[i].value){
        inputField[i].name = "";
      }
    }
});
```

## Replying comments
{: #replying-comments}

To be able to reply to a comment, we need to send an extra optional field to Staticman, called `parent`, with
the ID of the comment we are replying to. As it was seem [here](#replyTo-html-markup), when the user clicks
the *reply* link, we are calling a function called `replyTo();`{: .language-javascript} with the comment ID as an argument.

```javascript
//Fill reply field (parent field) => replyTo("ID"); to reply to ID and replyTo("") to cancel reply
function replyTo(id){
  var inputField = commentForm.getElementsByTagName("input");

  for(var i = 0; i < inputField.length; i++){
    if(inputField[i].name == "fields[parent]"){
      inputField[i].value = id;
    }
  }

  //Show cancel button if we are adding a reply, remove it if we are removing a reply
  if(id) {
    document.getElementById("cancel-reply").style.display = "block";
  } else {
    document.getElementById("cancel-reply").style.display = "none";
  }
}
```

The function simply adds the ID value to a readonly `<input>`{: .language-html} that is going
to send the `parent` value when the `<form>`{: .language-html} is submitted. It also displays a link to cancel the reply.
The cancel link calls this function with an empty argument, which by it turn recognizes the ID is empty, resets the
`<input>`{: .language-html} value and hides the cancel link.
{: .extra-margin}

I hope you enjoyed the articles and that they were helpful for you to start developing your own comment system. I tried to
discuss the most important points of the process, but if there are any doubts or if you think I should include some topic on the
articles please let me know! Any constructive criticism is also very welcome!

#### [Go back to part 2]({{ site.baseurl }}{{ page.previous.url }})
