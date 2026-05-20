---
title: "Spring IoC Container Part 3"
date: 2016-10-24
slug: "spring4-3"
tags:
  - "Java"
  - "Spring 4"
categories:
  - "Spring 4"
---

## DefaultSingletonBeanRegistry

Last time we find that in the `DefaultListableBeanFactory` it provides map to save bean definitions. But when getting bean instance, we need to step into `DefaultSingletonBeanRegistry`. First let’s see the document:

> Generic registry for shared bean instances, implementing the `SingletonBeanRegistry`. **Allows for registering singleton instances that should be shared for all callers of the registry, to be obtained via bean name**.

> Also supports registration of `DisposableBean` instances, (which might or might not correspond to registered singletons), to be destroyed on shutdown of the registry. Dependencies between beans can be registered to enforce an appropriate shutdown order.

> This class mainly serves as base class for `BeanFactory` implementations, factoring out the common management of singleton bean instances. Note that the `ConfigurableBeanFactory` interface extends the `SingletonBeanRegistry` interface.

> Note that this class assumes neither a bean definition concept nor a specific creation process for bean instances, in contrast to `AbstractBeanFactory` and `DefaultListableBeanFactory` (which inherit from it). Can alternatively also be used as a nested helper to delegate to.

In the class, it has private variable to cache the bean instance:

|  |  |
| --- | --- |
| ``` 1 2 ``` | ``` /** Cache of singleton objects: bean name --> bean instance */ private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256); ``` |

## AbstractBeanFactory

Again `AbstractBeanFactory` is very important `BeanFactory` implementation to get beans, and it is also a `SingletonBeanRegistry` interface implementation to save beans.

|  |  |
| --- | --- |
| ``` 1 2 3 4 ``` | ``` protected <T> T doGetBean(String name,                           Class<T> requiredType,                           Object[] args,                           boolean typeCheckOnly) ``` |

When `ApplicationContext` calls `doGetBean(...)`, it will call `getSingleton(...)` which is declared in `SingletonBeanRegistry` interface and implements in `DefaultSingletonBeanRegistry`.

> Return the (raw) singleton object registered under the given name.  
> Only checks already instantiated singletons; does not return an Object for singleton bean definitions which have not been instantiated yet.

> The main purpose of this method is to access manually registered singletons (see `registerSingleton(java.lang.String, java.lang.Object)`). Can also be used to access a singleton defined by a bean definition that already been created, in a raw fashion.

> NOTE: This lookup method is not aware of FactoryBean prefixes or aliases. You need to resolve the canonical bean name first before obtaining the singleton instance.

In this method, if the bean exists in concurrent map `singletonObjects<String, Object>` of `DefaultSingletonBeanRegistry`, it will call `getSingleton(...)` and return the bean object; Else the bean isn’t defined, the method will also call `getSingleton(...)`, if the singleton bean is null then it will generate the object of bean.

## Create Object of Bean

How does spring create object from bean definition file? In `AbstractBeanFactory`, it provide an abstract bean creation method:

|  |  |
| --- | --- |
| ``` 1 2 ``` | ``` protected abstract Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) 			throws BeanCreationException; ``` |

Also see the document:

> **Create a bean instance for the given merged bean definition (and arguments).** The bean definition will already have been merged with the parent definition in case of a child definition.

> All bean retrieval methods delegate to this method for actual bean creation.

This method is implemented in `AbstractAutowireCapableBeanFactory`:

> Abstract bean factory superclass that implements default bean creation, with the full capabilities specified by the `RootBeanDefinition` class. Implements the `AutowireCapableBeanFactory` interface **in addition to AbstractBeanFactory’s `createBean(java.lang.Class<T>)` method.**

> Provides bean creation (with constructor resolution), property population, wiring (including autowiring), and initialization. Handles runtime bean references, resolves managed collections, calls initialization methods, etc. Supports autowiring constructors, properties by name, and properties by type.

> The main template method to be implemented by subclasses is `AutowireCapableBeanFactory.resolveDependency(DependencyDescriptor, String, Set, TypeConverter)`, used for autowiring by type. In case of a factory which is capable of searching its bean definitions, matching beans will typically be implemented through such a search. For other factory styles, simplified matching algorithms can be implemented.

> Note that this class does not assume or implement bean definition registry capabilities. See `DefaultListableBeanFactory` for an implementation of the `ListableBeanFactory` and `BeanDefinitionRegistry` interfaces, which represent the API and SPI view of such a factory, respectively.

Now we just move to `createBean(...)` method and its main method is `doCreateBean(...)`:

> Actually create the specified bean. Pre-creation processing has already happened at this point, *e.g. checking* `postProcessBeforeInstantiation` *callbacks*.

> Differentiates between default bean instantiation, use of a factory method, and autowiring a constructor.

So we can know from the document that the method just create the specified bean which means it should get class reference in the definition file.

In this method, we find `BeanWrapper` class which is the central interface of Spring’s low-level JavaBeans infrastructure. The wrapper is initialized with `createBeanInstance(...)` method which can create a new instance for the specified bean, using an appropriate instantiation strategy: factory method, constructor autowiring, or simple instantiation. Then it call the core instantiation method `instantiateBean(...)` and it will step into `InstantiationStrategy` interface.

## InstantiationStrategy

> Interface responsible for creating instances corresponding to a root bean definition.

> This is pulled out into a strategy as various approaches are possible, including using CGLIB to create subclasses on the fly to support Method Injection.

Here we just talk about its `SimpleInstantiationStrategy` implement class. it has `instantiate(...)` which can return an instance of the bean with the given name in this factory. First it get the `Class<?> clazz` of the bean, and call `clazz.getDeclaredConstructor(...)` to get the `Constructor<?>` of the bean, then step into `BeanUtils` class and call `instantiateClass(...)` method which can call `constructor.newInstance(args)` and return a new object created by calling the constructor this object represents.

Finally, the object will be saved in the `singletonObjects<String, Object>` for `getBean(...)` method to get the object.

## What’s Next?

After reading the source code of Spring’s IoC, I find some TODO list.

* `ThreadLocal<?>`, `ConcurrentMap<K, V>` and *volatile* are often appeared in the source, so I need to deal with something about Java Threads;
* [Spring Reference](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/) is very useful document to read thus I need to focus more with it.
