### Thrift的数据类型、基本类型、结构类型
1. 基本类型（括号内为对应的Java类型）：
    * bool（boolean）: 布尔类型(TRUE or FALSE)
    * byte（byte）: 8位带符号整数
    * i16（short）: 16位带符号整数
    * i32（int）: 32位带符号整数
    * i64（long）: 64位带符号整数
    * double（double）: 64位浮点数
    * string（String）: 采用UTF-8编码的字符串

2. 特殊类型（括号内为对应的Java类型）：
    * binary（ByteBuffer）：未经过编码的字节流

3. Structs（结构）：
    > struct定义了一个很普通的OOP对象，但是没有继承特性。
    ```
    struct UserProfile {
    1: i32 uid,
    2: string name,
    3: string blurb
    }
    ```
    > 如果变量有默认值，可以直接写在定义文件里:
    ```
    struct UserProfile {
    1: i32 uid = 1,
    2: string name = "User1",
    3: string blurb
    }
    ```
4. 容器，除了上面提到的基本数据类型，Thrift还支持以下容器类型：
    * list(java.util.ArrayList)：
    * set(java.util.HashSet)：
    * map(java.util.HashMap)：

    > Thrift容器与类型密切相关，它与当前流行编程语言提供的容器类型相对应，采用java泛型风格表示的。Thrift提供了3种容器类型：

    * List<t1>：一系列t1类型的元素组成的有序表，元素可以重复
    * Set<t1>：一系列t1类型的元素组成的无序表，元素唯一
    * Map<t1,t2>：key/value对（key的类型是t1且key唯一，value类型是t2）。

    容器中的元素类型可以是除了service以外的任何合法thrift类型（包括结构体和异常）。

    用法如下：
    ```
    struct Node {
    1: i32 id,
    2: string name,
    3: list<i32> subNodeList,
    4: map<i32,string> subNodeMap,
    5: set<i32> subNodeSet
    }
    ```
    包含定义的其他Object:
    ```
    struct SubNode {
    1: i32 uid,
    2: string name,
    3: i32 pid
    }
    
    struct Node {
    1: i32 uid,
    2: string name,
    3: list<subNode> subNodes
    }
    ```
5. Services服务，也就是对外展现的接口：
    ```
    service UserStorage {
    void store(1: UserProfile user),
    UserProfile retrieve(1: i32 uid)
    }
    ```