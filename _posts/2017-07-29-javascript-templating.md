---
layout: default
title: "JavaScript Templating"
date: 2017-07-29
---

Overview
========

A Collection of useful JavaScript templating for various scenarios. I'm expecting the list to grow as new frameworks and/or techniques come along.

The objective is to find an ideal mechanism for creating HTML on the fly and bind JSON data to it. The ideal mechanism is likely to vary based on the context, e.g. if the page or project already includes JQuery, or how complex the HTML fragment or JSON data is.

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

Context: String concatenation
========

This is really just here to provide a comparison

```javascript
    $(function () {
        console.log("Loaded page...");

        // Load data
        $.getJSON('../data.json', function (json) {
            console.log(json);

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

**Note:** I'm super-impressed with IntelliJ IDEA for managing to autocomplete the HTML tags in the middle of the apostrophe and concatenation soup. 
