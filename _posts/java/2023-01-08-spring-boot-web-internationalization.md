---
layout: post
title: Spring Web/MVC Internationalization (i18n) with Example
date: 2023-01-08 14:42
category: [Java, Spring]
author: dkansh
tags: [java, i18n, internationalization, Localization, Spring, Spring boot, Web, thymeleaf]
summary: How to apply Internationalization (i18n) and Localization (L10n) on a Spring Web/MVC application
description: >-
    Any web application with users all around the world, internationalization (i18n) or localization (L10n) is very important for better user interaction. Most of the web application frameworks provide easy ways to localize the application based on user locale settings. Spring also follows the pattern and provides extensive support for internationalization (i18n) through the use of Spring interceptors, Locale Resolvers and Resource Bundles for different locales. Some earlier articles about i18n in java.
beforeToc: >-
    <p>A short time ago we looked into the <a href="/java-internationalization-the-basics">basics of Java i18n</a> and <a href="/spring-core-standalone-internationalization">the internationalization of Spring standalone application</a>. In this article, let’s take a step into the web application world and see how the Spring Boot framework handles internationalization i18n for Web/MVC applications.</p>

    <p>When developing a web application, we tend to code it using a collection of the most efficient, the most popular, and the most sought-after programming languages for both our front end and back end. But what about spoken languages? Most of the time, with or without our knowledge, we depend on the built-in translation engines of our customers’ browsers to handle the required translations. Don’t we?</p>

    <p>In the ever-globalizing world we live in, we need our web applications to reach as wide an audience as possible. Here enters the much-required concept of internationalization. In this article, we will be looking at how i18n works on the popular Spring Boot framework.</p>
toc: true
---

## Spring Web/MVC Internationalization (i18n)

We’ve seen how easy it is to do simple translations in a Spring project. Let’s go ahead and see how we can perform i18n on a Spring Boot Web application. Let’s create a simple Spring boot project where we will use the request parameter to get the user locale and based on that set the response page label values from locale-specific resource bundles. Create a Spring boot Project using [Spring Initializr](https://start.spring.io). Our final project with localization changes looks like the below image. We will look into all the parts of the application one by one.

![Project Structure](/assets/images/post/20221128/project_structure.png "Project Structure")

## Spring i18n Gradle Configuration

Our Spring boot `build.gradle` looks like below:

```groovy
plugins {
	id 'java'
	id 'org.springframework.boot' version '2.7.7'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'
}

group = 'com.techvigorous'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '19'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

Alright! Thanks to the magic of Spring Boot we have already completed building the skeleton of our Spring Boot internationalization example project. Now, it is time to give it some i18n functionalities.

## Spring Resource Bundle

For simplicity, let’s assume that our application supports only two locales - en and fr. If no user locale is specified, we will use English as the default locale. Let’s create spring resource bundles for both of these locales that will be used by the thymeleaf template.

messages.properties:

```properties
hello=Hello
welcome=Welcome to Tech Vigorous!
switch-en=Switch to English
switch-fr=Switch to French
```

messages_fr.properties:

```properties
hello=Bonjour
welcome=Bienvenue \u00e0 Tech Vigorous!
switch-en=Passer \u00e0 l'anglais
switch-fr=Passer au fran\u00e7ais
```

## Sample web page view using thymeleaf

Next, it’s time to create a simple View on our `java-i18n-spring-boot-web` application. Let’s make a `welcome.html` file within the project’s `resources/templates` directory, like so:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title th:text="#{welcome}"></title>
    </head>
    <body>
        <span th:text="#{hello}"></span>!<br>
        <span th:text="#{welcome}"></span><br>
        <button type="button" th:text="#{switch-en}" onclick="window.location.href='http://localhost:8080/?locale=en'"></button>
        <button type="button" th:text="#{switch-fr}" onclick="window.location.href='http://localhost:8080/?locale=fr'"></button>
    </body>
</html>
```

1. Make sure to declare Thymeleaf namespace to support `th:*` attributes.
2. The value for the `welcome` key is retrieved from the applicable language resource file of the specified locale and displayed as the title.
The button has the value of a `switch-fr` property key of the specified locale.

Upon clicking the button, the page is reloaded with an additional `locale=fr` parameter. This in turn causes our `LocaleChangeInterceptor` to kick in and resolve the template in the Italian language.

## It’s time to set up Spring Beans

Initially, let’s add some i18n related Spring beans to our `java-i18n-spring-boot-web` project.

Let's create a config class with name `I18nConfig` under `com.techvigorous` package. It should be just beside to `JavaI18nSpringBootWebApplication` class.

### Meet LocaleResolver

The [LocaleResolver](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/LocaleResolver.html) interface deals with locale resolution required when localizing web applications to specific locales. Spring aptly ships with a few LocaleResolver implementations that may come in handy in various scenarios:

- **FixedLocaleResolver:** Always resolves the locale to a singular fixed language mentioned in the project properties. Mostly used for debugging purposes.
- **AcceptHeaderLocaleResolver:** Resolves the locale using an “accept-language” HTTP header retrieved from an HTTP request.
- **SessionLocaleResolver:** Resolves the locale and stores it in the HttpSession of the user. But as you might have wondered, yes, the resolved locale data is persisted only for as long as the session is live.
- **CookieLocaleResolver:** Resolves the locale and stores it in a cookie stored on the user’s machine. Now, as long as browser cookies aren’t cleared by the user, once resolved the resolved locale data will last even between sessions. Cookies save the day!

### Use CookieLocaleResolver

Let’s see how we can use `CookieLocaleResolver` in our `java-i18n-spring-boot-web` application. Simply add a LocaleResolver bean within the `I18nConfig` class annotated with `@Configuration` and set a default locale. For instance:

```java
@Bean // <--- 1
public LocaleResolver localeResolver() {
    CookieLocaleResolver localeResolver = new CookieLocaleResolver(); // <--- 2
    localeResolver.setDefaultLocale(Locale.US); // <--- 3

    return localeResolver;
}
```

1. Bean annotation is added to mark this method as a Spring bean.
2. The LocaleResolver interface is implemented using Spring’s built-in CookieLocaleResolver implementation.
3. The default locale is set for this locale resolver to return in the case that no cookie is found.

### Add LocaleChangeInterceptor

Okay, now our application knows how to resolve and store locales. However, when users from different locales visit our app, who’s going to switch the application’s locale accordingly? Or in other words, how do we localize our web application to the specific locales it supports?

For this, we’ll add an interceptor – or interceptor? – bean that will intercept each request that the application receives, and eagerly check for a `locale` parameter on the HTTP request. If found, the interceptor uses the `localeResolver` we coded earlier to register the locale it found as the current user’s locale. Let’s add this bean within the `JavaI18nSpringBootWebApplication` class:

```java
@Bean
public LocaleChangeInterceptor localeChangeInterceptor() {
    LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
    // Defaults to "locale" if not set
    localeChangeInterceptor.setParamName("locale");

    return localeChangeInterceptor;
}
```

Now, to make sure this interceptor properly intercepts all incoming requests, we should add it to the Spring [InterceptorRegistry](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/InterceptorRegistry.html):

1. Set the i18n config class in your project, which is the `I18nConfig` class annotated with `@Configuration`, to implement [WebMvcConfigurer](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html), like so:

```java
@Configuration
public class I18nConfig implements WebMvcConfigurer {...}
```

2. Override the `addInterceptors` method and add our locale change interceptor to the registry. We can do this simply by passing its bean `localeChangeInterceptor` as a parameter to `interceptorRegistry.addInterceptor` method. Let’s add this overriding method to our config class `I18nConfig`. For example:

```java
@Override
public void addInterceptors(InterceptorRegistry interceptorRegistry) {
    interceptorRegistry.addInterceptor(localeChangeInterceptor());
}
```

## Create a Controller

Add a class named `WelcomeController` within the same package and annotate it with `@Controller`. This will mark this class as a [Spring Controller](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html) which holds Controller endpoints on Spring MVC architecture, as below:

Now, let’s add a GET mapping to the root URL. Add this to `WelcomeController`:

```java
@GetMapping("/")
public String welcome() { // <--- 1
    return "welcome"; // <--- 2
}
```

1. The method name is insignificant here since the Spring IoC Container resolves the mapping by looking at the annotation type, method parameters, and method return value.
2. The `welcome` View is called by the Controller.

## Test Functionality

Let’s see if our Spring Boot application correctly performs internationalization. Run the project, then open up a browser and hit the GET mapping URL we coded on our application’s Controller, which in this case would be the root URL `localhost:8080/`. Click different ‘language switch’ buttons to see if the page now reloads with its content properly localized in the requested locale.

![Output English](/assets/images/post/20221128/output1.png "Output English")

![Output French](/assets/images/post/20221128/output2.png "Output French")

As a nifty bonus, switch to one locale, close and reopen the browser, and navigate to the root URL again; since we used [CookieLocaleResolver](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/i18n/CookieLocaleResolver.html) as our `LocaleResolver` implementation, you’ll see that the chosen locale choice has been retained.

## Conclusion

In conclusion, in this tutorial, we looked into how we can localize to several locales and integrate internationalization into a Spring Boot project. We learned how to perform simple translations using `LocaleResolver` and `LocaleChangeInterceptor` classes to resolve languages using the details of incoming HTTP requests, and how we can switch to a different language at the click of a button in our internationalized Spring Boot web application.

And with that, I’ll be signing off. Don’t hesitate to leave a comment if you have any questions.

Till we meet again, have a safe day and happy coding!
