# Manual

{% for item in site.collection01 %}
  <h2>
    <a href="{{ item.url }}">
      {{ item.name }} - {{ item.position }}
    </a>
  </h2>
  <p>{{ item.content | markdownify }}</p>
{% endfor %}