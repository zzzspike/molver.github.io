---
layout: default
paginator.perpage: 5
paginator.page:0
---
<!-- This loops through the paginated posts -->
{% for post in paginator.posts %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  <p class="author">
    <span class="date">{{ post.date }}</span>
  </p>
  <div class="content">
    {{ post.content }}
  </div>
{% endfor %}

<!-- Pagination links -->
<div class="pagination">
  {% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path }}" class="previous">上一页</a>
  {% else %}
    <span class="previous">上一页</span>
  {% endif %}
  <span class="page_number "> {{ paginator.page }} / {{ paginator.total_pages }}</span>
  {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path }}" class="next">下一页</a>
  {% else %}
    <span class="next ">下一页</span>
  {% endif %}
</div>
