---
title: Composing Components
section: Cookbook
cookbook-section: Helpers and Components
layout: cookbook-recipe
---
<span class="recipe-label">Recipe:</span>
## {{ page.title }}
-----
Components organize applications best when the same input will
always result in the same behavior. This approach is often called
"data down, actions up" - components receive data and emit actions,
but themselves never change.

The tools to compose components in Ember are the `yield` method and
block params. Here is a preview of what this guide goes over:

`app/templates/application.hbs`

{% highlight html %}
{% raw %}
{{#user-list users=activeUsers sortBy="name" as |user|}}
  {{user-card user=user}}
{{/user-list}}
{% endraw %}
{% endhighlight %}

First, a review of how to use components.

### Component Blocks

Components can be used in two forms, just like regular HTML elements.

**Inline Form**

`app/templates/application.hbs`

{% highlight html %}
{% raw %}
{{user-list users=activeUsers}}
{% endraw %}
{% endhighlight %}

This is the basic blockless form of a component, which doesn't give the
consumer the ability to insert their own content into the given component.
It's even possible that a component can only be used in this form.
Make sure to consult a component's documentation for its available uses
and forms.

**Block Form**

`app/templates/application.hbs`

{% highlight html %}
{% raw %}
{{#user-list users=activeUsers}}
  {{!-- custom template here --}}
{{/user-list}}
{% endraw %}
{% endhighlight %}

To compose components, we must use the block form, but we must also
be able to distinguish from within our component which form is being used.
This can be done with the `hasBlock` property from within our component.

`app/templates/components/user-list.hbs`

{% highlight html %}
{% raw %}
{{#if hasBlock}}
  {{yield}}
{{else}}
  <p>No Block Specified</p>
{{/if}}
{% endraw %}
{% endhighlight %}

{% raw %}
Once we know that the component is being used in block form, we can yield
whatever the user has placed inside the block wherever we want via the `{{yield}}` helper.

This `{{yield}}` helper can be used in the component's template definition multiple times, allowing us to create a component that works like a list, where we output a headline and yielded body for each post.
{% endraw %}

`app/templates/components/post-list.hbs`

{% highlight html %}
{% raw %}
{{#each posts as |post|}}
  <h3>{{post.title}}</h3>
  <p>{{yield}}</p>
{{/each}}
{% endraw %}
{% endhighlight %}

Which can be used like so:

`app/templates/posts.hbs`

{% highlight html %}
{% raw %}
{{#post-list posts=headlinePosts}}
  Greatest post ever!
{{/post-list}}
{% endraw %}
{% endhighlight %}

And will result in the following HTML:

{% highlight html %}
<div id="ember123" class="ember-view">
  <h3>Tomster goes to town</h3>
  <p>Greatest post ever!</p>
  <h3>Tomster on vacation</h3>
  <p>Greatest post ever!</p>
</div>
{% endhighlight %}

{% raw %}
But what use is it to just output the same thing over and over? Don't we want to customize our posts, and display the right content? Sure we do. Lets explore the `{{yield}}` helper a bit.
{% endraw %}

### Data Down

{% raw %}
To accomplish composability beyond simple templates, we need to pass data to those templates. This can be done with the `{{yield}}` helper that was introduced above.

The `{{yield}}` defines where the content we defined inside our component block will yield in the component's layout,
as we saw in the previous section. Apart from that, the yield helper also allows us to send data down by yielding params back to the scope the component was invoked in.
{% endraw %}

{% highlight html %}
{% raw %}
{{yield}}
{{yield "post"}}
{{yield item}}
{{yield this "footer"}}
{% endraw %}
{% endhighlight %}

{% raw %}
By default yield does not send any data, but you can provide an arbitrary number of parameters. Also, the yielded values are ordered sequentially by how they are defined, so you access them in the same order as the component exposes them.
Sending data down allows the consumer of our component to utilize that data.
For the consumer to get access to the data, she needs to know what data a component exposes, this is where documentation is so crucial, and from there the yielded data can be accessed with the `as` operator. Let's take the following template as an example:
{% endraw %}

`app/templates/components/user-list.hbs`

{% highlight html %}
{% raw %}
{{#each users as |user|}}
  {{yield user}}
{{/each}}
{% endraw %}
{% endhighlight %}

{% raw %}
This `{{user-list}}` component will yield as many times as there are users.
We can consume it in the following way:
{% endraw %}

`app/templates/application.hbs`

{% highlight html %}
{% raw %}
{{#user-list users=users as |user|}}
  {{user-card user=user}}
{{/user-list}}
{% endraw %}
{% endhighlight %}

{% raw %}
Now `{{user-card}}` has access to the current user, which changes as the `{{user-list}}` iterates over each user
since the `{{yield user}}` is inside the `{{each}}` block. Although this is nice, what if we used our component
without a block, i.e. `{{user-list users=users}}`? This would make the component pretty useless since nothing is yielding,
but the users are still being iterated. Let's mix in the `hasBlock` attribute and see if we can make it more useful.
{% endraw %}

`app/templates/components/user-list.hbs`

{% highlight html %}
{% raw %}
{{#if hasBlock}}
  {{#each users as |user|}}
    {{yield user}}
  {{/each}}
{{else}}
  {{#each users as |user|}}
    <section class="user-info">
      <h3>{{user.name}}</h3>
      <p>{{user.bio}}</p>
    </section>
  {{/each}}
{{/if}}
{% endraw %}
{% endhighlight %}

This makes our component more useful due to it having sane defaults, but it also allows us to override those defaults. By using the `{{yield}}` helper we can pass
down our params for the consumer to utilize. This makes the concept of passing
data down very useful.

#### {% raw %}`{{component}}`{% endraw %} Helper

To understand the `{{component}}` helper, we will add the following new features to the `{{user-list}}` component:

* For each type of user, we want to use different layout.
* The list can also be toggled between basic and detailed mode.

Let's add a new type of user, a superuser, which will have different behavior in our application. How would the consumer use this new data to show a different
UI for each type of user? This is where the `{{component}}` helper comes into play. This helper allows us to render a component chosen at runtime.

Since we have two user types, 'public' and 'superuser', and we want to render
a component for each type plus for the simple and detailed modes. We want to have the following component names:

* `basic-card-public`
* `basic-card-superuser`
* `detailed-card-public`
* `detailed-card-superuser`

We can create these names by using the {% raw %}`{{concat}}`{% endraw %} helper in nested form.

{% highlight html %}
{% raw %}
{{component (concat "basic-card-" user.type)}}
{% endraw %}
{% endhighlight %}

Now that our component names are valid due to the inclusion of a dash in their
names, we can put together a full template with the two modes.

`app/templates/application.hbs`

{% highlight html %}
{% raw %}
{{#user-list users=users as |user basicMode|}}
  {{#if basicMode}}
    {{component (concat "basic-card-" user.type) user=user}}
  {{else}}
    {{component (concat "detailed-card-" user.type) user=user}}
  {{/if}}
{{/user-list}}
{% endraw %}
{% endhighlight %}

Now we have {% raw %}`{{basic-card-superuser}}` and `{{detailed-card-public}}`{% endraw %} rendering in their respective places
based on the `user.type` equaling 'superuser' and 'public' respectively. The use of the 'concat' helper in nested form allowed us to accomplish that. Nested helpers are evaluated before the parent helper
and then that helper can use the value returned by the nested helper. In this case it's just a concatenation of two values. You can
also experiment with other helpers in nested form, like the `if` helper.

{% raw %}
Apart from the use of `{{component}}`, we also have the second yielded value for our basic/detailed mode feature,
which we've added to our yield, i.e. `{{yield user basic}}`.
{% endraw %}

The consumer can decide how to name the yielded values. We could have named our yields like so:

`app/templates/application.hbs`

{% highlight html %}
{% raw %}
{{#user-list users=users as |userModel isBasic|}}
  {{! something creative here }}
{{/user-list}}
{% endraw %}
{% endhighlight %}

### Actions Up

Now that we know how to send data down, we probably want to manipulate that data via some user interaction,
like changing a user's avatar. We can accomplish this by using actions.

{% raw %}
We'll be looking at a `{{user-profile}}` component that implements a save action. By yielding the action as a parameter, we allow the consumer to use this component without having to know how to save, but still be able to trigger a save.
{% endraw %}

Here's our component's definition:

`app/components/user-profile.js`

```js
export default Ember.Component.extend({
  actions: {
    saveUser() {
      var user = this.get('user');

      user.save()
        .then(() => {
          this.set('saved', true);
        })
        .catch(error => {
          this.set('saveError', error);
        });
    }
  }
});
```

`app/templates/components/user-profile.hbs`

{% highlight html %}
{% raw %}
{{! most likely we have some markup here }}
{{yield profile (action "saveUser")}}
{% endraw %}
{% endhighlight %}

So now the consumer will have access to our data and the
action that we have defined. Lets see how this component could be consumed
in block form.

`app/templates/user/profile.hbs`

{% highlight html %}
{% raw %}
{{#user-profile user=user as |profile saveUser|}}
  {{user-avatar url=profile.imageUrl onchange=saveUser}}
{{/user-profile}}
{% endraw %}
{% endhighlight %}

{% raw %}
As you can see from this example, the consumer was able to hook into the
`{{user-profile}}` component's `saveUser` action, which we passed down
with the `(action "saveUser")` nested helper.  This helper reads a property off the actions hash of the current context, then creates a closure over that function and any arguments you pass.
{% endraw %}

We could also leverage this format to place actions on native HTML elements
like an input button:

`app/templates/user/profile.hbs`

{% highlight html %}
{% raw %}
{{#user-profile user=user as |profile saveUser|}}
  <button type="button" onclick={{action saveUser}}>Save</button>
{{/user-profile}}
{% endraw %}
{% endhighlight %}

This is very useful since we are reusing the existing event hooks that
are provided on these elements. This makes components much more reusable due to how we can compose
small components that do one thing well.

### Wrapping Up

Composing components is all about isolating functionality into reusable
chunks that are easy to reason about and easy to combine together so that they work well together.
This also has the added benefit of easier testing since we try to stay away from
monolithic components that do everything, and can isolate small parts of functionality.
