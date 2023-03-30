# Java

## 基本类型

### 整数类型

|长度|有符号|
|---|---|
|8位|`byte`|
|16位|`short`|
|32位|`int`|
|64位|`lang`|

```java
public class Main {  
    public static void main(String[] args) {  
        byte i = 1;  
        short i2 = 1;  
        int i3 = 1;  
        long i4 = 1;  
        System.out.printf("8位=%d 16位=%d 32位=%d 64位=%d", i, i2, i3, i4);  
    }  
}
```

### 浮点型

|长度|类型|
|---|---|
|32位|`float`|
|64位|`double`|

```java
public class Main {  
    public static void main(String[] args) {  
        float i = 1.111F;  
        double i2 = 1.111F;  
        System.out.printf("32位=%f 64位=%f", i, i2);  
    }  
}
```

布尔类型

```java
public class Main {  
    public static void main(String[] args) {  
        boolean i = false;  
        boolean i2 = true;  
        System.out.printf("32位=%b 64位=%b", i, i2);  
    }  
}
```

### 字符类型

```java
public class Main {  
    public static void main(String[] args) {  
        char i = 'x';  
        System.out.printf("字符=%s", i);  
    }  
}
```

### Number类

```java
public class Main {  
    public static void main(String[] args) {  
        Boolean i = true;  
        Byte i2 = 1;  
        Short i3 = 1;  
        Integer i4 = 1;  
        Long i5 = 1L;  
        Float i6 = 1.111F;  
        Double i7 = 1.11;  
  
        System.out.println(i.equals(i2));  
    }  
}
```

### Character 类

```java
public class Main {  
    public static void main(String[] args) {  
        Character i = 'a';  
        Character i2 = 'b';  
        System.out.println(i.equals(i2));  
    }  
}
```

### String 类

```java
public class Main {  
    public static void main(String[] args) {  
        String i = "a";  
        String i2 = "a";  
  
        System.out.println(i.equals(i2));  
    }  
}
```

### StringBuilder 类

```java
public class Main {  
    public static void main(String[] args) {  
        StringBuilder i =new StringBuilder("a");  
        i.append("b");  
        System.out.println(i);  
    }  
}
```

### 数组

```java
public class Main {  
    public static void main(String[] args) {  
        int[] ns = {1, 2, 3};  
        for (int i = 0; i < ns.length; i++) {  
            int n = ns[i];  
            System.out.println(n);  
        }  
    }  
}
```

## 变量类型

```java
public class ValueTest {  
    public int i = 18; // 实例变量  
    static int i2 = 20; // 类变量  
  
    public void method() {  
        int i3 = 21; // 局部变量  
    }  
}
```

### 修饰符可见性

|修饰符|当前类|当前包|子孙类|子孙类不同包|其他包|
|---|---|---|---|---|---|
|`public`|Y|Y|Y|Y|Y|Y|
|`protected`|Y|Y|Y|Y/N|N|N|
|`default`|Y|Y|Y|N|N|N|
|`private`|Y|N|Y|N|N|N|

## 流程控制

### 循环语句

```java
public class Main {  
    public static void main(String[] args) {  
        int i = 0;  
        while (i < 10) {  
            System.out.println(i);  
            i++;  
        }  
    }  
}
```

```java
public class Main {  
    public static void main(String[] args) {  
        for (int i = 0; i < 10; i++) {  
            System.out.println(i);  
        }  
    }  
}
```

### 条件语句

```java
public class Main {  
    public static void main(String[] args) {  
        int i = 10;  
        if (i < 10) {  
            System.out.println("小于10");  
        } else if (i > 10) {  
            System.out.println("大于10");  
        } else {  
            System.out.println("等于10");  
        }  
    }  
}
```

```java
public class Main {  
    public static void main(String[] args) {  
        int i = 1;  
        switch (i) {  
            case 1:  
                System.out.println("1");  
            case 2:  
                System.out.println("2");  
            default:  
                System.out.println("其他");  
  
        }  
    }  
}
```


## 面向对象


### 方法

```java
public class User { 

	// 类方法
    public int GetAge() {  
        return 18;  
    }  
}
```

### 构造方法

```java
public class Main {  
    public static void main(String[] args) {  
        User user = new User(17);  
  
        System.out.println(user.GetAge());  
    }  
}  
  
class User {  
    private int age;  

	// 构造方法
    public User(int age) {  
        this.age = age;  
    }  
  
    public int GetAge() {  
        return this.age;  
    }  
}
```

### 方法重载

```java
public class Main {  
    public static void main(String[] args) {  
        User user = new User(17);  
  
        System.out.println(user.GetAge(20));
        System.out.println(user.GetAge());  
    }  
}  
  
class User {  
    private int age;  
  
    User(int age) {  
        this.age = age;  
    }  
  
    int GetAge() {  
        return this.age;  
    }  
	// 方法名相同 实参不同  
    int GetAge(int age) {  
        return age;  
    }  
}
```

### 继承重写

```java
public class Main {  
    public static void main(String[] args) {  
        Student user = new Student();  
        System.out.println(user.getUser());  
    }  
}  
  
class Person {  
    private String name;  
    private int age;  
	@Override // 检查签名
    int getUser() {  
        return 123;  
    }  
}  
  
class Student extends Person {  
    // 重写方法
    int getUser() {  
        return 456;  
    }  
}
```


### 多态

实现多态的方式
- 抽象类
- 重写
- 接口

```java
public class Main {  
    public static void main(String[] args) { 
        // 多态 
        Person user = new Student();  
        System.out.println(user.getUser());  
    }  
}  
  
class Person {  
    private String name;  
    private int age;  
  
    int getUser() {  
        return 123;  
    }  
}  
  
class Student extends Person {  
    int getUser(int age) {  
        return age;  
    }  
  
    String getName() {  
        return "张三";  
    }  
}  
  
class Teacher extends Person{  
  
}
```

### 抽象类

```java
public class Main {  
    public static void main(String[] args) { 
        // 抽象类自身无法被实例化 
        Person user = new Teacher();  
        System.out.println(user.getUser());  
    }  
}  
  
abstract class Person {  
    private String name;  
    private int age;  
  
    int getUser() {  
        return 123;  
    }  
}  
  
class Student extends Person {  
    int getUser(int age) {  
        return age;  
    }  
  
    String getName() {  
        return "张三";  
    }  
}  
  
class Teacher extends Person{  
  
}
```


### 接口

```java
public class Main {  
    public static void main(String[] args) {  
        Person user = new Teacher("老师");  
        System.out.println(user.getName());  
    }  
}  
  
interface Person {  
    String getName();  
}  
  
class Student implements Person {  
    private String name;  
  
    Student(String name){  
        this.name = name;  
    }  

	// 实现接口
    @Override  
    public String getName() {  
        return this.name;  
    }  
}  
  
class Teacher implements Person{  
    private String name;  
  
    Teacher(String name){  
        this.name = name;  
    }  
    @Override  
    public String getName() {  
        return this.name;  
    }  
}
```


### 枚举

```java
public class Main {  
    public static void main(String[] args) {  
        Log log = Log.DEBUG;  
        if (log.level >= 3) {  
            System.out.println("日志级别是DEBUG");  
        }  
  
    }  
}  
  
enum Log {  
    DEBUG(3), INFO(2), ERROR(3);  
  
    public final int level;  
  
    Log(int i) {  
        this.level = i;  
    }  
  
}
```

## HashMap

```java
public class Main {  
    public static void main(String[] args) {  
        HashMap<String, Integer> m = new HashMap<String, Integer>();  
        m.put("key1", 1);  
        m.put("key2", 2);  
        m.remove("key2");  
  
        System.out.println(m.get("key1"));  
    }  
}
```

## 泛型

```java
public class Main {  
    static <E> void printArray(E[] inputArray) {  
        for (E element : inputArray) {  
            System.out.println(element);  
        }  
    }  
  
    public static void main(String[] args) {  
        Integer[] i = {1, 2, 3};  
        printArray(i);  
  
        String[] i2 = {"a", "b", "c"};  
        printArray(i2);  
    }  
}
```

```java
public class Main {  
    static <T extends Number> void printArray(T[] inputArray) {  
        // 约束T必须是Number 类型  
        for (T element : inputArray) {  
            System.out.println(element);  
        }  
    }  
  
    public static void main(String[] args) {  
        Integer[] i = {1, 2, 3};  
        printArray(i);  
  
        Float[] i2 = {1.1F, 1.2F, 1.3F};  
        printArray(i2);  
    }  
}
```

## 反射

```java
public class Main {  
    public static void main(String[] args) throws Exception {  
        Person p = new Student("张三");  
        System.out.println(p.getClass().getMethod("getName"));  
        System.out.println(p.getClass().getField("name"));  
        System.out.println(p.getClass().getDeclaredField("age"));  
  
        // 反射字段的值  
        Field name = p.getClass().getField("name");  
        Object value = name.get(p);  
        System.out.println(value);  
  
        // 调用方法  
        Method m = p.getClass().getMethod("getAge", int.class);  
        Object age = m.invoke(p, 20);  
        System.out.println(age);  
  
        // 调用构造方法  
        Constructor c = p.getClass().getConstructor(String.class);  
        Object p2 = c.newInstance("李四");  
        System.out.println(p2.getClass().getField("name").get(p2));  
  
        // 获取继承关系  
        System.out.println(p.getClass().getSuperclass());  
    }  
}  
  
class Person {  
    public String name;  
    private int age;  
  
    public Person(String name) {  
        this.name = name;  
    }  
  
    public String getName() {  
        return this.name;  
    }  
  
    public int getAge(int age) {  
        return age;  
    }  
}  
  
class Student extends Person {  
    private String name;  
    private int age;  
    public Student(String name) {  
        super(name);  
    }  
  
    @Override  
    public String getName() {  
        return "学生";  
    }  
}
```


## 异常处理

```java
public class Main {  
    static void process() {  
        // 抛出异常  
        Integer.parseInt(null);  
    }  
  
    static void process2() {  
        // 向上传播异常  
        process();  
    }  
  
    public static void main(String[] args) {  
        // 捕获异常  
        try {  
            process2();  
            System.out.println(1);  
        } catch (Exception e) {  
            System.out.println(e);  
        }  
    }  
}
```