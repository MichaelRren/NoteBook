# Spring

## Spring IoC

IoC (Inverse of Control)，控制反转

DI (Dependency Injection)，依赖注入

## 基于 XML 的 IoC

```java
public class SpringClient {
    public static void main(String[] args) {
       //读取资源位置。创建对应的BeanFactory，使用Reader将前两者构成联系。首先是指定Reader读取BeanFactory，其次Reader加载哪个资源
       //由 线程上下文类加载器或者系统类加载器加载。
        Resource resource = new ClassPathResource("applicationContext.xml");
      //忽略掉三个重要的接口
        DefaultListableBeanFactory defaultListableBeanFactory = new DefaultListableBeanFactory();
        BeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(defaultListableBeanFactory);
        beanDefinitionReader.loadBeanDefinitions(resource);
        Student student = (Student) defaultListableBeanFactory.getBean("student");

        System.out.println(student.toString());
    }
}
```

### 关于 Spring 容器管理 Bean 的过程以及加载模式：

1. 需要讲bean的定义信息声明在Spring的配置文件当中。
2. 需要通过Spring抽象出的各种Resource来指定对应的配置文件。
3. 需要显示声明一个Spring工厂， 该工厂用来掌握我们在配置文件中所声明的各种bean以及bean之间的依赖关系与注入关系。
4. 需要定义一个配置信息读取器，该读取器用来读取之前所定义的bean配置文件信息。
5. 读取器的作用是读取我们所声明的配置文件信息，并且将读取后的信息装配到之前所声明的工厂中。 
6. **需要将读取器与工厂以及资源对象进行相应的关联处理。**
7. 工厂所管理的全部对象装配完毕，可以供客户端直接调用，获取客户端想要使用的各种bean对象。

### Spring 对 Bean 管理的核心组件：

1. 资源抽象
2. 工厂
3. 配置信息读取器

### BeanFactoy

BeanFactory 是 Spring Bean 工厂最顶层的抽象。

### Spring Bean实例的注册流程

1. 定义好Spring 的配置文件
2. 通过Resource对象将Spring 配置文件进行抽象，抽象成一个具体的Resource对象(如ClassPathResource)
3. 定义好将要使用的Bean工厂(各种BeanFactory)
4. 定义好XmlBeanDefinitionReader对象，并将工厂对象作为参数传递进去，从而构建好二者之间的关联关系
5. 通过XmlBeanDefinitionReader对象读取之前所抽取出的Resource对象
6. 流程开始进行解析
7. 针对XML文件进行各种元素以及元素属性的解析，这里面，真正的解析是通过**BeanDefinitionParserDelegate**对象来完成的(委托模式)
8. `registry.regosterBeanDefinition(beanName, definitionHoler.getBeanDefinition())`
9. 通过BeanDefinitionParserDelegate对象在解析XML文件时，有时用到了模版方法设计模式(pre, process, post)
10. 当所有的bean标签元素都解析完毕后，开始定义一个BeanDefinition对象，该对象是一个非常重要的对象，里面容纳了一个Bean相关的所有属性。
11. BeanDefinition对象创建完毕后，Spring又会创建一个BeanDefinitionHolder对象来持有这个BeanDefinition对象
12. BeanDefinitionHolder对象主要包含两部分内容：beanName 与 beanDefinition
13. 工厂会将解析出来的Bean信息存放到内部的一个ConcurrentHashMap中，该Map的键是 beanName，值是BeanDefinition对象
14. 调用Bean解析完毕的触发动作，从而触发相应的监听器方法的执行(使用到了观察者模式)

> 如何解析XML元素以及各种属性，将Bean属性存放在BeanDefinition这个对象中。然后构造BeanDefinitionHolder这个对象，持有beanName和BeanDefintion，形成了 key value 对的形式的对象。最后将其put到底层的ConcurrentHashMap中。

### Spring Bean 实例的创建流程

1. doGetBean(name, null, null, false); 首先 `getSingleton`，检查是否之前生成过这个实例对象。  **单例对象都是来自于 singletonObjects 缓存中。**用一个ConcurrentHashMap对所有的对象进行缓存。beanName -> bean instance

2. 对于配置的 id 和 name 最终在Spring中都会被统一成 beanName 这个属性。

3. 第一次进行 getSingleton时，肯定是不存在这个实例，因此会检查 beanDefinition 是否存在于 ParentBeanFactory 中，会进行一次判断，如果不存在就进行下一步。然后，进行 markBeanAsCreated 将其标记为正在进行创建的状态。这样可以防止 bean 被重复的创建。存入一个 名为 alreadyCreated 的 Set 中。==注意，本次调用时，使用的是  `getSinlgeton(beanName)`仅仅根据名字获取。==

4. mergedBeanDefinition底层也是一个 ConcurrentHashMap。这个变量代表 Map from bean name to merged RootBeanDefinition

5. 之后，会创建一个 RootBeanDefinition这个 汇总的 Bean 的信息。内部还会维护一个 BeanDefinitionHolder  并进行其他 Bean 的生成。从 BeanDefinition中获取所有的解析出的 bean 信息，在 mergedBeanDefinition过程中拼接在一起。形成一个汇总的 RootBeanDefinition

6. 拿到 mbd 后，会进行是否是抽象的判定以及 dependsOn 依赖的检查。如果有依赖，那么还需要创建其他依赖的 bean

7. `if(mbd.isSingleton()) {}` 执行这一部分进行 bean 的创建。在这一部分，进行 bean 实例的创建。本次 使用的是重载的方法。额外接收一个 ObjectFactory 的参数

   ```java
   if(mbd.isSingleton()) {
   	sharedInstance = getSingleton(beanName, () -> {
       try {
         return createBean(beanName, mbd, args);
       }
     })
   }
   ```

8. getSinlgeton(beanName, ObjectFactory sF) 中，首先执行一个断言，检查beanName是否为空。然后会从 singleObjects 中尝试获取 beanName 对应的 Bean，如果为空，就会执行 beforeSingletonCreation 进行一些检查，是否还有其他地方在创建这个实例，如果有，就抛出异常。定义一个标识 `newSingleton` 表示是否创建成功。

9. lambda 表达式对应的是 getObject 的具体方法

10. 执行 createBean(beanName, mbd, args) ，首先在实例化前进行操作，然后执行 Object beanInstance = doCreateBean(beanName, mbd, args)

11. BeanWrapper 使用这个 wrapper 包装类进行包装与生成。如果没有创建过，那么就会执行 createBeanInstance 这个方法。根据不同的策略进行创建。

12. 最后执行 instantiateBean(beanName, mbd) 如果没有指定策略，就会使用默认的无参的构造方法进行构造。使用反射的方式。

13. 返回到 doGetBean，就可以得到了 被创建封装的实例。返回之后，执行后置处理器。然后判断是否是单例模式以及允许循环引用。

14.   缓存，这一步还没有将生成的实例加入到==singleObjects==中

    ```java
    synchronized (this.singetonObjects) {
      if (!this.singletonObjects.containsKey(beanName)) {
        //Cache of singleton factories, bean name to ObjectFactory
        this.singletonFactories.put(beanName, singletonFactory);
        //Cache of early singleton objects: bean name to bean instance HashMap
        this.earlySingletonObjects.remove(beanName);
        //Set if reguestered singetons, containing the bean names in registration order LinkedHashSet
        this.registeredSingletons.add(beanName);
      }
    }
    ```

15. populateBean 中完成对属性的装配。从mbd中获取属性值。`exposedObject = initializeBean(beanName, exposedObject, mbd); 

16. 加入到IoC的缓存中

    ```java
    if (new Singleton) {
      addSingleton(beanName, singeltonObject);
    }
    
    synchronized(this.singleObjects) {
      this.singletonObjects.put(beanName, singletonObject);
      this.singletonFactories.remove(beanName);
      this.earlySingletonObjects.remove(beanName);
      this.registeredSingletons.add(beanName);
      }
    ```

> Spring Bean 的创建流程
>
> 1. Spring所管理的Bean 实际上是缓存在一个 ConcurrentHashMap 中的 (singleObjects对象中)
> 2. 该对象本质是一个 key-value 对的形式，key指的是 bean 的名字 (id)，value 是一个 Object 对象，就是创建的 bean 对象
> 3. 在创建 Bean 之前，首先需要将该 Bean 的创建标识指定好，表示该 Bean 已经或是即将被创建，目的是增强缓存的效率
> 4. 根据 bean 的 scope 属性确定当前这个 bean 是一个 singleton 还是一个 prototype 的 bean，然后创建相应的对象。
> 5. 无论是 singleton 还是 prototype 的 bean，其创建的过程是一致的。
> 6. 通过 Java 反射机制来创建 Bean 的实例，在创建之前需要检查构造方法的访问修饰符，如果不是 public 的，则会调用 setAccessible(true)方法来突破 Java 的语言限制，使得可以通过非 public 构造方法来完成对象实例的创建。
> 7. 当对象创建完毕后，开始进行对象属性的注入。
> 8. 在对象属性注入的过程中，Spring 除去使用之前通过 BeanDefinition对象获取的 Bean 信息外，还会通过反射的方式获取到上面所创建的 Bean 中的真实属性信息(还包括一个 class 属性，表示该 Bean 所对应的 Class 类型)
> 9. 完成Bean属性的注入（或者抛出异常）
> 10. 如果Bean是一个单例的，那么将所创建出来的Bean添加到singletonObject对象中(缓存中)，供程序后续使用

## 基于注解的IoC

```java
@Configuration//对应 xml 中的 <beans>标签
//class com.tju.spring.PersonConfiguration$$EnhancerBySpringCGLIB$$361a20
//返回的是一个CGLIB生成的 PersonConfiguration的子类型
public class PersonConfiguration {
  //只能作用于方法和注解上。
  @Bean(name = "person")
  public Person person() {
    Person person = new Person();
    person.setId(20);
    person.setName("zhangsan");
    
    return person;
  }
}
```

```java
public class SpringClient {
  public static void main(String[] args) {
    //基于注解的工厂，对应前四行，创建工厂。构造方法会实例化 reader 和 scanner，对应 BeanDefinitionReader，用于读取配置/注解。scanner用于扫描组件。使用注解的方式，底层都是 DefaultListableBeanFactory,并指定组件的扫描。
    AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext();
    //类似于指定好配置文件，注册完成后，一定要 refresh，对新注册的信息进行处理。 
    //registerBean doRegisterBean
    //将 Configuration类注册到工厂中
    annotationConfigApplicationContext.register(PersonConfiguration.class);
    //注册后刷新，但是此时还没有创建实例。
    //触发bean的创建
    annotationConfigApplicationContext.refresh();
    
    Person person = (Person) annotationConfigApplicationContext.getBean("person");
  }
}
```

 `BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry)`通过这个方式完成 BeanDefinition 的注册，注册到了工厂中。

基于注解与基于XML配置的SpringBean在创建时机上存在唯一的不同之处：

1. 基于XML配置的方式，Bean对象的创建是在程序首次从工厂中获取该对象时才创建的
2. 基于注解配置的方式，Bean对象的创建是在注解处理器解析相应的@Bean注解时调用了该注解所修饰的方法，当该方法执行后，相应的对象自然就已经被创建出来了。这是，Spring将会将该对象纳入到工厂的管理范围之内，当我们首次尝试从工厂中获取到该Bean对象时，这时，该Bean对象实际上已经完成了创建并被纳入到了工厂的管理范围之内。

### refresh方法

1. 首先进行同步
2. prepareRefresh(): 比较重要的一步就是 初始化占位符的属性源、验证环境相关的属性、注册相应的监听器
3. ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();通知子类刷新内部 bean factory，然后 return 一个 DefaultListableFactory
4. prepareBeanFactory(beanFactory) 准备好后续上下文需要使用的 beanFactory，如类加载器或者忽略一些依赖如一些Aware接口。
5. postProcessBeanFactory(beanFactory): 允许 beanFactory 的后置处理
6. invokeBeanFactoryPostProcessors(beanFactory): 调用 beanFactory 的后置处理器 -> invokeBeanDefinitionRegistryPostPorcessors(curentRegistryProcessors, registry) -> `for(String beanName : candidateNames)`，对配置类进行解析-> `loadBeanDefinitionsForConfigurationClass`完成对于 Configuration 注解的类中包含的 bean 的解析。
7. registerBeanPostProcessors(beanFactory): 注册 Bean 本身的后置处理器
8. initMessageSource(): 消息来源的注册和解析
9. initApplicationEventMulticaster(): 用于进行事件的分发
10. onRefresh(): 初始化特殊的 bean
11. finishBeanFactoryInitialization(beanFactory): 实例化非延迟加载的 bean -> beanFactory.preInstantiateSingletons(); -> 遍历 beanNames 中的 beanName，首先获取到源数据信息，判断是否是FactoryBean，随后进入 getBean(beanName) -> doGetBean;取出每一个 beanName 并进行创建对应的实例。 
12. registerListeners(): 

### Bean 创建

Bean 对象创建时与 xml 不同。

```java
if (mbd.getFactoryMethodName() != null) {
  //fatoryBeanName = "personConfiguration" factoryMethodName = "getPeson"
  return instantiateUsingFactoryMethod(beanName, mbd, args);
}
Object result = factoryMethod.invoke(factoryBean, args);//执行的时间点

```

对于 Configuration 而言，在注解领域中，此注解相当于一个工厂。而被 Bean 标注的方法都是工厂方法。也就是说，由工厂返回指定的对象。

方法内部也是先创建一个 BeanWrapper，

