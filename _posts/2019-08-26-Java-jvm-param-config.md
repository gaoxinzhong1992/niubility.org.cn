---
layout: post
title: "jvm内存参数配置"
date: 2019-08-26 10:44:00
categories: Java
---

jvm内存参数配置

## 概述

基于jdk1.8，由于jdk1.8开始，没有了永久代的概念，所以在jvm参数配置上不再需要：

- XX:PermSize
- XX:MaxPermSize

