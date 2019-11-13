---
title: Java作用域访问权限
date: 1999-02-24
tags:
---

# Java作用域访问权限

不写的时候就是默认

| 作用域       | 当前类                | 同一包                | 子类                 | 其他包                |
| --------- | ------------------ | ------------------ | ------------------ | ------------------ |
| public    | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| protected | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |                    |
| default   | :heavy_check_mark: | :heavy_check_mark: |                    |                    |
| private   | :heavy_check_mark: |                    |                    |                    |
