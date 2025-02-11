# Rust

## 基本类型

### 整数类型

|长度|有符号类型|无符号类型|
|---|---|---|
|8位|`i8`|`u8`|
|16位|`i16`|`u16`|
|32位|`i32`|`u32`|
|64位|`i64`|`u64`|
|视架构而定|`isize`|`usize`|

```rust
fn main() {
    let x: i8 = -1;
    let x2: u8 = 1;
    println!("8位有符号{}, 8位无符号{}", x, x2)
}
```

### 浮点类型

|长度|类型|
|---|---|
|32位|`f32`|
|64位|`f64`|

```rust
fn main() {
    let x: f32 = -1.0;
    let x2: f32 = 1.0;
    println!("32位有符号{}, 32位无符号{}", x, x2)
}
```

### 字符类型

```rust
fn main() {
    let x: char = '中';
    println!("字符类型:{}", x)
}
```

### 布偶类型

```rust
fn main() {
    let x: bool = true;
    println!("布偶类型:{}", x)
}

```

### 单元类型

单元类型是`()` 类似Go语言中的`struct{}` 单元类型不占用内存 可以用于`map`中的占位值

### 函数

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    let x = add(1, 2);
    println!("{}", x)
}
```

```rust
fn add(a: i32, b: i32) -> i32 {
   let x = a + b;
   return x
}

fn main() {
    let x = add(1, 2);
    println!("{}", x)
}
```

```rust
fn print_add(a: i32, b: i32) {
    println!("a+b={}", a + b)
}

fn main() {
    print_add(1, 2);
}
```

#### 闭包

```rust
fn main() {
    let call = ||{
        1
    };

    println!("{}", call())
}
```

```rust
fn test_f(f: fn() -> i32) -> fn() -> i32 {
    f
}


fn main() {
    let f = || -> i32 {
        1
    };
    let x = test_f(f);

    println!("{}", x())
}

```

## 复合类型

### 字符串

- `str`是动态大小类型
- 类型大小不能确定
- 值大小可以确定
- 值不可被修改
- `&st`是对`str`的引用
- `&str`由长度和指针两部分组成
- 由于长度和指针的大小是确定的所以`&str`分配在栈

```rust
fn main() {
    let s: &str = "hello";
    println!("s的长度是{}", s.len())
}
```

- `str`是对`String`一部分或全部的引用切片
- `String`可以修改

```rust
fn main() {
    let s: String = String::from("hello");
    println!("s的长度是{} 切片{}", s, &s[0..2])
}
```

### 元组

```rust
fn main() {
    let tup: (i32, String) = (1, String::from("hello"));
    println!("元组的第一个元素{}", tup.0)
}
```

### 结构体

```rust
struct User {
    name: String,
    age: i32,
}

fn main() {
    let user = User {
        name: String::from("张三"),
        age: 18,
    };

    let user2 = User {
        age: 19,
        ..user
    };
    println!("{} {}", user2.name, user2.age)
}
```

### 数组

```rust
fn main() {
    let a = [1, 2, 3];
    let s = &a[0..2];
    println!("{}", s[1])
}
```

### 泛型

```rust
// 限定实现了 std::ops::Add 的类型
fn add_num<T: std::ops::Add<Output=T>>(a: T, b: T) -> T {
    a + b
}

fn main() {
    let x = add_num(1, 2);
    println!("{}", x)
}
```

### 枚举

#### 同类枚举

```rust
#[derive(Debug)]
enum Log {
    Debug,
    Info,
}

fn print_log(log: Log) {
    println!("日志类型是{:?}", log)
}

fn main() {
    print_log(Log::Debug);
    print_log(Log::Info)
}
```

#### 枚举整数

```rust
enum Log {
    Debug = 1,
    Info
}

fn main() {
    let x =  Log::Info as i32;
    println!("{}", x)
}
```

#### 处理空值

```rust
enum Option<T> {
    Some(T),
    None,
}

fn main() {
    let s = Option::Some(1);
    let n = Option::None;
}
```

### 集合数组

#### 动态数组

```rust
fn main() {
    let mut v: Vec<i32> = Vec::new();
    v.push(1);
    v.push(2);
    println!("{}", v[0]);

    let mut v2 = vec![1];
    v2.push(2);
    println!("{}", v2[1])
}
```

#### HashMap

```rust
use std::collections::HashMap;

fn main() {
    let mut my_map: HashMap<&str, i32> = HashMap::new();
    my_map.insert("key", 100);
    my_map.insert("key", 1000);
    let key: Option<&i32> = my_map.get("key");
    println!("{:?}", key)
}
```

## 所有权

- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

### 作用域

```rust
fn main() {
    //此处无效因为x尚未声明
    {
        let x = 1;
        // 使用x
    }
    // 此处无效因为x作用域已结束
}
```

### 转移所有权

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // 所有权已经转移给s2 s1不在有效

    println!("{}, world!", s1);  // 这里会编译报错
}
```

```rust
fn test_ownership(s: String) -> String {
    s
}

fn main() {
    let s1 = String::from("hello");
    let s2 = test_ownership(s1); // s1 所有权转移
    println!("{}", s2);
}
```

```rust
fn main() {
    let x = 5;
    let y = x;

    println!("{}{}", x, y); // 不会报错 因为x是基本类型有固定的大小会在栈上拷贝赋值
}

```

```rust
fn main() {
    let x: &str = "hello, world";
    let y = x;

    println!("{},{}", x, y); // 不会报错 因为y只是引用了字符串本身而不是引用的x
}
```

### 深拷贝

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("{} ,{}", s1, s2);
}
```

### 引用和借用

Rust 中把对变量指针的引用称之为`借用Borrowing`

```rust
fn main() {
    let x = 5;
    let y = &x; // 引用

    println!("{}", *y) // 解引用
}
```

#### 不可变引用

```rust
fn test_change(s: &String) -> &String {
    // 不能修改s 的值
    s
}

fn main() {
    let s = String::from("hello");
    test_change(&s);
    println!("{}", s);
}
```

#### 可变引用

```rust
fn test_change(s: &mut String){
    // 可以修改s的值
    s.push_str(",world");
}

fn main() {
    let mut s = String::from("hello");
    test_change(&mut s);
    println!("{}", s);
}
```

#### 借用原则

- 可变引用同时只能存在一个
- 可变引用和不可变引用同时只能存在一个

```rust
fn main() {
    let s = String::from("hello");
    let a = &s; // 没问题
    let b = &s; // 没问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &mut s;
    let b = &mut s;

    println!("{}{}", a, b) // 有问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &mut s;
    println!("{}", a) // 没问题
    let b = &mut s;

    println!("{}", b) // 没问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &s;
    let b = &mut s;

    println!("{}{}", a, b) // 有问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &s;
    println!("{}", a) // 没问题
    let b = &mut s;

    println!("{}", b) // 没问题
}
```

#### 原则

- 可变引用同时只能存在一个
- 可变引用和不可变引用同时只能存在一个

```rust
fn main() {
    let s = String::from("hello");
    let a = &s; // 没问题
    let b = &s; // 没问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &mut s;
    let b = &mut s;

    println!("{}{}", a, b) // 有问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &mut s;
    println!("{}", a) // 没问题
    let b = &mut s;

    println!("{}", b) // 没问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &s;
    let b = &mut s;

    println!("{}{}", a, b) // 有问题
}
```

```rust
fn main() {
    let mut s = String::from("hello");
    let a = &s;
    println!("{}", a) // 没问题
    let b = &mut s;

    println!("{}", b) // 没问题
}
```

### 生命周期

```rust
fn main() {
    let s1 = "abc";
    let s2 = "abcd";

    let result = longest(s1, s2);
    println!("最长的字符串 {}", result);
}


fn longest(x: &str, y: &str) -> &str {
    // 编译器不知道最终返回的是x还是y
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

```rust
fn main() {
    let s1 = "abc";
    let s2 = "abcd";

    let result = longest(s1, s2);
    println!("最长的字符串 {}", result);
}


fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

## 流程控制

### if

```rust
fn main() {
    let x = 5;
    if x > 10 {
        println!("大于10")
    } else if x == 10 {
        println!("等于10")
    } else {
        println!("小于10")
    }
}
```

### for 循环

```rust
fn main() {
    let a = [1, 2, 3, 4];
    for i in a {
        if i == 2 {
            continue;
        }
        println!("{}", i);
        if i == 3 {
            break;
        }
    }
}    
```

```rust
fn main() {
    for i in 1..=5 {
        println!("{}", i);
    }
}

```

### while 循环

```rust
fn main() {
    let mut n = 0;

    while n <= 5  {
        println!("{}!", n);

        n = n + 1;
    }

    println!("我出来了！");
}
```

### loop 循环

```rust
fn main() {
    let mut counter = 0;
    loop {
        counter += 1;
        if counter == 10 {
            println!("退出loop")
        }
    };
}
```

## 方法

### 结构体方法

```rust
struct Dog {
    name: String,
    age: i32,
}

impl Dog {
    fn get_age(&self) -> i32 {
        self.age
    }
}

fn main() {
    let dog = Dog { name: String::from("皮球"), age: 3 };
    println!("年龄是 {}", dog.get_age())
}
```

### 枚举方法

```rust
enum Log {
    Debug(String),
    Info(String),
}

impl Log {
    fn print(&self) {
        match self {
            Log::Debug(log) => {
                println!("debug: {}", log)
            }
            Log::Info(log) => {
                println!("info: {}", log)
            }
        }
    }
}

fn main() {
    let log = Log::Info(String::from("这是一个日志"));
    log.print()
}
```

## 匹配

### 解构枚举

#### 枚举匹配

```rust
enum Log {
    Debug,
    Info,
    Warn,
    Error,
}

fn main() {
    let log = Log::Debug;

    match log {
        Log::Debug => println!("debug"),
        Log::Info | Log::Warn => println!("info 或 warn"),
        _ => println!("error")
    }
}
```

#### 匹配赋值

```rust
enum Log {
    Debug,
    Info,
}

fn log_level(log: Log) -> i32 {
    match log {
        Log::Debug => 3,
        Log::Info => 2,
    }
}

fn main() {
    let level = log_level(Log::Debug);
    if level == 3 {
        println!("这是debug log")
    }
}
```

#### 嵌套解构

```rust
enum Errors {
    Fatal,
    Error,
}

enum Log {
    Debug,
    Info,
    Error(Errors),
}

fn log_level(log: Log) -> i32 {
    match log {
        Log::Debug => 3,
        Log::Info => 2,
        Log::Error(Errors::Error) => {
            1
        }
        Log::Error(Errors::Fatal) => {
            0
        }
    }
}

fn main() {
    let level = log_level(Log::Error(Errors::Fatal));
    if level == 0 {
        println!("[Fatal] 这是fatal log")
    }
}
```

### 解构 Option

```rust
fn log_level(log: &str) -> Option<i32> {
    match log {
        "debug" => Some(3),
        "info" => Some(2),
        _ => None
    }
}

fn main() {
    let level = log_level("info");
    match level {
        None => println!("不支持"),
        _ => println!("支持")
    }
}
```

### 解构结构体

```rust
struct User {
    name: String,
    age: i32,
}

fn main() {
    let user = User {
        name: String::from("张三"),
        age: 18,
    };
    match user {
        User { age: 18, .. } => println!("18岁的用户"),
        _ => ()
    }
}
```

## 特征

### 实现特征

```rust
pub trait Animal {
    fn get_name(&self) {}
}

struct Dog {
    pub name: String,
}

struct Cat {
    pub name: String,
}

impl Animal for Dog {
    fn get_name(&self) {
        println!("狗狗的名字叫 {}", self.name)
    }
}


impl Animal for Cat {
    fn get_name(&self) {
        println!("猫咪的名字叫 {}", self.name)
    }
}


fn main() {
    let dog = Dog { name: String::from("皮球") };
    dog.get_name();

    let cat = Cat { name: String::from("小花") };
    cat.get_name()
}
```

### 函数参数

```rust
pub trait Animal {
    fn get_name(&self) {}
}

struct Dog {
    pub name: String,
}

struct Cat {
    pub name: String,
}

impl Animal for Dog {
    fn get_name(&self) {
        println!("狗狗的名字叫 {}", self.name)
    }
}


impl Animal for Cat {
    fn get_name(&self) {
        println!("猫咪的名字叫 {}", self.name)
    }
}


pub fn get_animal_name(a: impl Animal) {
    a.get_name()
}

fn main() {
    let dog = Dog { name: String::from("皮球") };
    let cat = Cat { name: String::from("小花") };

    get_animal_name(dog);
    get_animal_name(cat)
}
```

### 特征约束

```rust
pub trait Animal {
    fn get_name(&self) {}
}

struct Dog {
    pub name: String,
}

struct Cat {
    pub name: String,
}

impl Animal for Dog {
    fn get_name(&self) {
        println!("狗狗的名字叫 {}", self.name)
    }
}


impl Animal for Cat {
    fn get_name(&self) {
        println!("猫咪的名字叫 {}", self.name)
    }
}


pub fn get_animal_name<T: Animal>(a: T, b: T) {
    a.get_name();
    b.get_name()
}

fn main() {
    let dog = Dog { name: String::from("皮球") };
    let dog2 = Dog { name: String::from("小花") };

    get_animal_name(dog, dog2);
}
```

## 智能指针

### Box

```rust
fn main() {
    // 使用Box创建的对象必然分配在堆上
    let x = Box::new(1);
    println!("{}", x)
}
```

### Dorp

- 手动释放内存

```rust
struct Foo {
    x: i32,
}
impl Drop for Foo {
    fn drop(&mut self) {
        println!("释放Foo");
    }
}
fn main() {
    let mut x = Foo { x: -7 };
    drop(x); // ok!
}
```

### Rc

- 允许一个对象可以有多个所有者
- 只能用于同一线程内部

```rust
use std::rc::Rc;
fn main() {
    let a = Rc::new(String::from("hello, world"));
    // 允许 b c 同时引用a
    let b = Rc::clone(&a);
    let c = Rc::clone(&a);
}
```

### Arc

- 允许一个对象可以有多个所有者
- 可以多线程使用

```rust
use std::sync::Arc;
use std::thread;
fn main() {
    let s = Arc::new(String::from("多线程漫游者"));
    for _ in 0..10 {
        let s = Arc::clone(&s);
        let handle = thread::spawn(move || {
            println!("{}", s)
        });
    }
}
```

### Cell

- 把编译期对借用原则的检查推迟到运行时
- 只能操作实现了`Coyp`特性的类型
- 因为是`Coyp`所以运行时不会panic

```rust
use std::cell::Cell;

fn main() {
    let c = Cell::new("asdf");
    let one = c.get();
    c.set("qwer");
    let two = c.get();
    println!("{},{}", one, two);
}
```

### RefCell

- 把编译期对借用原则的检查推迟到运行时
- 违背借用原则运行时会panic
- 使用引用操作数据

```rust
use std::cell::RefCell;

fn main() {
    let s = RefCell::new(String::from("hello, world"));
    let s1 = s.borrow();
    let s2 = s.borrow_mut();

    println!("{},{}", s1, s2);
    // 会发生运行时panic 因为没有遵守借用原则
}
```

```rust
use std::cell::RefCell;

fn main() {
    let s = RefCell::new(String::from("hello, world"));
    let mut s1 = s.borrow_mut();
    s1.push_str("123");

    println!("{}", s1);
}
```

```rust
// 配合 Rc 一起使用
use std::cell::RefCell;
use std::rc::Rc;
fn main() {
    let s = Rc::new(RefCell::new("我很善变，还拥有多个主人".to_string()));

    let s1 = s.clone();
    let s2 = s.clone();
    s2.borrow_mut().push_str(", on yeah!");

    println!("{:?}\n{:?}\n{:?}", s, s1, s2);
}
```

## 多线程


### 创建多线程

```rust
use std::thread;  
use std::thread::sleep;  
use std::time::Duration;  
use chrono::{Local};  
  
fn main() {  
    let x = thread::spawn(|| {  
        sleep(Duration::from_millis(1000));  
        let t = Local::now();  
        println!("我是子线程{}", t.format("%Y-%m-%d %H:%M:%S"));  
    });  
  
    let t = Local::now();  
    println!("我是主线程{}", t.format("%Y-%m-%d %H:%M:%S"));

	// 等待子线程结束
    x.join().unwrap()  
}
```

### 线程通信

```rust
use std::sync::mpsc;  
use std::thread;  
  
fn main() {  
    let (tx, rx) = mpsc::channel();  

	// move 可以转移某个值得所有权到线程内
    thread::spawn(move|| {  
        let s = "我是线程1".to_string();  
        tx.send(s).unwrap();  // 注意s 所有权会转移出去
    });  
  
    let x = thread::spawn(move|| {  
        let r = rx.recv().unwrap();  
        println!("线程2接收到消息: {}", r)  
    });  
    x.join().unwrap();  
}
```

### 互斥锁

```rust
use std::sync::{Arc, Mutex};  
use std::thread;  
use std::thread::sleep;  
use std::time::Duration;  
  
fn main() {  
  
    let mu = Arc::new(Mutex::new(0));  
    {  
        let mu = Arc::clone(&mu);  
        thread::spawn(move || {  
            // 获得锁  
            let l = mu.lock();  
            sleep(Duration::from_secs(1));  
            // 释放锁  
            drop(l);  
            sleep(Duration::from_secs(100));  
        });  
    }  
  
    {  
        let mu = Arc::clone(&mu);  
        let x = thread::spawn(move || {  
            let l = mu.lock();  
            println!("线程2拿到锁");  
            drop(l);  
        });  
        x.join().unwrap();  
    }  
}
```

## 全局变量

### 常量

```rust
const MAX_NUMBER: i32 = 123;  
  
fn main() {  
    println!("{}", MAX_NUMBER)  
}
```

### 静态变量

```rust
static mut COUNT: i32 = 0;  
  
fn main() {  
    for _ in 1..10 {  
        unsafe {  
            COUNT += 1;  
        }  
    }  
  
    unsafe { println!("{}", COUNT) }  
}
```


## 返回和错误


### 返回值

```rust
use std::error::Error;  
use std::fs::read_to_string;  
use std::io;  
  
fn get_file() -> Result<String, io::Error> {  
    let f = read_to_string("test.txt")?; // 若有错误直接返回  
    Ok(f) // 正常返回值  
}  
  
fn main() {  
    let x = get_file();  
    match x {  
        Ok(f) => println!("{}", f),  
        Err(e) => println!("{}", e)  
    }  
}
```

### 自定义错误

```rust
use std::fmt;  
  
#[derive(Debug)]  
struct AppError;  
  
impl fmt::Display for AppError {  
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {  
        write!(f, "自定义错误")  
    }  
}  
  
  
fn produce_error() ->Result<(), AppError> {  
    Err(AppError)  
}  
  
fn main() {  
    match produce_error() {  
        Err(e) => eprintln!("{}", e),  
        _ => println!("没错误")  
    }  
  
    println!("{:?}", produce_error())  
}
```

## 异步

### async

```toml
[dependencies]  
futures = "0.3"
```

```rust
use futures::executor::block_on;  
  
async fn hello_world() {  
    println!("hello, world!");  
}  
  
async fn hello_kit() {  
    println!("hello, kit!");  
}  
  
async fn async_main() {  
    hello_world().await; // 不阻塞线程 异步等待  
    hello_kit().await;  
    println!("我是主异步")  
}  
  
fn main() {  
    let future = async_main();  
    block_on(future) // 阻塞线程  
}
```