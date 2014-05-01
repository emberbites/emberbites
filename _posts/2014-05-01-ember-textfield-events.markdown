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

This app is just asking for your name and it promises to keep a safe distance at all times. Give it your name, and then click outside of the field. The page will change to this:

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


