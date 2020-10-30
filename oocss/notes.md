## OOCSS

Made by Nicole Sullivan. It was styling Facebook for a while.

## Problems with CSS

**Don't need to be an expert to get started.**
No-one knows what they're doing, it's possible to make something bad that works and we stick with it.

**File sizes keep getting bigger.**
This has performance implications and bandwidth issues.

**Code re-use is almost non existent.**
This means that we don't re-use things when we should, we just keep adding to the file size - same issues as before. Even though CSS classes are designed to be reusable - even on the same page.

**Code is too fragile.**
Changing one thing can have huge consequences. Unknown consequences.


Front-end devs makes really amazing modules that have no sense of the wider website. Whenever they add new functionality, they add more styles.

> [CSS] doesn't suck, you're just going it wrong.
_Douglas Crockford (and Nicole Sullivan)_

Designers don't think about how designs will scale, that's our job as developers.

There are lots of ~right~ wrong ways to code CSS.
Front-end architecture needs to be right.

Let's sort it out.

Metrics:
- HTTP requests
- Size of images
- Size of the CSS file.

Goal: create a component library so we can mix-and-match to make new modules.

### Separate Content and Container

Apply multiple classes.
Media object `^_^`

```html
<div class="media media--extended">
    <img src="myImage.jpg">
    <div class="media__body">
        <p>Lorem ipsum dolor ...</p>
    </div>
</div>
```

### Separate Skin from Structure

Nicole's common module pattern:

```html
<div class="module">
    <div class="inner">
        <div class="header"></div>
        <div class="body"></div>
        <div class="footer"></div>
    </div>
</div>
```

The skin makes it look pretty. The idea of the skin is to make it predictable.

Back in the days when CSS rounded corners were new and experimental and IE8 didn't support them, Nicole was recommending that we write rounded corners like this:

```css
.simple .inner {
    border: 1px solid gray;
    -webkit-border-radius: 7px;
    -moz-border-radius: 7px;
    border-radius: 7px;
}

.simple .body {
    *background-image: url("skin/mod/corners.png");
}
```

It's designed to be reused, it's designed to help consistency.

**Avoid location dependent styles**

A large thin heading shouldn't become a small bold heading in another part of the page.

A module should always look the same no matter where you place it on the page - whether it's in the main navigation, the sidebar, the body or the footer, it should look the same.
Otherwise you end up writing more rules to overwrite the crazy rules from before. (There was an old lady who swallowed a fly.)

For example, here's location dependent styles:

```css
#weatherModule h2 { /* ... */ }
#weatherModule h3 { /* ... */ }
#weatherModule p { /* ... */ }
#tabs h2 { /* ... */ }
#tabs h3 { /* ... */ }
#tabs p { /* ... */ }
```

+1 for modularity, but still broken. If you want the `h3` from the `#weatherModule` to appear in the `#tabs` you just can't do it. It's not possible with this code, we have to duplicate the code (wasteful) or combine selectors together (confusing).

Each module/element should be in control of its own destiny. Imagine a basic layout.

```
+-------------------------------------+
| Header                              |
+-------------------------------------+
+-------------------------------------+
| +----------+ +------+ +-----------+ |
| | Left col | | Body | | Right col | |
| +----------+ +------+ +-----------+ |
+-------------------------------------+
+-------------------------------------+
| Footer                              |
+-------------------------------------+
```

To make the left column thinner, we should only need to add a class to the left column

```
+-------------------------------------+
| Header                              |
+-------------------------------------+
+-------------------------------------+
| +------+ +----------+ +-----------+ |
| | Left | | Body     | | Right col | |
| +------+ +----------+ +-----------+ |
+-------------------------------------+
+-------------------------------------+
| Footer                              |
+-------------------------------------+
```

To remove the right column, we should only need to remove the right column element.

```
+-------------------------------------+
| Header                              |
+-------------------------------------+
+-------------------------------------+
| +------+ +------------------------+ |
| | Left | | Body                   | |
| +------+ +------------------------+ |
+-------------------------------------+
+-------------------------------------+
| Footer                              |
+-------------------------------------+
```

It shouldn't be any more complicated than that. This means we don't have to loop through the DOM to work out what someone's changed and alter different parts.

---

We learned DRY when we started programming. We didn't transfer that mentality to our CSS work.

When making a module, we tend to created nested styles.

```css
#weather h3 { font-size: 1.2em; }
```

Later on we might have another module.

```css
#weather h3 { font-size: 1.2em; }

#comments h3 { font-size: 1.2em; }
```

We now have duplicated code. We could combine those selectors, but that would add a new level of confusion.

```css
#weather h3,
#comments h3 { font-size: 1.2em; }
```

Are you expecting these styles to be in:
- weather.scss?
- comments.scss?
- headings.scss?
- Somewhere else?

Sadly, we've copied this mentality across to BEM.

```css
.weather__header { font-size: 1.2em; }

.comments__header { font-size: 1.2em; }
```

This _works_ ... but it only works for those modules. It doesn't work for other ones unless we duplicate the code. That leads to larger files, reduced performance and code that's harder to maintain.

A better idea would be to just have a generic `h3` class.

```css
h1, .h1 { font-size: 1.6em; }
h2, .h2 { font-size: 1.4em; }
h3, .h3 { font-size: 1.2em; }
```

This can change our markup.

```html
<div class="weather">
    <h3 class="weather__header">Hello world</h3>
</div>

<div class="comments">
    <h3 class="comments__header">Hello world</h3>
</div>
```

Instead, we can just have the `h3` without a class. We could even give it a different class if semantically it needs to be an `h3` but look like it's an `h2`:

```html
<div class="weather">
    <h3>Hello world</h3>
</div>

<div class="comments">
    <h3 class="h2">Hello world</h3>
</div>
```

You can also give an element multiple class names, so `h2 primary` could give it a font-size and a colour.

With a different idea, let's suppose we have an alert on our page.

```html
<div class="notice">Something happened</div>
```

It could have some simple styles.

```css
.notice {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
    background: url(icon.png) no-repeat 0 50%;
}
```

If you later decided that you needed an error notification that basically looks the same but has some differences, it could look like this.

```html
<div class="notice">Something happened</div>
<div class="error">Something bad happened</div>
```

```css
.notice {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
    background: url(icon.png) no-repeat 0 50%;
}

.error {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
    background: url(icon-error.png) no-repeat 0 50%;
}
```

Maybe we didn't write the `notice` styles to begin with? Maybe we're not sure what would be affected if we changed anything. It makes sense not to break existing styles, but we now have duplicated code. If we later need to add a "danger" variant, we have the same problem.

```css
.notice {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
    background: url(icon.png) no-repeat 0 50%;
}

.error {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
    background: url(icon-error.png) no-repeat 0 50%;
}

.danger {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
    background: url(icon-danger.png) no-repeat 0 50%;
}
```

We've totally forgotten about DRY.

How about somwthing like this instead?

```html
<div class="message notice">Something happened</div>
<div class="message error">Something bad happened</div>
<div class="message danger">Are you sure you want to do this?</div>
```

We now have very different CSS.

```css
.message {
    border: 1px dotted #666;
    padding: 1em;
    color: #292929;
}

.notice,
.error,
.danger {
    background-repeat: no-repeat;
    background-position: 0 50%;
}

.notice {
    background-image: url(icon.png);
}

.error {
    background-image: url(icon-error.png);
}

.danger {
    background-image: url(icon-danger.png);
}
```

Now if we want to change something like the padding, we only need to change it in one place.

- **Repair fragile code** code that is very attached to IDs or a module. Code that can only be used in one place.
- **Bring the OOP methodology into CSS** think in terms of objects so we can DRY up our code.
- **Create reusable components** to think in terms of objects, we make reusable components. We can use this in multiple locations.
- **Be weary of ids. Embrace classes** that way we can reuse the module multiple times on the same page. You can still use IDs as a JavaScript hook.
- **Think in terms of Lego bricks. Mix components to build your sites.** If you have lots of components, you can mix and match them to build your pages.

---

We want to reduce CSS. CSS is a blocking resource - everything stops while we load in CSS. Older browsers are really bad with this, they won't change page until the style sheets have downloaded.
We can't minify CSS like we can with JavaScript.

```js
var myLongVariableName = 1 + 2 + 3 + 4;
// minifies to:
var a=10;
```

```css
.my-long-class-name {
    text-decoration: underline;
}
/* Minifies to: */
.my-long-class-name{text-decoration:underline}
```

This means that we have to stop duplicating the CSS ourselves. It needs to be fast by default. CSS is something we just forget about, and it gets bigger and bigger.

Xigen site: 21KB
Jacksons M1: 73.5KB
Epson: 112KB + 33KB (145KB)

And all of those style sheets will only get bigger.

Kudzu was planted in the US to prevent soil erosion, but it exploded and is everywhere. No matter what we try, it just keeps growing. CSS is the same :(

A lot of companies say things like
> We like the idea of OOCSS but it's too late for us, we can't implement it now.

True, it's easier to do this from the start, but it's possible. Facebook went on a diet and tried to increase their performance. Implementing OOCSS was one of the things they did. In 6 months, they'd managed to reduce their response time in half.

OOCSS is an evolution of the language.

```css
        ul { /* ... */ }
     ul li { /* ... */ }
   ul li a { /* ... */ }
/*         |-----------|
   We spend a lot of time thinking about this part,
   getting everything to look right cross-browser.
*/
        ul { /* ... */ }
     ul li { /* ... */ }
   ul li a { /* ... */ }
/* |-----|
   But the architecture lives here. OOCSS focuses here
   to make much less code.
*/
```

OOCSS championed grids. Their styles were 574 bytes, 14 lines of code, infinitely nestable (not responsive). Bootstrap grids are 34KB.

```
GRANULARITY FAIL
+ STALE RULES
+ UNPREDICTABILITY
+ DUPLICATION
+ SPECIFICITY WARS

= MASSIVE CSS
```

## Granularity Fail

We shouldn't match CSS to the PHP object.
Lego is great - a few small bricks can build anything. Want to build a life-sized car? It's possible (although it takes a while). Want to build a life-sized sculpture of a person or a giant cartoon head of Albert Einstein? Totally possible.
CSS should work like that. Lots of little pieces to make a full website.

Most websites get the granularity wrong. Most of the time, we add CSS based on the PHP objects that we add. In WordPress with ACF, it's usually based on that field type. This is actually a terrible way of building CSS. Think of the [media object](http://www.stubbornella.org/content/2010/06/25/the-media-object-saves-hundreds-of-lines-of-code/) - there can be a lot of CSS objects in a single PHP object. Sandboxing CSS rules really back-fires when we look at performance. (Think of the headings.)

Do a visual inventory of the website. It's really hard to do with your own site, but easier with another one. Just look at it visually - don't look at the code.

When broken down correctly, you'll see that a lot of visual patterns are repeated throughout the website.

For example, what do the headings look like? Make a list of the headings and heading styles. `grep` is a good tool for this.

PHP Objects !== CSS objects! Beware, you will be tempted.

## Stale Rules

CSS gets crufty. Stuff gets old and just sits there. There are 2 kinds:
1. Truly state: rules that don't apply to anything any more.
2. Rules used on subsequent page views or activated after user action.

2 is difficult because we kinda need it. We can't reduce that to 0. We should delete 1 if we can.

"dust-me selectors" could be useful but it hasn't been updated for 7 years :(

## Unpredictability

Every object should be predictable. We should be able to pick it up and put it anywhere else on the website and it should look the same (or reasonably similar).

A large pink heading shouldn't become small and blue in another part of the page.

Test for things like this:

```css
#foo h3 { /* ... */ }
#bar h3 { /* ... */ }
#baz h3 { /* ... */ }
```

Grep for `h[1-6]`. Since we rely on BEM so much, it's easy to lost track of what elements they apply to. Also look for selectors like `heading` or `title` (e.g. `.my-module__title` or `.component__sub-heading`). If you have a lot, you need a site-wide heading solution.

Nicole Sullivan tested the top 1000 websites in 2009.
- The worse had 511 different heading styles.
- 56% had more than 10.
- 9% had more than 100.

10 doesn't sound like a lot, but how many different font sizes are actually readable by humans? Anything less than 7px is too small to read, anything too big to read is also out. So there aren't many font sizes available and more than 10 is still pretty big.

At one point, Facebook at 958 different heading selectors. After implementing OOCSS, they got that down to 25.

## Specificity Wars

Lobbing specificity bombs over the cubical wall. You end up with selectors that just get worse and worse:

- `h3`
- `.my-title`
- `.container h3`
- `.wrapper .module h3`
- `#holder h3`
- `{ !important }`
- `#holder h3 { !important }`

Use hacks sparingly! Less of an issue now, but we used to have additional classes for Internet Explorer. If you include hacks, don't change the specificity.

Don't style IDs. Only style classes. That keeps the specificity low. IDs can be used for JavaScript hooks.
And don't nest. Nested styles increase the specificity so you need to fight specificity wars just to modify the styles.

Don't use `!important` - once it's in, you'll never get it out.
Only ever use it on a "leaf node" - something that won't have any children, something that won't need to be modified later.

Worse offender in top 1000 websites in 2009 has 518 uses of `!important`. 12% of the websites had more than 50.

Style classes intead of elements. Style things without elements.
`.my-module__element` rather than `div.my-module__element`.
Try to keep to the same specificity. That will make working with the style sheet much easier.

## Duplication

Grep is your friend when you're looking for duplication. Look at the style sheet globally rather than locally.

- How many times are you changing your `margin`? If you're doing it too many times, you probably need a reset.css file.
- How many times are you using `float`? If you're using it too many times, you probably need a grid. `display: flex;` is another good thing to look for. Using it too often means a maintenance nightmare - we have to re-learn how every object works. "How did this dev decide to lay out their columns?"
- `font-size` is good to look for. Often it hides heading styles. The worst offender user 889 `font-size` declarations. 78% > 10, 23% > 100. There aren't that many usable font sizes on the web.

One example of duplication in Facebook's old layout was the media object - a layout piece with an image to the right of some text. They looked at it and worked out what they know and what they don't know.
They know:
- It can be nested. Comments and sub-comments do this.
- Optional right button to remove it.
- Must clearfix.
They don't know:
- The image size and decoration vary.
- Right content is unknown - it could just be text or a complex structure.
- The overall width. It could be in the sidebar or the main column. I can also just be the number of likes.

They separated the structure from the chrome. Some of them have a blue background, others have white. So the element can't have a background defined, it has to allow that to be altered.

The markup is really simple:

```html
<div class="media attribution">
    <a href="..." class="img">
        <img src="mini.jpg" alt="...">
    </a>
    <div class="bd">@Stubbornella 14 minutes ago</div>
</div>
```

The modern CSS is also very simple:

```css
.media { display: flex; }
.media .img { margin-right: 10px; }
.media .img img { display: block; }
.media .bd { flex-grow: 1; }
```

This is so well used, it's the poster child of OOCSS.

"Won't the HTML get bigger?"
Maybe, but careful planning usually reduces the size. Think of the length of a class name, for example. If you're re-using elements, you might not need `__` or `--` in the class names.
