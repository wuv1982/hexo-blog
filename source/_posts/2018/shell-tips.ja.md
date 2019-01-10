---
title: shell-tips
language: ja
date: 2018-08-13 12:04:28
tags:
---

## running script path
get the path relative the running script.
```
SCRIPT_PATH=$0
PROJ_HOME=`dirname $SCRIPT_PATH`/../..
```
