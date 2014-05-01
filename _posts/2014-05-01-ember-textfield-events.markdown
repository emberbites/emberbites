---
layout: post
title:  "#2 - Ember.TextField events"
date:   2014-05-01 08:00:00
---

[The application code for this bite is available on GitHub.](https://github.com/emberbites/2-textfield-events)

When using jQuery, you might've wanted to bind to the change event of a text field and cause something to happen from that, like putting the value of the field in some other element on the page. The code used might have looked like this:

```js
$('#some_element').change ->
  $('#some_other_element').html(this.value)
```

But how do you do that in Ember? Well, Ember provides us with some helpers that can assist us, like `Em.TextField`. `Em.TextField` extends `Em.Component`, which are reusable custom elements  within our application. This little helper can provide us with useful tasks such as data bindings, amongst other things.

Today we're going to be extending `Em.TextField` and building our own component from that. With this component, we'll trigger an on change event that will update the page with the content from the text field.

If you start up the example app for this bite and navigate to the homepage, you'll see this slightly stalker-esque prompt:

![What is your name?](/images/2014-05-01/name_prompt.png)

This app is just asking for your name and it promises to keep a safe distance at all times. Give it your name, and then hit enter. The page will change to this:

![Revelation](/images/2014-05-01/name_revelation.png)

Let's run through what's happening here. The Ember application starts out by routing to the IndexRoute, which is given to us for free by Ember. This route renders the template from `app/assets/javascripts/templates/index.hbs`, which contains this content:

```html
{%raw%}
<h1>What is your name?</h1>

<p>
  Please tell me your name: {{view EmberStore.NameField action='discoverName'}}
</p>

{{#if name}}
<h3>
  Oh, <em>you're</em> {{name}}!
</h3>
{{/if}}
{%endraw%}
```

In this template, we're using Ember's `view` helper to render a custom view of ours called `EmberStore.NameField`. I'll get to that in a moment. We're passing it the action key with the value of `discoverName`. Right after this, we're checking for the `name` property and if that exists then we'll show the "Oh, you're \{\{name\}\}!" message. Due to the fact that the `name` property is not set on the controller by default, the message is hidden.

The code that makes the message display starts off in the `EmberStore.NameField` view, which is located at `app/javascripts/templates/components/name_field.js.coffee`:

```coffee
EmberStore.NameField = Em.TextField.extend
  change: (e) ->
    @sendAction('action', @value)
```

This code defines a `change` function and calls `sendAction`. We'll see what that does in a moment, but first we must learn how this `change` function gets called. 

The `Em.TextField` component is defined in [this file](https://github.com/emberjs/ember.js/blob/v1.5.1/packages/ember-handlebars/lib/controls/text_field.js) and [down on line 30](https://github.com/emberjs/ember.js/blob/v1.5.1/packages/ember-handlebars/lib/controls/text_field.js#L30) we can clearly see that it extends `TextSupport`, which is defined in [this other file](https://github.com/emberjs/ember.js/blob/v1.5.1/packages/ember-handlebars/lib/controls/text_support.js). `Ember.TextSupport` is a mixin that is used for both `Em.TextField` and `Em.TextArea` components. In that mixin, we can see [this code](https://github.com/emberjs/ember.js/blob/v1.5.1/packages/ember-handlebars/lib/controls/text_support.js#L29-L37) which calls out to some jQuery functions to bind to certain events. These events are triggered just as per normal in jQuery-land, eventually ending up at our `change` function.

The `change` function calls the `sendAction` function, passing it the string
`'action'` and the current value of the field. This is where it can get
slightly confusing. The string `'action'` here is actually telling
`sendAction` the property to access from the view, not actually telling it to
send the action called `'action'`! We defined this property earlier with this code in our template:

```html
{%raw%}
{{view EmberStore.NameField action='discoverName'}}
{%endraw%}
```

Because components can be re-usable, Ember allows for the ability to specify different actions in different contexts. For instance, you might want to use this component to discover the first name of a person in one instance, and then their last name in another instance:

```html
{%raw%}
{{view EmberStore.NameField action='discoverFirstName'}}
{{view EmberStore.NameField action='discoverLastName'}}
{%endraw%}
```

What this code will do is run the `discoverFirstName` action with the code in the component for the first `NameField`, and it will run `discoverLastName` using the exact same component for the second `NameField`. All of this happens without having to have any code in the component itself which tells it to act in one way or the other depending on the circumstance. This is the beauty of components.

Back to our existing code:

```coffee
EmberStore.NameField = Em.TextField.extend
  change: (e) ->
    @sendAction('action', @value)
```

```html
{%raw%}
{{view EmberStore.NameField action='discoverName'}}
{%endraw%}
```

With these two pieces of code combined, the `discoverName` action gets run within the current context, which just so happens to be `IndexController`, because we're currently at the index route. The code for this controller is within `app/assets/javascripts/controllers/index.js.coffee`:

```coffee
EmberStore.IndexController = Ember.Controller.extend
  actions:
    discoverName: (name) ->
      this.set('name', name)
```

All this action has to do is take the argument it's given and set a property called `name` on the controller. Once that property's set, then the `\{\{if name\}\}` within our template evaluates to `true`, therefore Ember will display our content:

![Revelation](/images/2014-05-01/name_revelation.png)

That's all there is to change events for `Ember.TextField` components.