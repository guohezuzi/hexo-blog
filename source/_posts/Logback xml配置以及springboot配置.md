---
title: Logback xml配置以及springboot配置
date: 1999-02-24
tags:
---

# Logback xml配置以及springboot配置

## XML标签

- appender(定义日志输出的格式)

- include(包含的其他日志配置)

- springProfile(只针对spring)

- logger(特定的日志对应的水平)

- root(根日志对应的水平)

## Springboot application.xml配置

- logging.config 自定义的日志文件的位置

- logging.pattern.* 不同日志输出的格式

- logging.level 不同日志输出的日志水平

- logging.path 日志的路径

- logging.file.* 日志输出文件的一系列配置
