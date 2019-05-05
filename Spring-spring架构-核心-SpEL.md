## Spring Expression Language

Spring表达式语言(简称SpEL)是一种功能强大的表达式语言，支持在运行时查询和操作对象图。语言语法类似于统一EL，但提供了额外的功能，最显著的是方法调用和基本字符串模板功能。

虽然还有其他几种Java表达式语言可用——OGNL、MVEL和JBoss EL，仅举几个例子——但Spring表达式语言的创建是为了向Spring社区提供一种得到良好支持的表达式语言，可以跨Spring产品组合中的所有产品使用。

本章将介绍表达式语言的特性、API和语法。

1. Evaluation

    本节介绍SpEL接口的简单使用及其表达式语言。SpEL最常用的类和接口都在org.springframework.expression包及子包spel.support中。

    * SpEL API来计算字面字符串表达式Hello World

        ```java
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'Hello World'"); 
        String message = (String) exp.getValue();
        ```

        message最终的值为'Hello World'

    * SpEL API来合并字符串

        ```java
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'Hello World'.concat('!')"); 
        String message = (String) exp.getValue();
        ```

    * 获取String字节数组

        ```java
        ExpressionParser parser = new SpelExpressionParser();

        // invokes 'getBytes()'
        Expression exp = parser.parseExpression("'Hello World'.bytes"); 
        byte[] bytes = (byte[]) exp.getValue();
        ```

    1. 理解EvaluationContext

        EvaluationContext接口用于在计算表达式时解析属性、方法或字段，并帮助执行类型转换。Spring提供了两种实现。
        * SimpleEvaluationContext:为不需要完全使用SpEL语言语法的表达式类别公开了SpEL语言的基本特性和配置选项的子集，这些表达式应该受到有意义的限制。示例包括但不限于数据绑定表达式和基于属性的过滤器。
        * StandardEevalationcontext:公开了SpEL语言的全部特性和配置选项。您可以使用它来指定缺省根对象，并配置每个可用的与评估相关的策略。

        ```java
        class Simple {
            public List<Boolean> booleanList = new ArrayList<Boolean>();
        }

        Simple simple = new Simple();
        simple.booleanList.add(true);

        EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

        // "false" is passed in here as a String. SpEL and the conversion service
        // will recognize that it needs to be a Boolean and convert it accordingly.
        parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

        // b is false
        Boolean b = simple.booleanList.get(0);
        ```

    2. Parser配置

        可以使用解析器配置对象(org.springframework.expression.spel.SpelParserConfiguration)配置SpEL表达式解析器。配置对象控制一些表达式组件的行为。

        ```java
        class Demo {
            public List<String> list;
        }

        // Turn on:
        // - auto null reference initialization
        // - auto collection growing
        SpelParserConfiguration config = new SpelParserConfiguration(true,true);

        ExpressionParser parser = new SpelExpressionParser(config);

        Expression expression = parser.parseExpression("list[3]");

        Demo demo = new Demo();

        Object o = expression.getValue(demo);

        // demo.list will now be a real collection of 4 entries
        // Each entry is a new empty String
        ```

    3. SpEL编译

        Spring Framework 4.1包含一个基本的表达式编译器。表达式通常是解释的，这在评估期间提供了很多动态灵活性，但并没有提供最佳性能。对于偶尔使用表达式，这是可以的，但是，当被其他组件(如Spring Integration)使用时，性能可能非常重要，并且不需要真正的动态性。

        * 编译器配置

            默认情况下编译器不会打开，但是您可以用两种不同的方式打开它。
            * 可以通过使用解析器配置过程(前面讨论过)
            * 在SpEL用法嵌入到另一个组件中时使用系统属性来打开它。

            编译器可以在org.springframework.expression.spel中捕获的三种模式中进行操作。SpelCompilerMode枚举。模式如下:

            * OFF (默认):编译器已关闭。
            * IMMEDIATE:在立即模式下，表达式将尽快编译。这通常是在第一次解释评估之后。如果编译后的表达式失败(如前所述，通常由于类型更改)，则表达式求值的调用者将收到异常。
            * MIXED:在混合模式下，表达式会随着时间在解释模式和编译模式之间悄然切换。经过一些解释后，它们切换到编译后的表单，如果编译后的表单出现问题(如前面描述的类型更改)，表达式将自动切换回解释后的表单。稍后，它可能生成另一个编译后的表单并切换到它。基本上，用户在即时模式中获得的异常是在内部处理的。

        * 编译器限制

            自Spring Framework 4.1以来，基本编译框架已经就位。然而，该框架还不支持编译每种表达式。最初的重点是可能在性能关键上下文中使用的公共表达式。目前无法编译以下几种表达式:
            * 表达式包括赋值
            * 依赖于转换服务的表达式
            * 使用自定义解析器或访问器的表达式
            * 使用选择或投影的表达式
            
            将来可以编译更多类型的表达式。

2. Bean定义中的表达式

    您可以使用带有基于xml或基于注释的配置元数据的SpEL表达式来定义bean定义实例。在这两种情况下，定义表达式的语法形式都是#{<表达式字符串>}。

    1. XML配置

        属性或构造函数参数值可以使用表达式设置，如下例所示:

        ```xml
        <bean id="numberGuess" class="org.spring.samples.NumberGuess">
            <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

            <!-- other properties -->
        </bean>
        ```

        systemProperties变量是预定义的，所以您可以在表达式中使用它，如下面的示例所示:

        ```xml
        <bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
            <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>

            <!-- other properties -->
        </bean>
        ```

    2. 注解配置

        要指定默认值，可以将@Value注释放在字段、方法和方法或构造函数参数上。

        ```java
        public static class FieldValueTestBean

            @Value("#{ systemProperties['user.region'] }")
            private String defaultLocale;

            public void setDefaultLocale(String defaultLocale) {
                this.defaultLocale = defaultLocale;
            }

            public String getDefaultLocale() {
                return this.defaultLocale;
            }

        }
        ```

3. 语言参考

    本节描述Spring表达式语言如何工作。它包括下列主题:

    * 文字表达式
    * 属性，数组，列表，Map和索引
    * 内联列表
    * 内联Maps
    * 数组构造器
    * 方法
    * 运算符
    * 类型
    * 构造器
    * 变量
    * 行数
    * Bean引用
    * 三元操作符(if - then - else)
    * Elvis操作符
    * 安全导航栏操作符

    1. 文字表达式

        支持的文字表达式类型包括字符串、数值(int、real、hex)、布尔值和null。字符串由单引号分隔。要将单引号本身放在字符串中，请使用两个单引号字符。

        ```java
        ExpressionParser parser = new SpelExpressionParser();

        // evals to "Hello World"
        String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

        double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

        // evals to 2147483647
        int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

        boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

        Object nullValue = parser.parseExpression("null").getValue();
        ```

    2. 属性，数组，列表，Map和索引

        ```java
        int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

        String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
        
        // Officer's Dictionary

        Inventor pupin = parser.parseExpression("Officers['president']").getValue(societyContext, Inventor.class);

        // evaluates to "Idvor"
        String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(societyContext, String.class);

        // setting values
        parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(societyContext, "Croatia");
        ```

    3. 内联列表

        您可以使用{}符号直接在表达式中表示列表。

        ```java
        // evaluates to a Java list containing the four numbers
        List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

        List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
        ```

    4. 内联Maps

        您还可以使用{key:value}符号直接在表达式中表示映射。

        ```java
        // evaluates to a Java map containing the two entries
        Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

        Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
        ```

    5. 数组构造器

        您可以使用熟悉的Java语法构建数组，还可以选择提供初始化器，以便在构建时填充数组。

        ```java
        int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

        // Array with initializer
        int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

        // Multi dimensional array
        int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
        ```

    6. 方法

        您可以使用典型的Java编程语法调用方法。您还可以对文字调用方法。还支持变量参数。

        ```java
        // string literal, evaluates to "bc"
        String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

        // evaluates to true
        boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(societyContext, Boolean.class);
        ```

    7. 运算符

        * 关系运算符

            ```java
            // evaluates to true
            boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

            // evaluates to false
            boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

            // evaluates to true
            boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
            ```

        * 逻辑运算符

            SpEL支持以下的逻辑运算符:

            * and
            * or
            * not

            ```java
            // -- AND --

            // evaluates to false
            boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

            // evaluates to true
            String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
            boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

            // -- OR --

            // evaluates to true
            boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

            // evaluates to true
            String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
            boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

            // -- NOT --

            // evaluates to false
            boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

            // -- AND and NOT --
            String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
            boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
            ```

        * 数学运算符
        
            ```java
            // Addition
            int two = parser.parseExpression("1 + 1").getValue(Integer.class);  // 2

            String testString = parser.parseExpression(
                    "'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

            // Subtraction
            int four = parser.parseExpression("1 - -3").getValue(Integer.class);  // 4

            double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class);  // -9000

            // Multiplication
            int six = parser.parseExpression("-2 * -3").getValue(Integer.class);  // 6

            double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class);  // 24.0

            // Division
            int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class);  // -2

            double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class);  // 1.0

            // Modulus
            int three = parser.parseExpression("7 % 4").getValue(Integer.class);  // 3

            int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class);  // 1

            // Operator precedence
            int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class);  // -21
            ```

        * 赋值运算符

            ```java
            Inventor inventor = new Inventor();
            EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();

            parser.parseExpression("Name").setValue(context, inventor, "Aleksandar Seovic");

            // alternatively
            String aleks = parser.parseExpression("Name = 'Aleksandar Seovic'").getValue(context, inventor, String.class);
            ```

    8. 类型

        您可以使用特殊的T操作符来指定java.lang.Class的实例(类型)。静态方法也可以通过使用这个操作符来调用。
        
        ```java
        Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

        Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

        boolean trueValue = parser.parseExpression(
            "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
            .getValue(Boolean.class);
        ```

    9. 构造器

        您可以使用new操作符来调用构造函数。除了基本类型(int、float等)和字符串之外，应该对所有其他类型使用完全限定的类名。

        ```java
        Inventor einstein = p.parseExpression(
                "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
                .getValue(Inventor.class);

        //create new inventor instance within add method of List
        p.parseExpression(
                "Members.add(new org.spring.samples.spel.inventor.Inventor(
                    'Albert Einstein', 'German'))").getValue(societyContext);
        ```

    10. 变量

        可以使用#variableName语法引用表达式中的变量。变量是通过在EvaluationContext实现上使用setVariable方法设置的。

        ```java
        Inventor tesla = new Inventor("Nikola Tesla", "Serbian");

        EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();
        context.setVariable("newName", "Mike Tesla");

        parser.parseExpression("Name = #newName").getValue(context, tesla);
        System.out.println(tesla.getName())  // "Mike Tesla"
        ```

    11. 函数

        您可以通过注册可以在表达式字符串中调用的用户定义函数来扩展SpEL。函数通过EvaluationContext注册。

        ```java
        public abstract class StringUtils {

            public static String reverseString(String input) {
                StringBuilder backwards = new StringBuilder(input.length());
                for (int i = 0; i < input.length(); i++)
                    backwards.append(input.charAt(input.length() - 1 - i));
                }
                return backwards.toString();
            }
        }
        ```

        ```java
        ExpressionParser parser = new SpelExpressionParser();

        EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
        context.setVariable("reverseString",
                StringUtils.class.getDeclaredMethod("reverseString", String.class));

        String helloWorldReversed = parser.parseExpression(
                "#reverseString('hello')").getValue(context, String.class);
        ```

    12. Bean引用

        如果计算上下文已经配置了bean解析器，则可以使用@符号从表达式中查找bean。

        ```java
        ExpressionParser parser = new SpelExpressionParser();
        StandardEvaluationContext context = new StandardEvaluationContext();
        context.setBeanResolver(new MyBeanResolver());

        // This will end up calling resolve(context,"something") on MyBeanResolver during evaluation
        Object bean = parser.parseExpression("@something").getValue(context);
        ```

    13. 三元操作符(if - then - else)

        可以使用三元运算符在表达式中执行if-then-else条件逻辑。

        ```java
        String falseString = parser.parseExpression(
        "false ? 'trueExp' : 'falseExp'").getValue(String.class);
        ```

    14. Elvis操作符

        Elvis操作符是三元操作符语法的缩写，在Groovy语言中使用。使用三元运算符语法，通常需要重复一个变量两次

        ```java
        String name = "Elvis Presley";
        String displayName = (name != null ? name : "Unknown");
        ```

        相反，您可以使用Elvis操作符(得名于Elvis发型的相似性)。

        ```java
        ExpressionParser parser = new SpelExpressionParser();

        String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);
        System.out.println(name);  // 'Unknown'
        ```

    15. 安全导航操作符

        安全导航操作符用于避免NullPointerException，它来自Groovy语言。通常，当您拥有对对象的引用时，您可能需要在访问对象的方法或属性之前验证它是否为空。为了避免这种情况，安全导航操作符返回null，而不是抛出异常。

        ```java
        ExpressionParser parser = new SpelExpressionParser();
        EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

        Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
        tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

        String city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, tesla, String.class);
        System.out.println(city);  // Smiljan

        tesla.setPlaceOfBirth(null);
        city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, tesla, String.class);
        System.out.println(city);  // null - does not throw NullPointerException!!!
        ```

    16. 集合选择

        选择是一个功能强大的表达式语言特性，它允许您通过从源集合的条目中进行选择，将源集合转换为另一个集合。

        ```java
        List<Inventor> list = (List<Inventor>) parser.parseExpression(
        "Members.?[Nationality == 'Serbian']").getValue(societyContext);

        Map newMap = parser.parseExpression("map.?[value<27]").getValue();
        ```

    17. 集合投射

        投影让集合驱动子表达式的计算，结果是一个新的集合。投影的语法是.![projectionExpression]。

        ```java
        // returns ['Smiljan', 'Idvor' ]
        List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
        ```

    18. 表达式模板

        表达式模板允许将文本与一个或多个计算块混合使用。每个计算块由可以定义的前缀和后缀字符分隔。常见的选择是使用#{}作为分隔符

        ```java
        String randomPhrase = parser.parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext()).getValue(String.class);

        // evaluates to "random number is 0.7038186818312008"
        ```

4. 示例中使用的类

    本章示例中使用的类。用@Data来替代setter和getter方法。

    1. Inventor.java

        ```java
        package org.spring.samples.spel.inventor;

        import java.util.Date;
        import java.util.GregorianCalendar;

        @Data
        public class Inventor {

            private String name;
            private String nationality;
            private String[] inventions;
            private Date birthdate;
            private PlaceOfBirth placeOfBirth;

            public Inventor(String name, String nationality) {
                GregorianCalendar c= new GregorianCalendar();
                this.name = name;
                this.nationality = nationality;
                this.birthdate = c.getTime();
            }

            public Inventor(String name, Date birthdate, String nationality) {
                this.name = name;
                this.nationality = nationality;
                this.birthdate = birthdate;
            }

            public Inventor() {
            }
        }
        ```

    2. PlaceOfBirth.java

        ```java
        package org.spring.samples.spel.inventor;

        @Data
        public class PlaceOfBirth {

            private String city;
            private String country;

            public PlaceOfBirth(String city) {
                this.city=city;
            }

            public PlaceOfBirth(String city, String country) {
                this(city);
                this.country = country;
            }
        }
        ```

    3. Society.java

        ```java
        package org.spring.samples.spel.inventor;

        import java.util.*;

        @Data
        public class Society {

            private String name;

            public static String Advisors = "advisors";
            public static String President = "president";

            private List<Inventor> members = new ArrayList<Inventor>();
            private Map officers = new HashMap();
            public boolean isMember(String name) {
                for (Inventor inventor : members) {
                    if (inventor.getName().equals(name)) {
                        return true;
                    }
                }
                return false;
            }

        }
        ```