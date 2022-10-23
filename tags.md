---
layout: default
---

{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
{% assign vids = tag[1] | sort %}

{% if vids != empty %}

<h2 id="{{tag[0] | uri_escape | downcase}}">{{tag[0]}}</h2>
<p>
{% for p in vids %}
<a href="{{ p.url }}">{{ p.title }}</a> {{ p.date | date: "%Y-%m-%d" }}
<br />
{% endfor %}
</p>

{% endif %}

{% endfor %}