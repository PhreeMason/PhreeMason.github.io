---
layout: post
title:  "The Power Of Handlebars"
date:   2017-06-16 06:30:47 +0000
---

I finished my first rails app with a jQuery Front End and my biggest takeaway from this is that Handlebars is AWESOME! Without it, this would have been harder and taken much longer.

For those of you who don't know, what Handlebars is, it's a JavaScript tool that allows you to build html templates. But that's not the best part! With the templates made, you can effortlessly  insert a JavaScript object from an API or from your app. Here's an example using my calendar app.

Here is my template for a task that will render on the calendars' show page.

```
<script id="event-template" type="text/x-handlebars-template">
  <div class="event">
    <div class="event-desc" id="task-{{day}}">
        {{ content }}
    </div>
    <div class="event-time">
        {{ month_time }}
    </div>
  </div>
</script>
```

And this is what my tasks object looks like

```
Object {id: 1, content: "awesome stuff being done here", time: "Jun 22,  4:05 AM", day: 22, month_time: "06 05 AM"}
```

As you can see the template is written in html. Within the html are handlebars expressions indicated by two curly braces. JS objects can replace the expressions in the curly braces. This is where my tasks objects comes into play. My task object has a property of 'content' and 'month_time.'  The Handlebars template will grab the values of 'month_time' and 'content' and replace the expressions of the same name.

The output will look like this:

```
  <div class="event">
    <div class="event-desc" id="task-22}">
        awesome stuff being done here
    </div>
    <div class="event-time">
        06 05 AM
    </div>
  </div>
```

The html above can then be appended to an element on the page. This is great because without it, I would have to write all the html inside a JavaScript string. Then append the string to an element on the page. This can get messy if you need to input more than one line of html and specify classes, ids and other attributes. Also, your string will not be properly formatted in html which makes it hard to edit later if its long. 

Take a look at the same template using only JavaScript:

```
`<div class="event"> <div class="event-desc" id="task-${day}"> ${content} </div> <div class="event-time"> ${month_time} </div> </div>`
```

Gross! 


