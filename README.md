# RiotJS Style Guide

Opinionated *RiotJS Style Guide* for teams by [De Voorhoede](https://twitter.com/devoorhoede).


## Purpose

This guide provides a uniform way to structure your [RiotJS](http://riotjs.com/) code. Making it

* easier for developers / team members to understand and find things.
* easier for IDEs to interpret the code and provide assistance.
* easier to (re)use build tools you already use.
* easier to cache and serve bundles of code separately.

This guide is inspired by the [AngularJS Style Guide](https://github.com/johnpapa/angular-styleguide) by John Papa.


## Table of Contents

* [Module based development](#module-based-development)
* [Tag module names](#tag-module-names)
* [1 module = 1 directory](#1-module--1-directory)
* [Use `*.tag.html` extension](#use-taghtml-extension)
* [Use `<script>` inside tag](#use-script-inside-tag)
* [Keep tag expressions simple](#keep-tag-expressions-simple)
* [Avoid `tag.parent`](#avoid-tagparent)
* [Put styles in external files](#put-styles-in-external-files)
* [Use tag name as style scope](#use-tag-name-as-style-scope)
* [Add a tag demo](#add-a-tag-demo)


## Module based development

Always construct your app out of small modules which do one thing and do it well. 

A module is a small self-contained part of an application. The RiotJS micro-framework is specifically designed to help you create *view-logic modules*, which Riot calls *tags*.

### Why?

Small modules are easier to learn, understand, maintain, reuse and debug. Both by you and other developers.

### How?

Each riot tag (like any module) must be [FIRST](https://addyosmani.com/first/): *Focused* ([single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle)), *Independent*, *Reusable*, *Small* and *Testable*.

If your module does too much or gets too big, split it up into smaller modules which each do just on thing.
As a rule of thumb, try to keep each tag file less than 100 lines of code.
Also ensure your tag module works in isolation. For instance by adding a stand-alone demo.


## Tag module names

A tag module is a specific type of module, containing a Riot tag. 

Each module name must be:

* **Meaningful**: not overspecific, not overly abstract.
* **Short**: 2 or 3 words.
* **Pronouncable**: we want to be able talk about them.

Tag module names must also be:

* **Custom element spec compliant**: [include a hyphen](https://www.w3.org/TR/custom-elements/#concepts), don't use reserved names.
* **`app-` namespaced**: if very generic and otherwise 1 word, so that it can easily be reused in other projects.

### Why?

* The name is used to communicate about the module. So it must be short, meaningful and pronouncable.
* The `tag` element is inserted into the DOM. As such, they need to adhere to the spec.

### How?

```html
<!-- recommended -->
<app-header />
<user-list />
<range-slider />

<!-- avoid -->
<btn-group /> <!-- short, but unpronouncable. use `button-group` instead -->
<ui-slider /> <!-- all tags are ui elements, so is meaningless -->
<slider /> <!-- not custom element spec compliant -->
```


## 1 module = 1 directory

Bundle all files which construct a module into a single place.

### Why?

Bundling module files (Riot tags, tests, assets, docs, etc.) makes them easy to find, move and reuse.

### How?

Use the module name as directory name and file basename.
The file extension depends on the purpose of the file.

	modules/
		my-example/
			my-example.tag.html
			my-example.less
			...
			README.md

If your project uses nested structures, you can nest a module within a module.
For example a generic `radio-group` module may be placed directly inside "modules/". While `search-filters` may only make sense inside a `search-form` and may therefore be nested:

    modules/
        radio-group/
            radio-group.tag.html
        search-form/
            search-form.tag.html
            ...
            search-filters/
               search-filters.tag.html


## Use `*.tag.html` extension

Riot introduces a new concept called *tags*, and suggests to use a `*.tag` extension.
However in essence these tags are simply custom elements containing markup. Therefore you should **use the `*.tag.html` extension**.

### Why?

* Tells developers it's not just HTML, but a Riot tag element.
* Improves IDE support (signals how to interpret).

### How?

In case of [in-browser compilation](http://riotjs.com/guide/compiler/#in-browser-compilation):
```html
<script src="path/to/modules/my-example/my-example.tag.html" type="riot/tag"></script>
```

In case of [pre-compilation](http://riotjs.com/guide/compiler/#pre-compilation), set the [custom extension](http://riotjs.com/guide/compiler/#custom-extension):
```bash
riot --ext tag.html modules/ dist/tags.js
```


## Use `<script>` inside tag

While Riot supports writing JavaScript inside a tag element [without a `<script>`](http://riotjs.com/guide/#no-script-tag),
you should **always use `<script>`** around scripting. This is closer to web standards and prevents confusing developers and IDEs. 

### Why?

* Prevents markup being interpreted as script.
* Improves IDE support (signals how to interpret).
* Tells developers where markup stops and scripting starts.

### How?

```html
<!-- recommended -->
<my-example>
	<h1>The year is { this.year }</h1>
	
	<script>
		this.year = (new Date()).getUTCFullYear();
	</script>
</my-example>

<!-- avoid -->
<my-example>
	<h1>The year is { this.year }</h1>
	
	this.year = (new Date()).getUTCFullYear();
</my-example>
```


## Keep tag expressions simple

Riot's inline [expressions](http://riotjs.com/guide/#expressions) are 100% Javascript. This makes them extemely powerful, but potentially also very complex. Therefore you should **keep tag expressions simple**.

### Why?

* Complex inline expressions are hard to read.
* Inline expressions can't be reused elsewehere. This can lead to code duplication and code rot.
* IDEs typically don't have support for expression syntax, so your IDE can't autocomplete or validate.

### How?

Move complex expressions to tag methods or tag properties. 

```html
<!-- recommended -->
<my-example>
	{ year() + '-' + month() }
	
	<script>
		const twoDigits = (num) => ('0' + num).slice(-2);
		this.month = () => twoDigits((new Date()).getUTCMonth() +1);
		this.year  = () => (new Date()).getUTCFullYear();
	</script>
</my-example>

<!-- avoid -->
<my-example>
	{ (new Date()).getUTCFullYear() + '-' + ('0' + ((new Date()).getUTCMonth()+1)).slice(-2) }
</my-example>
```

## Avoid `tag.parent`

Riot supports [nested tags](http://riotjs.com/guide/#nested-tags) which have access to their parent context through `tag.parent`. Accessing context outside your tag module violates the [FIRST](https://addyosmani.com/first/) rule of [module based development](#module-based-development). Therefore you should **avoid using `tag.parent`**.

The exception to this rule are anonymous child tags in a [for each loop](http://riotjs.com/guide/#loops) as they are defined directly inside the tag module.

### Why?

* A tag module, like any module, must work in isolation. If a tag needs to access its parent, this rule is broken.
* If a tag needs access to its parent, it can no longer be reused in a different context. 
* By accessing its parent a child tag can modify properties on its parent. This can lead to unexpected behaviour.

### How?

* Pass values from the parent to the child tag using attribute expressions.
* Pass methods defined on the parent tag to the child tag using callbacks in attribute expressions.

```html
<!-- recommended -->
<parent-tag>
	<child-tag value="{ value }" /> <!-- pass parent value to child -->
</parent-tag>

<child-tag>
	<span>{ opts.value }</span> <!-- use value passed by parent -->
	<script></script>
</child-tag>

<!-- avoid -->
<parent-tag>
	<child-tag />
</parent-tag>

<child-tag>
	<span>value: { parent.value }</span> <!-- don't do this -->
</child-tag>
```
```html
<!-- recommended -->
<parent-tag>
	<child-tag on-event="{ methodToCallOnEvent }" /> <!-- use method as callback -->
	<script>this.methodToCallOnEvent = () => { /*...*/ };</script>
<parent-tag>

<child-tag>
	<button onclick="{ opts.onEvent }"></button> <!-- call method passed by parent -->
</child-tag>

<!-- avoid -->
<parent-tag>
	<child-tag />
	<script>this.methodToCallOnEvent = () => { /*...*/ };</script>
<parent-tag>

<child-tag>
	<button onclick="{ parent.methodToCallOnEvent }"></button> <!-- don't do this -->
</child-tag>
```
```html
<!-- allowed exception -->
<parent-tag>
	<button each="{ item in items }"
		onclick="{ parent.onEvent }"> <!-- okay, because button is not a riot tag -->
		{ item.text }
	</button>
	<script>this.onEvent = (e) => { alert(e.item.text); }</script>
</parent-tag>
```

## Put styles in external files

For developer convenience, Riot allows you to define a tag element's style in a [nested `<style>` tag](http://riotjs.com/guide/#tag-styling). While you can [scope](http://riotjs.com/guide/#scoped-css) these styles to the tag element, Riot does not provide true encapsulation. Instead Riot extracts these styles from the tags (JavaScript) and injects them into the document on runtime. Since Riot compiles nested styles to JavaScript and doesn't have true encapsulation, you should instead **put styles in external files**.

### Why?

* External stylesheets can be handled by the browser independently of Riot and tag files. This means styles can be applied to initial markup even if JavaScript errors occur or isn't loaded (yet).
* External stylesheets can be used in combination with pre-processors (Less, Sass, PostCSS, etc) and your own (existing) build tools.
* External stylesheets can be minified, served and cached separately. This improves performance.
* Riot expressions are not supported in nested `<style>`s so there's no added benefit in using them.

### How?

Styles related to the tag and its markup, should be placed in a separate stylesheet file next to the tag file, inside its module directory:

    my-example/
        my-example.tag.html
        my-example.(css|less|scss)    <-- external stylesheet next to tag file
        ...


## Use tag name as style scope

Riot tag elements are custom elements which can very well be used as style scope root.
Alternatively the module name can be used as CSS class namespace.

### Why?

* Scoping styles to a tag element improves predictability as its prevents styles leaking outside the tag element.
* Using the same name for the module directory, the Riot tag and the style root makes it easy for developers to understand they belong together.

### How?

Use the tag name as selector, as parent selector or as namespace prefix (depending on your CSS naming strategy):

```css
/* recommended */
my-example { }
my-example li { }
.my-example__item { }

/* avoid */
.my-alternative { } /* not scoped to tag or module name */
.my-parent .my-example { } /* .my-parent is outside scope, so should not be used in this file */
```


# Add a tag demo

Add a `*.demo.html` file with demos of the tag with different configurations, showing how the tag can be used.

## Why?

* A tag demo proofs the module works in isolation.
* A tag demo gives developers a preview before having to dig into the documentation or code.
* Demos can illustrate all the possible configurations and variations a tag can be used in. 

## How?

Add a `*.demo.html` file to your module directory:

	modules/
		city-map/
			city-map.tag.html
			city-map.demo.html
			city-map.css
			...

Inside the demo file:

* Include `riot+compiler.min.js` to also compile during runtime.
* Include the tag file (e.g. `./city-map.tag.html`).
* Create a `demo` tag (`<yield/>`) to embed your demos in (otherwise option attributes are not supported).
* Write demos inside the `<demo>` tags.
* As a bonus add `aria-label`s to the `<demo>` tags and style those as title bars.
* Initialise using `riot.mount('demo', {})`.

Example demo file in `city-tag` module:

```html
<!-- modules/city-map/city-map.demo.html: -->
<body>
    <h1>city-map demos</h1>
    
    <demo aria-label="City map of London">
        <city-map location="London" />    
    </demo>
    
    <demo aria-label="City map of Paris">
        <city-map location="Paris" />    
    </demo>
    
    <link rel="stylesheet" href="./city-map.css">
    
    <script src="path/to/riot+compiler.min.js"></script>
    <script type="riot/tag" src="./city-map.tag.html"></script> 
    <script>
        riot.tag('demo','<yield/>');
        riot.mount('demo', {});
    </script>
    
    <style>
    	/* add a grey bar with the `aria-label` as demo title */
    	demo:before {
        	content: "Demo: " attr(aria-label);
	        display: block;
        	background: #F3F5F5;
        	padding: .5em;
        	clear: both;
        }
    </style>
</body>
```
Note: this is a working concept, but could be much cleaner using build scripts.
