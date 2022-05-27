+++
title = "{{ replace .Name "-" " " | title }}"
outputs = ["Reveal"]
date = {{ .Date }}
draft = false
author = "杨武杰"
categories = [ "演示文稿" ]
tags = [ "Hugo" ]
year = "{{ dateFormat "2006" .Date }}"
month = "{{ dateFormat "2006/01" .Date }}"
+++

---

# Hello Hugo!

---