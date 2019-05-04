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

        ```java
        import lombok.Data;
        
        @Data
        public class Employee {

            private String name;
            private float salary;
        }
        ```

        以下代码片段展示了如何检索和操作实例化的公司和员工的一些属性:

        ```java
        BeanWrapper company = new BeanWrapperImpl(new Company());
        // setting the company name..
        company.setPropertyValue("name", "Some Company Inc.");
        // ... can also be done like this:
        PropertyValue value = new PropertyValue("name", "Some Company Inc.");
        company.setPropertyValue(value);

        // ok, let's create the director and tie it to the company:
        BeanWrapper jim = new BeanWrapperImpl(new Employee());
        jim.setPropertyValue("name", "Jim Stravinsky");
        company.setPropertyValue("managingDirector", jim.getWrappedInstance());

        // retrieving the salary of the managingDirector through the company
        Float salary = (Float) company.getPropertyValue("managingDirector.salary");
        ```

    2. 内置PropertyEditor实现

        Spring使用PropertyEditor的概念来实现对象和字符串之间的转换。用不同于对象本身的方式表示属性是很方便的。

        在Spring中使用PropertyEditor的几个例子:

        * 在bean上设置属性是通过使用PropertyEditor实现实现的。当您使用String作为在XML文件中声明的某个bean的属性的值时，Spring(如果相应属性的setter有一个类参数)使用ClassEditor尝试将该参数解析为一个类对象。

        * 在Spring的MVC框架中解析HTTP请求参数是通过使用各种PropertyEditor实现来完成的，您可以在CommandController的所有子类中手动绑定这些实现。

        内置PropertyEditor实现

        
        Class|Explanation
        --|--
        ByteArrayPropertyEditor|字节数组编辑器。将字符串转换为相应的字节表示形式。BeanWrapperImpl默认注册。
        ClassEditor|将表示类的字符串解析为实际类，反之亦然。当没有找到类时，抛出一个IllegalArgumentException。默认情况下，由beanwrapperimpl注册。
        CustomBooleanEditor|布尔属性的可自定义属性编辑器。默认情况下，注册了beanwrapperimpl，但是可以通过将它的自定义实例注册为自定义编辑器来覆盖它。
        CustomCollectionEditor|属性编辑器，将任何源集合转换为给定的targetCollection类型。
        CustomDateEditor|为java.util定制的属性编辑器。支持自定义日期格式。默认情况下未注册。必须根据需要以适当的格式进行用户注册
        CustomNumberEditor|可为任何数字子类(如Integer、Long、Float或Double)定制属性编辑器。默认情况下，由BeanWrapperImpl注册，但是可以通过将它的自定义实例注册为自定义编辑器来覆盖它。
        FileEditor|将字符串解析为java.io.File对象。默认情况下，由BeanWrapperImpl注册。
        InputStreamEditor|单向属性编辑器，它可以接受一个字符串并生成一个InputStream(通过一个中间ResourceEditor和Resource)，以便可以直接将InputStream属性设置为字符串。注意，默认用法不会为您关闭inputstream。默认情况下，由BeanWrapperImpl注册。
        LocaleEditor|可以将字符串解析为Locale对象，反之亦然(字符串格式为[country][变体]，与Locale的toString()方法相同)。默认情况下，由BeanWrapperImpl注册。
        PatternEditor|可以将字符串解析为java.util.regex.Pattern对象，反之亦然。
        PropertiesEditor|可以将字符串(格式为java.util.Properties类的javadoc中定义的格式)转换为属性对象。默认情况下，由BeanWrapperImpl注册。
        StringTrimmerEditor|属性编辑器，用于修剪字符串。可选地允许将空字符串转换为空值。默认情况下未注册-必须是用户注册。
        URLEditor|可以将URL的字符串表示形式解析为实际的URL对象。默认情况下，由BeanWrapperImpl注册。

4. Spring类型转换

    Spring 3引入了core.convert包，提供一般类型转换系统。
    
    系统定义了一个SPI来实现类型转换逻辑，以及一个API来在运行时执行类型转换。
    
    在Spring容器中，您可以使用这个系统作为PropertyEditor实现的替代方案，将外部化bean属性值字符串转换为所需的属性类型。您还可以在应用程序中任何需要类型转换的地方使用公共API。

    1. Converter SPI

        SPI实现类型转换逻辑简单而且是强类型的。

        ```java
        package org.springframework.core.convert.converter;

        public interface Converter<S, T> {

            T convert(S source);
        }
        ```

        要创建自己的转换器，请实现转换器接口，并将S作为要转换的类型参数化，将T作为要转换的类型参数化。

        为了方便，在core.convert中提供了几个转换器实现。

        ```java
        package org.springframework.core.convert.support;

        final class StringToInteger implements Converter<String, Integer> {

            public Integer convert(String source) {
                return Integer.valueOf(source);
            }
        }
        ```

    2. 使用ConverterFactory

        当您需要集中整个类层次结构的转换逻辑时(例如，当从String转换为Enum对象时)，您可以实现ConverterFactory。

        ```java
        package org.springframework.core.convert.converter;

        public interface ConverterFactory<S, R> {

            <T extends R> Converter<S, T> getConverter(Class<T> targetType);
        }
        ```

        参数化S为要转换的类型，R为定义可以转换为的类范围的基类型。然后实现getConverter(类<T>)，其中T是R的子类。以StringToEnumConverterFactory为例:

        ```java
        package org.springframework.core.convert.support;

        final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

            public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
                return new StringToEnumConverter(targetType);
            }

            private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

                private Class<T> enumType;

                public StringToEnumConverter(Class<T> enumType) {
                    this.enumType = enumType;
                }

                public T convert(String source) {
                    return (T) Enum.valueOf(this.enumType, source.trim());
                }
            }
        }
        ```

    3. 使用GenericConverter

        当您需要复杂的转换器实现时，请考虑使用GenericConverter接口。GenericConverter具有比转换器更灵活但类型更弱的签名，支持在多个源和目标类型之间进行转换。

        ```java
        package org.springframework.core.convert.converter;

        public interface GenericConverter {

            public Set<ConvertiblePair> getConvertibleTypes();

            Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
        }
        ```

        要实现GenericConverter，让getConvertibleTypes()返回支持的源→目标类型对。然后实现convert(对象、类型描述符、类型描述符)来包含转换逻辑。源类型描述符提供对保存转换值的源字段的访问。目标类型描述符提供对要设置转换值的目标字段的访问。

        GenericConverter的一个很好的例子是在Java数组和集合之间进行转换的转换器。这样的ArrayToCollectionConverter内省声明目标集合类型的字段，以解析集合的元素类型。这允许在目标字段上设置集合之前，将源数组中的每个元素转换为集合元素类型。

        * ConditionalGenericConverter

            有时，您希望仅在特定条件为真时才运行转换器。例如，您可能希望仅在目标字段上出现特定注释时才运行转换器，或者仅在目标类上定义了特定方法(例如静态valueOf方法)时才运行转换器。ConditionalGenericConverter是GenericConverter和ConditionalConverter接口的结合，允许您定义这样的自定义匹配标准

            ```java
            public interface ConditionalConverter {

                boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
            }

            public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
            }
            ```

    4. ConversionService API

        ConversionService定义了一个用于在运行时执行类型转换逻辑的统一API。转换器通常在以下外观接口之后执行

        ```java
        package org.springframework.core.convert;

        public interface ConversionService {

            boolean canConvert(Class<?> sourceType, Class<?> targetType);

            <T> T convert(Object source, Class<T> targetType);

            boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

            Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

        }
        ```

        大多数ConversionService实现还实现了ConverterRegistry，它为注册转换器提供了SPI。在内部，ConversionService实现委托给它注册的转换器来执行类型转换逻辑。

    5. 配置ConversionService

        ConversionService是一个无状态对象，设计用于在应用程序启动时实例化，然后在多个线程之间共享。在Spring应用程序中，通常为每个Spring容器(或ApplicationContext)配置一个ConversionService实例。Spring获取ConversionService，并在框架需要执行类型转换时使用它。您还可以将此转换服务注入任何bean并直接调用它。

        *如果没有向Spring注册ConversionService，则使用原始的基于PropertyEditor的系统。*

        ```xml
        <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean"/>
        ```

        默认的转换服务可以在字符串、数字、枚举、集合、映射和其他常见类型之间进行转换。要使用自己的自定义转换器补充或覆盖默认转换器，请设置转换器属性。

        ```xml
        <bean id="conversionService"
        class="org.springframework.context.support.ConversionServiceFactoryBean">
            <property name="converters">
                <set>
                    <bean class="example.MyCustomConverter"/>
                </set>
            </property>
        </bean>
        ```

    6. 以编程方式使用ConversionService

        ```java
        @Service
        public class MyService {

            @Autowired
            public MyService(ConversionService conversionService) {
                this.conversionService = conversionService;
            }

            public void doIt() {
                this.conversionService.convert(...)
            }
        }
        ```

5. Spring字段格式化

    如前所述，core.convert是一种通用类型转换系统。它提供了统一的ConversionService API以及用于实现从一种类型到另一种类型转换逻辑的强类型转换器SPI。Spring容器使用此系统绑定bean属性值。此外，Spring表达式语言(SpEL)和DataBinder都使用这个系统绑定字段值。

    1. Formatter SPI

        用于实现字段格式化逻辑的Formatter SPI简单且具有强类型。

        ```java
        package org.springframework.format;

        public interface Formatter<T> extends Printer<T>, Parser<T> {
        }
        ```

        Formatter从Printer和Parser构建块接口扩展而来。

        ```java
        public interface Printer<T> {

            String print(T fieldValue, Locale locale);
        }
        ```

        ```java
        import java.text.ParseException;

        public interface Parser<T> {

            T parse(String clientValue, Locale locale) throws ParseException;
        }
        ```

        要创建自己的格式化程序，请实现前面显示的Formatter接口。参数化T为您希望格式化的对象类型，例如，java.util.Date。

        * 实现print()操作来打印一个T实例，以便在客户机区域设置中显示。
        * 实现parse()操作，从客户机区域设置返回的格式化表示形式中解析T的实例。
        
        如果解析尝试失败，格式化程序应该抛出ParseException或IllegalArgumentException。请注意确保格式化程序实现是线程安全的。

        ```java
        package org.springframework.format.datetime;

        public final class DateFormatter implements Formatter<Date> {

            private String pattern;

            public DateFormatter(String pattern) {
                this.pattern = pattern;
            }

            public String print(Date date, Locale locale) {
                if (date == null) {
                    return "";
                }
                return getDateFormat(locale).format(date);
            }

            public Date parse(String formatted, Locale locale) throws ParseException {
                if (formatted.length() == 0) {
                    return null;
                }
                return getDateFormat(locale).parse(formatted);
            }

            protected DateFormat getDateFormat(Locale locale) {
                DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
                dateFormat.setLenient(false);
                return dateFormat;
            }
        }
        ```

    2. 注解驱动格式化

        字段格式可以通过字段类型或注释配置。要将注释绑定到格式化程序，请实现AnnotationFormatterFactory。

        ```java
        package org.springframework.format;

        public interface AnnotationFormatterFactory<A extends Annotation> {

            Set<Class<?>> getFieldTypes();

            Printer<?> getPrinter(A annotation, Class<?> fieldType);

            Parser<?> getParser(A annotation, Class<?> fieldType);
        }
        ```

        下面的示例AnnotationFormatterFactory实现将@NumberFormat注释绑定到格式化程序，以便指定数字样式或模式

        ```java
        public final class NumberFormatAnnotationFormatterFactory implements AnnotationFormatterFactory<NumberFormat> {

            public Set<Class<?>> getFieldTypes() {
                return new HashSet<Class<?>>(asList(new Class<?>[] {
                    Short.class, Integer.class, Long.class, Float.class,
                    Double.class, BigDecimal.class, BigInteger.class }));
            }

            public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
                return configureFormatterFrom(annotation, fieldType);
            }

            public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
                return configureFormatterFrom(annotation, fieldType);
            }

            private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) {
                if (!annotation.pattern().isEmpty()) {
                    return new NumberStyleFormatter(annotation.pattern());
                } else {
                    Style style = annotation.style();
                    if (style == Style.PERCENT) {
                        return new PercentStyleFormatter();
                    } else if (style == Style.CURRENCY) {
                        return new CurrencyStyleFormatter();
                    } else {
                        return new NumberStyleFormatter();
                    }
                }
            }
        }
        ```

        要触发格式化，可以使用@NumberFormat注释字段，如下面的示例所示:

        ```java
        public class MyModel {

            @NumberFormat(style=Style.CURRENCY)
            private BigDecimal decimal;
        }
        ```

    3. FormatterRegistry

        FormatterRegistry是一个用于注册格式化程序和转换器的SPI。FormattingConversionService是FormatterRegistry的一个实现，适用于大多数环境。

        ```java
        package org.springframework.format;

        public interface FormatterRegistry extends ConverterRegistry {

            void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

            void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

            void addFormatterForFieldType(Formatter<?> formatter);

            void addFormatterForAnnotation(AnnotationFormatterFactory<?, ?> factory);
        }
        ```

    4. FormatterRegistrar SPI

        FormatterRegistry是一个SPI，用于通过FormatterRegistry注册格式化程序和转换器。下面的清单显示了它的接口定义

        ```java
        package org.springframework.format;

        public interface FormatterRegistrar {

            void registerFormatters(FormatterRegistry registry);
        }
        ```

6. 配置全局的日期时间格式

    可以自定义定义自己的全局格式。

    为此，您需要确保Spring没有注册默认格式器。相反，您应该手动注册所有格式化程序。使用org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar或org.springframework.format.datetime.DateFormatterRegistrar类。

    ```java
    @Configuration
    public class AppConfig {

        @Bean
        public FormattingConversionService conversionService() {

            // Use the DefaultFormattingConversionService but do not register defaults
            DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

            // Ensure @NumberFormat is still supported
            conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

            // Register date conversion with a specific global format
            DateFormatterRegistrar registrar = new DateFormatterRegistrar();
            registrar.setFormatter(new DateFormatter("yyyyMMdd"));
            registrar.registerFormatters(conversionService);

            return conversionService;
        }
    }
    ```

    xml的方式如下

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd>

        <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
            <property name="registerDefaultFormatters" value="false" />
            <property name="formatters">
                <set>
                    <bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
                </set>
            </property>
            <property name="formatterRegistrars">
                <set>
                    <bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
                        <property name="dateFormatter">
                            <bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
                                <property name="pattern" value="yyyyMMdd"/>
                            </bean>
                        </property>
                    </bean>
                </set>
            </property>
        </bean>
    </beans>
    ```

7. Spring Validator

    Spring 3为其验证支持引入了几个增强。首先，完全支持JSR-303 Bean验证API。其次，当以编程方式使用时，Spring的DataBinder可以验证对象并绑定到它们。第三，Spring MVC支持声明性验证@Controller输入。

    1. JSR-303 Bean验证API概述

        JSR-303标准化了Java平台的验证约束声明和元数据。通过使用此API，您可以使用声明性验证约束对域模型属性进行注释，并由运行时强制执行它们。您可以使用许多内置的约束。您还可以定义自己的自定义约束。

        ```java
        public class PersonForm {
            private String name;
            private int age;
        }
        ```

        JSR-303允许您根据这些属性定义声明性验证约束

        ```java
        public class PersonForm {

            @NotNull
            @Size(max=64)
            private String name;

            @Min(0)
            private int age;
        }
        ```

    2. 配置Bean验证提供程序

        Spring提供了对Bean验证API的完全支持。这包括方便地支持将JSR-303或JSR-349 Bean验证提供者引导为Spring Bean。这允许您注入javax.validation.ValidatorFactory或javax.validation。在应用程序中需要验证的地方使用验证器。

        ```xml
        <bean id="validator"
        class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
        ```

        1. 注入一个Validator

            LocalValidatorFactoryBean同时实现了javax.validation.ValidatorFactory和javax.validation.Validator，以及Spring的org.springframework.validation.Validator。

            您可以将引用注入到javax.validation.Validator，如果你喜欢直接使用Bean验证API。

            ```java
            import javax.validation.Validator;

            @Service
            public class MyService {

                @Autowired
                private Validator validator;
            }
            ```

        2. 配置自定义约束

            每个bean验证约束由两个部分组成:

            * 一个@Constraint注释，声明约束及其可配置属性。
            * javax.validation的实现。实现约束行为的ConstraintValidator接口。

            要将声明与实现关联，每个@Constraint注释引用一个对应的ConstraintValidator实现类。在运行时，当域模型中遇到约束注释时，ConstraintValidatorFactory实例化引用的实现。

            ```java
            @Target({ElementType.METHOD, ElementType.FIELD})
            @Retention(RetentionPolicy.RUNTIME)
            @Constraint(validatedBy=MyConstraintValidator.class)
            public @interface MyConstraint {
            }
            ```

            ```java
            import javax.validation.ConstraintValidator;

            public class MyConstraintValidator implements ConstraintValidator {

                @Autowired;
                private Foo aDependency;

                ...
            }
            ```

    3. 配置DataBinder

        自Spring 3以来，您可以使用验证器配置DataBinder实例。一旦配置好，您就可以通过调用bind.validate()来调用验证器。任何验证错误都会自动添加到绑定器的BindingResult中。

        下面的例子展示了如何使用DataBinder编程调用绑定到目标对象后的验证逻辑

        ```java
        Foo target = new Foo();
        DataBinder binder = new DataBinder(target);
        binder.setValidator(new FooValidator());

        // bind to the target object
        binder.bind(propertyValues);

        // validate the target object
        binder.validate();

        // get BindingResult that includes any validation errors
        BindingResult results = binder.getBindingResult();
        ```

    4. Spring MVC 3校验

        参考Spring MVC的[Validation](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/web.html#mvc-config-validation)

参考：https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html#validation