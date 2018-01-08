---
layout: default
title: "Kotlin For Production"
date: 2018-01-08
---

# Summary
I've been thinking about proposing adoption of Kotlin as a companion language for Java in our dev shop.
We have a ton of Java web apps, but Kotlin seems to be turning many heads (including mine) as a better alternative
to plain old Java.
To be clear, I do not advocate writing or re-writing everything in Kotlin, that wouldn’t make any sense.
This document is intended to promote the language and features of Kotlin.
There are a number of pros and cons to using Kotlin compared with Java for production web applications:
## Pros
1. Kotlin fixes a series of issues that Java suffers from: Null references, Array variance, checked exceptions,
boilerplate verbosity.
2. Kotlin has a number of features over and above Java: lambda expressions, extension functions, smart
casting, properties, first-class delegation, type inference for variables (though this is coming in Java 10) and properties, range expressions,
operator overloading, data classes, and others.
3. Kotlin and Java are fully [interoperable](https://kotlinlang.org/docs/reference/java-interop.html), meaning Java’s vast ecosystem can be leveraged, including JEE,
Spring, Hibernate, JDBC, logging, Maven, app servers, utility libraries etc. The Java standard library types,
collections, etc. are all reused and augmented for greater usability.
4. Kotlin and Java can coexist in the same application, allowing for gradual adoption and retrofitting Kotlin into
older apps.
5. All of our current libraries can be used as-is by a Kotlin application, meaning we don’t have to start from
scratch with domain models, security set-up etc.
6. Spring 5 and Spring Boot 2 (at Milestone 7 as of writing) have dedicated Kotlin [support](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0).
The entirety of the core framework have been retrofitted with `@Nullable` annotations specifically to allow Kotlin to infer the nullability of the Spring APIs.
7. Kotlin is a popular language (#39 on the TIOBE index and #46 on Redmonk ranking), with special mentions on the [TIOBE index for January 2018](https://www.tiobe.com/tiobe-index/) and the Redmonk
ranking for June 2017.
8. Kotlin support in IntelliJ is top notch. Basically on par with Java for static analysis, code completion, and
debugging.
9. Kotlin is a supported language for the Android platform – suggests strong backing and longevity.
10. Kotlin is branching out to other platforms – JavaScript and Native / iOS.

## Cons
1. There is a learning curve for any new language. However (a) Kotlin is sufficiently similar to Java as to not
prove overwhelming, and (b) the IntelliJ IDE has excellent automatic Java to Kotlin support.
2. It will take even longer for a Java developer to be able write fully idiomatic Kotlin code utilising best
practices.
3. Kotlin support may vanish over time, as with any other language. In this case Kotlin code can be decompiled
to Java to help migration back to Java if required.

## Some Useful Resources
… on how ace Kotlin is
1. https://www.wired.com/story/kotlin-the- upstart-coding- language-conquering- silicon-valley/
2. https://blog.heroku.com/rise-of- kotlin
3. https://plus.google.com/+JakeWharton/posts/WSCoqkJ5MBj

# Drivers
The primary drivers for adopting Kotlin are:

## Null Safety
The null reference (in any language, not just Java) has been referred to as the [billion dollar mistake](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions) by its ‘inventor’
Tony Hoare. Kotlin has null safety built into the type system, meaning it is impossible to generate a null pointer
exception.

## Concise and Elegant
Kotlin allows a developer to write very concise code, with much less verbosity than Java for standard constructs. For
example a Kotlin ‘data’ class is similarly sized to a Lombok annotated Java POJO (It should be noted that Java 10 may include so-called value types).
Inline functions, extension functions, operator overloading and a host of other language features allow for writing
very concise, yet readable, code. This may not be a huge productivity boost, but instead should be looked at as a way
of exposing the wood from the trees.

## Low Risk
Given that it is 100% interoperable with Java, and can be mixed with Java within a single project, Kotlin is perhaps
the first language that could be adopted for production web applications with such a small risk.

# Proof Of Concept
Using Spring Boot 2 (pulls in Spring 5) the following code snippets have been used to create a secured
web application.

## Security
Configure web and service method security using Spring Security. Access to the `/report` end point is secured using a granted authority `GA_1`:

```kotlin
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
class WebSecurityConfig : CommonWebSecurityConfig(), WebMvcConfigurer {
    override fun configureHttpSecurity(http: HttpSecurity?) {
        http?.let {
            it.authorizeRequests()
            .antMatchers("/report").access("hasAuthority('GA_1')")
            .anyRequest().authenticated()
            .and().exceptionHandling().accessDeniedPage("/403")
        }
    }
//...
}
```

## Domain Modelling

A simple report row is modelled using a data class, much like a Lombok annotated POJO:

```kotlin
data class ReportRow(
    val postedDate: String,
    val name: String,
    val region: String
    // ...
)
```

## Data Access
### With JDBC

Using JDBC to access a Database with Spring’s `JdbcTemplate`.

```kotlin
@Repository
class ReportRepository(private val jdbcTemplate: NamedParameterJdbcTemplate)
: ReportRepository {
    // Maps a ResultSet row to a ReportRow domain object (like RowMapper)
    private val reportRowMapper: (rs: ResultSet, rowNum: Int) -> ReportRow = { rs, _ ->
        ReportRow(
            rs["POSTED_DATE"],
            rs["DIVISION"],
            rs["REGION"]
            //...
        )
    }
    
    fun runReport(options: ReportOptions): List<ReportRow> {
        val source = MapSqlParameterSource()
        source["id"] = options.user.id
        val sql = generateSql() // "SELECT * FROM XXX WHERE ID = :id ...
        return jdbcTemplate.query(sql, source, reportRowMapper)
    }
}
```

### With HTTP
For example, a HTTP call to Oracle REST Data Services. This could use
Spring’s `RestTemplate` or Apache’s `HttpClient` in place of the `JdbcTemplate`. The following also includes setting a
HMAC header.

```kotlin
@Repository
class RestReportRepository(
    restTemplateBuilder: RestTemplateBuilder,
    @Value("\${endpoint.target}") private val target: String)
: ReportRepository {

    private val template: RestTemplate = restTemplateBuilder.build()

    fun runReport(options: ReportOptions): List<ReportRow> {
        val url = "http://$target/report"
        val headers = HttpHeaders()
        headers["Auth-HMAC"] = calculateHmacAuthHeader(/* ... */)
        val request: HttpEntity<Any> = HttpEntity(headers)
        val response = template.exchange(
            url, GET, request, object : ParameterizedTypeReference<List<ReportRow>>() {}
        )
        return response.body ?: emptyList()
    }
}
```

## Service Layer
Nothing much of interest in the service layer. A useful spot to introduce access logging though.

```kotlin
@Service
class ReportService (
    private val reportRepository: ReportRepository,
    private val loggingService: MyLoggingService
) {

    @PreAuthorize("hasAuthority("GA_1")")
    fun runReport(options: ReportOptions): List<ReportRow> {
        // Access Logging in the database
        loggingService.log(options, "Running report ${options.report}")
        // Application logging
        log.debug("Run ${options.report} for user with ID ${options.user.id}")
        return reportRepository.runReport(options)
    }
    
    companion object {
        val log: Logger = LoggerFactory.getLogger(ReportService::class.java)
    }
}
```

## Web Component
The web layer is simply a Spring Controller that binds HTTP query parameters to a command object, passing it to the
service to generate the report data. A JSP or similar could pick up the report data for presentation.

```kotlin
@Controller
class ReportController(private val reportService: ReportService) {

    @GetMapping("/report")
    fun writtenCommCredit(@Valid options: ReportOptions, model: Model): String {
        model.addAttribute(reportService.runReport(options))
        return "report"
    }
}
```

## Build
The application can be built with a standard Maven `pom.xml` file. All
that is required for Kotlin is a build plugin.