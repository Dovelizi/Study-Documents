# Spring-Framework

1. ### IOC容器

   1. Bean的定义

      | Property              | Explained in…                                                |
      | --------------------- | ------------------------------------------------------------ |
      | Class                 | [Instantiating Beans](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-class) |
      | Name                  | [Naming Beans](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-beanname) |
      | Scope                 | [Bean Scopes](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-scopes) |
      | Constructor arguments | [Dependency Injection](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-collaborators) |
      | Properties            | [Dependency Injection](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-collaborators) |
      | Autowiring mode       | [Autowiring Collaborators](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-autowire) |
      | 延迟初始化模式        | [Lazy-initialized Beans](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-lazy-init) |
      | Initialization method | [Initialization Callbacks](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-lifecycle-initializingbean) |
      | Destruction method    | [Destruction Callbacks](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-lifecycle-disposablebean) |

      ​		除了包含有关如何创建特定 bean 的信息的 bean 定义之外，`ApplicationContext`实现还允许注册在容器外部(由用户)创建的现有对象。这是通过`getBeanFactory()`方法访问 ApplicationContext 的 BeanFactory 来完成的，该方法返回 BeanFactory `DefaultListableBeanFactory`的实现。 `DefaultListableBeanFactory`通过`registerSingleton(..)`和`registerBeanDefinition(..)`方法支持此注册。但是，典型的应用程序只能与通过常规 bean 定义元数据定义的 bean 一起使用。

   2. 命名 Bean

      ​		每个 bean 具有一个或多个标识符。这些标识符在承载 Bean 的容器内必须唯一。一个 bean 通常只有一个标识符。但是，如果需要多个，则可以将多余的别名视为别名。

      ​		在基于 XML 的配置元数据中，您可以使用`id`属性和`name`属性，或同时使用这两者来指定 bean 标识符。 `id`属性可让您精确指定一个 ID。按照惯例，这些名称是字母数字(“ myBean”，“ someService”等)，但它们也可以包含特殊字符。如果要为 bean 引入其他别名，还可以在`name`属性中指定它们，并用逗号(`,`)，分号(`;`)或空格分隔。作为历史记录，在 Spring 3.1 之前的版本中，`id`属性定义为`xsd:ID`类型，该类型限制了可能的字符。从 3.1 开始，它被定义为`xsd:string`类型。请注意，bean `id`的唯一性仍由容器强制执行，尽管不再由 XML 解析器执行。

      您不需要为 Bean 提供`name`或`id`。如果未明确提供`name`或`id`，则容器将为该 bean 生成一个唯一的名称。但是，如果要按名称引用该 bean，则通过使用`ref`元素或[Service Locator](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-servicelocator)样式查找，必须提供一个名称。不提供名称的动机与使用[inner beans](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-inner-beans)和[autowiring collaborators](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-autowire)有关。

      Bean 命名约定

      ​		约定是在命名 bean 时将标准 Java 约定用于实例字段名称。也就是说，Bean 名称以小写字母开头，并从那里用驼峰式大小写。这样的名称的示例包括`accountManager`，`accountService`，`userDao`，`loginController`等

      > ​	**Note**
      >
      > 通过在 Classpath 中进行组件扫描，Spring 会按照前面描述的规则为未命名的组件生成 Bean 名称：从本质上讲，采用简单的类名称并将其初始字符转换为小写。但是，在(不寻常的)特殊情况下，如果有多个字符并且第一个和第二个字符均为大写字母，则会保留原始大小写。这些规则与`java.beans.Introspector.decapitalize`(Spring 在此使用)定义的规则相同。

   3. Bean的实例化

      1. 使用静态工厂方法实例化

         ​		以下 bean 定义指定通过调用工厂方法来创建 bean。该定义不指定返回对象的类型(类)，而仅指定包含工厂方法的类。在此的示例`createInstance()`方法必须是静态方法

         ```java
         <bean id="clientService"
             class="examples.ClientService"
             factory-method="createInstance"/>
         ```

         具体的实例化(单例)：

         ```java
         public class ClientService {
             private static ClientService clientService = new ClientService();
             private ClientService() {}
         
             public static ClientService createInstance() {
                 return clientService;
             }
         }	
         ```
         
      2. 使用静态工厂方法实例化
      
         ​		与通过[静态工厂方法](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference/core.html#beans-factory-class-static-factory-method)实例化类似，使用实例工厂方法实例化从容器中调用现有 bean 的非静态方法以创建新 bean。要使用此机制，请将`class`属性留空，并在`factory-bean`属性中，在当前(或父容器或祖先容器)中指定包含要创建该对象的实例方法的 bean 的名称。使用`factory-method`属性设置工厂方法本身的名称
      
         ```java
         <!-- the factory bean, which contains a method called createInstance() -->
         <bean id="serviceLocator" class="examples.DefaultServiceLocator">
             <!-- inject any dependencies required by this locator bean -->
         </bean>
         
         <!-- the bean to be created via the factory bean -->
         <bean id="clientService"
             factory-bean="serviceLocator"
             factory-method="createClientServiceInstance"/>
         ```
      
         具体的实例化(单例)：
      
         ```java
         public class DefaultServiceLocator {
         
             private static ClientService clientService = new ClientServiceImpl();
         
             public ClientService createClientServiceInstance() {
                 return clientService;
             }
         }
         ```
      
         一个工厂类也可以包含一个以上的工厂方法
      
         ```java
         <bean id="serviceLocator" class="examples.DefaultServiceLocator">
             <!-- inject any dependencies required by this locator bean -->
         </bean>
         <!-- 第一个 Bean 对象 -->
         <bean id="clientService"
             factory-bean="serviceLocator"
             factory-method="createClientServiceInstance"/>
         <!-- 第二个 Bean 对象 -->
         <bean id="accountService"
             factory-bean="serviceLocator"
             factory-method="createAccountServiceInstance"/>
         ```
      
         具体代码实现：
      
         ```java
         public class DefaultServiceLocator {
         
             private static ClientService clientService = new ClientServiceImpl();
         
             private static AccountService accountService = new AccountServiceImpl();
         		// 使用单例模式，直接使用该方法获取实例化对象
             public ClientService createClientServiceInstance() {
                 return clientService;
             }
         
             public AccountService createAccountServiceInstance() {
                 return accountService;
             }
         }
         ```
         
      3. 有参数的Bean对象注入
      
         ​		当引用另一个 bean 时，类型是已知的，并且可以发生匹配(与前面的示例一样)。当使用简单类型(例如`<value>true</value>`)时，Spring 无法确定值的类型，因此在没有帮助的情况下无法按类型进行匹配
      
         ```java
         public class ExampleBean {
         
             // Number of years to calculate the Ultimate Answer
             private int years;
         
             // The Answer to Life, the Universe, and Everything
             private String ultimateAnswer;
         
             public ExampleBean(int years, String ultimateAnswer) {
                 this.years = years;
                 this.ultimateAnswer = ultimateAnswer;
             }
         }
         ```
      
         1. 如果使用`type`属性显式指定了构造函数参数的类型，则容器可以使用简单类型的类型匹配
      
         ```java
         <bean id="exampleBean" class="examples.ExampleBean">
             <constructor-arg type="int" value="7500000"/>
             <constructor-arg type="java.lang.String" value="42"/>
         </bean>
         ```
      
         2. 可以使用`index`属性来显式指定构造函数参数的索引(索引从0开始)
      
         ```java
         <bean id="exampleBean" class="examples.ExampleBean">
             <constructor-arg index="0" value="7500000"/>
             <constructor-arg index="1" value="42"/>
         </bean>
         ```
         3. 还可以使用构造函数参数名称来进行匹配
         
         ```java
         <bean id="exampleBean" class="examples.ExampleBean">
             <constructor-arg name="years" value="7500000"/>
             <constructor-arg name="ultimateAnswer" value="42"/>
         </bean>
         ```
         
      3. 
      
      
