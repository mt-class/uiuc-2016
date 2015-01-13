---
layout: ling490_frame
img: bruegel_babel
img_link: http://en.wikipedia.org/wiki/Tower_of_babel
img_credit: Image by Pieter Brueghel the Elder (1526/1530–1569). Public Domain via Wikimedia Commons.
caption: The Tower of Babel
title: Syllabus
active_tab: syllabus
---



<table class="table table-striped"> 
  <tbody>
    <tr>
      <th>Date</th>
      <th>Topic</th>
      <th>Readings (<i class="fa fa-heart"></i>=graduate level; <i class="fa fa-asterisk"></i>=optional)</th>
    </tr>
    {% for lecture in site.data.ling490_syllabus %}
    <tr>
      <td>{{ lecture.date | date: "%b %d" }}</td>
      <td>
        {% if lecture.slides %}<a href="{{ lecture.slides }}">{{ lecture.title }}</a>
        {% else %}{{ lecture.title }}{% endif %}
	{% if lecture.language %}
	<br/><a href="lin10.html">Language in 10</a>: <a href="{{ lecture.language_slides }}">{{ lecture.language }}</a>
        {% endif %}
      </td>
      <td>
        {% if lecture.reading %}
          <ul class="fa-ul">
          {% for reading in lecture.reading %}
            <li>
            {% if reading.grad_level %}<i class="fa-li fa fa-graduation-cap"> </i>
            {% elsif reading.optional %}<i class="fa-li fa fa-info-circle"> </i>
            {% else %}<i class="fa-li fa"> </i> {% endif %}
            {{ reading.author }},
            {% if reading.url %}
            <a href="{{ reading.url }}">{{ reading.title }}</a>
            {% else %}
            {{ reading.title }} 
            {% endif %}
            {% if reading.pages %}
            (p. {{ reading.pages }})
            {% endif %}
            </li>
          {% endfor %}
          </ul>
        {% endif %}
      </td>
    </tr>
    {% endfor %}

  </tbody>
</table>
