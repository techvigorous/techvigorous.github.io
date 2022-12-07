---
layout: post
title: Internationalization (i18n) and Localization (L10n) in a Spring standalone application
date: 2022-11-28 19:46
category: [Java, Spring]
author: dkansh
tags: [java, i18n, internationalization, Localization, Spring, Spring boot]
summary: Internationalization (i18n) and Localization (L10n) in a Spring standalone application
description: >-
    This blog takes you through all the steps you'll ever need to get started with implementing internationalization in any spring application! Internationalization is a process of preparing an application to support various linguistic, regional, cultural or political-specific data. It is an essential aspect of any modern multi-language application.
beforeToc: >-
    <p>A the previous blog we looked into the <a href="/java-internationalization-the-basics">basics of Java i18n</a>. In this blog, let’s take a step into the framework realm and see how the Spring framework handles internationalization i18n and Localization (L10n).</p>

    <p>For Any web application with users all around the world, internationalization (i18n) or localization (L10n) is very important for better user interaction. Most of the web application frameworks provide easy ways to localize the application based on user locale settings. Spring also follows the pattern and provides extensive support for internationalization (i18n) through the use of Spring Resource Bundles for different locales.a</p>
toc: true
---

## What is Internationalization (i18n)

Internationalization is a process of preparing an application to support various linguistic, regional, cultural or political-specific data. It is an essential aspect of any modern multi-language application.

For further reading, we should know that there's a very popular abbreviation (probably more popular than the actual name) for internationalization – i18n due to the 18 letters between ‘i' and ‘n'.

When developing a web application, we tend to code it using a collection of the most efficient, the most popular, and the most sought-after programming languages for both our front end and back end. But what about spoken languages? Most of the time, with or without our knowledge, we depend on the built-in translation engines of our customers’ browsers to handle the required translations. Don’t we?

In the ever-globalizing world we live in, we need our web applications to reach as wide an audience as possible. Here enters the much-required concept of internationalization. In this blog, we will be looking at how i18n works on the popular Spring framework.

## What is Localization (L10n)

Within Java, we have a fantastic feature at our disposal called the Locale class.

It allows us to quickly differentiate between cultural locales and format our content appropriately. It's essential for the internationalization process. The same as i18n, Localization also has its abbreviation – L10n.

The main reason for using Locale is that all required locale-specific formatting can be accessed without recompilation. An application can handle multiple locales at the same time so supporting a new language is straightforward.

Locales are usually represented by language, country, and variant abbreviation separated by an underscore:

- de (German)
- it_CH (Italian, Switzerland)
- en_US_UNIX (United States, UNIX platform)

## Prerequisites

- Spring Framework 5.2.7+
- Spring Boot 2.3.1+
- Java SDK 8+
- Gradle 5.0+

## I18n on Spring Boot

First off, let us create a simple Spring Boot example project using Gradle to get a grasp of how internationalization works in Spring.

Let’s go ahead and make a new Spring Boot application named java-i18n-spring-boot. To achieve this, head over to [Spring Initializr](https://start.spring.io/) and generate a new Spring Boot project with the following setup:

```text
Group:     com.techvigorous
Artifact:  java-i18n-spring-standalone
Packaging: Jar
Java:      17
```

Save the generated ZIP file and extract it to a local directory of your choice. Next, simply start your favorite IDE and open the extracted `java-i18n-spring-standalone` project as a Gradle project.

The source code is available on [GitHub](https://github.com/dkansh/java-i18n-spring-standalone).

## Add Language Resources

Firstly, we need to add some language resources to our app. In the project resources, create a new i18n directory. Create a simple Java properties file messages.properties inside this, and add a few words or sentences that you plan to internationalize on your application:

```text
hello=Hello!
welcome=Welcome to Tech Vigorous.
passion=I love coding.
```

Here 'messages' will act as the base name for our set of language resources. Make sure to take note of this term as we will be using it quite frequently as we advance through the tutorial. Also, note that your base name can be whatever you like.

messages.properties portrays the default resource file that our Spring application will resort to in the case that no match was found.

Secondly, let’s add another messages_fr.properties file to the same lang directory to hold localization resource data for a French locale. Duplicate the same keys of the default resource file on the messages_fr.properties file. As for the values, add the corresponding French translations according to each of those keys as follows:

```text
hello=Bonjour!
welcome=Bienvenue sur Tech Vigorous.
passion=J'adore coder.
```

## Meet MessageSource

Spring developers created the [MessageSource](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/MessageSource.html) interface for internationalization purposes within Spring applications. We will be using its `ResourceBundleMessageSource` implementation for our language resource resolving purposes.

`ResourceBundleMessageSource` acts as somewhat of an extension to the standard [ResourceBundle](https://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html) class in Java. If you take a quick look into its source code, you’ll notice it uses the Java inbuilt [Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html) class-type parameters within its methods.

### Language Resource Naming Rules

I’m sure by now you must have wondered.

What’s with all the 'messages', 'messages_fr' blah blah blah? Can’t I just name them whatever I want?

A glance at the source code of [Spring Boot’s Auto-configuration class for MessageSource](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/context/MessageSourceAutoConfiguration.java) itself answers this question, see below:

```java
String basename = context.getEnvironment().getProperty("spring.messages.basename", "messages");
```

As you can see, Spring picks the base name from a Spring property named `spring.messages.basename`, or in the absence of this, it tries to pick it up from a properties file named `messages.properties`. Therefore, since none of our language resource files are named `messages`, we will have to add a Spring property to inform Spring where and what our base name is.

A new `spring.messages.basename` property in the `application.properties` file indicating the base name for the language resources. Make sure to include the path within the `resources` directory leading to this default resource file, like so:

```properties
spring.messages.basename=i18n/messages
```

Since the `ResourceBundleMessageSource` class relies on the underlying JDK’s `ResourceBundle` implementation, you will also have to follow the same [naming rules as used by Java’s built-in i18n functions](/java-internationalization-the-basics/#rules-to-follow) when naming language resource files:

- All resource files must reside in the same package.
- All resource files must share a common base name.
- The default resource file should simply have the base name:

```text
messages.properties
```

- Additional resource files must be named following this pattern:

```text
base name _ language suffix
messages_fr.properties
```

- Let’s assume that at least one resource file with a language suffix already exists. However, for a particular language, you might like to narrow down the target locale to specific countries as well. In this case, you can add more resource files with additional country suffixes. For example:

```text
base name _ language suffix _ country suffix
messages_fr_CA.properties
```

- Likewise, following the same logic, you may narrow it down to resource files with an additional variant suffix as well. For instance:

```text
base name _ language suffix _ country suffix _ variant suffix
messages_fr_CA_CA.properties
```

### Test Out ResourceBundleMessageSource

Let’s see how ResourceBundleMessageSource works. Insert this code snippet into the main method of the java-i18n-spring-boot application and run it:

```java
ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
messageSource.setBasenames("i18n/messages");

System.out.println(messageSource.getMessage("hello", null, Locale.FRENCH));
```

You’ll see it successfully retrieves the localized value for the 'hello' key and prints it out on the console.

## Conclusion

In conclusion, in this blog, we looked into how we can localize to several locales and integrate internationalization into any Spring project (including the Spring standalone project). We learned how to perform simple translations using MessageSource implementations to resolve languages using the Locale passed to ResourceBundleMessageSource, and how we can switch to a different language in our internationalized Spring application.
