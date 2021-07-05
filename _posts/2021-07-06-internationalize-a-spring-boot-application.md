---
title: "Internationalize a Spring Boot Application"
categories: [spring-boot]
date: 2021-07-06 00:00:00 +1000
modified: 2021-07-06 00:00:00 +1000
author: Edoardo Fiorini
excerpt: "Let's learn one of the most fundamental practices to make our spring boot application more accessible and inclusive: internationalization. In this tutorial, I'm going to guide you through applying this feature by creating a working example that localizes visitors and provides the appropriate translation to them. Users can also change the language."
---

## Introduction
Let's learn one of the most fundamental practices to make our spring boot application more accessible and inclusive: internationalization.
In this tutorial, I'm going to guide you through applying this feature by creating a working example that localizes visitors and provides
the appropriate translation to them. Users can also change the language.

## Code Example
This article is accompanied by a working code example on [Github](https://github.com/thombergs/code-examples/internationalize-a-spring-boot-application).

## Why to Internationalize an Application?
For brevity, we often refer to internationalization as "i18n" as it contains 18 letters between "i" and "n" and to "localization", the practice of getting user location, as "l10n", as it contains 10 letters between "l" and "n".
Supporting multiple languages has great effects: our application is going to spread faster and users will feel at home.

## Adding the Dependencies
Start off by creating a new Spring Boot project and adding the following dependencies to the ```pom.xml```:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## Translation Setup
The standard way for organizing translations in Java is to have all the strings in a ```.properties``` file. That's what we are going to do, too.
Create a ```lang``` folder in src/main/resources and add three files:

+ ```res.properties``` this file contains the standard translation (in English):

```properties
greeting=Good Morning
```
+ ```res_en.properties``` this file contains the translation for English:

```properties
greeting=Good Morning
```

+ ```res_de.properties``` this file contains the translation for German:

```properties
greeting=Guten Morgen
```

## Translation on Console

Let's now test the translation at the console stage. Add this code to the ```Application.java``` file:

```java
@SpringBootApplication
@EnableAutoConfiguration

public class Application implements WebMvcConfigurer { // remember to implement WebMvcConfigurer

      public static void main(String[] args) throws IOException, InterruptedException {

      SpringApplication.run(Application.class, args);

      ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();

      messageSource.setBasenames("lang/res");

      System.out.println(messageSource.getMessage("greeting", null, Locale.getDefault()));

      }

}
```

In the above class, we created a new ResourceBundleMessageSource instance, set its basename to our default translation file, and printed the ```greeting``` value
corresponding to the user's default locale. Setting the basename is fundamental here because Spring Boot will try to compose the path where to look for the translation by
adding the user's default locale to ```lang/res``` plus ```.properties``` at the end. For example, if the user's default locale returns ```en```, Spring Boot
is going to search, into the ```lang``` folder, for the ```res_en.properties``` file.

## Translation on Web
The final step is to do the same procedure but on the web stage. Let's add these two beans to the ```Application``` class:

```java
@Bean

public LocaleResolver localeResolver() {

      CookieLocaleResolver localeResolver = new CookieLocaleResolver();

      localeResolver.setDefaultLocale(Locale.getDefault());

      return localeResolver;

}
```

```java
@Bean
public LocaleChangeInterceptor localeChangeInterceptor() {

      LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();

      localeChangeInterceptor.setParamName("languagecookie");

      return localeChangeInterceptor;

}
```

Always in the ```Application``` class add this:

```java
@Override
public void addInterceptors(InterceptorRegistry interceptorRegistry) {
      interceptorRegistry.addInterceptor(localeChangeInterceptor());
}
```

Now, we will create a ```greeting.html``` file in the ```src/main/resources/templates``` folder:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>App</title>
</head>
<body>
<p style="text-align: center; font-size: 30px;" th:text="#{greeting}"></p>

<p style="text-align: center;">
<a href="?languagecookie=en">English</a>
</p>

<p style="text-align: center;">
<a href="?languagecookie=de">Deutsche</a></p>
</body>
</html>
```

And a GreetingController class:

```java
@Controller
public class GreetingController {

@GetMapping("/")
public String home() {
      return "greeting.html";
}

}
```

Finally, add this line to your application.properties:
```properties
spring.messages.basename=lang/res
```

Let's now run the program. The output you'll see is "Good Morning" in your language, plus two links to change language either to English or german. Let's explain how we did this.
Whenever someone visits a page of the app, we set a new cookie, ```languagecookie```, which stores the user default language (this happens in the localeResoler bean). In the
HTML template we ask Spring Boot to get the ```greeting``` value from the translation file. Also, we give users the possibility of changing language by providing two links to
```?languagecookie=en``` and ```?languagecookie=de```.
If user language is not yet supported by our app the default translation, ```res.properties``` will load.

## Adding New Languages
Unfortunately German and English don't cover the world population, but adding support for new languages with i18n is very easy.
Let's add a third language, say Dutch. Simply create a new ```res_nl.properties``` file under the ```lang``` folder:

```properties
greeting=Goedemorgen
```

That's it. Now Dutch visitors, who formerly got a standard English translation, are going to get a Dutch language page.

## Conclusion

In this article, we learned basic concepts of internationalization and localization by putting them into practice in a brand new spring boot application.
As an exercise, you can try to implement more languages by yourself. You can do an even better job by providing a link to
switch languages.

You can refer to all the source code used in the article on [Github](https://github.com/thombergs/code-examples/internationalize-a-spring-boot-application).
