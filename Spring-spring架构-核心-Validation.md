## Validation

数据绑定对于将用户输入动态绑定到应用程序的域模型(或用于处理用户输入的任何对象)非常有用。Spring提供了正确命名的DataBinder来实现这一点。验证器和DataBinder组成验证包，验证包主要用于但不限于MVC框架。

1. 使用Spring的Validator接口进行验证

    Spring提供了一个Validator 接口，您可以使用它来验证对象。验证器接口通过使用错误对象来工作，以便在进行验证时，验证器可以向错误对象报告验证失败。

    ```java
    public interface Validator {
        boolean supports(Class<?> clazz);
        void validate(Object target, Errors errors);
    }
    ```

    ```java
    public class Person {

        private String name;
        private int age;

        // the usual getters and setters...
    }
    ```

    ```java
    public class PersonValidator implements Validator {

        /**
        * This Validator validates *only* Person instances
        */
        public boolean supports(Class clazz) {
            return Person.class.equals(clazz);
        }

        public void validate(Object obj, Errors e) {
            ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
            Person p = (Person) obj;
            if (p.getAge() < 0) {
                e.rejectValue("age", "negativevalue");
            } else if (p.getAge() > 110) {
                e.rejectValue("age", "too.darn.old");
            }
        }
    }
    ```

    虽然可以实现单个验证器类来验证富对象中的每个嵌套对象，但最好将每个嵌套对象类的验证逻辑封装在自己的验证器实现中。可以用内嵌验证器的方式实现。

    ```java
    public class CustomerValidator implements Validator {

        private final Validator addressValidator;

        public CustomerValidator(Validator addressValidator) {
            if (addressValidator == null) {
                throw new IllegalArgumentException("The supplied [Validator] is " +
                    "required and must not be null.");
            }
            if (!addressValidator.supports(Address.class)) {
                throw new IllegalArgumentException("The supplied [Validator] must " +
                    "support the validation of [Address] instances.");
            }
            this.addressValidator = addressValidator;
        }

        /**
        * This Validator validates Customer instances, and any subclasses of Customer too
        */
        public boolean supports(Class clazz) {
            return Customer.class.isAssignableFrom(clazz);
        }

        public void validate(Object target, Errors errors) {
            ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
            ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
            Customer customer = (Customer) target;
            try {
                errors.pushNestedPath("address");
                ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
            } finally {
                errors.popNestedPath();
            }
        }
    }
    ```

2. 将代码解析为错误消息

    当您从Errors接口调用(通过使用ValidationUtils类直接或间接地)rejectValue或其他拒绝方法时，底层实现不仅注册您传入的代码，还注册了许多额外的错误代码。MessageCodesResolver确定错误接口寄存器的错误代码。默认情况下，使用DefaultMessageCodesResolver，它(例如)不仅用您提供的代码注册消息，还注册包含传递给reject方法的字段名的消息。

    可参考[MessageCodesResolver](https://docs.spring.io/spring-framework/docs/5.1.6.RELEASE/javadoc-api/org/springframework/validation/MessageCodesResolver.html)和[DefaultMessageCodesResolver](https://docs.spring.io/spring-framework/docs/5.1.6.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html)。

3. Bean操作和BeanWrapper

    org.springframework.beans包遵循javabean标准。JavaBean是一个具有默认无参数构造函数的类。

    bean包中一个非常重要的类是BeanWrapper接口及其相应的实现(BeanWrapperImpl)。正如javadoc所引用的，BeanWrapper提供了设置和获取属性值(单独或批量)、获取属性描述符以及查询属性以确定它们是否可读或可写的功能。

    1. 设置和获取基本和嵌套属性

        设置和获取属性是通过使用setPropertyValue、setPropertyValues、getPropertyValue和getPropertyValues方法来完成的，这些方法带有两个重载的变量。其约定如下

        表达式|解释
        --|--
        name|指示与getName()或isName()和setName(..)方法相对应的属性名。
        account.name|指示属性account的嵌套属性名，该属性名对应于(例如)getAccount(). setname()或getAccount(). getname()方法
        account[2]|指示索引属性帐户的第三个元素。索引属性可以是数组、列表或其他自然有序集合的类型。
        account[COMPANYNAME]|指示按account map属性的COMPANYNAME键索引的映射条目的值。

        ```java
        import lombok.Data;
        
        @Data
        public class Company {

            private String name;
            private Employee managingDirector;
        }
        ```


4. Spring类型转换

5. Spring字段格式化

6. 配置全局的日期时间格式

7. Spring Validator

参考：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#validation