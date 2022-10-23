# Manual

{% for item in site.collection01 %}
  <h2>
    <a href="{{ item.url }}">
      {{ item.relative_path }} - {{ item.date }}
    </a>
  </h2>
  <p>{{ item.content | markdownify }}</p>
{% endfor %}