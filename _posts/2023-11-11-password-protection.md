---
layout: password
title: 密码验证
date: 2023-11-11 10:00:00 +0800
---

{% if page.password == site.password %}
  <meta http-equiv="refresh" content="0;url={{ page.redirect_to | default: page.url }}">
{% else %}
  <h2>密码错误</h2>
{% endif %}
