---
layout: default
title: "BEM CSS conventions"
date: 2017-11-01
---

# Overview
Typically CSS is treated as a second class citizen, with little effort on organisation, standardisation or maintainability. By applying a very simple set of conventions this can be improved.
 
There are a few naming schemes to choose from OOCSS, SMACSS, SUITCSS, Atomic, but my favourite so far is BEM.
 
# BEM
There are 3 parts to BEM - Blocks, Elements and Modifiers (hence the name).
 
**Block** – Standalone component that is meaningful on its own, e.g. a status badge, such as the Bootstrap badge.

**Element** – A part of a block component that has no standalone meaning and is semantically tied to its block, e.g. an input field in a form.

**Modifier** – A flag on a block or element. Used to change appearance or behaviour, e.g. Bootstrap button sizing via `btn-sm`.
 
Below is a simple example of the convention for a made-up badge component:
 
Example badges in HTML:

```html
<span class="badge badge--info">
 <span class="badge__count">3</span>
 <span class="badge__text">new messages</span>
</span>
<span class="badge badge--danger">
 <span class="badge__count">7</span>
 <span class="badge__text">deadly sins</span>
</span>
<span class="badge">
 <span class="badge__count">99</span>
 <span class="badge__text">red balloons</span>
</span>
``` 
 
Example CSS:

```css
  /*Block*/
  .badge {
    border: 1px solid #CCCCCC;
    border-radius: 0.3rem;
    padding: 0 0.5rem;
    margin: 0.1rem;
  }

  /*Elements*/
  .badge__text {
    margin-left: 0.2rem;
  }
  .badge__count {…}
 
  /*Modifiers*/
  .badge--danger {
    background-color: #D9534F;
    border-color: #D9534F;
    color: #FFFFFF;
  }
  .badge--info {…}
```

Giving something like this:

![BEM badges](https://raw.githubusercontent.com/cowinr/moostey/master/images/BEM-badges.png)
 
As you can see `badge` is the class name of the Block and a pseudo-namespace for the element and modifier classes.
 
An Element class is prefixed with `<namespace>__` E. g. `badge__text`.
 
A Modifier class is prefixed with `<namespace>--` E.g. `badge--danger`.
 
A scheme such as this is better than none at least!
 
Perhaps the strongest case for using any CSS convention based on namespacing is that it unambiguously defines which CSS belongs to a piece of user interface and so using it gives answers to questions "Can I remove this piece of CSS?" and "What happens and which interface parts will be affected if I change this piece of CSS?".

## BEM Resources
* [BEM naming rules](http://getbem.com/naming/)
* [FAQ](http://getbem.com/faq/)

# Alternative – Bootstrap-esque BEM
 
Whilst Bootstrap doesn’t follow BEM, it comes close.
 
Blocks are named as per BEM, e.g. `btn` for a button.
 
Elements are prefixed with `<namespace>-` instead of `<namespace>__`. E.g. `card-title` instead of `card__title`.
 
However, Modifiers are treated the same as Elements, e.g. `btn-danger` instead of `btn--danger`.

Personally I prefer BEM...
