---
title: "Json to Html table"
date: "2019-10-18"
tags: [
    "html",
    "javascript"
]
draft: false
---

 Convert a JSON Doc or JSON-Schema into a HTML Table 

```html
<!-- JS-Code taken from https://gist.github.com/fbstj/1199018 -->
<!-- See live demo at: https://jsfiddle.net/peLjkfrm/3/ -->
<style>
  table {
    border-collapse: collapse;
  }

  table, th, td {
    border: 1px solid black;
  }
</style>

<div id="mytab"></div>


<script>
const schema = {
    "$schema": "http://json-schema.org/draft-04/schema#",
    "type": "object",
    "title": "Book",
    "properties": {
        "id": {"type": "string"},
        "title": {"type": "string"},
        "author": {"type": "string"},
        "year": {"type": "integer"},
        "publisher": {"type": "string"},
        "website": {"type": "string"},
        "subDoc": {
            "type": "object",
            "properties": {
                "internal": {
                    "type": "object",
                    "properties": {
                        "ean": {
                            "type": "integer"
                        },
                        "sn": {
                            "type": "string"
                        }
                    },
                    "required": [
                        "ean"
                    ]
                }
            },
            "required": [
                "internal"
            ]
        }
    },
    "required": ["id", "title", "author", "year", "publisher", "subDoc"]
};

function HTMLON(o, e) {
    function ne(name) {
        return $(document.createElement(name));
    }

    function nr(i, v) {
        return ne('tr').append(ne('th').text(i)).append(HTMLON(v, ne('td')));
    }
    if ($.isArray(o)) {
        var ul = ne('ul');
        $.each(o, function(i, v) {
            ul.append(HTMLON(v, ne('li')));
        });
        e.append(ul);
    }
    else if (typeof o == 'object') {
        var t = ne('table');
        $.each(o, function(i, v) {
            t.append(nr(i, v));
        });
        e.append(t);
    }
    else if(typeof o == 'function')
    {
        e.append(ne('code').text(o.toString()));
    }
    else {
        e.text(o);
    }
    return e;
}

HTMLON(schema, $('#mytab'));
</script>
```
