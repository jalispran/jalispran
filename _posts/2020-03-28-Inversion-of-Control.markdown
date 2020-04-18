---
title: "Inversion of Control"
layout: post
date: 2020-03-28
tag: 
    - Inversion of Control
    - Dependency Injection
    - Design Pattern
    - Source Code Architecture
description: "This is my take on Inversion of Control"
category: blog
author: pranjal
---

### Disclaimer

I recently understood what Inversion of Control means. Now, I admit that I may be not-so-accurate on some concepts, so I advise you to use your best judgement. Also, feel free to reach out to me in case you want to discuss these concepts or just want to criticize me about writing this terrible piece of article.

Since I am predominantly a Java Developer and I have substantial experience with Spring and Spring Boot, I will use these in the example. However, the concepts remain the same for other languages and other frameworks as well.

This blog post touches upon *dependency injection* as well. However, I won't be discussing the various injection types in here.

### Example

Let us not reinvent the wheel and just take two existing examples as our boiler plate:

1. [Hello World in Java][1]
2. [Hello World in Spring Boot][2]

In the first example, its hardly 5 lines of code. There is a `main` method that prints our message to the console. In contrast, its a much more complicated code in the second example. In second example, you need to define a controller first. This controller then prints the message. Why the difference?

The reason being, in the first example, you are in total control of how the user interacts with the application (let us call this *Main Control of the Application*). Hence, you can just show the message on the console right away. However, in the second example, its a much more complicated setup. You no longer control how the user interacts with the application. Here spring is in control of how the user interacts with the application (using a web browser in this specific example). Now, you have to work in accordance with the rules that Spring has laid down. Did you notice the difference? The *Main Control of the Application* is inverted here. The *Main Control of the Application* is moved away from the application developer to the framework. This can be termed as one type of *Inversion of Control*.  

### Yet Another Example (Application's perspective)

Let us dive into another example that will be more helpful.

Let us say I want to create a plugin that other people will use in their applications. I provide some functionalities as a part of my plugin. I expect application developers who use my plugin to implement some functionalities on which my plugin depends. These functionalities can be anything like fetching data from a database or reading a QR Code etc. Naturally, what I do is, I write an interface for the functionality that I expect the application developers to implement. The application developers will typically write a class that implements this interface.

> Interfaces form a contract between the class and the outside world - [Java Docs][3]

![Example Scenario](..\assets\inversion-of-control\example-ioc.jpg)

Now, the question is, **HOW** does my plugin will come to know about the existance of this new implementation class? In other words, **HOW** do I inject this implementation class into plugin.jar? There are two approaches here:

1. either the application developer writes `Contract contract = new ContractImpl();` somewhere in their code and provide it to the appropriate class in plugin.jar
2. or the Spring Container somehow finds the implementation class and provides it to the appropriate class in plugin.jar

In the first approach, the **application developer controls HOW** the plugin finds the implementation class, whereas, in the second approach the **container controls HOW** the plugin finds the implementation class. This is called **Inversion of Control**. 

> The control of **HOW** an implementation class is found, is taken from the developer and is given to the framework/container. 

Let me try and be more clear.

The problem we have is **HOW** do I (plugin developer) make that link between an interface and its implementation so that my plugin is ignorant of `ContractImpl` but can still talk to `ContractImpl` to do its work. Let me declare what I mean by the term "ignorant" in this context. By "ignorant" I mean, my plugin does not `import` the implementation class. Otherwise, I (plugin developer) can not redistribute my code as a plugin.

What we are essentially doing here is, we are trying to inject a dependency, `ContractImpl`, into plugin.jar. 

> "Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various Inversion of Control advocates we settled on the name Dependency Injection." - [Martin Fowler][4]

### Yet Another Example (Container's perspective)

For containers like Spring, the inversion is about **HOW** they lookup a plugin implementation (implementation of an interface defined in the plugin). Needless to say, there can be multiple such plugins in any given application.

So the core problem we (Spring container) are trying to solve is **HOW** do we (Spring container) assemble these plugins into an application? There are two approaches here:

1. Using *Dependency Injection* 
2. Using *Service Locator*

The important difference between the two patterns (Dependency Injection and Service Locator) is about **HOW** that implementation is provided to the application class. 

- With *service locator* the application developer asks for the implementation class explicitly. This is done by a message to the locator, something like a `getService(Contract.class)`.
- With injection there is no explicit request. The service just appears in the application class, something like `@Autowired Contract contract`. Hence *inversion of control*. The control is moved from application developer to the framework.

### About Martin Fowler's Definition

Martin Fowler, in one of his blog posts, says, "*Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various Inversion of Control advocates we settled on the name Dependency Injection.*" However, I am not fully convinced by this.

A more appropriate statement would be, I think, "*Inversion of Control is the way containers, like Spring, implement Dependency Injection*". As I understand, *Inversion of Control* is applicable only in cases where containers and plugins are involved. However, in an application where no plugins are involved, I can use dependency injection but I won't be inverting the control (the way an implementation class is looked up).

I may be wrong about this statement but I am open to discusion. Feel free to get in touch if you wish to discuss this.

### Enough chitchat

Enough chitchat, let me show you some code now.

{% gist jalispran/088ee3c4da5f74cdcd8914f8ec48914b %}

`PluginApplication.java` and `Contract.java` belong to plugin project and hence will be packaged into `plugin.jar` However, `Application.java` belongs to application project and hence will be packaged into `application.jar`

As we can see, plugin does not provide a concrete implementation of `Contract.java` Providing a concrete implementation of the `Contract` interface is the responsibility of application project. and hence the `@Bean` annotation is used.

In `PluginApplication.java`, Spring will take care of finding the implementation of `Contract` interface. The developer need not ask for/find the implementation. The control of **HOW** this implementation is found is taken away from the developer and to the framework. Whereas in a service locator pattern, the plugin developer would have had to say `getService(Contract.class)`. This is *Inversion of Control*.

You can head over to [github][5] and check out this code.



[1]: https://www.javatpoint.com/simple-program-of-java	"Hello World Java tutorial"
[2]: https://dzone.com/articles/hello-world-program-spring-boot	"Hello World Spring Boot tutorial"
[3]: https://docs.oracle.com/javase/tutorial/java/concepts/interface.html	"What Is an Interface?"
[4]: https://martinfowler.com/articles/injection.html#InversionOfControl	"Inversion Of Control - Martin Fowler"
[5]: https://github.com/jalispran/inversion-of-control	"Github Project"

