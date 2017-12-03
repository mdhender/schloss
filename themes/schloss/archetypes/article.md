---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
menu:
    articles:
        name: "{{ replace .TranslationBaseName "-" " " | lower }}"
        weight: 999
---

