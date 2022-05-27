+++
title = "{{ replace .Name "-" " " | title }}"
date = {{ .Date }}
draft = false
author = "杨武杰"
categories = [ "随想", "工具" ]
tags = [ "Hugo", "rust" ]
year = "{{ dateFormat "2006" .Date }}"
month = "{{ dateFormat "2006/01" .Date }}"
+++

概要...
<!--more-->
更多内容...