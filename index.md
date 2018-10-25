---
layout: about
---


<div class="about-title">Qianrui Zhang</div>

---

<div class="about-nav">
<a href="https://github.com/owen6314" target="_blank">GitHub</a>
<a href="https://drive.google.com/file/d/1IRQExna0gWKOf4UkZj91Ma0iXc38yxOV/view?usp=sharing" target="_blank">Resume</a>
</div>

---


Updates:
<nav>
<ul>
  {% for update in site.data.updates %}
  <li>
[ {{ update.date }} ] {{ update.title }}!
  </li>
  {% endfor %}
</ul>
</nav>
