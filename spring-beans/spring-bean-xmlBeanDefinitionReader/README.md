## XmlBeanDefinitionReader

- [XmlBeanDefinitionReader](#xmlbeandefinitionreader)
  - [一、知识储备](#一知识储备)
  - [二、基本描述](#二基本描述)
  - [三、主要功能](#三主要功能)
  - [四、主要实现](#四主要实现)
  - [五、最佳实践](#五最佳实践)
  - [六、源码分析](#六源码分析)
  - [七、与其他组件的关系](#七与其他组件的关系)
  - [八、常见问题](#八常见问题)


### 一、知识储备

1. **Bean定义**
   + 了解Bean的概念以及如何定义和配置Bean是非常重要的。这包括Bean的ID、类名、属性注入、依赖关系等。
2. **`DocumentLoader`**
   + `XmlBeanDefinitionReader`依赖于`DocumentLoader`来加载和解析XML配置文件，以便可以将XML文件中的Bean定义信息转化为Spring容器内部的Bean定义对象。 [点击查看DocumentLoader接口](https://github.com/xuchengsheng/spring-reading/tree/master/spring-resources/spring-resource-documentLoader)

### 二、基本描述

`XmlBeanDefinitionReader`是Spring Framework中的一个类，用于加载和解析XML格式的Bean定义配置文件，将配置文件中定义的Bean元数据信息提取为Spring容器内部的Bean定义对象，进而实现IOC容器的构建和管理。这类负责读取XML配置文件，解析Bean的定义信息（包括ID、类名、属性、依赖等），并将这些定义注册到Spring应用程序上下文，使我们能够方便地配置和管理应用程序中的各种Bean组件。

### 三、主要功能

1. **加载XML配置文件**
   + `XmlBeanDefinitionReader`能够加载XML格式的Spring配置文件，通常使用Resource对象来指定配置文件的位置，如`ClassPathResource`、`FileSystemResource`等。
2. **解析XML文件**
   + 它将XML配置文件解析为Spring容器内部数据结构，使用`DocumentLoader`来实现。这包括将XML元素和属性转化为Spring的Bean定义信息。
3. **注册Bean定义**
   + 一旦`XmlBeanDefinitionReader`成功解析XML文件中的Bean定义信息，它会将这些信息注册到Spring容器的Bean工厂，以便容器能够创建和管理这些Bean实例。

### 四、主要实现

`BeanDefinitionHolder` 不是一个接口，也不需要其他实现类。它是一个具体的类，用于加载和解析XML格式的Bean定义配置文件，将配置文件中定义的Bean元数据信息提取为Spring容器内部的Bean定义对象，进而实现IOC容器的构建和管理。

### 五、最佳实践

首先创建了一个Spring容器（`DefaultListableBeanFactory`），然后使用`XmlBeanDefinitionReader`来加载和解析名为"`beans.xml`"的XML配置文件，将其中定义的Bean元数据信息注册到容器中。随后，通过容器获取名为"`myBean`"的Bean实例，最后将该Bean实例打印出来。这样的操作实现了Spring容器的初始化、XML配置文件的解析，以及Bean的获取和使用。

```java
public class XmlBeanDefinitionReaderDemo {

    public static void main(String[] args) {
        // 创建一个DefaultListableBeanFactory作为Spring容器
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();

        // 创建XmlBeanDefinitionReader实例用于解析XML配置
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        // 加载XML配置文件
        reader.loadBeanDefinitions(new ClassPathResource("beans.xml"));

        // 获取并使用Bean
        MyBean myBean = factory.getBean("myBean", MyBean.class);
        System.out.println("myBean = " + myBean);
    }
}
```

`ClassPathResource("sample.xml")` 将加载类路径下名为 "`sample.xml`" 的资源文件的内容。在我们的示例配置文件中，这个资源文件定义了一个名为 "`myBean`" 的 Spring Bean，该 Bean 具有一个属性 "message"，其值设置为 "Hello World"。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myBean" class="com.xcs.spring.bean.MyBean">
        <property name="message" value="Hello World"/>
    </bean>

</beans>
```

`MyBean` 的Java类，代表了一个简单的Java Bean。

```java
public class MyBean {

    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    @Override
    public String toString() {
        return "MyBean{" +
                "message='" + message + '\'' +
                '}';
    }
}
```

### 六、源码分析

在`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(resource)`方法中，又调用了 `loadBeanDefinitions(encodedResource)` 方法，同时将 `resource` 包装成一个 `EncodedResource` 对象。

```java
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(new EncodedResource(resource));
}
```

在`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(encodedResource)`方法中，首先是创建一个 `InputSource` 对象，用于解析XML。最后调用 `doLoadBeanDefinitions` 方法，实际的XML解析和Bean定义加载。

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    // ... [代码部分省略以简化]

    try (InputStream inputStream = encodedResource.getResource().getInputStream()) {
        // 创建一个 InputSource 对象，用于解析XML
        InputSource inputSource = new InputSource(inputStream);
        
        // 如果 encodedResource 包含字符编码信息，设置 InputSource 的编码
        if (encodedResource.getEncoding() != null) {
            inputSource.setEncoding(encodedResource.getEncoding());
        }
        
        // 调用 doLoadBeanDefinitions 方法，实际的XML解析和Bean定义加载
        return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
    }
    catch (IOException ex) {
        // ... [代码部分省略以简化]
    }
    finally {
        // ... [代码部分省略以简化]
    }
}
```

在`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions`方法中，主要用于加载XML配置文件，解析其中的Bean定义，并注册这些Bean定义到Spring容器中

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
            throws BeanDefinitionStoreException {
    try {
        // 解析XML文档
        Document doc = doLoadDocument(inputSource, resource);
        // 注册Bean定义并获取已加载的Bean数量
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    } 
    
    // ... [代码部分省略以简化]
}
```

在`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadDocument`方法中，主要由`DocumentLoader`执行实际的XML加载和解析操作，并将解析后的`Document`对象返回。[点击查看DocumentLoader接口](https://github.com/xuchengsheng/spring-reading/tree/master/spring-resources/spring-resource-documentLoader)

```java
protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
    return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
                                            getValidationModeForResource(resource), isNamespaceAware());
}
```

在`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#registerBeanDefinitions`方法中，首先创建一个 `BeanDefinitionDocumentReader` 实例，然后调用它的 `registerBeanDefinitions` 方法，将XML文档中的Bean定义注册到Spring容器中，最后返回成功注册的Bean数量。

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

在`org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#registerBeanDefinitions`方法中，又调用 `doRegisterBeanDefinitions` 方法，开始实际的Bean定义注册过程。

```java
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    doRegisterBeanDefinitions(doc.getDocumentElement());
}
```

在`org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions`方法中，核心内容是，XML的预处理、Bean定义的解析和XML的后处理。

```java
protected void doRegisterBeanDefinitions(Element root) {
    // ... [代码部分省略以简化]
    
    // 调用 preProcessXml 方法，执行XML预处理操作
    preProcessXml(root);

    // 调用 parseBeanDefinitions 方法，解析Bean定义
    parseBeanDefinitions(root, this.delegate);

    // 调用 postProcessXml 方法，执行XML后处理操作
    postProcessXml(root);
    
    // ... [代码部分省略以简化]
}
```

在`org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseBeanDefinitions`方法中，主要解析XML文档中的Bean定义元素，根据元素所属的命名空间（默认命名空间或自定义命名空间）分别调用合适的方法进行解析。

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    if (delegate.isDefaultNamespace(root)) {
        // 如果根元素属于默认命名空间，则遍历子元素
        NodeList nl = root.getChildNodes();
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;
                if (delegate.isDefaultNamespace(ele)) {
                    // 如果子元素也属于默认命名空间，解析为默认元素
                    parseDefaultElement(ele, delegate);
                }
                else {
                    // 否则解析为自定义元素
                    delegate.parseCustomElement(ele);
                }
            }
        }
    }
    else {
        // 如果根元素不属于默认命名空间，解析为自定义元素
        delegate.parseCustomElement(root);
    }
}
```

在`org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseDefaultElement`方法中，主要负责根据元素的类型选择合适的方法进行处理，从而实现了XML配置文件中不同元素的解析和Bean定义的注册。

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    // 根据元素的名称进行不同的处理
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        // 如果是 <import> 元素，导入其他Bean定义资源
        importBeanDefinitionResource(ele);
    }
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        // 如果是 <alias> 元素，处理Bean的别名注册
        processAliasRegistration(ele);
    }
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        // 如果是 <bean> 元素，处理普通Bean定义
        processBeanDefinition(ele, delegate);
    }
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // 如果是 <beans> 元素，递归注册内部Bean定义
        doRegisterBeanDefinitions(ele);
    }
}
```

在`org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#processBeanDefinition`方法中，主要是解析 `<bean>` 元素，注册Bean定义到Spring容器。

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    // 调用 BeanDefinitionParserDelegate 的 parseBeanDefinitionElement 方法，解析 <bean> 元素
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    
    if (bdHolder != null) {
        // 如果解析成功，调用 delegate.decorateBeanDefinitionIfRequired 方法，如果需要的话装饰 Bean 定义
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);

        try {
            // 注册最终装饰后的 Bean 定义
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        } catch (BeanDefinitionStoreException ex) {
            // 处理注册时的异常，如果注册失败，抛出 BeanDefinitionStoreException 异常
            getReaderContext().error("Failed to register bean definition with name '" +
                                     bdHolder.getBeanName() + "'", ele, ex);
        }

        // 发送 Bean 注册事件
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```

在`org.springframework.beans.factory.support.BeanDefinitionReaderUtils#registerBeanDefinition`方法中，将Bean定义注册到Spring容器的Bean定义注册表中，并处理别名的注册。

```java
public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}
```

### 七、与其他组件的关系

1. **`ApplicationContext`的实现类**
   + Spring的`ApplicationContext`接口有多个实现，其中一些实现类在其内部使用`XmlBeanDefinitionReader`来处理XML配置文件。例如，`ClassPathXmlApplicationContext`、`FileSystemXmlApplicationContext`和`XmlWebApplicationContext`都使用`XmlBeanDefinitionReader`。
2. **Custom XML配置加载**
   + 如果我们编写自己的Spring应用程序或自定义的Spring容器，我们可以使用`XmlBeanDefinitionReader`来实现自定义的XML配置加载逻辑。
3. **单独的`XmlBeanDefinitionReader`使用**
   + 有时，我们可能需要在应用程序中手动创建`XmlBeanDefinitionReader`实例，然后使用它来加载和注册Bean定义。

### 八、常见问题

1. **XML文件路径正确性**
   + 确保XML配置文件的路径和名称正确，相对路径或绝对路径都要检查，否则将导致文件无法找到或读取。
2. **XML文件格式和编码**
   + XML文件必须符合XML语法规则，包括标签嵌套、属性格式和XML声明。同时，确保文件使用正确的字符编码，与指定的编码一致，否则可能导致解析错误或乱码。
3. **Bean定义的正确性**
   + XML文件中的Bean定义必须正确，包括Bean的属性、名称、类路径等。不正确的Bean定义可能导致Spring容器初始化失败。
4. **版本兼容性**
   + 确保所使用的Spring框架版本与XML配置文件中的XML schema（XSD）版本兼容。不同版本可能需要不同的XSD版本。
5. **错误处理和日志记录**
   + 在出现问题时，查看Spring框架的错误日志和异常堆栈信息，以便更好地识别和解决配置问题。及时的错误处理和日志记录有助于定位问题并加快排查过程。