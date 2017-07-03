# Purepoint SCSS guide
> 📚 Guide to writing a modern, maintainable SCSS framework

<img src="https://user-images.githubusercontent.com/12545/27768577-cba84eaa-5f0e-11e7-8a84-dc480af816c1.png" width="100%">

Writing SCSS can be hard. In larger projects it's easy to end up with fragmented messy code. This guide focuses on patterns for writing maintainable SCSS on large projects with multiple developers.

1. [Architecture](#1-architecture)
    - [Framework](#framework)
    - [Components](#components)
    - [Helpers](#helpers)
    - [Modifiers](#modifiers)
    - [State](#state)
    - [Media Queries](#media-queries)
    - [Javascript Hooks](#javascript-hooks)
2. [Putting It Together](#2-putting-it-together)
3. [Style](#3-style)
4. [Best Practices](#4-best-practies)
5. [Performance](#5-performance)
6. [Cross Browser](#6-cross-browser)
7. [Further Reading](#7-further-reading)
    - [Thanks](#thanks)
    - [Contributing](#contributing)
8. [Licence](#8-licence)
9. [About Purepoint](#9-about-purepoint)

## 1. Architecture

> Our goal is to write SCSS that is entirely generic. A good way to envisage this is by considering a framework like [Bootstrap](https://v4-alpha.getbootstrap.com).

To achieve this we break our SCSS down into a `framework`, `components` and `helpers`.

- The `framework` sets the bases rules, usually driven by the brand and design. Fonts, margins, colors, default element styling, grid etc.
- A `component` is an encapsulated piece of functionality. It has an isolated use, and can be thrown away without impacting anything else. Navbars, hero units, unique components etc.
- A `helper` is a tool that allows us to quickly manipulate display properties. Background colors, text sizes, flexbox properties etc.

In many cases we will use an open source SCSS `framework` such as Bootstrap, and use `$variables` to customise it. In the base of bootstrap it already follows the above architecture, and we get many `components` and `helpers` 'for free'.

**Structure:**
```
framework.scss
/components
  ↳ _specific_component.scss
/helpers
  ↳ _specific_helper.scss
```

Once we have a good implementation of the above pattern on a project it means that we only have to write SCSS in unique situations.

Make sure **every selector is a class**. There should be no reason to use id or element selectors. No underscores or camelCase. Everything should be lowercase.

There should **never** be SCSS written for a 'page' or for a specific instance of a component. So `about-us.scss` should simply not exist. Or `about-navbar.scss` should just be `sub-navbar.scss` so it can be reused elsewhere.

### Framework

> Use the framework to define everything related to the design and brand up-front

Think of the framework as a set of rules everything else will follow. We use `$variables` to define everything custom. In a larger project this may be several hundred unique variables.

``` SCSS
// Body
$body-bg                : $gray;
$body-color             : $black;

// Links
$link-color             : $blue;
$link-decoration        : none;

// Fonts
$font-size-base         : 1.125rem; //18px on most devices
$font-size-base-md      : .875rem;  //14px on most devices
$line-height-base       : 1.35;
$font-family-sans-serif : "Bariol", "Arial", sans-serif;

// [... etc]
```

This approach means that we never use things like hex values, or specific number values in our components or helpers. If we need to change a color, a margin, or a visual effect we need only change it in one place: the `$variable` in the framework.

### Components

> Use the `.component-descendant-descendant` pattern

A component creates an encapsulated piece of visual functionality. It has it's own unique, descriptive namespace. 

Components shouldn’t know anything about each other and should be reusable in other places. Everything you need to know about the component should be in the file. Inversely, you shouldn’t overwrite or include component styles in other components. This has a lot of advantages:

1. You can see everything about the component in the file.
2. You don’t have to worry about other components overriding a style.
3. You can reuse the component elsewhere.
4. Components are small and readable.

Using the `component-descendant` pattern helps stop us writing 'super-components' which are too big and unwieldy. A good 'rule of thumb' is that if you have more than 4 descendants, you should consider how to break your component up into smaller, individual components.

An example of this may be breaking a `header` component up into separate `global-header`, `navbar` and `submenu` components.

We consider it good practice when writing a component to also create a matching HTML partial and assets folder. For example:

 - `navbar.scss`
 - `_navbar.html`
 - `/assets/navbar/`

This makes is easy to throw it away, or rewrite it. But this differs depending upon the language and framework in use.

When writing a component, it's important to remember that it will be used in conjunction with helper classes. Depending upon the helpers you have you may want to reduce the specificity as much as possible.

Here’s an example of name spacing using the `.component-descendant-descendant` pattern:

``` LESS
.global-header {
  padding: $spacer;

   &-logo {
    display: block;

    &-img {
      height: 40px;
    }
  }
}
```

This results in 3 independent classes:

``` CSS
.global-header
.global-header-logo
.global-header-logo-img
```

This is opposed to using descendent nested selectors. which we **do not do**:

``` LESS
.global-header {
  padding: $spacer;

   .logo {
    display: block;

    img {
      height: 40px;
    }
  }
}
```

There are a few reasons we follow this pattern:
 - It makes classes much more understandable in the HTML
 - It forces clean encapsulation of components
 - It makes it very obvious when you are building a 'super component' that should be broken down into smaller components

### Helpers

> Use the `.description-option` pattern

Helpers are tools that let us add additional classes to an object or a component to manipulate them visually.

Generally helpers will be built using more complex SCSS to generate a range of options automatically.

An example of this may be a `.bg-` color helper:

``` LESS
// Utility to generate a `bg-x` color helpers

@mixin make-bg-colors($brand-colors) {
  @each $color in $brand-colors {
    &-#{nth($color, 1)} {
      background: nth($color, 2);
    }
  }
}

.bg {
  @include make-bg-colors($brand-colors);
}
```

``` LESS
// Colours defined in the main Framework
$brand-colors: (
  'orange' : #ff7a00,
  'gray'   : #33495b,
  'blue'   : #3d61e8
);
```

This results in 3 independent classes:

``` CSS
.bg-orange
.bg-gray
.bg-blue
```

The advantage of this approach is that if we add a new color, say `red`, to the main framework our helper will automatically just let us use a `bg-red` class to change a components background to red.

### Modifiers

> Use the `&.mod-modifier` pattern for modifier classes.

Modifiers are useful to change the state of a component in a specific instance. A special 'sign up' button with a shadow in a navbar for example. Modifiers should be reserved for instances where using generic helpers would be too unwieldy. (Creating a generic helper for a specific drop shadow would be overkill).

``` HTML
<a class="global-header-nav-item mod-sign-up">
  Sign Up
</a>
```

``` LESS
...
&-item {
  display: block;

  &.mod-sign-up {
    box-shadow: 10px 10px 5px 0px rgba(0,0,0,0.75);
  }
}
```

**You should never write a bare `.mod-` class**. It should always be nested inside a component. We could be using a `.mod-` in another component and we wouldn’t want to override.

### State

> Use the `&.is-state` pattern for state. Manipulate `.is-` classes in JavaScript (but not presentation classes).

State classes show that something is enabled, expanded, hidden, loading etc. They denote a temporary state, usually influenced by an external factor.

``` LESS
...
&-submit-button {
  background: url("logo.png");

  &.is-loading {
    background: url("logo-loading.gif");
  }
}
```

The `.component.is-state` pattern decouples state and presentation concerns so we can add state classes without needing to know about the presentation class.

Like modifiers, it’s possible that the same state class will be used on different components. You don’t want to override or inherit styles, so it’s important that **every component defines its own styles for the state**. They should never be defined on their own. They should always be nested.

### Media Queries

> Use media query variables inside your component classes.

Media queries breakpoints should be defined in your `framework` and then any `helpers` added so they can be easily use. If you are using a framework like bootstrap, you get this setup out-the-box.

``` SCSS
@include media-breakpoint-up(xs) { ... }
@include media-breakpoint-up(sm) { ... }
@include media-breakpoint-up(md) { ... }
@include media-breakpoint-up(lg) { ... }
@include media-breakpoint-up(xl) { ... }

// Example usage:
.some-class {
  @include media-breakpoint-up(sm) {
    display: block;
  }
}
```

### Javascript Hooks

> Separate style and behaviour concerns by using a data tag for javascript selectors `data-js="open-contact-menu"`

Naturally, this depends entirely on your javascript framework. Many frameworks will be opinionated on how they integrate with your html and css. In some cases, like React, they may be entirely bound-together. However, if you are working in a project without an opinionated framework, and maybe only a simple tool like jQuery, we suggest you follow this pattern.

``` HTML
<a href="#" class="content-nav-button" data-js="open-content-menu">
  Menu
</a>
```

``` JavaScript
// JavaScript (with jQuery)
$("[data-js='open-content-menu']").on("click", function(e){
  openMenu();
});
```

## 2. Putting it together

Here is an example of a simple component that uses all the above patterns:

``` SCSS
// Global Header, at the top of every page
.global-header {
  padding: $spacer;

   &-logo {
    // Only display logo above the sm breakpoint
    display: none;
    @include media-breakpoint-up(sm) {
      display: block;
    }

    &:hover {
      background: $black;
    }

    &-img {
      height: 40px;
    }
  }

  // Minimised header state, when user scrolls down
  &.is-minimised {
    padding: $spacer / 2;

    &-logo-img {
      height: 30px;
    }
  }

  // Change the logo position on partner pages
  &.mod-partner-page {
    &-logo {
      float: right;
    }
  }
}
```

``` HTML
<header class="global-header bg-white">
  <a href="/" class="global-header-logo">
    <img src="logo.png" class="global-header-logo-img">
  </a>
</header>
```

The `<header>` element can have `is-minimised` and `mod-partner-page` added as additional classes to change its behaviour.

The header may have other components such as `navigation` inside it. But those would be treated separately and with no dependency.

## 3. Style

> Use the sass-lint style guide

It's possible to write SCSS in many different ways. To keep things simple we follow the `sass-lint` rules:

> [sass-lint rules](https://github.com/sasstools/sass-lint/tree/master/docs/rules)

It is not nessesary to remember these, as the style and formatting should be automatically enforced in your IDE, and in the CI using tools like [Code Climate](https://codeclimate.com/)

**IDE Integration**

* [Atom](https://atom.io/packages/linter-sass-lint)
* [Sublime Text](https://github.com/skovhus/SublimeLinter-contrib-sass-lint)
* [TextMate 2](https://github.com/jjuliano/SCSS-Lint.tmbundle)
* [Brackets](https://github.com/petetnt/brackets-sass-lint)
* [IntelliJ IDEA, RubyMine, WebStorm, PhpStorm, PyCharm](https://github.com/idok/sass-lint-plugin)
* [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=glen-84.sass-lint)
* [Vim](https://github.com/gcorne/vim-sass-lint)

## 4. Best Practices

- Comments are almost always a good thing
- Document your components and helpers, try to approach this as if you are writing an open source framework
- Don't worry about being too terse. Modern compression makes this a moot point.
- No body classes
- Don't worry about long class values `global-header-nav-item bg-blue text-white` is fine. It's descriptive, and has no impact upon performance.
- 

## 5. Performance

> Don't worry about selector performance

For years we've been taught to use efficient selectors, like `id`s. However practically speaking, using specific `class` selectors for components has negligible performance impact in modern browsers.

Unless you are going into the realms of complex javascript powered single page apps with thousands of DOM nodes, readability and maintainability are simply more important.

The only area of note is when dealing with hardware accelerated CSS3 effects. This would need a guide on it's own, so is outside the scope of this document. In general, if working with animations or transforms pay very close attention to performance.

## 6. Cross Browser

> Never write -browser- prefixes by hand

We universally use [autoprefixer](https://github.com/postSCSS/autoprefixer) in our projects, so we don't have to worry about writing browser prefixed declarations, like `-webkit-feature`.

## 7. Further Reading

### Thanks

This guide builds on some basic concepts from the now defunct [Trello CSS guide](https://gist.github.com/bobbygrace/9e961e8982f42eb91b80). Our approach is also heavily inspired by the internals of [Bootstrap 4](https://github.com/twbs/bootstrap/tree/v4-dev).

### Contributing

This is a work in progress. It is far from finished or perfect!

1. Fork it (<https://github.com/Purepoint/scss-guide/fork>)
2. Create your change branch (`git checkout -b change/fooBar`)
3. Commit your changes (`git commit -am 'Add some fooBar'`)
4. Push to the branch (`git push origin change/fooBar`)
5. Create a new Pull Request

## 8. Licence

MIT License

Copyright (c) 2017 Purepoint

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

## 9. About Purepoint

<img src="https://user-images.githubusercontent.com/12545/27732807-fe3defbc-5d8a-11e7-861c-6e6671a495f1.png" width="400">

SCSS Guide is a [Purepoint](https://purepoint.io) Open Source project.

We deliver innovative software that solves big business problems. Maybe you'd be interested in working with us?
