## spring FactoryBean与BeanFactory

1. BeanFactory

    BeanFactory定义了 IOC 容器的最基本形式，并提供了 IOC 容器应遵守的的最基本的接口，也就是 Spring IOC 所遵守的最底层和最基本的编程规范。

    BeanFactory仅是个接口，并不是IOC容器的具体实现，具体的实现有：如 DefaultListableBeanFactory 、 XmlBeanFactory 、 ApplicationContext 等。

    ```java
    public interface BeanFactory {
        //FactoryBean前缀
        String FACTORY_BEAN_PREFIX = "&";

        //根据名称获取Bean对象
        Object getBean(String name) throws BeansException;

        ///根据名称、类型获取Bean对象
        <T> T getBean(String name, Class<T> requiredType) throws BeansException;

        //根据类型获取Bean对象
        <T> T getBean(Class<T> requiredType) throws BeansException;

        //根据名称获取Bean对象,带参数
        Object getBean(String name, Object... args) throws BeansException;

        //根据类型获取Bean对象,带参数
        <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

        //是否存在
        boolean containsBean(String name);

        //是否为单例
        boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

        //是否为原型（多实例）
        boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

        //名称、类型是否匹配
        boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;

        //获取类型
        Class<?> getType(String name) throws NoSuchBeanDefinitionException;

        //根据实例的名字获取实例的别名
        String[] getAliases(String name);
    }
    ```

2. FactoryBean

    ```java
    public interface FactoryBean<T> {

        //FactoryBean 创建的 Bean 实例
        T getObject() throws Exception;

        //返回 FactoryBean 创建的 Bean 类型
        Class<?> getObjectType();

        //返回由 FactoryBean 创建的 Bean 实例的作用域是 singleton 还是 prototype
        boolean isSingleton();
    }
    ```

3. 示例

    1. 普通bean

        ```java
        public class Dog {
            private String msg;

            public Dog(String msg){
                this.msg=msg;
            }
            public void run(){
                System.out.println(msg);
            }
        }
        ```

    2. 实现了FactoryBean的bean

        ```java
        public class DogFactoryBean implements FactoryBean<Dog>{

            public Dog getObject() throws Exception {
                return new Dog("DogFactoryBean.run");
            }

            public Class<?> getObjectType() {
                return DogFactoryBean.class;
            }

            public boolean isSingleton() {
                return false;
            }
        }
        ```

    3. 配置文件

        ```xml
        <bean id="dog" class="com.bean.Dog" >
            <constructor-arg value="Dog.run"/>
        </bean>

        <bean id="dogFactoryBean" class="com.bean.DogFactoryBean" />
        ```

    4. 测试

        ```java
        @Test
        public void testBean() throws Exception {
            ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

            Dog dog1 = (Dog) ctx.getBean("dog");
            dog1.run();

            Dog dog2 = (Dog) ctx.getBean("dogFactoryBean");
            dog2.run();

            //使用&前缀可以获取FactoryBean本身
            FactoryBean dogFactoryBean = (FactoryBean) ctx.getBean("&dogFactoryBean");
            Dog dog3= (Dog) dogFactoryBean.getObject();
            dog3.run();
        }
        ```

        结果输出：

        ```java
        Dog.run
        DogFactoryBean.run
        DogFactoryBean.run
        ```

4. 总结

    通过以上源码和示例来看，基本上能印证以下结论，也就是二者的区别。

    BeanFactory是个Factory，也就是 IOC 容器或对象工厂，所有的 Bean 都是由 BeanFactory( 也就是 IOC 容器 ) 来进行管理。

    FactoryBean是一个能生产或者修饰生成对象的工厂Bean，可以在IOC容器中被管理，所以它并不是一个简单的Bean。**当使用容器中factory bean的时候，该容器不会返回factory bean本身，而是返回其生成的对象。**

原文：https://www.cnblogs.com/ninth/p/6405366.html