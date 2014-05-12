---
layout: post
title:  "#4 - Ember View Heirarchy"
permalink: 4-ember-view-heirarchy
date: 2014-05-14
---

[The application code for this bite is available on GitHub.](https://github.com/emberbites/4-ember-view-heirarchy)

How Ember decides what to render for a route can sometimes be confusing, especially if you've come from frameworks such as Rails where `index` has meant the template for displaying a collection of a resource, rather than how Ember defines it, which is very different.

Here's two images that might help you wrap your head around how Ember's views work:

![View Heirarchy](/images/2014-05-12/view-heirarchy.png)

In this first example, we're visiting a route that's something such as `/orders/1`. Ember knows by default to render the application view (if it's available), and that's what it does. This gets us the "Ember Store" title of the page. Next, Ember will render the `order.hbs` template, to show us the "Order 1" title underneath the "Ember Store" title. Finally, it will render the "order/index.hbs" template, giving us the "Add Item" link.


![View Heirarchy 2](/images/2014-05-12/view-heirarchy-2.png)

If we click that "Add Item" link, then we get taken to the page shown above, at the route `orders/1/items/new`. Instead of rendering `order/index.hbs`, Ember is now rendering `items/new.hbs`, which is exactly what we want it to do because we're now at a different route. The link to add an item has now been replaced with a "TODO: Add New Item Form" message which is from the `items/new.hbs` template. 

This all works because Ember automatically generates an `index` route for us for our resources. The routes for this Ember app are defined like this:

```coffee
EmberStore.Router.map ->
  this.resource 'order', path: '/orders/:order_id', -> 
    this.resource 'items', ->
      this.route 'new'
```

When we go to a path that matches `/orders/:order_id`, Ember will assume that we want to render the order template, which is the middle template in both of the above images. It assumes this based on the resource's name. The `order/index.hbs` template is being rendered because Ember has automatically generated an `OrderIndex` route and has routed to that. That route's default behaviour is to render `order/index.hbs`, and it does that dutifully.

When we make a request to `/orders/:order_id/items/new`, Ember will still assume that we want to render the order template. It's not going to render `order/index.hbs` because we're routing to a sub-resource of order now. Therefore it will render `items/new.hbs`.

To jump between the index and the new item route, there is this code in the application's templates:

**app/assets/javascripts/templates/order/index.hbs**

```html
{%raw%}
<p>{{#link-to 'items.new'}}Add Item{{/link-to}}</p>
{%endraw%}
```

**app/assets/javascripts/templates/items/new.hbs**

```html
{%raw%}
<p>{{#link-to 'order.index'}}Back to order{{/link-to}}</p>
{%endraw%}
```

Ember is smart enough to know here that, in the first instance, we want to go to the "/items/new" route for this order. It's also smart enough, in the second instance, to know which order want to go back to.





