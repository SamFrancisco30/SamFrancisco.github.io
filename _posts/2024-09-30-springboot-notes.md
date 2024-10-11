---
layout: post
title: Spring Boot笔记
categories: SpringBoot
description: Spring Boot笔记
keywords: SpringBoot
excerpt: Spring Boot Notebook
---

# Spring Core
## Inversion of Control
a **design principle** where the control of **object creation and management** is transferred from the application code to the framework.

## Dependency Injection
a **design pattern** that enables objects to receive their dependencies from external sources during runtime rather than creating them internally.

Ways to realize: constructor, setter, fields

## Benefits of IoC and DI*
1. **decouple** the creation and management of objects from the business logic,makes the code **more flexible and easier to maintain**

2. Improved Code **Reusability**: DI encourages breaking down functionality into small, reusable components (beans)

3. **reduce boilerplate code**

## Spring Bean
an object that is managed by the Spring IoC container