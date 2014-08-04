---
layout: post
title: Why are we creating unusable components?
---

In several spots in my [Ember.js](http://emberjs.com) applications I have ran into a need for using a select box.  Since we no longer live in the 90's I rather have my users have a rich experience and the built-in [input-helpers](http://emberjs.com/guides/templates/input-helpers/) were not cutting it.  

Having ran across the [chosen](https://harvesthq.github.io/chosen/) select box in the past I wanted a similar experience for my users. This lead me to try and integrate Addepar's [Ember Widgets](https://addepar.github.io/#/ember-widgets/overview), but yet it just felt too bulky and I hate the idea that they weren't completely Ember components.  So I attempted to roll my own, and have it so every piece of the select box would be a Ember.js component.  

In my attempts to make the component I have an epiphany around customization of components. We should be able to customize without having to mess too much with component.  We should never have to extend the component to make the changes required.  So let us make our components allow the users define how the component works outside of the core functionality like in the following example.

<script src="https://gist.github.com/KnownSubset/0acb02742b475987d603.js"></script> 

A component like this allows for defining custom behavior for hover, drag, popover, clicks, and anything else since the display elements for the content would be their own components. With the ability to use components in this manner we can create a set of components with pre-defined core functionality like a grid, table, tree, and many more that anyone can use within their projects while still adding whatever additional functionality is required.Luckily this is all be possible by making use of [layout property](http://emberjs.com/api/classes/Ember.Component.html#property_layout). We can do this by setting the layout during the init of the component.  

<script src="https://gist.github.com/KnownSubset/e0792572dcb01ddecfb5.js"></script>

Once we have it working, we can do something like this prototype which showcases multiple stylings of the same component.

<a class="jsbin-embed" href="http://emberjs.jsbin.com/qazamo/3/embed?output">JS Bin Example</a><script src="http://static.jsbin.com/js/embed.js"></script>

We should abadon or extend any component that does not support custom usage in favor of ones that we can use beyond a single project.  I would like to see a community effort grow into producing a library of components and examples in the fashion of [Bootstrap for Ember](https://ember-addons.github.io/bootstrap-for-ember/).  
