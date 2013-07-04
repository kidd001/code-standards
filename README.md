Howdy, Developer. You're here because you're building a Springload web project. To make it easy for everyone to understand your code and maintain it later, you'll need to stick to the house rules.

This is a living document, so check back often.

# QuickStart
If you're not the RTFM type, stick to these basic principles:
* Comment everything (that isn't obvious).
* Be consistent.
* Classes with underscores.
* Don't use JS to replicate HTML/CSS functionality unless a puppy's life depends on it.
* Don't do crazy SaSS nesting (3 levels at most)
* Limit nested blocks to 50 lines
* No styles on IDs
* All JS hooks should be prefixed with js. E.g, `.js_show_nav`
* Always, always always check your compiled CSS output. This is what matters.
* Indent using 4 spaces. No tabs.
* Don't fight the browser.
* Put the seat down.
* Do the dishes.


# Table of contents
* [CSS & SaSS](#css)
* [JavaScript](#js)
* [File naming conventions](#files)

<a id='css'></a>
# CSS & SaSS

### Headings and typography

Try and create the type for your site using only the top-level declarations: H1-6 and p. You'll find you won't spend time fighting the vertical rhythm if your typographic palette has fewer moving parts.

Consistent typography is easiest if you keep the number of point-sizes down. Talk to a designer about this.

In general, don't use more than 10 font-sizes in a project:
* H1-H6
* P
* small
* .intro
* .legal

Try and only define your `h1-h6` styles once in the stylesheet. Don't rely on source order (like, when `h2` is inside `.some_selector`). This is brittle, it means your styles are tightly coupled to particular markup.

```
h2 {
    font-size: 1.5em;
}
.some_selector h2 {
    font-size: 2em;
}
.other_selector h2 {
    font-size: 1.4em;
}
```

Instead, apply a class that modifies the size:

```
h2 {
    font-size: 1.5em;
}
.widget_title {
    font-size: 2em;
}
.footer_title {
    font-size: 1.4em;
}
```


### Apply margins to the top
Use `margin-top` to set vertical margins. This makes margins easy and predictable, and lets you use the adjacent selector from CSS2.1 `h2 + .intro` to adjust the spacing.

This also means you can use `:first-child` to take the margin-top off the first element if needed, and has slightly wider browser support than `:last-child`.

```
h2 {
	margin-top: $base-spacing-unit * 2;
}
/* Tighten up spacing between h2 and intro paragraph */
h2 + .intro {
        margin-top: $base-spacing-unit / 2;
}

h2:first-child {
	margin-top: 0;
}
```

### Don't use units on line-heights
Don't do this:
```
h2{
    line-height: 1.25em;
}
```
Do this:
```
h2{
    line-height: 1.25;
}
```

### Be consistent with vertical spacing.

Avoid setting arbitrary margins on items. Defer these to a variable, and use multiples of that variable to control your vertical spacing. Our designs will often change right up til the end, so it's worth making stuff really easy to change.

Here's a great [article](http://webtypography.net/Rhythm_and_Proportion/Vertical_Motion/2.2.2/) about vertical rhythm.

Don't do this:
```
.block {
	margin-top: 1.25em;
}

.footer {
	margin-top: 1.25em;
}

.block--feature {
	margin-top: 2.75em;
}
```

Do this:

```
/**
 * $base-spacing-unit: 1.25em;
 * Declare common properties once only, or use an @extend in SaSS
 */
.m, .block, .footer {
	margin-top: 1.25em;
}

.block--feature {
        /* $base-spacing-unit times two. */
	margin-top: 2.5em;
}
```

### Leave `html, body` font-size at 100%
Resist the urge to tweak the body font-size. Filament group have a [good article](http://filamentgroup.com/lab/how_we_learned_to_leave_body_font_size_alone/) about this.

Even for responsive sites, it's nice knowing that `1em` is always 16px. Computers calculate things in base2 rather than base10. This is a fundamentally awesome feature of the browser and we shouldn't break it.

**Always set font-sizes, margins, padding, and media queries in `ems`**


## Selectors
* Classnames with underscores by default `.my_awesome_class`.
* Modifiers with double-dashes, in the BEM style: `.my_awesome_class--fancy`
* Sub-components with double-underscores: `.my_awesome_class__sidebar`

### Don't use IDs in your CSS. Ever.
They're too specific. It takes 255 classes to cascade over styling set on an ID. It doesn't matter how 'unique' that element is. If you see IDs in a project and you can refactor them safely, please do.

### Don't over-qualify selectors
Over-qualified selectors typically look like `ul.list`. But what if you want to use `.list` on a `div`? You're much better off leaving base elements off your classnames. Use comments to help other developers know which sorts of elements the selector is intended for.

Don't do this:
```
ul.list {
	list-style: none;
}

body#home_loans figure.banner {
	background: #333;
}
```

Do this:
```
.list {/*ul, ol, block-level elements*/
	list-style: none;
}

.banner--home {/* figure */
	background: #333;
}
```

### Don't use HTML5 elements when a `div` will do.
HTML5 has some nice new tricks, like `canvas`, `video` and `input[type='tel']`. Use them all you want (via [Feature Detection](#feature-detection)). But don't use HTML5 layout elements `figure, article, section, etc..`. A div is three characters long and it renders in every browser without a JS hack.


### Nesting
Be careful not to have unnecessary __nesting__. It may make the SASS look tidier, but it makes the CSS files larger, the parsing of the CSS will slow and the styles will be harder to override when they are too specific. But most importantly, it will mess with __the cascade__, and styles will be very hard to override.

Don’t do this:

```SCSS
.something_detail {
    ul {
        list-style: none;
        li {
            padding-left: 1em;
            a {
                display: block;
                span {
                    font-weight: bold;
                }
            }
        }
    }
}

/* Outputs */
.something_detail ul { list-style: none; }
.something_detail ul li { padding-left: 1em; }
.something_detail ul li a { display: block; }
.something_detail ul li a span { font-weight: bold; }
```
Instead, do this:

```CSS
.something_detail {
    ul {
        list-style: none;
    }
    li {
        padding-left: 1em;
    }
    a {
        display: block;
        span {
            font-weight: bold;
        }
    }
}

/* Outputs */
.something_detail ul { list-style: none; }
.something_detail li { padding-left: 1em; }
.something_detail a { display: block; }
.something_detail a span { font-weight: bold; }
```

Even better, use BEM to keep the specificity down:

```CSS
.something_detail__list {
    list-style: none;
}
.something_detail__list__item {
    padding-left: 1em;
}
.something_detail__link {
    display: block;
}
.something_detail__link--bold {
    font-weight: bold;
}
```

### Indent sub-components in CSS
Rather than doing crazy SaSS nesting, indent CSS rules for child elements:

```
.widget{
    margin-top: 1em;
    border: solid 1px #eee;
}
    
    /**
     * The widget title is a heading with an icon
     */
    .widget__title{
        font-size: 2em;
    }

        .widget__title i{
            display: inline-block;
        }

    /**
     * The body of the widget gets some padding, and we tweak the spacing of
     * paragraphs so they're a little tighter
     */
    .widget__body{
        padding: 0.5em;
    }

        .widget__body p{
            margin-top: 0.5em;
        }

```


### Don't add styles too early

In general, think about your classes like functions on the linux command line. They all do one thing, and do it well. If you're too specific to start with, you'll find you have to undo the styling later on, and you'll get stuck in the specificity arms-race.

Only abstract out the bits that are absolutely unique:

Don't do this:

```
.active {
    color: red;
    margin-top: 1em; /* hmm...does every active thing really need a margin-top? */
}
.nav--item {
    color: blue;
}
.nav--item.active {
    margin-top: 0;  /* resetting margin because `.active` was too specific */
}
```

Do this:

```
.active {
    color: red !important; /* the class only does one thing, but does it well.*/
}
.nav--item {
    color: blue;
}
.nav--item.active {
    /* this gets the default active state, so we don't even need this rule now :) */
}
```

Think about the most essential function of the class(es), and your CSS will scale much better. You'll write less of it.





<a id='comments'></a>
## Comments
We work in SaSS. This means you get to **comment everything!**. You're not always going to be around to explain how you've done something. All the cleverness is really obvious to you right now, but you might not be the one doing maintenance in 6 months time.

Write comments in the DocBlockr style, and limit lines to 80 chars:

```
/**
 * Oh hai there, I'm a comment! We write comments in the jsDoc|DocBlockr style,
 * so that our CSS and JS comments all look the same. You can read about jsDoc
 * over at the repo: https://github.com/spadgos/sublime-jsdocs
 */

/**
 * Oh hai there, I'm a comment with a title!
 * ----------------------------------------------------------------------------
 *
 * Wrap lines that are longer than 80 chars. A comment might need a title block
 * which should be underscored by 76 dashes so it comes to 80 chars total.
 *
 * In General, butt the comment block up against the CSS item that follows it,
 * like so:
 *
 */
.some-selector {
    /* I'm a line comment in a CSS selector. */
}
```

Make use of CSS comments rather than SaSS `// line-comment` style in your general styles. This makes it easy for us to comment consistently across projects that are in SaSS or vanilla CSS, and means the non-minified CSS output still has all the context explaining what you've done.

Use the `// line-comment` style in SaSS mixins, since they shouldn't really make it to the CSS output:

```
// This mixin converts pixels to ems, assuming a base em-size of 16px:
@function pxem($px-val, $base: 16) {
    @return #{ ($px-val / $base) }em;
}
```

Rather than qualifying selectors, suggest/imply the element type with a comment:

```
.list { /*ul, ol*/
    list-style:none;
}

.link {/*a, button, input*/
    text-decoration: none;
}
```

Let developers know when classes should extend other classes,
```
/**
 * Throw a list into a row of inline-block elements.
 * @extends .list in _lists.scss
 */
.nav { /*ul, ol, div*/
    overflow: hidden;
}
    .nav > li {
        display: inline-block;
    }
```

## Documenting the CSS project

Write a helpful comment block at the top of your main stylesheet. This should include:
* The project name
* Who's worked on it
* Which browsers are supported
* Notes (this can include links to our wiki documentation)
* The structure of the SaSS/CSS project
* A link to the House Rules document.
* A friendly salutation to your fellow devs (bonus points for beautiful prose).

```
/**
 * AwesomeTown™ Stylesheet
 * ----------------------------------------------------------------------------
 * Authors:
 *
 * 		Joe Bloggs 		| May 2013
 * 		Blog Joes 	        | June 2013
 *
 * ----------------------------------------------------------------------------
 * Supported platforms:
 *
 * 		IE8+
 * 		Android 2.3+
 * 		Mobile Safari (iOS 4+)
 * 		Chrome, Firefox, Safari
 *
 * ----------------------------------------------------------------------------
 * Notes:
 *
 * 		Build this project using `[project-root]/tools/css.sh`.
 *
 * 		This wraps config.rb, allows us to keep the SaSS config out of the
 * 		web-root, and extends our config.sh which maintains all paths for
 * 		our asset toolchain (sprites/uglifyJS etc)
 *
 *              Project documentation at http://wiki.springload.co.nz/project-name
 *
 * Structure:
 *
 * 		1. Variables
 * 		2. Base HTML/Reset
 * 		3. Libraries
 * 		4. Objects
 * 		5. UI Elements
 * 		6. Page specific styles
 * 		7. Shame.css
 *
 *
 * House Rules:
 *
 * 		https://github.com/springload/css-design-patterns/wiki/House-Rules
 *
 * ----------------------------------------------------------------------------
 *
 * Good luck and happy hunting!
 *
 */
```


## SaSS

* Keep an eye on your compiled CSS.
* Make it beautiful before you minify it.
* Watch out for duplicate rules. (Do a `grep` or search for common offenders, like font-size);

All __mixins__, __functions__, __variables__ and __placeholder selectors__, should use hyphen (-) ’s, in keeping with SASS and Compass mixins and functions.


## Whizzy new browser features!
New things are in the Chrome Canary and FireFox Nightly builds all the time. There's some really cool stuff out there, and you shoud be using it on your project if you can!

Be aware that rendering, performance, and support will all need to be assessed before you roll out a feature on your site, like flexbox for instance. Create a [JSPerf](http://jsperf.com/) test and run it every browser you're supporting (including all the phones). Check what's going on in [WebKit's Timeline](http://perfectionkills.com/profiling-css-for-fun-and-profit-optimization-notes/). Then, write your progressive enhancement to include it (via Modernizr or Browser.js) accordingly.



<a id='js'></a>
# JavaScript
* Read the [Writing Idiomatic JavaScript](https://github.com/rwldrn/idiomatic.js/) article.
* Always concatenate and minify your scripts in production. From the command line, you can use `uglifyjs`, or let Yak handle it (default).

## Watch your variable scope
If you don't use the `var` keyword, your variable is in the global scope! This breaks things, and isn't advisable.

```
myFunction = function() {
	me = this;
}

/* is equivalent to: */
myFunction = function() {
	window.me = this;
}

/* make sure `me` only exists in the scope of myFunction */
myFunction = function() {
	var me = this;
}
```

You can throw JavaScript into `strict` mode to generate errors when you don't scope your variables properly:
```
(function() {
    "use strict";
    var myFunction = function() {
         /* This will generate an error in strict mode*/
         self = this;
    }
    return myFunction;
})();
```




### Write stuff in closures
Always write your modules in a [Closure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures) for good encapsulation. Scope your variables. Provide a close method to unbind/remove your event listeners.

```
(function(window, document) {
    var myModule = {
        myFunction: function() {
            var self = this;
            window.addEventListener("load", this.myEventListener, false);
        },
        myEventListener: function(e) {
            var self = this;
            // do something awesome. Single line comments use the // comment style.
        },

        /**
         * Closes the module and unbinds event listeners.
         * Block comments/function comments in DocBlockr style.
         * @returns {void}
         */
        close: function() {
            var self = this;
            window.removeEventListener("load", self.myEventListener, false);
        }
    };

    return myModule;
})(window, document);
```
## Boot up your app in one place

Don't do this:
```
/* in module.js */
$(document).ready(function() {
	myModule.init();
});

/* in otherModule.js */
$(document).ready(function() {
	myModule.init();
});
```

Do this:

```
/* init all the modules */
$(document).ready(function() {
	myModule.init();
	myOtherModule.init();
});
```


## Cache your jQuery selectors

Don't do this:
```
$("body").on("click", function() {
	$("body").addClass("clicked");
});
```
Do this:
```
var $body = $("body");

$body.on("click", function() {
	$body.addClass("clicked");
});
```


## Use a `self` variable to manage scope

Try and avoid accessing top-level/global objects inside your methods. Instead, refer to the module by `var self = this;`

```
var SITE = {
	isResponsive: true
};

/* Don't do this */
SITE.myModule = function() {
	if(SITE.isResponsive) {
		// do something
	}
}

/* Do this */
SITE.myModule = function() {
        var self = this;
	if(self.isResponsive) {
		// do something
	}
}
```

## Make `this` point to the right thing

It can be really hard to maintain the scope of `this` in JS functions. Luckily, there's a magical method for applying the scope of `this` in a function: `call()`.

Here's how it works:
```
/* Alert a message after a given timeout */
SITE = {
	message: "done, woohoo!",
	timeout: 500
};
SITE.done = function() {
	var self = this;
	alert(self.message);
}

/* Don't do this */
setTimeout(function() {
	SITE.done();
	/* `this` will refer to the global window object */
}, SITE.timeout);

/* Do this */
setTimeout(function() {
	SITE.done.call(SITE);
	/* `this` will refer to SITE */
}, SITE.timeout);
```



## Feature detection

Feature detect any fancy things! It might seem like a pain in the ass when you're writing the code, but you'll save lots of testing time, and your project will enhance/degrade gracefully to the level of the user's browser. You can also be more confident that keeping up with the latest browsers won't break old browsers.

Use custom Modernizr builds or our own Browser.js. This is a good habit:

```
if (Browser.is("modern")) {
    // Do fancy things for modern browsers.
} else {
    // Do less fancy things.
}
```

## Unit Testing

Test your functions. James and Tama should write something here!!






<a id='files'></a>
# File naming conventions
File names should be all lowercase and using hyphen (-) for spaces. For versioning and minified, use period.

__Images:__ Try and prefix file names with what they are: I.e. if it’s a sprite for buttons, call it `sprite-buttons.png`. If it’s an icon for Facebook, call it `icon-facebook.png` etc.

__SASS:__ The same goes for SASS: If a partial only contains a mixin for a slider, call it `_mixin-slider.scss`.

    JS:
    jquery-1.8.1.js
    jquery-1.8.1.min.js
    jq-collection-slider.js
    jq-collection-slider.min.js

    Images:
    icon-facebook.png
    sprite-buttons.png
    sprite-buttons-retina.png

    SASS:
    styles.scss
    _partial-name.scss
    _mixin-slider.scss
    _function-px-to-em.scss
