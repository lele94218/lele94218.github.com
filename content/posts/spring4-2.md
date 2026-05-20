---
title: "Spring IoC Container Part 2"
date: 2016-10-21
slug: "spring4-2"
tags:
  - "Java"
  - "Spring 4"
categories:
  - "Spring 4"
---

## Log4J

When using Spring framework, we must use [Apache Common Logging](https://commons.apache.org/proper/commons-logging/) to print log in console or binary files. As we known, Spring log configuration can change in the `web.xml` when it works on server. But in these series of articles, we try to analyze the Spring core techniques without works on server. So there should have another way to configure log without `web.xml` because we want to see the debug informations.

Spring gives `Log4jConfigurer.initLogging()` method (has been deprecated in Spring 4.2.1) to initialize log4j from the given file location. Note that we always use Maven to build project, but maven might mash `src` and `resources` folders to one. Thus we should use `classpath:` ,**without slash**, to represent `resources` folder in the project.

Now Apache Commons Logging (JCL) provides thin-wrapper Log implementations for other logging tools, including Log4J. But `Log4jConfigurer.initLogging()` calls methods from log4j 1.x. It means that we should add log4j 1.x dependency to make sure that our log configuration file works smoothly.

Here is the configure file `log4j.properties`, and just opening debug function in `org.springframework.beans` because we want to know the IOC process:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 ``` | ``` log4j.rootCategory=INFO, stdout  log4j.appender.stdout=org.apache.log4j.ConsoleAppender log4j.appender.stdout.layout=org.apache.log4j.PatternLayout log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c:%L - %m%n  log4j.category.org.springframework.beans=DEBUG ``` |

As last article, we run the same code but add our log configuration before:

|  |  |
| --- | --- |
| ``` 1 2 3 ``` | ``` Log4jConfigurer.initLogging("classpath:log4j.properties"); ApplicationContext context = new ClassPathXmlApplicationContext("services.xml"); PetStoreServiceImpl petStoreService = context.getBean("petStore", PetStoreServiceImpl.class); ``` |

Now we can follow the log information to know the IOC clearly.

## ClassPathXmlApplicationContext

|  |  |
| --- | --- |
| ``` 1 ``` | ``` 17:23:15,265  INFO main ClassPathXmlApplicationContext org.springframework.context.support.AbstractApplicationContext.prepareRefresh:581 - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@42dafa95: startup date [Fri Oct 21 17:23:15 EDT 2016]; root of context hierarchy ``` |

First we step into `ClassPathXmlApplicationContext` constructor and it calls `refresh()`. This method is declared in the interface `ConfigurableApplicationContext`, implements in `AbstractApplicationContext`, finally called in `ClassPathXmlApplicationContext` which is a subclass of `AbstractApplicationContext`.

> This method can load or refresh the **persistent representation** of the configuration, which might an XML file, properties file, or relational database schema. As this is a startup method, it should destroy already created singletons if it fails, to avoid dangling resources. In other words, after invocation of that method, either all or no singletons at all should be instantiated.

As we see in the document, `refresh()` is used to load and refresh persistent representation configuration. It should be done first of all.

## XmlBeanDefinitionReader

|  |  |
| --- | --- |
| ``` 1 ``` | ``` 17:23:15,555  INFO main XmlBeanDefinitionReader org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions:317 - Loading XML bean definitions from class path resource [services.xml] ``` |

The `refresh()` method calls `obtainFreshBeanFactory()` in which can tell the subclass to refresh the internal bean factory. So how to let a class to call the method (in this case it is `refreshBeanFactory()`) in its subclass? It is a very classical design pattern.

First, the `AbstractApplicationContext` declares:

|  |  |
| --- | --- |
| ``` 1 ``` | ``` protected abstract void refreshBeanFactory() ``` |

This method is implemented in `AbstractRefreshableApplicationContext` which is a subclass of `AbstractApplicationContext`:

|  |  |
| --- | --- |
| ``` 1 2 ``` | ``` @Override protected final void refreshBeanFactory() ``` |

Therefore the father class can call the method which implemented in its subclass.

Here is `XmlBeanDefinitionReader`:

> Bean definition reader for XML bean definitions. Delegates the actual XML document reading to an implementation of the `BeanDefinitionDocumentReader` interface. Typically applied to a  
> `DefaultListableBeanFactory` or a `GenericApplicationContext`.

> This class loads a DOM document and applies the `BeanDefinitionDocumentReader` to it.

> The document reader will register each bean definition with the given bean factory, talking to the latter’s implementation of the `BeanDefinitionRegistry` interface.

`loadBeanDefinition()` method is used to load bean definitions from the specified XML file.

## DefaultDocumentLoader

`XmlBeanDefinitionReader` calls `doLoadDocument()` and it has local variable `DocumentLoader` documentLoader which calls `loadDocument()`.

|  |  |
| --- | --- |
| ``` 1 ``` | ``` 19:38:23,603 DEBUG main DefaultDocumentLoader org.springframework.beans.factory.xml.DefaultDocumentLoader.loadDocument:73 - Using JAXP provider [com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl] ``` |

Here is `DefaultDocumentLoader`:

> Spring’s default {@link DocumentLoader} implementation. Simply loads `Document` documents using the standard JAXP-configured XML parser. If you want to change the `DocumentBuilder` that is used to load documents, then one strategy is to define a corresponding Java system property when starting your JVM. For example, to use the Oracle `DocumentBuilder`, might start your application like as follows:
>
> `java -Djavax.xml.parsers.DocumentBuilderFactory=oracle.xml.jaxp.JXDocumentBuilderFactory MyMainClass`

## DefaultBeanDefinitionDocumentReader

Here is `DefaultBeanDefinitionDocumentReader` which is used to read bean definitions:

> Default implementation of the `BeanDefinitionDocumentReader` interface that reads bean definitions according to the “spring-beans” DTD and XSD format (Spring’s default XML bean definition format).

> The structure, elements, and attribute names of the required XML document are hard-coded in this class. (Of course a transform could be run if necessary to produce this format). `<beans>` does not need to be the root element of the XML document: this class will parse all bean definition elements in the XML file, regardless of the actual root element.

And `registerBeanDefinitions(doc, readerContext)`:

> Read bean definitions from the given DOM document and register them with the registry in the given reader context.

> `doc` the DOM document `readerContext` the current context of the reader (includes the target registry and the resource being parsed)

## DefaultListableBeanFactory

> Default implementation of the `ListableBeanFactory` and `BeanDefinitionRegistry` interfaces: a full-fledged bean factory based on bean definition objects.  
> **Typical usage is registering all bean definitions first (possibly read from a bean definition file), before accessing beans**. Bean definition lookup is therefore an inexpensive operation in a local bean definition table, operating on pre-built bean definition metadata objects.

> Can be used as a standalone bean factory, or as a superclass for custom bean factories. Note that readers for specific bean definition formats are typically implemented separately rather than as bean factory subclasses: see for example `PropertiesBeanDefinitionReader` and `XmlBeanDefinitionReader`.

> For an alternative implementation of the `ListableBeanFactory` interface, have a look at `StaticListableBeanFactory`, which manages existing bean instances rather than creating new ones based on bean definitions.

It implements the bean register method `registerBeanDefinition()` which is declared in interface `BeanDefinitionRegistry`.

> Register a new bean definition with this registry. Must support `RootBeanDefinition` and `ChildBeanDefinition`.

Now we look at `DefaultListableBeanFactory` and find that it has some private variables.

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 ``` | ``` /** Map of bean definition objects, keyed by bean name */ private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(256);  /** List of bean definition names, in registration order */ private volatile List<String> beanDefinitionNames = new ArrayList<String>(256); ``` |

Note that `beanDefinitionMap` is a `java.util.concurrent.ConcurrentHashMap` and `beanDefinitionNames` is a volatile variable.

|  |  |
| --- | --- |
| ``` 1 2 ``` | ``` this.beanDefinitionMap.put(beanName, beanDefinition); this.beanDefinitionNames.add(beanName); ``` |

TODO: `java.util.concurrent.ConcurrentHashMap` and volatile variable.
