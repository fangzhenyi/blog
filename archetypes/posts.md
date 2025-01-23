---
title: {{ replace .TranslationBaseName "-" " " | title }}
date: {{ .Date }}
slug: {{ substr .File.UniqueID 0 7 }}
description:
keywords:
tags:
  - blog
categories:
  - blog
collections:
  - blog  
# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

<!--more-->
