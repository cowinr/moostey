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

There are [concerns](https://thoughtworks.github.io/p2/issue12/lombok/) [raised](https://stackoverflow.com/questions/3852091/is-it-safe-to-use-project-lombok) with using Lombok, and not everyone is happy to do so, so check with any teammates if you want to use it.

I've included it in the Getting Started section as you will need to '[install](https://projectlombok.org/setup/intellij)' Lombok to be able to use it with your IDE.

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

Turning on the goodies one-by-one, starting at the front with a simple `@Controller`:

Getting in Control
==================

I'm not writing a REST API, so I'm sticking with a simple `@Controller`:

```java
@Controller public class SimpleController {
  @GetMapping("/") public String getRootPage() {
    return "root";
  }
}
```

Nothing much new for me here other than `@GetMapping` (and pals `@PostMapping`, `@PutMapping` etc.) which makes the code easier to grok if nothing else. However, it is easy to overlook a few things:
- I haven't written a `xxx-servlet.xml` config file
- or a `web.xml`
- I can choose from a growing number of optional method [args](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-arguments).
- No need to tell Spring where my page templates are

I could go on...

Securing What We Have
=====================

[Security integration](http://docs.spring.io/spring-boot/docs/1.5.4.RELEASE/reference/htmlsingle/#boot-features-security) is where Boot's web support really shines:

> If Spring Security is on the classpath then web applications will be secure by default with ‘basic’ authentication on all HTTP endpoints. To add method-level security to a web application you can also add `@EnableGlobalMethodSecurity` with your desired settings.

Erm, so no code example needed.

However, adding custom config requires a little code:

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .antMatchers("/", "/home").permitAll()
          .anyRequest().authenticated()
          .and().formLogin().loginPage("/login").permitAll()
          .and().logout().permitAll();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
          .withUser("user").password("password").roles("USER");
    }
}
```

... again this saves an extraordinary amount of coding, giving us:
- a flexible way of securing URLs
- form based login
- a simple user database
- method level security via `@EnableGlobalMethodSecurity`
- access to the security details within page templates

Working With the Domain
=======================

The domain layer usually requires ORM mapping to a database using JPA. Spring has a long history of working with Hibernate out of the box -- with Spring Data JPA and Boot, the convenience is notched up a gear.

Add `@EnableJpaRepositories` and you're off with implementation-less JPA repositories backed by Hibernate. What could be easier than defining a full CRUD + paging + sorting DAO with an expressive DSL for queries using:

```java
@Repository
public interface PersonRepository
  extends PagingAndSortingRepository<Person, Long>,
  QueryDslPredicateExecutor<Person> {}
```

... jaw-droppingly concise.

The powerful QueryDsl support can be used to assemble various reusable predicate expressions, such as:

```java
public static BooleanExpression isPending() {
    return QPerson.person.nextReview.after(LocalDate.now());
}
```

This could be combined with other expressions to filter queries, or used by itself, as below, to find a page of persons with a pending review date:

```java
Page<Person> persons = personRepo.findAll(isPending(), pageable);
```

The domain Entity itself can be stripped of all boilerplate code using Lombok:

```java
@Entity
@Getter @Setter @NoArgsConstructor @Builder
public class Person {
    @Id
    private Long id;
    
    private String name;
    
    @DateTimeFormat(iso = ISO.DATE)
    private LocalDate nextReview; // (1)
}
```
(1) Java 8 date/time support prior to JPA 2.2 requires jerry-rigging in Spring Boot via `Jsr310JpaConverters`.

With the `@Builder` Lombok annotation you get a handy companion builder (inner) class, as below. This allows for a more fluid style of coding:

```java
Person p =
    Person.builder()
    .name("Doctor, The")
    .nextReview(LocalDate.now().plusDays(7L))
    .build();
```

Generating HTML
===============

There are a number of web template engines that work with Boot out-of-the-box -- Freemarker, Mustache, Groovy templates and JSPs. Unfortunately support for my favourite go-to engine, Velocity, has been dropped due to inactivity on the project.

I'd never tried [Thymeleaf](http://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html) before, so gave it a shot. The biggest advantages of Thymeleaf is being able to craft the HTML templates outside of a running container (so-called natural templates). However, I couldn't persuade my fellow developers to adopt it - they still prefer good *old* JSPs, despite the [obvious benefits](http://www.thymeleaf.org/doc/articles/thvsjsp.html) of Thymeleaf.

Some simple examples, for reference:

```html
<p th:text="#{msgs.headers.name}">Name</p> <!-- (1) -->

<table>
  <tr th:each="prod: ${allProducts}"> <!-- (2) -->
    <td th:text="${prod.name}">Oranges</td>
    <td th:text="${#numbers.formatDecimal(prod.price, 1, 2)}">0.99</td> <!-- (3) -->
  </tr>
</table>

<form th:object="${person}">
    <input type="text" th:field="*{name}"> <!-- (4) -->
</form>

<span sec:authorize="hasAuthority('SOME_AUTH')">…</span> <!-- (5) -->
```

| (1) | Message from resource bundle for localisation |
| (2) | Simple each loop over a list of Products |
| (3) | Built-in formatting |
| (4) | `*{xxx}` for reading properties of `person` var |
| (5) | [Security](http://www.thymeleaf.org/doc/articles/springsecurity.html) object use |

My favourite (re-)discovery as part of this exercise though are [webjars](http://www.webjars.org/) which allow you to easily manage client-side JavaScript libraries in a Maven build:

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.1.1</version>
</dependency>
```

With this dependency in place a page can pull in the JQuery resources with a URL like `/webjars/jquery/3.1.1/jquery.min.js`. As a bonus the [webjars-locator](https://github.com/webjars/webjars-locator-core) library allows you define this URL without the version path part (`/webjars/jquery/jquery.min.js`), making upgrades simpler.

Deployment
==========

One of the best parts about Boot? Deployment is as easy as running the `main()` method of the application, or `java -jar myapp-1.0.0.jar` for the packaged uber-jar, or `mvn spring-boot:run` using Maven. Take your pick, and have the app running using the embedded Tomcat server in seconds.

References
==========

Some well-worn references from my journey...

## Spring
https://spring.io/docs/reference
http://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/
http://docs.spring.io/spring-data/jpa/docs/1.11.1.RELEASE/reference/html/
http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/
http://docs.spring.io/spring-security/site/docs/4.2.2.RELEASE/reference/htmlsingle/
https://github.com/spring-projects/spring-data-examples/tree/master/jpa

## Thymeleaf
http://www.thymeleaf.org/doc/articles/springsecurity.html
http://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html
https://ultraq.github.io/thymeleaf-layout-dialect/
https://github.com/thymeleaf/thymeleaf-extras-java8time
http://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html

# Others I used
http://www.webjars.org/
http://dbunit.sourceforge.net/
http://fontawesome.io/icons/
http://hibernate.org/orm/documentation/5.0/
https://v4-alpha.getbootstrap.com/getting-started/introduction/
https://jsoup.org/
https://projectlombok.org/