---
layout: post
title:  "#3 - Ember and Select2"
permalink: 3-ember-and-select2
date: 2014-05-02
---

[The application code for this bite is available on GitHub.](https://github.com/emberbites/3-ember-and-select2)

In this bite, we'll be building upon our [previous example](http://emberbites.com/2014/05/01/ember-textfield-events.html). In that example, we had an `Ember.Textfield` that had a `change` event which displayed the name that was entered. With this example, we'll extend that to be a select2 field from which we can pick people's names and then it will display information about whoever we choose.

![Info box](/images/2014-05-02/info_box.png)

These are all fake people in the boxes and any similarities to real people is purely coincidental.

You can setup the app by cloning it from GitHub and then running these commands:

```
bundle
bundle exec rake db:setup
```

When we navigate to the root of this application, Ember handles the request with it's `IndexRoute` which renders the template from `app/assets/javascripts/templates/index.hbs`:

```html
{%raw%}
<h1>Who do you want to know about?</h1>

<p>
  {{view EmberStore.NameField person=person}}
</p>

{{#if person}}
  <h3>
    Here's some info about {{person.first_name}} {{person.last_name}}:
  </h3>

  <dl>
    <dt>Age:</dt>
    <dd>{{person.age}}</dd>
    <dt>Job Title:</dt>
    <dd>{{person.job_title}}</dd>
  </dl>
{{/if}}
{%endraw%}
```

This template renders the familiar `EmberStore.NameField` component from Bite #2, but instead of calling an action, it links the `person` property on the component to this template's property of the same name by doing `person=person`.

The idea for this template is the same as the previous example: the `showInfo` action should set the `person` property, and then once that's set we should be able to display some content in the page.

Let's take a look at the `EmberStore.NameField` component, located at `app/assets/templates/components/name_field.js.coffee`:

```coffee
EmberStore.NameField = Em.TextField.extend
  didInsertElement: ->
    view = this
    $("##{this.elementId}").select2
      width: '20em'
      ajax: 
        url: '/api/people'
        data: (name) ->
          q:
            first_name_or_last_name_cont: name
        results: (results) ->
          { results: results.people }
      formatResult: (person) ->
        "#{person.first_name} #{person.last_name}"
      formatSelection: (person) ->
        view.set('person', person)
        @formatResult(person)
```

Inside this component, we're using `didInsertElement`, which is callled as soon as the element has been inserted into the page by Ember.

The `didInsertElement` method finds the current element's ID and calls `select2` on it, setting that up to make an AJAX call to an API endpoint at `/api/people`. The data returned by this endpoint is restricted by the `data` option we've passed here, which tells the API we only want people who have first names or last names that contain the text we've entered into the text field. The `results` function just formats the results in a way that Select2 can understand. The `formatResult` function tells Select2 what to display for our results in the dropdown list, and `formatSelection` tells Select2 how to display the item once selected. 

We're causing this `formatSelection` function to have a second purpose, which is to set the `person` property once that selection has been made. This is in place of the `change` event, which would only have the `id` of the person we've selected, rather than the whole `person` object that's available in `formatSelection`. While the `change` event was fine when all we wanted was the value of the field, now we want more, and so we do it differently. 

Once the property is set, the check for the `person` property will now evaluate to `true`, and so the content will be displayed:

```html
{% raw %}
{{#if person}}
  <h3>
    Here's some info about {{person.first_name}} {{person.last_name}}:
  </h3>

  <dl>
    <dt>Age:</dt>
    <dd>{{person.age}}</dd>
    <dt>Job Title:</dt>
    <dd>{{person.job_title}}</dd>
  </dl>
{{/if}}
{% endraw %}
```

