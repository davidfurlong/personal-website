---
template: post
title: Creating unique stable selectors from arbitrary DOM elements
slug: creating-unique-stable-selectors-dom-elements
draft: false
date: 2021-03-05T11:31:47.705Z
description: >-
  A little exploration into the problem of trying to create selectors for DOM
  elements that uniquely select that element across page reloads and try to
  future proof them
category: engineering
tags:
  - engineering
---
While building *erase distractions* (https://chrome.google.com/webstore/detail/erase-distractions/aojacabljkpaglimemogalnbmillgplm?hl=en) I wanted to be able to select any element on the page and create a DOM selector that uniquely selects this DOM element across page reloads.

It can't be that hard, right? 

Just get the element, loop over its parentNode's and use these elements tagnames, classnames, ids to construct a selector like `html > body > div > div.container > span#nav > div`. This is basically an `XPath` if you know what that is.

Well turns out that doesn't work, no.

**A) Many modern sites have non stable classnames. The same DOM element often has a different class name across a reload. This is true of most popular sites (twitter, ...) This appears to be because of XYZ**

So we can a snapshot a reload 

**B) Sometimes the above selector is not unique, particularly if the DOM element has siblings**

So we could use nth-child() selectors or sibling selectors?

Unfortunately due to modern frameworks like React, element are often added to the DOM in a different order, for instance while data is loading some element aren't added. This means our child selector selects the wrong element temporarily.

---

We can optimize the selectors by chopping off anything before a unique selector like *tagname* *body* or when an *id* attribute is defined, right? 

You should know that by now the answer is its not that simple. I tried this and it didn't always work. It didn't work because although ids are unique in the spec, some sites just ignore this. For instance [Youtube.com](http://youtube.com) has non unique ids on their home page and not within shadow doms. I have no idea why. I guess we can't rely on these. Even if sites had unique ids, ids can be non unique on a page by using a shadow dom. We could select to the shadow dom then use the id uniquely within it, but that probably isnt worth the hassle

---

Okay perhaps we can generate all the possible selectors, sort them from most specific to least specific and try them all until one works? This works, but unfortunately this will often create a lot of selectors. Each additional attribute or child or tag doubles the number of selectors. Ideally we would reload the page in a bunch of variations, including browser widths, incognito (to clear any cookies set by A/B tests etc) and then compare our selector graphs to find a selector that matches in each case that is both unique and stable. 

It's unclear whether this should be the most specific unique selector or the least one, or somewhere in between? 
It depends on how the site uses classnames and what they are likely to change. Some sites use classnames for individual css rules, by using a css tool like tailwind.css. Any single change to the style of the element or its parents would change the selector's css classnames, breaking it. So perhaps we shouldn't rely on classnames at all? Or perhaps we should only rely on classnames that are unique - suggesting that this classname is coupled to this DOM element in the code rather than a utility class name? 

If we simply ignore class names then perhaps we risk our selector breaking by becoming non unique or selecting a different element without it being clear that has happened. For instance a selector like `body > div:nth-child(2) > div:nth-child(4) > div:nth-child(1)` is likely to select the wrong element if the site changes the code to insert a `div` in many places, or if they dynamically insert `div`s based on some sort of logic.

Perhaps a step back would help at this point. If we can't programmatically create reliable unique stable selectors, but we can reliably point to the same element on the page each time then we might be missing something. We can reliably point to the same element (perhaps the sidebar, or a button) because of its visual placement on the page (perhaps its on the top right side), and perhaps because of its contents. Could we make a stable selector that uses these human properties to identify elements in a stable way? 

Well this is also tricky - what should we consider essential properties of something? Is a button's location stable? If it's moved we would still be able to point to it. The content of the button can also be changed without it becoming a "different" button. In automated tests in cypress it's recommended you don't depend tests on selectors using text contents as these are prone to changes and fixing tests because copy changed is annoying. Perhaps instead the behaviour is essential? The fact that this button links to this page or does this action could be used. The problem here is that sometimes this behaviour isn't unique on a page, or we're trying to select an element which doesn't trigger some sort of event.

When writing cypress tests sometimes developers add `data-testid` properties to DOM elements in order to uniquely select them for tests. This adds a bit of verbosity to the code but guarantees a selector. Unfortunately this is only possible if you have control over the source code. 

Unfortunately we can't create reliable, stable unique DOM element selectors as a third party as the structure or contents can always change in slight ways that are hard to automatically assimilate. 

---

What other kinds of companies have this problem?

- Browser automation libraries (RPA) for example Browserflow
- Automated testing libraries like cypress
- Ad blocking extensions
- Augmentation extensions (refined github, ...)
