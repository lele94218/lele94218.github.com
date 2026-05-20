---
title: "4.2  Using Resource"
date: 2016-09-28
slug: "2016-01-22-42--using-resource"
tags:
  - "Java"
  - "Spring 3"
categories:
  - "Spring 3"
---

### Using Resource

#### 1. ByteArrayResource

Junit test:

```java
import org.junit.Test;
import org.springframework.core.io.ByteArrayResource;
import org.springframework.core.io.Resource;

public class ResourceTest {

	@Test
	public void testByteArraryResorce() {
		Resource resource = new ByteArrayResource("Hello World!".getBytes());
		if (resource.exists()) {
			dumpStream(resource);
		}
		Assert.assertEquals(false, resource.isOpen());
	}

	private void dumpStream(Resource resource) {
		InputStream inputStream = null;
		try {
			inputStream = resource.getInputStream();
			byte [] descByte = new byte [inputStream.available()];
			inputStream.read(descByte);
			System.out.println(new String(descByte));
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			try {
				inputStream.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

**NOTE:** ***ByteArrayResource*** can be used repeatedly, because ***isOpen()*** always returns *false*.

#### 2. InputStreamResource:

```java
@Test
public void testInputStreamResource() {
		ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream("Hello World!".getBytes());
		Resource resource = new InputStreamResource(byteArrayInputStream);
		if (resource.exists()) {
			dumpStream(resource);
		}
		Assert.assertEquals(true, resource.isOpen());
}
```

**NOTE:** ***InputStreamResource*** can be used only one time, because ***isOpen()*** always returns *true*.

#### 3. FileSystemResource:

```java
@Test
public void testFileResource() {
		File file = new File("d:/test.txt");
		Resource resource = new FileSystemResource(file);
		if (resource.exists()) {
			dumpStream(resource);
		}
		Assert.assertEquals(false, resource.isOpen());
}
```

**NOTE:** ***FileSystemResource*** can be used repeatedly, because ***isOpen()*** always returns *false*.

#### 4. ClassPathResource:

***ClassPathResource*** replaces ***Class*** and ***ClassLoader*** to give a uniform implement. The resource can be used repeatedly, because ***isOpen()*** always returns *false*.

Here are three constructors in ***ClassPathResource***:

- ***public ClassPathResource(String path)***: using default ***ClassLoader*** to load resource.
- ***public ClassPathResource(String path, ClassLoader classLoader)***: using specified ***ClassLoader*** to load path resource.
- **_ public ClassPathResource(String path, Class<?> clazz)_**: using *Class* path to load resource.

> For example, if the Class path is “cn.javass.spring.ResourceTest”, the path is “cn/javass/spring/ResourceTest”.

1. ***public ClassPathResource(String path)***:

```java
@Test
public void testClasspathResourceBydefaultClassLoader() throws IOException {
		Resource resource = new ClassPathResource("cn/javass/spring/chapter4/test.properties");
		if (resource.exists()) {
			dumpStream(resource);
		}
		System.out.println("path:" + resource.getFile().getAbsolutePath());
		Assert.assertEquals(false, resource.isOpen());
}
```

1. ***public ClassPathResource(String path, ClassLoader classLoader)***:

```java
@Test
public void testClasspathResourceByClassLoader() throws IOException {
		ClassLoader classLoader = this.getClass().getClassLoader();
		Resource resource = new ClassPathResource("cn/javass/spring/chapter4/test.properties", classLoader);
		if (resource.exists()) {
			dumpStream(resource);
		}
		System.out.println("path:" + resource.getFile().getAbsolutePath());
		Assert.assertEquals(false, resource.isOpen());
}
```

1. **_ public ClassPathResource(String path, Class<?> clazz)_**:

```java
@Test
public void testClasspathResourceByClass() throws IOException {
		Class clazz = this.getClass();
		Resource resource1 = new ClassPathResource("cn/javass/spring/chapter4/test.properties", clazz);
		if (resource1.exists()) {
			dumpStream(resource1);
		}
		System.out.println("path:" + resource1.getFile().getAbsolutePath());
		Assert.assertEquals(false, resource1.isOpen());

		Resource resource2 = new ClassPathResource("cn/javass/spring/chapter4/test.properties", this.getClass());
		if (resource2.exists()) {
			dumpStream(resource2);
		}
		System.out.println("path:" + resource2.getFile().getAbsolutePath());
		Assert.assertEquals(false, resource1.isOpen());
}
```

If not provide the full path, it will search in the ***resource*** folder. If not found in the ***resource*** folder, it will search in other package folders.

```java
@Test
public void classpathResourceTestFromJar() throws IOException {
		Resource resource = new ClassPathResource("overview.html");
		if (resource.exists()) {
			dumpStream(resource);
		}
		System.out.println("path:" + resource.getURL().getPath());
		Assert.assertEquals(false, resource.isOpen());
}
```

### 5. UrlResource:

***isOpen()*** always return *false*. Supporting three ways to load resources:

- **http**
- **ftp**
- **file:** For example, **new *UrlResource*(“*file:d:/test.txt*“)**.

### 6. ServletContextResource:

To simplize the ***ServletContext*** of servlet.

### 7. VfsResource:

JBoss AS is now provide on [http://wildfly.org/downloads/](http://wildfly.org/downloads/).

```java
@Test
public void testVfsResourceForRealFileSystem() throws IOException {
		//New virtual path
		VirtualFile home = VFS.getChild("/home");
		//Mapping physical path
		VFS.mount(home, new RealFileSystem(new File("d:")));
		//Load resource from virtual path
		VirtualFile virtualFile = home.getChild("test.txt");
		Resource resource = new VfsResource(virtualFile);
		if (resource.exists()) {
			dumpStream(resource);
		}
		System.out.println(resource.getFile().getAbsolutePath());
		Assert.assertEquals(false, resource.isOpen());
}
```
