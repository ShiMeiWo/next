---
title: Migration Guide
layout: detail
description: Migration guide from Riot.js 2 and 3
---

# 🚧🚧🚧 WIP 🚧🚧🚧

## Introduction

Riot.js 4 is a complete rewrite, I will provide a detailed explanation about it on [Medium soon so please stay in touch](https://medium.com/@gianluca.guarini){:target="_blank"}.

Migrating older applications written in Riot.js 3 is not recommended because the legacy Riot.js versions will still get security patches and they are enough stable.

You can use this guide to learn how to write components for Riot.js 4 coming from Riot.js 3 and 2.

## Component syntax

The components syntax was updated to match the modern javascript standards avoiding any possible ambiguity.
Less magic means more clarity and interoperability, Riot.js 4 components are designed to be completely future proof!

### The script tag

In the previous Riot.js versions you could just extend to your component instance via `this` keyword in your `<script>` tags. This syntax sugar was removed because the components public API can be clearer defined using standard javascript ES2018 syntax:

**old**

```html
<my-component>
  <p onclick={onClick}>{message}</p>

  <!-- optional <script> tag -->

  onClick() {
    this.message = 'hello'
  }
</my-component>
```

**new**

```html
<my-component>
  <p onclick={onClick}>{message}</p>

  <!-- mandatory <script> tag -->

  <script>
    export default {
      onClick() {
        this.message = 'hello'
        this.update()
      }
    }
  </script>
</my-component>
```

In this way your editor, and other compilers like `typescript` will not get confused bye your component javascript logic and can be used along the way without any special concerns.

It's worth to mention that this change was driven by the new [Riot.js philosophy]({{ '/'|prepend:site.baseurl }}#conclusion):

> ...In the face of ambiguity, refuse the temptation to guess.<br/>
There should be one– and preferably only one –obvious way to do it.<br/>
Although that way may not be obvious at first unless you’re Dutch...

<aside class="note note--info">
Notice how the use of the <code>&#x3C;script&#x3E;</code> tag becomes mandatory to split your components templates from their logic.
</aside>


### Template shortcuts

The template shortcuts were completely removed in favor of pure and more explicit javascript expressions. Let's see how it's simple to achieve the same results in a cleaner way in Riot.js 4.

**old**

```html
<my-component>
  <p class={ active: isActive, disabled: isDisabled }>hello</p>
</my-component>
```

**new**

```js
/**
 * Convert object class constructs into strings
 * @param   {Object} classes - class list as object
 * @returns {string} return only the classes having a truthy value
 */
function classNames(classes) {
  return Object.entries(classes).reduce((acc, item) => {
    const [key, value] = item

    if (value) return [...acc, key]

    return acc
  }, []).join(' ')
}

// install the classNames plugin
riot.install(function(component) {
  // add the classNames helper to all the riot components
  component.classNames = classNames

  return component
})
```

```html
<my-component>
  <p class={
    classNames({
      active: isActive,
      disabled: isDisabled
    })
  }>hello</p>
</my-component>
```

Even better, you can use the `classNames` directly in your components logic keeping your templates clean avoiding the use of `riot.install` for example:

```html
<my-component>
  <p class={getParagraphClasses()}>hello</p>

  <script>
    import classNames from 'classnames'

    export default {
      getParagraphClasses() {
        return classNames({
          active: this.isActive,
          disabled: this.isDisabled
        })
      }
    }
  </script>
</my-component>
```

and don't forget to remember that:

> ...Explicit is better than implicit...