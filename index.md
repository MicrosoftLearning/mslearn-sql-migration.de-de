---
title: Übungen zu Azure OpenAI
permalink: index.html
layout: home
---

# SQL Server-Migrationsübungen

Die folgenden Übungen wurden entwickelt, um die Module im Lernpfad [Migrieren von SQL Server-Workloads zu Azure SQL](https://learn.microsoft.com/training/paths/migrate-sql-workloads-azure/) in Microsoft Learn zu unterstützen.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}