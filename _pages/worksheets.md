---
layout: archive
title: Worksheets
permalink: /worksheets/
---

{% for worksheet in site.worksheets %}
{% if worksheet.optional == nil %}
{% if worksheet.local != "true" %}
[{{ worksheet.title }}]({{ worksheet.url | prepend: site.docsurl }})
{% else %}
[{{ worksheet.title }}]({{ worksheet.url | prepend: site.baseurl }})
{% endif %}
{% endif %}
{% endfor %}

## Optional

{% assign optional-worksheets = site.worksheets | where: "optional","true" %}
{% for worksheet in optional-worksheets %}
[{{ worksheet.title }}]({{ worksheet.url | prepend: site.docsurl }})
{% endfor %}
