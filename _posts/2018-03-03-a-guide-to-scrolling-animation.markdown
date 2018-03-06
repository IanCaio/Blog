---
layout: blogpost
title:  "A guide to using Scrolling Animation"
author: "Ian Caio"
date:   2018-03-04 02:49:31 -0300
category: "Front-End"
header-image: "/imgs/posts/headerFrontEnd.png"
---
While working on a webpage, I had interest in using animations activated by the scrolling
to display some content in a more dynamic way. This is not an uncommon effect on websites, and there
are some third-party plugins out there that will get the job done. However, I was interested
in writing my own implementation for simple animations. In this article, I'll present the script
I wrote and instructions of how to use it!

## Where to get it?

Github Project Page - [https://github.com/IanCaio/ScrollingAnimation](https://github.com/IanCaio/ScrollingAnimation)  
Project Description Page (with demonstration) - [https://iancaio.github.io/ScrollingAnimation](https://iancaio.github.io/ScrollingAnimation)

## Why did you write it?

Since there are other third-party scripts out there that provide similar functionality, a question that comes
up is why I decided to write my own.

First reason is a bit irrational: When I can solve a problem in a reasonable
amount of time I'll give preference to writing my own code than using third-party code. Most of the time this
will make webdevelopment more time consuming for me, but I want to believe it pays off in terms of maintainability.
Which brings me to my second point:

Third-party plugins will usually try to be as complete as possible and address a big range of possibilities.
This adds complexity that can make the website harder to maintain if some functionality breaks because of some
dependency or bug. Having a lighter script that would get the job done and whose code I'd be more familiar with
sounded reasonable.

## Introduction

[Scrolling Animation](https://iancaio.github.io/ScrollingAnimation) is a pure Javascript plugin, that
allows easy implementation of scrolling based animations. Right now, there are only a couple CSS properties
that can be animated using it: _top_, _bottom_, _left_, _right_, _opacity_, _backgroundColor_ and _color_.
Other CSS properties that are only represented as a single numeric value can also be animated (like the _opacity_).
There's the possibility of some other properties being added, but until now those seemed like the ones that are
most likely used in animations.

Before showing what can be done with the script (and how it can be done), I need to explain a little bit
of how it works. Every animation will controll one or more DOM elements, which are called _controlled objects_.
Those will be the DOM elements whose style properties will change according to the scrolling position. The
properties are defined in two Javascript objects that represent the state of the _controlled objects_ before the
animation begins and after it ends: the _animation beginning state_ and _animation ending state_.

<figure class="center large">
<img src="{{ site.baseurl }}/imgs/posts/4/costates.svg" alt="Controlled Object States" />
<figcaption>Controlled Object States</figcaption>
</figure>

When setting the animation, we will also define triggering points, that will indicate where the animation starts
and where it ends. They are called _beginning triggering point_ and _ending triggering point_, whose values can
be either absolute positions in pixels or DOM elements (and optionally an offset in pixels from those elements).
Usually, elements will be used because they respond to changes in the screen resolution, making it easier to build
animations that will work correctly when the page is responsive.

When the viewport is above the beginning triggering point, the animation is in the _Inactive_ state, and the
controlled object state will be equal to the animation beginning state. When it's between the beginning triggering point
and the ending triggering point the animation is in the _Active_ state, and the controlled object state will be calculated
according to the current position and its distance to each triggering point. When the viewport hits the ending triggering point,
the animation is in the _Done_ state, where the controlled object state will be equal to the animation ending state.

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/4/animationtriggers.svg" alt="Animation Triggers" />
<figcaption>Animation Triggers and States</figcaption>
</figure>

So, to set up an animation, we just need to specify the controlled objects, the animation beginning and ending states,
the beggining and ending triggering points and any _extra configurations_ of the animation. Currently there are two
supported configurations:

_killOnEnd_: Determines if the animation will be killed once we reach the _Done_ state.
If it's not set to true, once the user scroll back up the controlled object state will change again until it reaches
the animation beginning state. Default is _false_.

_callback_: Provides a callback function that will be called by the animation everytime the animation state or ratio
changes, parsing both as arguments. Default is _null_.

## What can be done with Scrolling Animation?

The way it was designed, allows Scrolling Animation to be used either to create __position dependent__ animations or
__position triggered__ animations. The former is basically an animation which doesn't play all at once, but instead
goes playing as the user scrolls down/up. To add a position dependent animation the path is a little more intuitive:
We just define the positions where the animation starts and ends, the CSS properties for each state and the objects
being animated. As the user scrolls between the triggering points the CSS properties will gradually change.

Position triggered animations are the ones that play instantly once the user reaches a triggering point. To add one of
those we just need to enable _CSS transitions_ for the property we want to animate, define both animation states and
set the triggering points __placing them in the same position__. That way, the animation state will jump from _Inactive_
to _Done_ instantly and CSS transitions will take care of making the state change smoothly.

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/4/positiontriggered.svg" alt="Position Triggered Animations" />
<figcaption>Position Triggered Animations</figcaption>
</figure>

## How to use it

To use Scrolling Animation, we first need to source the script to the webpage:

```html
<script type="text/javascript" src="/res/scripts/scrollinganimation.js"></script>
```

Then, for every animation we want to create, we use the following constructor:

```javascript
new ScrollingAnimation(ControlledObjects, AnimationBeginningState, AnimationEndingState, BeginningTriggeringPoint, EndingTriggeringPoint, [Configuration]);
```

Controlled Objects will be either a single ID or an array of IDs representing all DOM elements that will be animated.  
Example: `"my-div"`{: .language-javascript } or `[ "my-div1", "my-div2" ]`{: .language-javascript }.

Animation Beginning State and Animation Ending State will be both _Javascript Objects_ with the same keys, each representing
a property that will be animated and its value.  
Example: `{ left: "-100px", opacity: 0 }`{: .language-javascript }.

Beginning Triggering Point and Ending Triggering Point will each represent a triggering point of the animation. They are
_Javascript Objects_ and can either have one or both of the fields `id` and `posY`. If we only have the `id` field, the
triggering point will be the element whose ID is the one provided. If we only have the `posY` field, the triggering point
will be the absolute position in pixels. If we have both, the triggering point will be the element whose ID is the one
provided with an offset equal to the `posY` field.  
Example: `{ id: "trigger" }`{: .language-javascript }, `{ posY: 180 }`{: .language-javascript } or
`{ id: "trigger", posY: -150 }`{: .language-javascript }.

Finally, Configuration, which is optional, will be a _Javascript Object_ with options for the animation.  
Example: `{ killOnEnd: true }`{: .language-javascript } or `{ callback: myFunction }`{: .language-javascript }.

## Examples

Say you want an element to gradually fade into the screen while the user is scrolling between the
_about-us_ section and the _contact-us_ section of your website. You could create an animation with the _about-us_ `<div>` as the
beginning triggering point and the _contact-us_ `<div>` as the ending triggering point. The animation
beginning state would have `opacity: 0` as a property and the animation ending state `opacity: 1`.
That would be as easy as:

```javascript
new ScrollingAnimation("AnimatedElement", { opacity: 0 }, { opacity: 1}, { id:"about-us-div-id" }, { id:"contact-us-div-id" });
```

But then you realize you want the animation to start just slightly after you reach the _about-us_ section, and to end
slightly before you reach the _contact-us_ section. You can just add an offset to the triggering points.

```javascript
new ScrollingAnimation("AnimatedElement", { opacity: 0 }, { opacity: 1}, { id:"about-us-div-id", posY: 150 }, { id:"contact-us-div-id", posY: -150 });
```

And now the animation will start 150px after the _about-us_ `<div>` and end 150px before the _contact-us_ `<div>`.

Finally, you decide that once the element is visible, you don't want it to fade out again when the user scrolls back up. Just
add the `killOnEnd` option to the animation and as soon as the animation reaches the _Done_ state it will stop.

```javascript
new ScrollingAnimation("AnimatedElement", { opacity: 0 }, { opacity: 1}, { id:"about-us-div-id", posY: 150 }, { id:"contact-us-div-id", posY: -150 }, { killOnEnd: true });
```

Voilà, that's all you need!

If you want to use absolute positions, the code would change a little bit. Say you want the animation to start on the beginning of
the page and ending after 300 pixels:

```javascript
new ScrollingAnimation("AnimatedElement", { opacity: 0 }, { opacity: 1}, { posY: 0 }, { posY: 300 }, { killOnEnd: true });
```

You can also use a DOM element as a triggering point and a position as another.

```javascript
new ScrollingAnimation("AnimatedElement", { opacity: 0 }, { opacity: 1}, { posY: 0 }, { id:"my-div" }, { killOnEnd: true });
```

## Animation beginning and ending states

### Position properties

Position properties can be expressed as either a numeric value or a string. However, some rules must be understood to avoid
unwanted behavior.

First, if we express both states (animation beginning and ending states) position properties as numeric values, they will be
interpreted as _pixel_ positions:

```javascript
new ScrollingAnimation("Obj", { left: -100 }, { left: 0 }, { posY: 0 }, { posY: 300 });
```

On the code above, the DOM element will be moved from "-100px" to "0px" during the animation.

Second, if we express one state as a string with a CSS unit and the other one as a numeric value, both will have the same unit:
The one specified in the string:

```javascript
new ScrollingAnimation("Obj", { left: "-100vw" }, { left: 30 }, { posY: 0 }, { posY: 300 });
```

On the code above, the DOM element will go from "-100vw" to "30vw", since the unit from the animation ending state is implicitly
defined by the unit in the animation beginning state.

Third, __we absolutely cannot use different units for each state__. If you do use it, the animation beginning state unit
will be used in the _Inactive_ and _Active_ states of the animation, but the unit from the animation ending state will be
used in the _Done_ state. That's almost certainly not what you desire.

```javascript
// THIS IS WRONG!!!
new ScrollingAnimation("Obj", { left: "-100vw" }, { left: "30px" }, { posY: 0 }, { posY: 300 });
```

### backgroundColor and color properties

_backgroundColor_ and _color_ properties are expressed as a Javascript object with _red_, _green_ and _blue_ values in
decimal notation (0 to 255) and the _alpha_ value represented by a floating point (0 to 1). During the animation, the backgroundColor/color
of the DOM element will gradually change from the animation beginning state to the animation ending state.

There are not any secrets about using those properties, __except that we must specify all 4 values__. That means that
`backgroundColor:{red:255,green:0,blue:0}`{: language-javascript } is __invalid__ because we are omitting the _alpha_ channel
value.

```javascript
new ScrollingAnimation("Obj", { backgroundColor: { red:255,green:0,blue:0,alpha:0.5 } }, { backgroundColor: { red:0,green:255,blue:0,alpha:1} }, { posY: 0 }, { posY: 300 });
```

```javascript
new ScrollingAnimation("Obj", { color: { red:255,green:0,blue:0,alpha:0.5 } }, { color: { red:0,green:255,blue:0,alpha:1} }, { posY: 0 }, { posY: 300 });
```

On the codes above, the elements background color (or color, respectively) will change from a sligthly transparent red to a solid green.

## Using callback functions to extend Scrolling Animation

Sometimes we might need some more complex response to the scrolling than it's possible using the animation beginning and ending states.
For those, we can use the _callback_ configuration. When it's set, the animation will call the provided callback function on every animation state
or ratio change, parsing them as arguments. We are then free to react to those changes the way we want.

We can change some CSS property that is not supported by the plugin, or maybe run through a set of images depending on the animation ratio.
The possibilities are endless.

The state value is 0 for _Inactive_, 1 for _Active_ and 2 for _Done_, while ratio is a floating point number between 0 and 1.

```javascript
var myFunction = function(state, ratio){
	const numberOfImages = 12;
	var currentImage;

	if (state === 0){
		currentImage = 0;
	} else if (state === 2){
		currentImage = numberOfImages - 1;
	} else {
		currentImage = parseInt(ratio * (numberOfImages - 1));
	}

	document.getElementById("myImage").src = "/imgs/myImage" + currentImage;
}
new ScrollingAnimation("myImage", {}, {}, { id: "start-trigger" }, { id: "end-trigger" }, { callback: myFunction });
```

## Limitations

Scrolling Animation was written to be fairly simple to use and be a general light approach to animating DOM elements according to the
window scrolling. For that reason, it's not going to be able to address any situation and be a _one-size-fits-all_ tool. It can't animate
things like gradients color stops, or activate a _@keyframe CSS animation_ for example. It's a script to be used when you need something
simple and wish to use a lighter approach than some other more complex scrolling animation scripts.

However, with the `callback` state property, you can extend the plugins functionality by programming the state changes yourself, while having
a simple API to return the current animation state and ratio. That keeps the script simplicity, while giving more
advanced users the tools to make more complex animations.

Hope you enjoy it!
