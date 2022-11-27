---
layout: post
title: "Java internationalization (i18n): Translate your Java App/Website in multiple languages"
date: 2022-11-26 13:24
category: [java]
author: dk
tags: [java, i18n, internationalization, GoogleCloud]
summary: "Java internationalization (i18n): Translate your Java App/Website in multiple languages"
beforeToc: >-
    <p>Internationalization has become an invaluable method for capturing the attention of Java app users, making them feel right at home when using one. In this article, let’s look into how we can perform translations in our apps or websites and how to make sure our applications support multiple languages using Java internationalization.</p>

    <p>We will be covering the following topics in this tutorial:</p>
    <ul>
        <li>Java i18n.</li>
        <li><code>Locale</code> and <code>ResourceBundle</code> classes.</li>
        <li>Switching between locales.</li>
        <li>Pluralization.</li>
        <li>Date-time localization.</li>
        <li>Using Cloud Translation API.</li>
    </ul>
toc: true
---

## Internationalization in Java
So, you have an exciting idea for a Java app/website or maybe you use a seasoned Java application in your workplace. But what if your Java application plans to support multiple languages? It needs to operate swiftly, not just in the common English language, but also in a plethora of other languages.

That’s where Java internationalization comes in.

### How Java Supports Internationalization

Primarily we talk about three concepts when it comes to localization in Java. For localization purposes we will be needing:

-   [Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html) class objects representing the specific geographical, political, and cultural regions we plan to support.
-   Resources holding locale-specific data in the form of **classes or properties files**.
-   [ResourceBundle](https://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html) class objects fetching data for the relevant locales from the respective resources.

Note that these Java classes are already built into the `java.util` package.

### Rules to Follow

So, when adding language resources, it is essential to follow these rules:

-   All resource files must reside in the same package.
-   All resource files must share a common base name. Please remember this term as we’ll be using it later on in this tutorial. Note that your base name can be whatever you like.
-   The default resource should simply have the base name:
    > **bundle.properties** or **Bundle.java**
-   Additional properties files must be named following this pattern: `base name _ language suffix`
    > **bundle_fr.properties** or **Bundle_fr.java**
-   Let’s assume that at least one resource file with a language suffix already exists. However, for a particular language, you might like to narrow down the target locale to specific countries as well. In this case, you can add more resource files with additional country suffixes. For example: `base name _ language suffix  _ country suffix`
    > **bundle_en_US.properties** or **Bundle_en_US.java**
-   Likewise, following the same logic, you may narrow it down to resource files with an additional variant suffix as well. For instance: `base name _ language suffix _ country suffix _ variant suffix`
    > **bundle_th_TH_TH.properties** or **Bundle_th_TH_TH.java**

### Put Them to Use

Let’s build a basic Java localization example utilizing the previously mentioned features to get a proper idea of how internationalization works.

Create a new Java Application named `java-i18n`.

The source code is available on [GitHub](https://github.com/dkansh/java-i18n).

### Create a Resource Bundle

First off, let’s add some language resources for Java internationalization.

**Resources in Property File Form**

We can create simple Java `properties` files to hold our resources. These will be handled by the concrete class [PropertyResourceBundle](https://docs.oracle.com/javase/8/docs/api/java/util/PropertyResourceBundle.html) for Java localization purposes.

Create a new `resources` package within the main package and create a `bundle.properties` file inside it.

In the properties file, add a few words or sentences to be internationalized on your Java app in key-value form.

```properties
hello=Hello
welcome=Welcome to my app
ok=OK
cancel=Cancel
tryagain=Please try again
```

Here, `'bundle'` will act as the **base name**, and the `bundle.properties` file will act as the **default resource file** that our Java application will resort to in the case that no match is found.

Note that this is the serialized form in which we can keep our resource files. It provides us with the useful advantage of not requiring any project recompilations on resource file updates. Nonetheless, at the same time, the serialized form has the disadvantage of resource value types being restricted to strings.

**Resources in Java Class Form**

Alternatively, we can achieve the same result by creating the resource using a class extending [ListResourceBundle](https://docs.oracle.com/javase/8/docs/api/java/util/ListResourceBundle.html) abstract class.

In the `resources` package, create a `Bundle` class extending the `ListResourceBundle` class and override its `getContents` method.

```java
package resources;

import java.util.ListResourceBundle;

public class Bundle extends ListResourceBundle {
    @Override
    protected Object[][] getContents() {
        return new Object[][]{
                {"hello", "Hello"},
                {"ourfeatures", new String[]{"Collaborative Translation", "Localization Workflow Management", "Localization Process Automation"}}
        };
    }
}
```

Note that this resource is in a deserialized form and hence provides all the benefits of non-static objects such as fewer overheads and proper garbage collection to name but a few.

```java
{"ourfeatures", new String[] {"Collaborative Translation", "Localization Workflow Management", "Localization Process Automation"}}
```

The most notable advantage would be the ability to hold any type of object as a resource value unlike when using property files.

**Add More Resources**

Let’s say we need to localize our app to cater to Italian-speaking users as well. Simply add another properties file containing the same keys as in `bundle.properties` but with their Italian language counterparts as values.

In this case, let’s add a new `bundle_it_IT.properties` file in the same package and add the corresponding language values.

```properties
hello=Ciao
welcome=Benvenuti nella mia app
ok=OK
cancel=Annulla
tryagain=Per favore riprova
```

Like last time, instead of using a property file, we can create a `Bundle_it_IT` class to hold the resources as well. Make a new `Bundle_it_IT` class in the same package as follows:

```java
package resources;

import java.util.ListResourceBundle;

public class Bundle_it_IT extends ListResourceBundle {
    @Override
    protected Object[][] getContents() {
        return new Object[][] {
                {"hello", "Ciao"},
                {"ourfeatures", new String[] {"Traduzione Collaborativa", "Gestione del flusso di lavoro di localizzazione", "Automazione del processo di localizzazione"}}
        };
    }
}
```

The multiple resources together form a **resource bundle** that can be used for our i18n purposes.

### Add Some Locale Class Objects

Let’s create a Locale object within the main method of our `java-i18n` project. This will simply represent an Italian-speaking geographical region in Italy.

```java
Locale locale_it_IT = new Locale("it", "IT");
```

Refer to this list for [locales supported on Java 8](https://www.oracle.com/java/technologies/javase/jdk8-jre8-suported-locales.html).

### Use ResourceBundle Class

Now it’s time to fetch some language-specific data from our resource bundle for the program. Note that we must always provide the **fully-qualified class name** of the base name when providing the first parameter of the `ResourceBundle.getBundle` static method.

```java
ResourceBundle resourceBundle = ResourceBundle.getBundle("resources.bundle", locale_it_IT);
System.out.println(resourceBundle.getString("welcome"));
```

This should properly localize and print out the resource value corresponding to the `it-IT` locale.

But, what if someone asked for data from a locale that is absent from our resource bundle?

```java
Locale locale_ru_RU = new Locale("ru", "RU");
ResourceBundle resourceBundle = ResourceBundle.getBundle("resources.bundle", locale_ru_RU);
System.out.println(resourceBundle.getString("welcome"));
```

You’ll see that this prints out a value belonging to an en-US locale. This is because, as we discussed earlier when initially creating property resources, our Java application will always resort to the default resource file if no match was found. In this case, it would be the `bundle.properties` file.

### Switch Between Locales

How about we create a simple GUI app to get a visual idea of how locale switching looks in the User Interface of a Java I18n application? For this, let’s create a Java Swing GUI application that utilizes the language resources we created earlier. There will be an additional `Switch` button that switches the language of the Java app each time the user presses it.

Prerequisites

-   NetBeans IDE
-   Maven

I will be using NetBeans IDE 11.3 for Java GUI development purposes.

Create a new Maven project named `java-i18n-gui`. Create a `gui` package in the project and make a new JFrame form named `SwitchLang` inside it.

Add GUI Components
Let’s add some GUI components as follows:

-   First, add two `JLabel` components named `jLabelHello` and `jLabelWelcome`.
-   Next, add two `JButton` components named `jButtonOk` and `jButtonCancel`.
-   Finally, add another `JButton` named `jButtonSwitchLang` with a text value of `Switch`.

**Add Language Resources**

Now we need to create a resource bundle containing a few language resources and synchronize it with our `SwitchLang JFrame` form. Follow these steps to achieve this:

1.  First off [following Maven conventions](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html), click on the `Files` tab in Netbeans and create a new `resources` folder in the `src/main/` directory of the project. This `src/main/resources` path will automatically be marked as a `resource` directory the next time we build the project using Maven.
2.  Create a new `bundle.properties` file within `resources`. This will act as the default resource file of our resource bundle.
3.  Open the `SwitchLang` form from the `Projects` window and click on the `Design` tab in the `Editor`.
4.  In the Navigator section, in the lower-left corner, click on the root node in the upper corner. This should be mentioned as `Form SwitchLang` in our case.
5.  Next, in the `Properties` window on the right, look for an option called `Properties Bundle`. For its value browse `src/main/resources` and select the default resource file `bundle.properties` we created a little while ago.
6.  In the same window, mark the `Automatic Internationalization` option as checked.
7.  Open up the bundle.properties file again and replace its content as follows:

```properties
SwitchLang.jLabelHello.text=Hello
SwitchLang.jLabelWelcome.text=Welcome to my app
SwitchLang.jButtonOk.text=OK
SwitchLang.jButtonCancel.text=Cancel
SwitchLang.jButtonSwitchLang.text=Switch
```

Note that we simply set resource keys representing names of the Swing elements we created earlier and put their corresponding resource values on an `en-US` locale.

8.  Finally, create a new `bundle_it_IT.properties` file within resources and put these lines as its content:

```properties
SwitchLang.jLabelHello.text=Ciao
SwitchLang.jLabelWelcome.text=Benvenuti nella mia app
SwitchLang.jButtonOk.text=OK
SwitchLang.jButtonCancel.text=Annulla
SwitchLang.jButtonSwitchLang.text=Interruttore
```

Here we added a resource file on an `it-IT` locale.

Now, our resource bundle is nicely synced with the `SwitchLang JFrame` form elements.

**Code Switch Button Functionality**

Finally, we need to code the functionality of `jButtonSwitchLang`.

Double-click on the `jButtonSwitchLang` button in the `Editor` to get us into the `jButtonSwitchLangActionPerformed` private method which will hold the action performed upon clicking the `Switch` button. Let’s set the button click action to make the `java-i18n-gui` app change its locale to Italian.

First off, we need to retrieve the `it-IT` locale resource bundle. Code this in the `jButtonSwitchLangActionPerformed` private method:

```java
Locale locale_it_IT = new Locale("it", "IT");
ResourceBundle resourceBundleIT = ResourceBundle.getBundle("bundle", locale_it_IT);
```

Make sure to import the required classes from the `java.util` package.

As the final step, let’s add these lines to change the text of our `java-i18n-gui` Swing elements to the `it-IT` locale.

```java
jLabelHello.setText(resourceBundleIT.getString("SwitchLang.jLabelHello.text"));
jLabelWelcome.setText(resourceBundleIT.getString("SwitchLang.jLabelWelcome.text"));
jButtonOk.setText(resourceBundleIT.getString("SwitchLang.jButtonOk.text"));
jButtonCancel.setText(resourceBundleIT.getString("SwitchLang.jButtonCancel.text"));
jButtonSwitchLang.setText(resourceBundleIT.getString("SwitchLang.jButtonSwitchLang.text"));
```

That’s it! Run the app and press the Switch button to check that switching between locales works properly.

### A Few Other Java Internationalization Features

Let’s look at a few more Java features that could come in handy when dealing with localization in Java.

**Pluralization**

Handling the proper pluralization of text can become quite a necessity when dealing with Java i18n.

For instance, let’s assume we need to handle text representing some apples based on a provided quantity. So for the English language, it would take this form:

-   0 apples
-   1 apple
-   2 apples

We can code a Java method to handle this, like so:

```java
public static String getAppleCountMsg(int count) {
    if (count == 1) {
        return "1 apple";
    } else if (count == 0 || count > 1) {
        return count + " apples";
    } else {
        return "You don't count apples in minus numbers! Please input a whole number";
    }
}
```

However, as you can see, this is still a tedious amount of work just to handle some text about apples on a Java app.

**ChoiceFormat Helps a Bit**

Java provides a [ChoiceFormat](https://docs.oracle.com/javase/8/docs/api/java/text/ChoiceFormat.html) class that we can use to make the previous code look a bit less messy.

```java
public static ChoiceFormat getAppleCountMsgChoiceFormat() {
    double[] minAppleCount = {0, 1, 2};
    String[] appleCountFormat = {"apples", "apple", "apples"};  
    return new ChoiceFormat(minAppleCount, appleCountFormat);
}
```

The ChoiceFormat class accepts two parameters.

The first parameter holds an array of primitive doubles marking the starting points of a set of intervals. For instance, {0, 1, 2} in our example represents three intervals in ascending order:

-   The first interval ranges from 0 to 1 including double value 0 and excluding double value 1.
-   Similarly, the second interval ranges from 1 to 2 including 1 and excluding 2.
-   And finally, the third interval ranges from 2 to upwards including 2.

The second parameter holds a String array with values to be used according to each of those intervals. Therefore in our case, it should consecutively hold `apples`, `apple`, `apples`.

We can pass the `ChoiceFormat` object returned from `getAppleCountMsgChoiceFormat()` to a separate method like this to retrieve our apple count messages in much the same way.

```java
public static String getAppleCountMsg(ChoiceFormat appleCountChoiceFormat, int count) {
    return count + " " + appleCountChoiceFormat.format(count);
}
```

Note that since the pluralization forms for different languages could deviate from the pluralization form in English, you would eventually have to create separate `ChoiceFormat` objects manually for each of the languages you plan to support in your internationalized Java application.

**Date and Time**

We can use `getDateTimeInstance` method provided in the [DateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/DateFormat.html) class to handle localization in Java. For example, we can use the following to get the current date and time for a given locale in the SHORT date and time formatting style:

```java
public String getCurrentDateAndTime(final Locale locale) {
    DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT, locale);
    return dateFormat.format(new Date());
}
```

## Meet Cloud Translation API

As a bonus, let me show you how to work with Google Translate API in your Java app.

Google Cloud Platform provides an API, namely [Cloud Translation API](https://cloud.google.com/translate/docs), which we can easily employ for the previously mentioned use case of performing basic text translations.

Be sure to have [Maven](https://maven.apache.org/install.html) installed locally on your computer before continuing.

These will be the steps needed to integrate with the Google Cloud Platform:
1\. Create a Google Cloud project.
2\. Enable Google Cloud Translation API.
3\. Set up credentials.

Let’s see how we can do this one step at a time.

### Create Google Cloud Project

This step can be skipped if you have an existing Google Cloud project we can utilize for our translation purposes. Nevertheless, in case you don’t have an existing Google Cloud project, you can follow the steps mentioned in the [Google Cloud documentation](https://cloud.google.com/resource-manager/docs/creating-managing-projects) to create one.

**Enable Google Cloud Translation API**

Now we can get started, let’s enable the Cloud Translation API on our project. Head over to the [Translation API homepage](https://console.cloud.google.com/apis/library/translate.googleapis.com), select your Google Cloud project from the dropdown and click `Enable`.

**Set Up Credentials**

We have to set up some credentials to allow communications between Cloud Translation API and the `Translator` app to take place effectively. These authentication steps are described in the [Authentication section of the Google Cloud documentation](https://cloud.google.com/docs/authentication/getting-started).

### Simple Translation App

As an example, let’s create a basic Java application performing translations through the Cloud Translation API Java client library. We will be calling it the `Translator` application from here on out for easier reference.

The source code is available on [GitHub](https://github.com/dkansh/java-i18n-google-translate).

**Create Maven Project**

Create a new Maven project by running:

    mvn archetype:generate -DgroupId=com.techvigorous -DartifactId=java-i18n-google-translate -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

**Add Dependencies to Project**

Adding the following dependency will introduce the [Java client for Google Cloud Translation API](https://github.com/googleapis/java-translate) to our \`Translator project.

```xml
<dependency>
   <groupId>com.google.cloud</groupId>
   <artifactId>google-cloud-translate</artifactId>
   <version>2.6.0</version>
</dependency>
```

Note that the dependency version mentioned here might have changed over time. Refer to the [Maven repository](https://mvnrepository.com/artifact/com.google.cloud/google-cloud-translate/) to get the latest version.

**Translator Application**

```java
import com.google.cloud.translate.Translate;
import com.google.cloud.translate.Translate.TranslateOption;
import com.google.cloud.translate.TranslateOptions;
import com.google.cloud.translate.Translation;
public class Translator {
    public static void main(String[] args) {
        //Initiate google cloud translation service
        Translate translate = TranslateOptions.getDefaultInstance().getService(); //1
        String textToTranslate = "Localization in Java is fun";
        //Perform translation
        Translation translation = translate.translate(textToTranslate, TranslateOption.sourceLanguage("en"), //2
                                                      TranslateOption.targetLanguage("it"));
        String translatedText = translation.getTranslatedText();
        System.out.println(translatedText);
    }
}
```

1.  The `TranslateOptions.getDefaultInstance` method forces the Google Cloud Translation Java client library to retrieve credentials through the `GOOGLE_APPLICATION_CREDENTIALS` environment variable we set when setting up credentials for Cloud Translation API locally.
2.  Note that providing the source language here is optional since Cloud Translation API can automatically detect languages. Hence, the `TranslateOption.sourceLanguage("en")` parameter is optional; but just to be on the safe side, it is best to manually provide the source language to avoid misinterpretations.

Consequently, running this program should print out an Italian translation of the `textToTranslate` on the console.

See here for a list of [languages supported](https://cloud.google.com/translate/docs/languages) by Google Cloud Translation API.

## Conclusions on Java I18n

In this tutorial, we got the hang of Java internationalization and localization for your app or website. We discovered how to approach translations using Google Cloud Translation API, set up Translation API on a Google Cloud Project, and how we can carry out simple text translations using the Java client of Google Cloud Translation API on a simple Maven project.

Additionally, we looked into adding multi-language support to Java applications. We explored Java’s built-in Locale & ResourceBundle classes for Java localization purposes, and how we can utilize them in a basic Java internationalization example.

So, that marks the end of this tutorial. We hope we helped you get kick-started on Java localization through this article. Until next time, keep coding!
