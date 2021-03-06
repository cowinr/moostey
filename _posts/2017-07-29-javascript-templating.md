---
layout: default
title: "JavaScript Templating"
date: 2017-07-29
---

Overview
========

A Collection of useful JavaScript templating techniques for various scenarios. I'm expecting the list to grow as new frameworks and/or techniques come along.

The objective is to find an ideal mechanism for creating HTML on the fly and bind JSON data to it. The *ideal* mechanism is likely to vary based on the context, e.g. if the page or project already includes a JavaScript library, or how complex the HTML fragment or JSON data is.

The example JSON I've used for each case is:

File data.json:
```javascript
{
  "name": "Richard",
  "foods": [
    {
      "id": 1,
      "name": "apple"
    },
    {
      "id": 2,
      "name": "pie"
    },
    {
      "id": 4,
      "name": "custard"
    }
  ]
}
```

It's rather simplistic, but my assertion is that the JSON should be pretty much ready for rendering without too much processing. Data processing *should* be done server-side as far as possible.

String Concatenation
====================

This is really just here to provide a comparison to the later examples. JQuery is used only to attach event handling.

```javascript
    $(function () {

        // Load data
        $.getJSON('../data.json', function (json) {

            var html = '<div class="panel"><span>' + json.name + '</span><ol>';

            $.each(json.foods, function (index, food) {
                html += '<li id="' + food.id + '">' + food.name + '</li>';
            });

            html += '</ol></div>';

            $('#section').html(html);

            $('.panel').click(function () {
                var s = "<p style='margin: 0;'>I've been clicked</p>";
                $(this).append(s);
                $(this).toggleClass("test");
            });

        });
    });
```

This is awkward, and I can't really advocate a situation in which you'd create HTML via string concatenation like this.

**Note:** I'm super-impressed with [IntelliJ IDEA](https://www.jetbrains.com/idea/) for managing to autocomplete the HTML tags in the middle of the apostrophe and concatenation soup.

JQuery
======

Where [JQuery](http://code.jquery.com/jquery-3.2.1.min.js) (I used 3.2.1) is already available.

```javascript
    $(function () {
        $.getJSON('../data.json', function (json) {

            var $div = $('<div/>', {
                'class': 'panel',
                'click': function () {
                    $(this).toggleClass("test");
                }
            }).append($('<span/>').text(json.name));

            $div.click(function () {
                $div.append($('<p/>', {"style": "margin: 0"}).text("I've been clicked"));
            });

            var $ol = $('<ol/>');
            $.each(json.foods, function (index, food) {
                $ol.append($('<li/>', {
                    "id": food.id
                }).text(food.name));
            });

            $ol.appendTo($div);

            $('#section').html($div);

        });
    });
```

This feels much more natural (to a programmer at least).

Note that:
* `$(...).text(food.name)` call will escape any HTML in the JSON.
* The click handler can be bound at element creation if required.
* Best practice? Use of a `$` prefix for element variables created using JQuery.
* Best practice? Closing the HTML tags in `$('<html/>')`. Not necessary, but conveys intention better.

Still it's fairly verbose, and not strictly a templating mechanism!

Mustache
========

Finally :) a purpose-built templating mechanism. [Mustache](http://mustache.github.io/) is one of the more popular libraries, with various language implementations, including [Java](https://github.com/spullara/mustache.java) and [JavaScript](https://github.com/janl/mustache.js). Here I've used [mustache-2.3.0.js](https://cdnjs.cloudflare.com/ajax/libs/mustache.js/2.3.0/mustache.min.js).

```javascript
    $(function () {

        var panelTemplate = $('#panelTemplate').html();
        Mustache.parse(panelTemplate);

        var messageTemplate = $('#messageTemplate').html();
        Mustache.parse(messageTemplate);
        var s = Mustache.render(messageTemplate, {
            "time": (new Date()).toLocaleTimeString("en-GB")
        });

        // Load data
        $.getJSON('../data.json', function (json) {

            $('#section').html(Mustache.render(panelTemplate, json));
            $('.panel').click(function () {
                $(this).append(s);
                $(this).toggleClass("test");
            });

        });
    });
```

And the Mustache templates themselves can be kept in `<script/>` blocks in HTML.

{% raw %}
```html
<script id="messageTemplate" type="x-tmpl-mustache">
    <p style="margin: 0;">I was rendered at {{time}}</p>
</script>

<script id="panelTemplate" type="x-tmpl-mustache">
    <div class="panel">
        <span>{{name}}</span>
        <ol>
            {{#foods}}
            <li id="{{id}}">{{name}}</li>
            {{/foods}}
        </ol>
    </div>
</script>
```
{% endraw %}

Alternatively the templates can be loaded via AJAX calls, as per the following section.

Mustache with templates loaded via AJAX
---------------------------------------

As mentioned above the Mustache templates can be loaded dynamically along with the JSON data. The example below shows this in action, with the neat JQuery function chain `.when(...).done(...).fail(...)` ensuring that both the data and template are loaded before rendering begins:

```javascript
    $(function () {

        $.when(
            // Load template
            $.ajax({
                url: "panelTemplate.mustache",
                dataType: "text"
            }),
            // Load data
            $.getJSON('../data.json')
        ).done(function (panelTemplate, json) {
            // Parse and Compile Mustache template
            Mustache.parse(panelTemplate[0]);

            // Merge template with data and write to #section
            $('#section').html(Mustache.render(panelTemplate[0], json[0]));

            // Some simple jQuery event handling
            $('.panel').click(function () {
                $(this).toggleClass("test");
            });
        }).fail(function () {
            $('#section').text('Template and/or data loading failed!');
        });

    });
```

And the template file `panelTemplate.mustache`:

{% raw %}
```handlebars
<div class="panel">
    <span>Name: {{name}}</span>
    <ol>
        {{#foods}}
        <li id="{{id}}">Food: {{name}}</li>
        {{/foods}}
    </ol>
</div>
```
{% endraw %}

**Note:** A Note on partials:
Mustache supports partials, which can be used to embed template fragments inside other templates. This makes it ideal for complex templates that can be logically split.

For example:
```
TODO
```

Alternatives
------------

There are a number of alternatives templating frameworks out there, most notably [Handlebars](http://handlebarsjs.com/), which is syntactically very similar to Mustache. I prefer Mustache as it favours keeping things simple, minimising the amount of logic kept in the templates themselves.

ES6 Template Literals
=====================

ES6 has built in support for simple templating within string creation via [template literals](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals).

```javascript
    $(function () {

        // Load data
        $.getJSON('../data.json', function (json) {

            let html =
                `<div class="panel">
                    <span>${json.name}</span>
                    <ol>`;

            $.each(json.foods, function (index, food) {
                html += `<li id="${food.id}">${food.name}</li>`;
            });

            html +=
                    `</ol>
                 </div>`;

            $('#section').html(html);

            // Some simple jQuery event handling
            $('.panel').click(function () {
                $(this).append(`<p style="margin: 0">I was rendered at ${(new Date()).toLocaleTimeString('en-GB')}</p>`);
                var myMargin = 0;
                let str = `
<div>
    <p style="margin: ${myMargin}">I was rendered at ${(new Date()).toLocaleTimeString('en-GB')}</p>
</div>
`;
                $(this).toggleClass("test");
            });

        });
    });
```

Without the list iteration in the middle of the string, this would have been a neat solution. Unfortunatley string template literals don't really help if *any* logic is required in the template.

Summary
=======

| Technique | Useful for | Avoid |
|---------|
| String concatenation | When you have no libraries to play with! <br/>You've forgotten how to use JQuery (or everything else). | Under any other circumstances. |
| JQuery | When you feel like a programmer. <br/>Keeping event handling logic next to your HTML creation. <br/>Complex data processing on the client. | If anyone on the team is not familiar with JQuery or programming (e.g. designer) |
| Mustache | Large complex templates. <br/>Need to reuse templates across multiple pages. <br/>Non programmers need to create templates. | May be overkill for smaller templates. |
| ES6 templates literals | Simple HTML creation only. <br/>When ES6 is available. | There is a need for logic in the template. |
