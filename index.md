---
title: 面向 Azure 开发人员的练习
permalink: index.html
layout: home
---

## 概述

以下练习旨在提供动手学习体验，你在其中探索开发人员在 Microsoft Azure 上生成和部署解决方案时执行的常见任务。

> **备注**：要完成这些练习，你需要 Azure 订阅，其中包含足够的权限和配额以预配必要的 Azure 资源。 如果还没有 Azure 订阅，请注册 [Azure 帐户](https://azure.microsoft.com/free)。 

部分练习可能会有额外或不同的要求。 这些内容将包含一个特定于该练习的“准备工作”一节****。

## 主题区域
{% assign exercises = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign grouped_exercises = exercises | group_by: "lab.topic" %}

<ul>
{% for group in grouped_exercises %}
<li><a href="#{{ group.name | slugify }}">{{ group.name }}</a></li>
{% endfor %}
</ul>

{% for group in grouped_exercises %}

## <a id="{{ group.name | slugify }}"></a>{{ group.name }} 

{% for activity in group.items %} [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) <br/> {{ activity.lab.description }}

---

{% endfor %} <a href="#overview">返回页首</a> {% endfor %}

