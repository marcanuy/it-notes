---
title: 5 Steps To Add Bootstrap 4 To Jekyll The Right Way
description: Guide to add Bootstrap 4 to Jekyll with focus on having also a CSS stylesheet using its own variables and custom ones.
---

## Overview

One of the keys to use Bootstrap successfully is to be able to use and
redefine its variables in our custom designs. We should not simply add
Bootstrap's javascript and CSS stylesheets to use its components, we
need to change them and not making all the web look boringly the same.

This is a guide to make it easy to use Bootstrap 4 with a Jekyll
website and be able to use and customize its variables as well as
defining new ones.

## Background

Jekyll provides built-in support for *syntactically awesome
stylesheets* ([Sass]). 

Sass is a CSS extension language, it provides:

- Variables
- Nesting elements
  - Loops
  - Arguments
- Selector inheritance

It consists of two syntaxes:

- the original, indented, syntax uses the `.sass` extension.
- the newer syntax, more similar to CSS, uses `.scss` extension.

To make Jekyll process these SASS files, we need to create files with
the proper extension name (`.scss` or `.sass`) and start the file
contents with two lines of triple dashes.

A file named `assets/main.scss` will be rendered like `assets/main.css`.

As Bootstrap switched from `Less` to `Sass`[^bs-blog] now we can use
it directly without relying in parallel projects like [bootstrap-sass](https://github.com/twbs/bootstrap-sass)
needed for Bootstrap 3.

## Installing Bootstrap 4

We will be using the package manager [Yarn] to install Bootstrap. At
our Jekyll website root folder we run <kbd>yarn install</kbd>. In this
case I will be using **Bootstrap  v4.0.0** but you can find the
latest one
at <https://getbootstrap.com/docs/4.0/getting-started/download/>.

Previously I used [Bower] in this tutorial but as it won't be
maintained anymore, [Yarn] is a better, more robust solution.
{: .alert .alert-info}

<pre class="shell">
<samp>
<span class="shell-prompt">$</span> <kbd>yarn install</kbd>
yarn install v1.5.1
info No lockfile found.
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
info Lockfile not saved, no dependencies.
Done in 0.16s.

<span class="shell-prompt">$</span> <kbd>yarn add bootstrap@4.0.0</kbd>
yarn add v1.5.1
info No lockfile found.
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
warning " > bootstrap@4.0.0" has unmet peer dependency "jquery@1.9.1 - 3".
warning " > bootstrap@4.0.0" has unmet peer dependency "popper.js@^1.12.9".
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 1 new dependency.
info Direct dependencies
└─ bootstrap@4.0.0
info All dependencies
└─ bootstrap@4.0.0
Done in 1.16s.

<span class="shell-prompt">$</span> <kbd>yarn add jquery</kbd>
yarn add v1.5.1
warning package.json: No license field
warning No license field
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
warning " > bootstrap@4.0.0" has unmet peer dependency "popper.js@^1.12.9".
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 1 new dependency.
info Direct dependencies
└─ jquery@3.3.1
info All dependencies
└─ jquery@3.3.1
warning No license field
Done in 1.15s.

<span class="shell-prompt">$</span> <kbd>yarn add popper.js</kbd>
yarn add v1.5.1
warning package.json: No license field
warning No license field
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 1 new dependency.
info Direct dependencies
└─ popper.js@1.14.3
info All dependencies
└─ popper.js@1.14.3
warning No license field
Done in 1.02s.

</samp>
</pre>

So far, we have added this structure (excluding jekyll files):

<pre class="shell">
<samp>
<span class="shell-prompt">$</span> <kbd>tree -L 2</kbd>
.
├── node_modules
│   ├── bootstrap
│   ├── jquery
│   └── popper.js
├── package.json
└── yarn.lock

4 directories, 2 files
</samp>
</pre>

If you aren't working in a Jekyll project, you can add it now to the
current directory with <kbd>jekyll new . --force</kbd>.

## Adding new Sass load paths

We need to add this new path so Jekyll can process its Sass
files. 

Jekyll will look at the folder specified by the `sass_dir`
configuration key (`/_sass` by default), but it also supports extending
it, so it will process other folders too.

Instead of setting a custom sass folder with:

~~~ yaml
sass:
    sass_dir: _sass
~~~

we use the `load-paths`[^load-paths] key in `_config.yml` to add more paths:

~~~ yaml
sass:
    load_paths:
        - _sass
        - node_modules
~~~

`load_paths` only works when **not** in safe mode[^safe-mode] (i.e. it
won't work with Github Pages.)
{: .alert .alert-danger}

### Whitelist node_modules

Jekyll 3.3 started excluding the `node_modules` directory [by default](https://github.com/jekyll/jekyll/pull/5210),
so we have to make sure it is included in the final site to be able to
use its files:

~~~
exclude: []
~~~

## Add javascript

Add Bootstrap JavaScript at the end of the document so the pages load
faster, just before the `</body>` HTML tag.

We add them in the default layout at `_layouts/default.html` or in
`footer.html` in the `_includes` folder:

~~~ liquid
<html>
<body>
...
	<script src="{% raw %}{{'/node_modules/jquery/dist/jquery.min.js' | prepend: site.baseurl}}"{% endraw %}></script>
	<script src="{% raw %}{{'/node_modules/popper.js/dist/umd/popper.min.js' | prepend: site.baseurl}}{% endraw %}"></script>
	<script src="{% raw %}{{'/node_modules/bootstrap/dist/js/bootstrap.min.js' | prepend: site.baseurl}}{% endraw %}"></script>
</body>
</html>
~~~



## Import Bootstrap and use Sass variables

### Create variables Sass partial

To be able to define new variables and reuse the ones defined in
Bootstrap, we create a new partial Sass file `_sass/_variables.scss`.

1. We define our variables
2. "Overwrite" the ones we want from Bootstrap
`node_modules/bootstrap/scss/_variables.scss` before loading them
and then 
3. we import the Bootstrap variables.

All the variables defined in Bootstrap 4 have the **`!default`**[^scss_default]
property at the end.
When Jekyll process each Scss file, **it only defines the variables that do not
have been assigned any value yet**, so we can define Bootstrap's
variables before Bootstrap itself define them. It is important to do
this **before importing the variables** because there are many of them
depending on each other to calculate CSS properties values.
{: .alert .alert-danger }

In `_sass/_variables.scss`:

~~~ scss
$custom-font-size: 20px;
@import "../node_modules/bootstrap/scss/variables";
~~~

### Import variables from main Sass file

After we have our variables, we import them from our main style sheet:

In `assets/main.scss` we import them and then work with our styles,
using the above variables:

~~~ scss
---
---

@import "bootstrap/scss/functions";
@import "variables";
@import "bootstrap/scss/bootstrap";

.content {
  font-size: $custom-font-size;
}
~~~

Don't miss the triple dashes at the beginning of the file to ensure
Jekyll reads the file to be transformed into CSS later
{: .alert .alert-danger}

## Add css to layout

After we have our `assets/main.scss`, Jekyll will process it and
generate the final CSS file: `assets/main.css`.

That is the path we need to add to our layout, in the `<head>`
section of `_layouts/default.html` we include the css: `<link
rel="stylesheet" href="/assets/main.css">`

~~~html
<html>
<head>
<!-- site css -->
<link rel="stylesheet" href="{{'/assets/main.css' | prepend: site.baseurl}}">
</head>
</html>
~~~

## All together

This is the basic flow Jekyll follows processing these Scss files to
generate `assets/main.css`:

<div class="mermaid">
graph TB
        STYLESSCSS["Jekyll reads <strong>/assets/main.scss</strong>"]-->partials{"imports Sass partials"}
        partials--> |"import variables"|VARS
        partials--> |"import bootstrap/scss/bootstrap"|BS_SCSS
        VARS["/_sass/_variables.scss"]-->BS_VARIABLES
        BS_VARIABLES["@import '../node_modules/bootstrap/scss/variables';"]-->STYLESCSS
        BS_SCSS["/node_modules/bootstrap/scss/bootstrap.scss"]-->STYLESCSS
        STYLESCSS["Generates /assets/main.css"]
</div>

## Other

### Site

You’ll find
a ****minimal example site*** hosted on Github showing the result of
following this tutorial at <https://github.com/marcanuy/jekyll-bootstrap4>).

### Video

There is also a step by step video using the previous package manager
used here **Bower** instead of **Yarn** and a previous version of
Bootstrap, so you can get a general idea of how this works but it
won't reflect the current status of the tutorial:

<div class="embed-responsive embed-responsive-16by9 m-1">
  <iframe class="embed-responsive-item" src="//www.youtube.com/embed/0EI1V_Whgto" allowfullscreen></iframe>
</div>

### Skel

It is also part of a Jekyll starter
site [jekyll-skeleton](https://github.com/marcanuy/jekyll-skeleton).

## OPTIONAL: Keep node_modules out of _site

You probably don't want to expose all the package files in your
website, nor do I, so let's see how to serve just only the needed
files.

To make this we will copy the files that we are including directly
from the `node_modules` directory to a new one containing just
these files in each build, that means, we have to set up a script
replacement for <kbd>jekyll build</kbd> and <kbd>jekyll serve</kbd>.

In this case I will be using the
classic [make](https://www.gnu.org/software/make/manual/make.html)
program, each task is pretty self explanatory and can also be ported
easily to [Grunt](https://gruntjs.com) or any other task automation solution.

### Script

Create a file called `Makefile` at root level with this content (if
using Bundler):

~~~ make
SHELL := /bin/bash
BUNDLE := bundle
YARN := yarn
VENDOR_DIR = assets/vendor/
JEKYLL := $(BUNDLE) exec jekyll

PROJECT_DEPS := Gemfile package.json

.PHONY: all clean install update

all : serve

check:
	$(JEKYLL) doctor
	$(HTMLPROOF) --check-html \
		--http-status-ignore 999 \
		--internal-domains localhost:4000 \
		--assume-extension \
		_site

install: $(PROJECT_DEPS)
	$(BUNDLE) install --path vendor/bundler
	$(YARN) install

update: $(PROJECT_DEPS)
	$(BUNDLE) update
	$(YARN) upgrade

include-yarn-deps:
	mkdir -p $(VENDOR_DIR)
	cp node_modules/jquery/dist/jquery.min.js $(VENDOR_DIR)
	cp node_modules/popper.js/dist/umd/popper.min.js $(VENDOR_DIR)
	cp node_modules/bootstrap/dist/js/bootstrap.min.js $(VENDOR_DIR)

build: install include-yarn-deps
	$(JEKYLL) build

serve: install include-yarn-deps
	JEKYLL_ENV=production $(JEKYLL) serve
~~~

It is just a wrapper of Jekyll build and install commands handling
dependencies.

Remember that those spaces are <kbd>TAB's</kbd> or `make` will fail.
{: .alert .alert-danger}

To make it easier, I've created
a
[gist](https://gist.github.com/marcanuy/fefafbd617201aee2892666ee2d28761) with
the above script in two flavours:

- Makefile a `Gemfile` (not using Bundler).
- Makefile using Bundler.

Now we will use `make build` and `make serve` to work with Jekyll as
wrappers of `jekyll build` and `jekyll serve` respectively.

Remember that Makefile content uses **TABS** to indent recipes instead
of *spaces*.
{: .alert .alert-danger}

### Using new paths

It just remain to update our paths in the layout, in `default.html`
use them as:

~~~ html
<script src="{%raw%}{{'/assets/vendor/jquery.min.js' | absolute_url}}{%endraw%}"></script>
<script src="{%raw%}{{'/assets/vendor/popper.min.js' | absolute_url}}{%endraw%}"></script>
<script src="{%raw%}{{'/assets/vendor/bootstrap.min.js' | absolute_url}}{%endraw%}"></script>
~~~

### Exclude unwanted directories

Tell Jekyll to not include the `node_modules` directory in the final
site, in `_config.yml`:

~~~ 
exclude:
 - node_modules
~~~

This is the default behaviour from Jekyll 3.3
{: .alert .alert-info}

### Final

Now we are just including in our website the files we chose from the
`node_modules` folder, placing them in `assets/vendor` and avoiding to
have any other unnecessary files in the final website.

## Conclusion

When we build our site, Jekyll will process the `.scss` files with our
custom variables and we will have them in our `_site/assets/main.css`. In
this example its content starts with the Bootstrap code and ends with
our custom `_main.scss` processed, looking like:

~~~ css
/*!
 * Bootstrap v4.0.0 (https://getbootstrap.com)
 * Copyright 2011-2018 The Bootstrap Authors
 * Copyright 2011-2018 Twitter, Inc.
 * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE)
 */
:root {
  --blue: #007bff;
  --indigo: #6610f2;
  --purple: #6f42c1;

...

.content {
  font-size: 20px; }
~~~

## References

- Sass (stylesheet language) <https://en.wikipedia.org/wiki/Sass_(stylesheet_language)>
- Sass project website <http://sass-lang.com/>
- Bootstrap 4 customization <http://v4-alpha.getbootstrap.com/getting-started/options/>

[Sass]: http://sass-lang.com/
[Yarn]: https://yarnpkg.com/lang/en/docs/install
[Bower]: https://bower.io
[^load-paths]: [Issue](https://github.com/jekyll/jekyll/issues/3366) referring the code at <https://github.com/jekyll/jekyll-sass-converter/blob/master/lib/jekyll/converters/scss.rb#L77>
[^tether]:
    [Tether](http://github.hubspot.com/tether/) is a small,
    focused JavaScript library for defining and managing the position of
    user interface (UI) elements in relation to one another on a web page
    used by Bootstrap
	
*[CSS]: Cascading Style Sheets

[^bs-blog]:
    <http://blog.getbootstrap.com/2015/08/19/bootstrap-4-alpha/>
	
	> Moved from Less to Sass. Bootstrap now compiles faster than ever
    > thanks to Libsass, and we join an increasingly large community of
    > Sass developers.

[^safe-mode]:
    Safe mode disables custom plugins, and ignores symbolic links. 
	
	> The reason multiple load paths are shut off in safe mode is
	> because of a fundamental inability to trust non-user content.
	> 
	> <footer class="blockquote-footer"> <cite>parkr comment in <a href="https://github.com/jekyll/jekyll-sass-converter/issues/44#issuecomment-169858494">Allow multiple load_paths in safe mode</a></cite></footer>
	{: class="blockquote" cite="https://github.com/jekyll/jekyll-sass-converter/issues/44#issuecomment-169858494"}
	

[^scss_default]:
	SASS Variable Defaults: !default <http://www.sass-lang.com/documentation/file.SASS_REFERENCE.html#variable_defaults_>

	> You can assign to variables if they aren’t already assigned by adding the !default flag to the end of the value. This means that if the variable has already been assigned to, it won’t be re-assigned, but if it doesn’t have a value yet, it will be given one.

	> For example:

    ~~~scss
	$content: "First content";
	$content: "Second content?" !default;
	$new_content: "First time reference" !default;

	#main {
	  content: $content;
	  new-content: $new_content;
	}
	~~~

	> is compiled to:

	~~~css
	#main {
	  content: "First content";
	  new-content: "First time reference"; }
	~~~
