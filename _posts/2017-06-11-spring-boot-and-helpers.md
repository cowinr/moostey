---
layout: default
title: "Spring Boot and Helpers"
date: 2017-06-11
---

Overview
========

The journey from Spring with XML to Spring Boot. A tale of writing less code...

Intro
=====

I've been using Spring since version 1.2.6 way back in 2006, and I got very comfortable with configuring up my apps with XML. It just felt like the right thing to do, separating the config from the code and keeping it all in one neat place - a blueprint for my app. I could knock out an `application.xml` file pretty quick - no problem.

Then along came the Java config mechanism... I've been fighting using it for years. As annotations crept in to Spring I slowly got used to losing that central application blueprint, and having to search through the codebase to see how it fit together. Tools help of course, but nothing beats looking at the code right?

I finally gave up the XML habit when I had the chance to investigate Spring Boot. Now I'm hooked. The Spring guys have done an _amazing_ job with auto-configuration such that what little config I have to declare myself seems insignificant and fits neatly into a bunch of config-specific `@Configuration` classes. With the main Spring Boot app at the top-level package, and domain-specific config in sub-packages, I have my central (ish) blueprint back in a terse, compile-time checked, fashion.

The rest of this post documents what is needed to set up a Spring Boot app for a simple secured web application with a database at the back-end. I'll introduce a few 'friends' of Boot that helped along the way.

Getting Started
===============

Starting with Boot is super-easy. An app starter can be generated using <http://start.spring.io>. You can even share a URL for a particular selection of dependencies - for example [this link](http://start.spring.io/starter.zip?type=maven-project&language=java&bootVersion=1.5.3.RELEASE&baseDir=demo&groupId=com.example&artifactId=demo&name=demo&description=Demo+project+for+Spring+Boot&packageName=com.example.demo&packaging=jar&javaVersion=1.8&autocomplete=&style=security&style=devtools&style=lombok&style=web&style=thymeleaf&style=data-jpa&style=actuator&generate-project=") will generate a Maven project with some of the helpers mentioned below.

For reference I investigated Boot 1.5.3 with Java 8, Hibernate 5, Spring Data JPA, and Thymeleaf 3. Some other dependencies of note: [Bootstrap v4.0.0-alpha.6](https://v4-alpha.getbootstrap.com/) (alpha! never again will I try to use an alpha quality library), [webjars](http://www.webjars.org/), [Lombok](https://projectlombok.org/) and [JSoup](https://jsoup.org/).

## Lombok

[Lombok](https://projectlombok.org/) is one of those jewels I stumbled across along the path to Boot. Watch the video on their website for a few minutes and you will see why I smiled - no more getters, setters, `toString()`, `equals()`, or `hashcode()` worries.

There are concerns raised with using Lombok, and not everyone is happy to do so, so check with any teammates if you want to use it.

I've included it in the Getting Started section as you will need to 'install' Lombok to be able to use it with your IDE.

## Bootstrapping Boot

It's as easy as unpacking the zip from start.spring.io, loading it into an IDE and running the main class:

```java
@SpringBootApplication
public class MoosteySpringApplication {
  public static void main(String[] args) {
    SpringApplication.run(MoosteySpringApplication.class, args);
  }
}
```

## What's Next?

Turning on the goodies one-by-one, starting at the front with a simple `@Controller`.

Getting in Control
==================

I'm not writing a REST API, so I'm sticking with a simple `@Controller`:

Nothing much new for me here other than `@GetMapping` (and pals `@PostMapping`, `@PutMapping` etc.) which makes the code easier to grok if nothing else.
