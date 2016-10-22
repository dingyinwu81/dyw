---
title: 归档
permalink: /booklist/
nav: true
---

<script type="text/javascript">
// prepare data from jekyll
var $booklist = {
  baseUrl: "{{ site.baseurl }}/booklist/?label=",
  staticUrl: "{{ site.static_url }}",
  labels: [
    "显示全部",
    {% for post in site.posts %}
      {% if post.release %}
        {% for label in post.labels %}
          "{{ label }}",
        {% endfor %}
      {% endif %}
    {% endfor %}
  ],
  posts: [
    {% for post in site.posts %}
      {% if post.release %}
      {
        title: "{{ post.title }}",
        date: "{{ post.date | date: "%Y-%m-%d" }}",
        link: "{{ post.url | prepend: site.baseurl }}",
        labels: [
        {% for label in post.labels %}
          "{{ label }}",
        {% endfor %}
        ]
      },
      {% endif %}
    {% endfor %}
  ]
};
</script>
<script src="/assets/js/lib/react/react.js"></script>
<script src="/assets/js/lib/react/JSXTransformer.js"></script>
<script type="text/jsx" src="/pages/booklist.js"></script>

<div id="main"></div>
