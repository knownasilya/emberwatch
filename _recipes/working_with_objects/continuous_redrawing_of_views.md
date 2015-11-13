---
title: Continuous Redrawing of Views
section: Cookbook
cookbook-section: Working with Objects
layout: cookbook-recipe
author: Bill Heaton
email: pixelhandler@gmail.com
github: pixelhandler
website: pixelhandler.com
---
<span class="recipe-label">Recipe:</span>
## {{ page.title }}
-----
### Problem
You'd like to redraw your views every few seconds/minutes e.g. to update
relative timestamps (like on twitter.com).

### Solution
Have a `clock` service with a `pulse` attribute in your application which
increments using a timed interval. You want to let UI bound to the `pulse`
attribute update every time the value increments.

The `clock` service can be used to create new instances for binding to new views
generated within the application, like a list of comments.

### Discussion

<a class="jsbin-embed" href="http://jsbin.com/jihivevafo/1/edit?output">
Cookbook: Continuous Redrawing of Views
</a><script src="http://static.jsbin.com/js/embed.js"></script>

#### Clock Service

This `ClockService` is an `Ember.Service` which is a singleton and can be
used throughout your application.

During initialization the `tick` method is called which uses `Ember.run.later`
with a time of 250 milliseconds as the interval. A property is set at the end
of the interval. Since the `tick` method observes the incremented property
another interval is triggered each time the property increases.

`app/services/clock.js`

```javascript
import Ember from 'ember';

const { on, run, computed, observer } = Ember;

export default Ember.Service.extend({
  pulse: computed.oneWay('_seconds').readOnly(),
  _seconds: 0
  
  tick: on('init', observer('_seconds', function () {
    run.later(() => {
      var seconds = this.get('_seconds');
      
      if (typeof seconds === 'number') {
        this.set('_seconds', seconds + (1/4));
      }
    }, 250);
  }))
});
```

#### Accessing the `clock` service

Since this is a service, we can use `Ember.inject.service` to access the `clock`
service from our controller.

The controller can set any computed properties based on the `pulse` property of
the injected `clock` service.

In this case the `seconds` property is bound to the `pulse` property of the
controller's `clock`.

The controller has (session) data to display `seconds` to visitors, as well as
a handful of properties used as conditions in the Handlebars template.

`app/controllers/interval.js`

```javascript
import Ember from 'ember';

const { inject, computed } = Ember;

export default Ember.Controller.extend({
  clockService: inject.service('clock'),
  seconds: computed.alias('clockService.seconds'),
  
  fullSecond: computed('seconds', function () {
    return (this.get('seconds') % 1 === 0);
  }),
  
  quarterSecond: computed('seconds', function () {
    return (this.get('seconds') % 1 === 1/4);
  }),
  
  halfSecond: computed('seconds', function () {
    return (this.get('seconds') % 1 === 1/2);
  }),
  
  threeQuarterSecond: computed('seconds', function () {
    return (this.get('seconds') % 1 === 3/4);
  })
});
```

A controller for a list of comments, each comment will have a new clock
instance when added to the list. The comment item controller sets up
the `seconds` binding, used by the template to show the time since the
comment was created.

`app/controllers/comment-item.js`

```javascript
export default Ember.ObjectController.extend({
  seconds: Ember.computed.oneWay('clock.pulse').readOnly()
});
```

`app/controllers/comments.js`

```javascript
import ClockService from '../services/clock';

export default Ember.ArrayController.extend({
    itemController: 'commentItem',
    comment: null,
    actions: {
      add: function () {
        this.addObject(Em.Object.create({
          comment: this.get('comment'),
          clock: ClockService.create()
        }));
        this.set('comment', null);
      }
    }
  });
```

#### Handlebars template which displays the `pulse`

The `seconds` value is computed from the `pulse` attribute. And the controller
has a few properties to select a component to render, `fullSecond`,
`quarterSecond`, `halfSecond`, `threeQuarterSecond`.

`app/templates/interval.hbs`

```handlebars
{{#if fullSecond}}
  {{nyan-start}}
{{/if}}
{{#if quarterSecond}}
  {{nyan-middle}}
{{/if}}
{{#if halfSecond}}
  {{nyan-end}}
{{/if}}
{{#if threeQuarterSecond}}
  {{nyan-middle}}
{{/if}}
<h3>You&apos;ve nyaned for {{digital-clock seconds}} (h:m:s)</h3>
{{render 'comments'}}
```

A template for a list of comments

`app/templates/comments.hbs`

{% highlight html %}
{% raw %}
<form {{action "add" on="submit"}}>
  {{input value=comment}}
  <button>Add Comment</button>
</form>
<ul>
{{#each item in this}}
  <li>{{item.comment}} ({{digital-clock item.seconds}})</li>
{{/each}}
</ul>
{% endraw %}
{% endhighlight %}

#### Handlebars helper to format the clock display (h:m:s)

This helper is used in the template like so `{{digital-clock seconds}}`,
`seconds` is the property of the controller that will be displayed (h:m:s).

`app/helpers/digital-clock.js`

```javascript
export default Ember.Handlebars.makeBoundHelper(function(seconds) {
    var h = Math.floor(seconds / 3600);
    var m = Math.floor((seconds % 3600) / 60);
    var s = Math.floor(seconds % 60);
    var addZero = function (number) {
      return (number < 10) ? '0' + number : '' + number;
    };
    var formatHMS = function(h, m, s) {
      if (h > 0) {
        return '%@:%@:%@'.fmt(h, addZero(m), addZero(s));
      }
      return '%@:%@'.fmt(m, addZero(s));
    };
    return new Ember.Handlebars.SafeString(formatHMS(h, m, s));
});
```

#### Note

To explore the concept further, try adding a timestamp and updating the clock's
pulse by comparing the current time. This would be needed to update the pulse
property when a user puts his/her computer to sleep then reopens their browser
after waking.

#### Links

The source code:

* <http://jsbin.com/jihivevafo/1/edit?html,js,output>

Further reading:

* [Ember Object](http://emberjs.com/api/classes/Ember.Object.html)
* [Ember Application Initializers](http://emberjs.com/api/classes/Ember.Application.html#toc_initializers)
* [Method Inject](http://emberjs.com/api/classes/Ember.Application.html#method_inject)
* [Conditionals](../../templates/conditionals/)
* [Writing Helpers](../../templates/writing-helpers/)
* [Defining a Component](../../components/defining-a-component/)
* [Ember Array Controller](http://emberjs.com/api/classes/Ember.ArrayController.html)
