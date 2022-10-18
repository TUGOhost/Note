# 容器的基本实现

## 容器基本用法

bean是Spring中最核心的东西，因为Spring就像是个大水桶，而bean就像是容器中的水，水桶脱离了水便也没什么用处了，那么我们先看看bean的定义。

```java
public class MyTestBean{
    private String testStr = "testStr";
    public String getTestStr(){
        return testStr;
    }
    public void setTestStr(String testStr)
}
```

接下来是配置文件：

```xml
<beans><bean id="myTestBean" class="bean.MyTestBean"/></beans>
```

测试代码：

```java
@SuppressWarnings("deprecation")
public class BeanFactoryTest{
    @Test
    public void testSimpleLoad(){
        BeanFactory bf = new XmlBeanFactory(new ClassPathResource("beanFactoryTest.xml"));
        MyTestBean bean = (MyTestBean) bf.getBean("myTestBean");
        assertEquals("testStr",bean.getTestStr());
    }
}
```

这段测试代码完成的功能无非就是以下几点：

1. 读取配置文件`beanFactoryTest.xml`。
2. 根据`beanFactoryTest.xml`中的配置找到对应的类的配置，并实例化。
3. 调用实例化后的实例。

## Spring的结构组成

### 核心类介绍

1）`DefaultListableBeanFactory`

`XmlBeanFactory`继承自`DefaultListableBeanFactory`，而`DefaultListableBeanFactory`是整个bean的加载的核心部分，是Spring注册及加载bean的默认实现，而对于`XmlBeanFactory`与`DefaultListableBeanFactory`不同的地方其实是在`XmlBeanFactory`中使用了自定义的XML读取器`XmlBeanDefinitionReader`，实现了个性化的`BeanDefinitionReader`读取，`DefaultListableBeanFactory`继承了`AbstractAutowireCapableBeanFactory`并实现了`ConfigurableListableBeanFactory`以及`BeanDefinitionRegistry`接口。以下是`DefaultListableBeanFactory`的相关类图：

![](../image/282.png)

- AliasRegistry：定义对alias的简单增删改等操作。
- SimpleAliasRegistry：主要使用map作为alias的缓存，并对接口AliasRegistry进行实习。
- SingletonBeanRegistry：定义对单例的注册及获取。
- BeanFactory：定义获取bean及bean的各种属性。
- DefaultSingletonBeanRegistry：对接口SingletonBeanRegistry各函数的实现。
- HierarchicalBeanFactory：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持。
- BeanDefinitionRegistry：定义对BeanDefinition的各种增删改操作。
- FactoryBeanRegistrySuport：在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。
- ConfigurableBeanFactory：提供配置Factory的各种方法。
- ListableBeanFactory：根据各种条件获取bean的配置清单。
- AbstractBeanFactory：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。
- AutowireCapableBeanFactory：提供创建bean、自动注入、初始化、以及应用bean的后处理器。
- AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并对接口AutowireCapableBeanFactory进行实现。
- ConfigurableListableBeanFactory：BeanFactory配置清单，指定忽略类型及接口等。
- DefaultListableBeanFactory：综合上面所有功能，主要是对bean注册后的处理。

XmlBeanFactory对DefaultListableBeanFactory类进行了扩展，主要用于从XML文档中读取BeanDefinition，对于注册及获取bean都是使用从父类DefaultListableBeanFactory继承的方法去实现，而唯独与父类不同的个性化实现就是增加了XmlBeanDefinitionReader类型的reader属性。在XmlBeanFactory中主要使用reader属性对资源文件进行读取和注册。

2）`XmlBeanDefinitionReader`
XML配置文件的读取是Spring中重要的功能，因为Spring的大部分功能都是以配置作为切入点的，那么我们可以从XmlBeanDefinitionReader中梳理一下资源文件读取、解析及注册的大致脉络，首先我们看看各个类的功能。
- ResourceLoader：定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource。
- BeanDefinitionReader：主要定义资源文件读取并转换为BeanDefintion的各个功能。
- EnvironmentCapable：定义获取Environment方法。
- DocumentLoader：定义从资源文件加载到转换为Document的功能。
- AbstractBeanDefinitionReader：对EnvironmentCapable、BeanDefinitionReader类定义的功能进行实现。
- BeanDefinitionDocumentReader：定义读取Document并注册BeanDefinition功能。
- BeanDefinitionParserDelegate：定义解析Element的各种方法。

1. 通过继承自AbstractBeanDefinitionReader中的方法，来使用ResourceLoader将资源文件路径转换为对应的Resource文件。
2. 通过DocumentLoader对Resource文件进行转换，将Resource文件转换为Document文件。
3. 通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析。