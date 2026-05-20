---
title: "Spring IoC Container Part 1"
date: 2016-10-10
slug: "spring4-1"
tags:
  - "Java"
  - "Spring 4"
categories:
  - "Spring 4"
---

Let’s look a simple code within Spring below:

```java
 // create and configure beans
ApplicationContext context =
    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

First of all, we should know the class hierarchy in `org.springframework.context.support`:

![](http://www.plantuml.com/plantuml/svg/fP7D2i8m3CVFxYbwd8Xx1zdLW-X540zPpMQnQqiIFs3uxcwh5qLbmq03JVx9Bqq3w1DKg3nL6GSohYe9QhnwEcNQEy6RP7mEmPCqIQ9QCssGIj2eSkzAKvq92ekB4ApH8CQxF9QfffTtDhjuSr249I4VqsaVl_SU2XO2BKfRO5QBv_L3DrC7YpKwuN-v8y7DWB9rH7mPg3te3hyW8nbn0GD8F-UAqCby8zstx_PAzssNFgeufcUCahycqZVXaCLkQpjktczlR9B9anG1UUTN8CTXXC-gs3IvFm00)

`ClassPathXmlApplicationContext` implements `ApplicationContext` and calls its constructor as below:

```java
 /**
 * Create a new ClassPathXmlApplicationContext, loading the definitions
 * from the given XML file and automatically refreshing the context.
 * @param configLocation resource location
 * @throws BeansException if context creation failed
 **/
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
	this(new String[] {configLocation}, true, null);
}
```

Step into `ClassPathXmlApplicationContext(String[], boolean, ApplicationContext)`：

```java
/**
 * Create a new ClassPathXmlApplicationContext with the given parent,
 * loading the definitions from the given XML files.
 * @param configLocations array of resource locations
 * @param refresh whether to automatically refresh the context,
 * loading all bean definitions and creating all singletons.
 * Alternatively, call refresh manually after further configuring the context.
 * @param parent the parent context
 * @throws BeansException if context creation failed
 * @see #refresh()
 **/
public ClassPathXmlApplicationContext(String[] configLocations, ￼boolean refresh, ApplicationContext parent)
		throws BeansException {

	super(parent);
	setConfigLocations(configLocations);
	if (refresh) {
		refresh();
	}
}
```

In this method, it first call `super(parent)`, which is `AbstractXmlApplicationContext(parent)` –> `AbstractRefreshableConfigApplicationContext(parent)` –> `AbstractRefreshableApplicationContext(parent)` –> `AbstractApplicationContext(parent)`.

Then, for the `setConfigLocations(configLocations)`, we need move to `AbstractRefreshableConfigApplicationContext`, the superclass of `ClassPathXmlApplicationContext`. It has `setConfigLocations(configLocations)` that deals with the locations String and save to its private value `String[] configLocations`.

Now, we get to the `refresh()` method belong to `AbstractApplicationContext` that implements the method in the interface `ConfigurableApplicationContext`

```java
/**
 * Load or refresh the persistent representation of the configuration,
 * which might an XML file, properties file, or relational database schema.
 * <p>As this is a startup method, it should destroy already created singletons
 * if it fails, to avoid dangling resources. In other words, after invocation
 * of that method, either all or no singletons at all should be instantiated.
 * @throws BeansException if the bean factory could not be initialized
 * @throws IllegalStateException if already initialized and multiple refresh
 * attempts are not supported
 */
 @Override
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
		// Prepare this context for refreshing.
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// Prepare the bean factory for use in this context.
		prepareBeanFactory(beanFactory);
		// remainder...
		}
}
```

For this line call `obtainFreshBeanFactory()` that then calls `refreshBeanFactory()`, subclass `AbstractXmlApplicationContext` to refresh bean factory and load bean definitions by calling `loadBeanDefinitions(beanFactory)`, and then it calls `loadBeanDefinitions(reader)`, in which reader is `XmlBeanDefinitionReader` implements `BeanDefinitionReader`. This class loads a DOM document and applies the reader to it. The document reader will register each bean definition with the given bean factory, talking to the latter’s implementation of the `BeanDefinitionRegistry` interface.

```java
// Tell the subclass to refresh the internal bean factory.
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

The reader has the `loadBeanDefinitions(configLocations)` method and the inner method of `XmlBeanDefinitionReader` provides a `loadBeanDefinitions(encodedResource)` where `EncodedResource` is the resource descriptor for the XML file, allowing to specify an encoding to use for parsing the file.

Method `doLoadDocument(inputSource, resource)` actually load bean definitions from the specified XML file. `DefaultDocumentLoader` that implements `DocumentLoader` loads a `Document` document from the supplied `InputSource` source by calling method `loadDocument(InputSource, EntityResolver, ...)` (that is a classical design pattern).

Further topics: `InputSource` of Spring Resource,`ThreadLocal` of Java SE concurrency, `context.getBean(...)` of how to get instantialization of service.
