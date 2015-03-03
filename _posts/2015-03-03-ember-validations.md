---
layout: post
title: Dynamic validations via Ember-Validations
---

[Ember-Validations](https://github.com/dockyard/ember-validations) is a great library to allow for validations.  A small problem with it is that it expects the validations to be static.  This is usually not a problem if you are making some standard web forms where the content is always the same.  However in some of my internal projects the content of the form can be quite dynamic.  To resolve this issue, we store the validations to run against something inside our database and load it via Ember Data.

If you look at the internals of the [validator mixin](https://github.com/dockyard/ember-validations/blob/master/addon/mixin.js) you will see that the everything for validation is only set up during the init method for the mixin.  So how do you solve the problem of having dynamic validations?  Once you know what validations are required you can create the object that will be validated.

```javascript
var dynamicValidations = this.get('validations'); //can be from anywhere (model, property, controller)
var container = this.get('container'); //needed by Ember Validations to resolve validators
var validations = {obj: dynamicValidations};
var properties = {validations: validations, obj: null, container: container};
var validateableObject = Ember.Object.createWithMixins(EmberValidations.Mixin, properties);
```

You now have a validateable object that you can use/reference from your templates.  The bad part is that you will have to manually set the property you are validating against on the validatable object whenever there is a change.
