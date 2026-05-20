---
title: "4.3  Access Resource"
date: 2016-09-28
slug: "2016-01-22-43--access-resource"
tags:
  - "Java"
  - "Spring 3"
categories:
  - "Spring 3"
---

### Access Resource

#### 1. *ResourceLoader*

Spring has a ***DefaultResourceLoader*** which can return ***ClassPathResource***, ***UrlResource***, and ***ServletContextResourceLoader*** which extends all functions of ***DefaultResourceLoader***.

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 ``` | ``` @Test public void testResourceLoad() throws IOException { 	ResourceLoader loader = new DefaultResourceLoader(); 	Resource resource = loader.getResource("classpath:cn/javass/spring/chapter4/test.txt"); 	Assert.assertEquals(ClassPathResource.class, resource.getClass()); } ``` |

In ***ApplicationContext*** also can implement ***ResourceLoader***:

* ***ClassPathXmlApplicationContext***
* ***FileSystemXmlApplicationContext***
* ***WebApplicationContext***

#### 2. *ResourceLoaderAware*

Injection via ***ApplicationContext***:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 ``` | ``` import org.springframework.context.ResourceLoaderAware; import org.springframework.core.io.ResourceLoader;  public class ResourceBean implements ResourceLoaderAware { 	private ResourceLoader resourceLoader;  	@Override 	public void setResourceLoader(ResourceLoader resourceLoader) { 		this.resourceLoader = resourceLoader; 	}  	public ResourceLoader getResourceLoader() { 		return resourceLoader; 	} } ``` |

|  |  |
| --- | --- |
| ``` 1 ``` | ``` <bean class="cn.javass.spring.chapter4.ResourceBean" /> ``` |

JUnit test:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 ``` | ``` @Test public void test() { 		ApplicationContext context = new ClassPathXmlApplicationContext("chapter4/resourceLoaderAware.xml"); 		ResourceBean bean = context.getBean(ResourceBean.class); 		ResourceLoader loader = bean.getResourceLoader(); 		Assert.assertTrue(loader instanceof ApplicationContext); } ``` |

#### 3. Resource Injection

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 12 13 ``` | ``` import org.springframework.core.io.Resource;  public class ResourceBean3 { 	private Resource resource;  	public Resource getResource() { 		return this.resource; 	}  	public void setResource(Resource resource) { 		this.resource = resource; 	} } ``` |

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 ``` | ``` <bean id="resourceBean1" class="cn.javass.spring.chapter4.ResourceBean3"> 	<property name="resource" value="cn/javass/spring/chapter4/test.properties"/> </bean>  <bean id="resourceBean2" class="cn.javass.spring.chapter4.ResourceBean3"> 	<property name="resource" value="classpath:cn/javass/spring/chapter4/test.properties"/> </bean> ``` |

JUnit test:

|  |  |
| --- | --- |
| ``` 1 2 3 4 5 6 7 8 9 10 11 ``` | ``` @Test public void test() { 		ApplicationContext context = new ClassPathXmlApplicationContext("chapter4/resourceInject.xml");  		ResourceBean3 resourceBean1 = context.getBean("resourceBean1", ResourceBean3.class);  		ResourceBean3 resourceBean2 = context.getBean("resourceBean2", ResourceBean3.class);  		Assert.assertTrue(resourceBean1.getResource() instanceof ClassPathResource); 		Assert.assertTrue(resourceBean2.getResource() instanceof ClassPathResource); } ``` |
