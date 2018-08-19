# jee-Spring

## 1. IOC
注入方法:
1. 构造函数注入: 通过调用类的构造函数,将接口实现类通过构造函数变量传入;
2. 属性注入: 通过setter方法完成调用类所需依赖的注入,更方便;
3. 接口注入: 将调用类所有依赖注入的方法抽取到一个接口, 调用类通过实现该接口提供相应的注入方法.

## 2. 资源访问:
![ Resource](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fuahsjssmvj30gx069tam.jpg)

## 3. ApplicationContext

由BeanFactory派生而来, ApplicationContext 容器包括 BeanFactory 容器的所有功能,其实现类如下:

1. ClassPathXmlApplicationContext
2. FileSystemXmlApplicationContext
3. ConfigurableApplicationContext(新增了refresh(), close())

## 3. spring配置文件

1. 配置元数据可以通过 XML，Java 注释或 Java 代码来表示;

2. <beans></beans>  : 在其内部定义bean;(<bean>标签可以是单标签,也可以是双标签)

3. <import resource='other_xml.xml' />: 引入其他配置文件的bean的定义;

4. <bean></bean>: 在其内部定义bean, 如: `<bean id="bean1" class="xxx"></bean>`;

5. <alias />: 定义bean的别名, 如: `<alias alias="bean2" name="bean1"/>` ==> bean2 和bean1是同一个bean; 

6. [p-namespace 的作用](https://www.cnblogs.com/gmq-sh/articles/4807204.html): 简化property标签的配置: 如下两个配置方式等效: 

   使用property属性:

   ![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fubb4osdgmj30g505njs5.jpg)

   

   注1: 使用p属性传递值: 

   ![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fubb5ixuaij30hq04dq3o.jpg)

   注2: 使用p-namespace传递引用:

   <property name="spouse" ref="jane"/>   <==>  <bean p:spouse-ref="jane"  ..../>

7. bean配置中常用的属性, 如图:

   > 1. class, id, name
   >
   > 2. lazy-init="true"   <!-- 延迟初始化 -->
   >
   > 3. init-method="..." <!--该属性指定一个方法，在实例化 bean 时，立即调用该方法-->
   >
   > 4. destroy-method="..."  <!--该属性指定一个方法，只有从容器中移除 bean 之后，才能调用该方法-->
   >
   > 5. default-init-method="iniMethod"  <!--beans 标签属性. 用于初始化bean(可能有多个)的有一个特点: 都具有一个初始化方法iniMethod  -->
   >
   > 6. default-destroy-method="desMethod" <!--beans 标签属性. 用于销毁bean(可能有多个)的有一个特点: 都具有一个初始化方法desMethod  -->
   >
   > 7. parent="父类bean的id" , <!--在配置子类bean时指定其父类, 前提时在java代码中也存在继承关系--> usage: `<bean id="helloIndia" class="com.tutorialspoint.HelloIndia" p`
   >
   > 8. abstract="true", <!--定义抽象父bean, 本身并不能实例化, 供子bean定义时使用-->
   >
   > 9. ref="spellChecker" <!--使用构造器实例化时, 为构造器传入一个其他bean的引用-->
   >
   > 10. value="xxx" <!--使用构造器实例化时, 为构造器传入一个值-->
   >
   >     usage: `<constructor-arg index="0" value="2001"/>`
   >
   > 11. index=int_num,, <!--构造函数参数的方式，使用 index 属性来显式的指定构造函数参数的索引-->
   >
   >     usage: `<constructor-arg index="0" value="2001"/>`

   init-method:

   ![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fubdwfyulnj30la07qdgs.jpg)

   destory-method:

   ![](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fubdywh5s1j30ke08ijsf.jpg)

   


   ## 4. Sping容器高层视图

![Bean的配置,Bean的实现类,Spring容器,应用程序四者的关系](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fual8turs8j30u40ecwk9.jpg)

## 5. Bean的命名
* 配置文件:

![命名方式](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fuaodtsm7wj30kl08kjxk.jpg)

* 命名方式:

![配置文件](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fuaocu1snmj30if07ddhq.jpg)

## 6. Spring Bean的实例化的三种方式
1. 使用构造器实例化Bean(默认构造和带参构造均可)

     * 配置:

     ![构造器实例化bean--配置](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fuauxk1g9aj30p30d5tet.jpg)
         
     * 实例化:

     ![构造器实例化bean--实例化](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fuav6pzg8tj30oc0b5tf1.jpg)

2. 使用静态工厂方式实例化Bean; (<u>需指定class属性, 并用 `factory-method`属性来指定实例化的方法, 允许指定方法参数 </u>)

     ​       注: 需要新建一个静态工厂类, 该类的某个方法返回实例化的bean.
     * 配置: 注意, 1class`属性指定的是静态工厂的类名!!而不是需要实例化的类的类名:

     ![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fuavge79xsj30p2094n2h.jpg)
         
     ![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fuavj5qo1yj30k203xac7.jpg)
         
     * 静态工厂:

     ![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fuavcuczx2j30gx06umyz.jpg)
         
     * 实例化:

     ![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fuavnkn0ydj30kp05tado.jpg)

3. 使用实例工厂方法实例化Bean;(<u> 不能指定`class`属性, 使用`factory-bean`来指定工厂名, ` factory-method`属性来指定实例化的方法, 允许指定方法参数 </u>)

        注: 需要新建一个实例工厂类, 该类的某个方法返回实例化的bean.

     * 配置

       ![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fuavw1lp44j30p409lgs1.jpg)

       ![](http://wx3.sinaimg.cn/mw690/0060lm7Tly1fuaw3dnidyj30n403t77d.jpg)

     * 实例工厂类

      ![](http://wx2.sinaimg.cn/mw690/0060lm7Tly1fuaw0tq7y4j30ee07ftae.jpg)

    * 实例化

![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fuaw5dlvwxj30l505rdji.jpg)

## 7. Spring Bean的作用域(scope)
> 指的是, 在Spring容器中, 其创建的Bean对象相对于其他Bean对象的请求可见范围 

### 7.1 五种作用域
1. `scope="singleton"`, 默认作用域. Spring容器将只会创建该bean的唯一实例, 并缓存在单例缓存中, 后续针对该bean的请求及引用都将返回该实例.无状态的实例应当使用singleton, 如: DAO.
  `<bean id="xxx" class="xxxx" scope="singleton"></bean>`

2. `scope="prototype"`, Spring容器会在对该bean请求时(<u>getBean()</u>)都创建该bean的新的实例. 默认情况下, Spring容器启动时, 不会实例化prototype的bean. 因此, 对于有状态的实例应当使用prototype, 无状态的实例应当使用singleton. 
  `<bean id="xxx" class="xxxx" scope="prototype"></bean>`

3. `scope="request"`,  根据每次Http请求, Spring容器会创建一个全新的实例, 且该实例只在当前request内有效.
  `<bean id="xxx" class="xxxx" scope="request"></bean>`

4. `scope="session"`,   根据每次Http session, Spring容器会创建一个全新的实例, 且该实例只在当前Http session内有效.
   `<bean id="xxx" class="xxxx" scope="session"></bean>`

5. `scope="globalSession"`,  该作用域将 bean 的定义限制为全局 HTTP 会话
  `<bean id="xxx" class="xxxx" scope="globalSession"></bean>`

注: 当使用WebApplicationContext时, 可以使用后3种作用域, 同时需要一些额外配置, 如下: (分高低版本web容器)
![](http://wx4.sinaimg.cn/mw690/0060lm7Tly1fubamjk8h6j30ii09fdk6.jpg)

## 8. 整合多个配置文件

配置文件应该按模块等方式放在模块内部. 然后在应用层面通过一个全局的配置文件将各个模块下的配置文件import

`<import resource="module1/xxx.xml" />`

如下:

![](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fubbe7fx6fj30kw0bs78c.jpg)

## 9. 集合注入(List, Set, Map, Properties)

### 9.1 注入"值"

配置文件:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="collectionInject" class="com.huh.coll.CollectionInject">
        <property name="myList">
            <list>
                <value>"hello"</value>
                <value>"spring"</value>
            </list>
        </property>
        <property name="mySet">
            <set>
                <value>"hello"</value>
                <value>"hello"</value>
            </set>
        </property>
        <property name="myMap">
            <map>
                <entry key="1" value="hello"/>
                <entry key="2" value="hello"/>
                <entry key="2" value="hello"/>
            </map>
        </property>
        <property name="myProp">
            <props>
                <prop key="1">"hello"</prop>
                <prop key="2">"hello"</prop>
            </props>
        </property>
    </bean>
</beans>
```

java类:
```
package com.huh.coll;
import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;
public class CollectionInject {
    private List<String> myList;
    private Set<String> mySet;
    private Map<String, String> myMap;
    private Properties myProp ;
    public List<String> getMyList() {
        return myList;
    }
    public Set<String> getMySet() {
        return mySet;
    }
    public Map<String, String> getMyMap() {
        return myMap;
    }
    public Properties getMyProp() {
        return myProp;
    }
    public void setMyList(List<String> myList) {
        this.myList = myList;
    }
    public void setMySet(Set<String> mySet) {
        this.mySet = mySet;
    }
    public void setMyMap(Map<String, String> myMap) {
        this.myMap = myMap;
    }
    public void setMyProp(Properties myProp) {
        this.myProp = myProp;
    }
}

```

** 除此之外, 还能向集合注入bean的"引用". **

## 10. 注解

### 10.1 配置文件头(当使用注解时, 应使用下面的配置头文件)

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-3.0.xsd">
<context:annotation-config/>
<!-- bean definitions go here -->

</beans>
```

### 10.2 注解

|      注解      |      参数      |                             功能                             |
| :------------: | :------------: | :----------------------------------------------------------: |
|   @Required    |       无       | 用于 bean 属性的 setter 方法，它表明受影响的 bean 属性在配置时必须放在 XML中, <br />否则容器就会抛出一个 BeanInitializationException 异常. |
|   @Autowired   |                |         可以在 setter 方法中被用于自动连接 bean</br>         |
|   @Qualifier   |                | 使用 @Qualifier 注释和 @Autowired 注释通过指定哪一个真正的 bean 将会被装.<br/> usage: @Autowired<br/> @Qualifier("bean") |
| @PostConstruct |                |          作为初始化回调函数的一个替代(init-method)           |
|  @PreDestroy   |                |          作为销毁回调函数的一个替代(destory-method)          |
|   @Resource    | name="bean_id" |   使用一个 ‘name’ 属性，该属性以一个 bean 名称的形式被注入   |

### 10.3 基于java的配置
	基于 Java 的配置选项，可以使你在不用配置 XML 的情况下编写大多数的 Spring



|      注解      |                       参数                        |                             功能                             |
| :------------: | :-----------------------------------------------: | :----------------------------------------------------------: |
| @Configuration |                                                   | 带有 @Configuration 注解的类，表示这个类可以使用 Spring IoC 容器<br />作为 bean 定义的来源 |
|     @Bean      | (initMethod = "init", destroyMethod = "cleanup" ) | 一个带有 @Bean 的注解方法将返回一个对象，<br />该对象应该被注册为在 Spring 应用程序上下文中的bean. 然后使用｀AnnotationConfigApplicationContext｀的getbean法获取实例． |
|    @import     |                       class                       |        @import注解允许从另一个配置类中加载 @Bean 定义        |
|     @Scope     |                 ("prototype")....                 |                                                              |


##  11. 事件
通过 ApplicationEvent 类和 ｀ApplicationListener｀ 接口来提供｀ApplicationContext｀ 中处理事件。如果一个 bean 实现 ｀ApplicationListener｀，那么每次 ApplicationEvent 被发布到 ApplicationContext 上，那个 bean 会被通知.

自定义事件的流程：
1. 定义事件． 创建一个事件类 CustomEvent，继承ApplicationEvent,这个类必须定义一个默认的构造函数，它应该从 ApplicationEvent 类中继承的构造函数．
    ```
        public class CustomEvent extends ApplicationEvent{
            public CustomEvent(Object source) {
            super(source);
        }
            public String toString(){
            return "My Custom Event";
        }
    }
    ```
2. 事件发布．定义｀EventClassPublisher｀, 实现｀ApplicationEventPublisherAware｀接口，用于发布事件．并在配置文件中声明这个类作为一个bean.
    ```
        public class CustomEventPublisher implements ApplicationEventPublisherAware {
                private ApplicationEventPublisher publisher;
                public void setApplicationEventPublisher(ApplicationEventPublisher publisher){
                this.publisher = publisher;
       		 }
            public void publish() {
                CustomEvent ce = new CustomEvent(this);
                publisher.publishEvent(ce);　　// 发布事件
        	}
        }
    ```

3. 事件处理，　定义｀EventClassHandler｀ 并实现 ｀ApplicationListener ｀接口，而且实现了自定义事件的 ｀onApplicationEvent 方法｀，　

    ```
    public class CustomEventHandler
        implements ApplicationListener<CustomEvent>{
            public void onApplicationEvent(CustomEvent event) {
           	 	System.out.println(event.toString());
        }
    }
    ```
4. 配置文件
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
        <bean id="customEventHandler"　class="com.tutorialspoint.CustomEventHandler"/>
        <bean id="customEventPublisher"　class="com.tutorialspoint.CustomEventPublisher"/>
    </beans>
    ```

５．　测试

    ```
    public class MainApp {
        public static void main(String[] args) {
            ConfigurableApplicationContext context =
            new ClassPathXmlApplicationContext("Beans.xml");
            CustomEventPublisher cvp = (CustomEventPublisher)context.getBean("customEventPublisher");
            cvp.publish();
            cvp.publish();
        }
    }
    ```
