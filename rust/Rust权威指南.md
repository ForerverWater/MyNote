### 第1章 

Linux安装
```
curl https://sh.rustup.rs -sSf | sh
```

设置环境变量
```
export PATH="$HOME/.cargo/bin:$PATH"
```

第一个hello world程序
```rust
// hello_world.rs
fn main() {
    println!("hello world,你好");
}
```

以!结尾都是宏而不是函数

Cargo是Rust工具链中内置的构建系统及包管理器

```
// 查看是否安装cargo
cargo --version

// cargo创建一个项目
cargo new hello_cargo
```

**Cargo.toml**
[package]是一个区域标签，表明接下来的语句会被用于配置当前的程序包
[dependencies]也是区域标签，表明随后的区域会被用来声明项目的依赖

Rust中把代码的集合称作包(crate)
crate是Rust中最小的编译单元，package是单个或多个crate的集合，crate和package都可以叫作包，因为单个crate也是一个package，但package通常倾向于多个crate的组合。

```
// 构建命令
cargo build
```

运行构建任务后会生成一个名为Cargo.lock的新文件，该文件记录了当前项目所有依赖库的具体版本号。

```
// 运行
cargo run
```

```
// release模式
cargo build --release
```

上面命令进入release模式，会优化并生成可执行程序，放在target/release。一般build的模式下，是放在target/debug目录下


### 第2章

```rust
// 猜数字游戏
use std::io;

fn main() {
    println!("please input your guess.!");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("Failed to read line");

    println!("your guessed : {} ", guess);
}

```

```rust
let foo = 5; // foo是不可变
let mut foo1 = 5; // foo是可变
```

String::new中的::语法表明new是String类型的一个*关联函数*(associated function)。相对于其他语言中的*静态方法*

read_line(&mut guess) 中的&意味着当前的参数是一个*引用*
引用跟变量一样，默认情况也是不可变得，因此需要&mut来声明一个可变引用

read_line会将用户输入的内容存储到传入的字符串中，还会返回一个io::Result值。在Rust标准库中，可以找到许多以Result命令的类型，它们通常是各个子模块中Result泛型的特定版本，比如io::Result

Result是一个*枚举*类型。枚举类型由一系列固定的值组合而成，这些值被称作枚举的*变体*。

Result拥有Ok和Err两个变体。Ok表明当前的操作执行成功，并附带代码产生的结果值；Err变体表明当前操作执行失败，并附带引发失败的具体原因

```println!("your guessed : {} ", guess);``` 中的{}是一个占位符

遇到```cargo build``` 出现 Blocking waiting for file lock on package cache会一直卡住，需要去把用户下面.cargo文件夹下面的.package-cache文件删除掉，接着源改成国内的源

更换国内源，在.cargo文件夹下面加一个config文件，写入如下配置
```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'

# 中国科学技术大学
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
```

```rust
// 猜数字并比较
use std::io;
use rand::Rng;
use std::cmp::Ordering;
fn main() {
    loop {
        println!("please input your guess.!");

        let secret_nmumber = rand::thread_rng().gen_range(1, 101);

        let mut guess = String::new();

        io::stdin().read_line(&mut guess).expect("Failed to read line!");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("secret_number : {} ", secret_nmumber);

        println!("your guessed : {} ", guess);

        match guess.cmp(&secret_nmumber) {
            Ordering::Greater => println!("Too big!"),
            Ordering::Less => println!("Too small!"),
            Ordering::Equal => { 
                println!("Your win!");
                break;
            }
        }
    }
    
}
```

上面代码有几个注意点
1. 引入Ordering枚举类型，它有Less, Greater, Equal三个变体
2. 使用同名的变量guess当新旧变量，，通常被用在转换值的场景
3. match表达式
4. loop循环
5. break退出语句
6. parse会返回一个Result类型，通过match表达式处理成功和失败，Ok就会返回num值绑定到guess变量；Err(_)的下划线是一个通配符，代表匹配所有可能的Err值
7. continue使程序转至下次循环


### 第3章

```rust
// const关键字定义常量
const MAX_POINTS: u32 = 100;
```

4种基础标量类型
1. 整数
2. 浮点数
3. 布尔值
4. 字符

**Rust中整数类型**
| 长度 | 有符号 | 无符号 |
| --- | --- | --- |
| 8-bit | i8 | u8 |
| 16-bit | i16 | u16 |
| 32-bit | i32 | u32 |
| 64-bit | i64 | u64 |
| arch | isize | usize |

有符号类型，数值需要一个符号来表示当前是否为正；无符号类型，数值永远为正

整数范围参考P50

isize和usize长度取决程序运行的平台。64位架构的就是64，32位架构就是32

**浮点数类型**
两种基础浮点数类型: f32和f64

由于两种运行效率相差无几，f64拥有更高的精度。所有Rust默认会将浮点数字面量推导为f64

```rust
let x = 2.0; // 隐式推导f64
let y: f32 = 1.0; //显示声明f32
```

**布尔类型**
```rust
let t = true; 
let f: bool = false; // 显示类型标注
```

**字符类型**

char类型使用**单引号**指定，Rust中char类型占4个字节，是一个Unicode标量值，意味着它可以表示很多字符内容。中文、日文，甚至emoji表情都可以作为有效的char类型值


**复合类型**

Rust内置了两种复合类型：*元组(tuple)*和*数组(array)*

元组可以将其他**不同类型**的多个值合租
元组还拥有一个固定长度的特性，无法在声明结束后增加或减少其中的元素数量

```rust
let tup: (i32, f64, u8) = (500, 6.4 1); // 定义元组

let (x, y, z) = tup; // 解构元组

println!("The value of y is : {}", y);

// 通过索引访问元组中的值
let a =  tup.0;
let b =  tup.1;
let c =  tup.2;
```

数组与元组不同，数组中每一个元素都必须相同的类型
Rust中的数组拥有固定的长度，一旦声明就再不能随意改变大小，这与其他语言有所不同

```rust
// 数组里面元素类型为i32且数组长度为5
let a: [i32; 5] = [1, 2, 3, 4, 5];

let b = [3; 5];
// 等价于
let b = [3, 3, 3, 3, 3];

// 获取数组元素
let first = a[0];
```

**函数**

```rust
fn add(x: i32, y:i32) -> i32 {
    x + y
}
```

1. -> 后面的i32指定返回值的类型
2. 大多数函数都隐式地返回最后一个表达式，最后一个表达式不能加;号
3. 如果没有返回值，Rust默认返回一个空元组


```rust
// 这段程序会编译错误，因为Rust不会自动将非布尔类型转换成布尔类型
let number = 3;

if number {
    ...
}
```

**循环语句**
```rust
// loop循环中的返回值
let mut counter = 0;

let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};
```


```rust
// while循环
let mut counter = 3;

while counter != 0 {
    counter = counter - 1;
    ....
};
```

```rust
// for循环
let a = [1, 2, 3, 4, 5];

for element in a.iter() {
    println!("value is {}", element);
}
```

### 第4章

栈会以放入值得顺序来存储，并以相反的顺序将值取出。也就所谓的“后进先出”策略。存储在栈中的数据必须有一个已知且固定的大小。
编译器无法确定大小的数据，只能存储在堆中。堆空间分配一块足够大的空间，标记为已使用，并把这个空间地址的指针返回。
由于多了指针跳转这个环节，所以访问堆上的数据要慢于访问栈上的数据

**所有权规则**
* Rust中的每一个值都有一个对应的变量作为它的所有者
* 在同一时间内，值有且仅有一个所有者
* 当所有者离开自己的作用域时，它持有的值就会被释放

```rust
{
    let s = String::from("hhh"); // 变量s变得有效
}   // 作用域到这里结束，变量s失效
```

Rust在变量离开作用域时，会调用一个drop的特殊函数


```rust
let s1 = String::from("hhh");
let s2 = s1;

println!("{}, qqq", s1); // 这里报错，因为变量s1已经废弃
```

上面语句的内存布局，s2只复制了s1存储在栈中的指针、长度及容量，不会复制在堆上的数据
如果s1跟s2都离开自己的作用域时，会尝试去释放相同的内存，从而导致二次释放。为了确保内存安全，Rust在这种场景下会简单地将s1废弃，不再视其为一个有效变量。在s2离开再进行释放。一般这种在Rust是移动操作，可以说s1被移动到s2中。

Rust永远不会自动创建数据的深度拷贝

```rust
// 深度拷贝示例
let s1 = String::from("hhh");
let s2 = s1.clone();

println!("s1 : {}, s2 : {}", s1, s2); // 正常运行
```

Rust提供了一个名为Copy的trait。一旦某种类型拥有了Copy这种trait，那么它的变量就可以在赋值给其他变量之后保持可用性。如果某种类型实现了Drop这种trait，就不允许实现Copy

下面是一些拥有Copy trait的类型
* 所有的整数类型，如u32
* 布尔类型
* 字符类型
* 所有的浮点类型
* 包含的字段类型都是Copy的元组，如(i32, i32)，但(i32, String)不是


```rust
let s = String::from("hello"); // 变量s进入作用域
takes_ownership(s); //s的值被移动进函数
println!("s : {}", s); // 这里变量s已经被废弃，无效

let x = 6; // 变量x进入作用域
takes_ownership(x); //x的值被传递进函数
println!("x : {}", x); // 由于x是i32是有Copy trait，所以后面依然可以用x
```

```rust
let s1 = String::from("hello");
let len = calculate_length(&s1);
println!("The length if {} is {}", s1, len);

fn calculate_length(s: &String) -> usize { // s是一个指向String的引用
    s.len()
}// 到这里，s离开作用域。但是有它并不持有自己所指向值的所有权，所以这里没有特殊的事情发生
```

&代表引用的语义，它允许你在不获取所有权的前提下使用值
引用默认是*不可变*的，Rust不允许我们去修改引用指向的值

**可变引用**
```rust
let mut s = String::from("hello");
change(&mut s)
fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

```rust
// 可变引用一次只能声明一个可变引用，以下代码违背这一限制
let mut s = String::from("hello");
let r1 = &mut s;
let r2 = &mut s;
```

```rust
// 通过花括号来创建新的作用域范围，使我们可以创建多个可变引用
let mut s = String::from("hello");
{
    let r1 = &mut s;
} // 由于r1离开作用域就释放，所以后面可以合法地再创建一个可变引用
let r2 = &mut s;
```

```rust
// 不能在 拥有不可变引用的同时创建可变引用
let mut s = String::from("hello");
let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 错误
```

```rust
// 悬垂引用错误示例
let refer_to_nothing = dangle();

fn dangle() -> &String {
    let s = String::from("hello");
    &s // 将指向s的引用返回给调用者
}// 变量s在离开作用域并随之被销毁，它指向的内存自然不再有效

// 正确
fn dangle() -> String {
    let s = String::from("hello");
    s // 所有权被转移出函数，不会涉及释放
}
```

**引用规则**
* 在任何一段给定的时间里，要么只能拥有一个可变引用，要么只能拥有任意数量的不可变引用
* 引用总是有效的


##### 切片(slice)
切片：另一种不持有所有权的数据类型，切片允许我们引用集合中某一段连续的元素序列，而不是整个集合

```rust
let mut s = String::from("hello world");

let word = first_word(&s);

println!("word is {}", word);

s.clear();

fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

1. 使用```as_bytes```方法将String转换为字节数组
2. ```iter```方法创建一个可以遍历字节数组的迭代器
3. ```enumerate```将```iter```每个输出作为元素逐一封装在对应的元组中返回
4. i是元组的索引部分，```&item```则是元组中指向集合元素的引用
5. ```item == b' '```匹配数组中空格的字节，匹配成功返回索引位置
6. ```s.len()``` for循环完了还没匹配，直接返回字符串的长度


```rust
// 切片示例
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];

println!("first is {}", hello);
println!("second is {}", world);

//语法糖
let slice = &s[0..2]; // 相当于 &s[..2]; 
let slice = &s[3..11]; // 相当于 &s[3..];
let slice = &s[0..11]; // 相当于 &s[..];

```

```rust
// first_world改写，使用字符串切片作为参数
fn first_word(s: &str) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

let my_string = String::from("hello world");
let word = first_word(&my_string[..]); // 接收String对象切片

let my_string_leteral = "hello world";
let word2 = first_word(&my_string_leteral[..]); // 接收字符串字面量切片

let word3 = first_word(my_string_leteral); // 由于字符串字面量本身就是切片，直接传入函数也可以
```

### 第5章 使用结构体来组织相关联的数据

结构，或者结构体，是一种自定义数据类型，允许我们命名多个相关的值并将它们组成一个有机的结合体

```rust
// 定义一个User结构体
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// 创建一个User结构体的实例
let mut user = User {
    email: String::from("xxxx@xxx.com"),
    username: String::from("xxx123"),
    sign_in_count: 1,
    active: true,
};

// 通过.号赋值并使用
user.email = String::from("yyyy@yyyy.com");

println!("user.email is {}", user.email);


// 接收两个参数的创建User函数
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}

// 变量名跟字段名相同时，可以简化成如下写法
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

// 使用user中某些值创建一个新的User实例
let mut user1 = User {
    email: String::from("2222@222.com"),
    username: String::from("222123"),
    sign_in_count: user.sign_in_count,
    active: user.active,
};

// 下面简写，除了修改的字段，其他都用user的值
let mut user1 = User {
    email: String::from("2222@222.com"),
    username: String::from("222123"),
    ..user
};

```

**元组结构体**

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 2, 4);
let origin = Point(6, 3, 4);
```

```rust
// 使用元组计算长方形面积
let rect1 = (50, 30);

println!("The area is {}", area(rect1));

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

let rect2 = Rectangle {width: 40, height: 50};

println!("rect2 is {:#?}", rect2);

println!("The area is {}", area2(&rect2));

fn area2(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

1. 通过定义结构体来计算长方形
2. 在结构体上加上```#[derive(Debug)]```跟用```println!```宏跟```{:#?}```打印结构体的内容


```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

// 在结构体中定义area方法
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }

    fn square(size: u32) -> Rectangle {
        Rectangle {width: size, height: size}
    }
}

let rect2 = Rectangle {width: 40, height: 50};

println!("rect2 is {:#?}", rect2);

println!("The area is {}", rect2.area());

let sq = Rectangle::square(3);


println!("The square is {:?}", sq);
```

1. square这种不接收```self```参数的函数，称为关联函数，因为它不会作用于某个具体的结构体实例
2. 关联函数常常被用作构造器来返回一个结构体的新实例
3. 在类型名称后面用```::```来调用


### 第6章 枚举与模式匹配

```rust
// 通过枚举定义IpAddr
#[derive(Debug)]
enum IpAddrKind {
    V4,
    V6,
}

let ip4 = IpAddrKind::V4;

#[derive(Debug)]
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),

};

println!("Home IpAddr is {:?}", home);
```

```rust
enum Message {
    Quit,
    Move {x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

1. 上面的Message枚举内嵌了4个不同类型数据变体
2. Quit没有任何关联数据
3. Move包含一个匿名结构体
4. Write包含一个String
5. ChangeColor包含了3个i32的值

**Option<T>**

Option<T>枚举也只是一个普通的枚举类型，Some<T>和None是它的变体

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

```rust
// Option使用match表达式函数
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

**_通配符**

```rust
// 由于match必须穷举所有情况，如果我们只关注其中几种情况，剩下的可以用_通配符表示，这里通配符返回空元组，表示什么都不会发生
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    2 => println!("two"),
    10 => println!("ten"),
    55 => println!("fifty-five"),
    _ => (),
}
```

**if let**

```rust
// if let可以视作match的语法糖，只在值满足某一特定模式时运行代码，而忽略其他所有的可能性
let some_u8_value = Some(1u8);
// match some_u8_value {
//     Some(1) => println!("one"),
//     _ => (),
// }

if let Some(3) = some_u8_value {
    println!("three");
}

// 也可以用else处理其他情况
if let Some(3) = some_u8_value {
    println!("three");
} else {
    println!("other");
}
```

### 第7章 使用包、单元包及模块来管理日渐复杂的项目

##### 包与单元包

包的规则
* 一个包中只能有最多一个库单元包
* 包可以拥有任意多个二进制单元包
* 包内必须存在至少一个单元包(库单元包或二进制单元包)

Cargo会默认将src/main.rs视作一个二进制单元包的根节点

src/lib.rs视作与包同名的库单元包的根节点

```
// 创建带lib.rs的项目
cargo new --lib restaurant
```

```rust
// 在lib.rs输入，创建模块
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();
    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

路径的两种形式
* 使用单元包或字面量crate从根节点开始的绝对路径
* 使用self、super或内部标识符从当前模块开始的相对路径
* 绝对路径与相对路径都由至少一个标识符组成，标识符之间使用双冒号分隔


```rust
// super示例
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}


        mod back_of_house {
            fn fix_incorrect_order() {
                cook_order();
                super::serve_order();
            }

            fn cook_order() {}
        }
    }
}
```
1. ```fix_incorrect_order```跟```cook_order```同级，所以可以直接调用
2. ```super```就是```fix_incorrect_order```的父级，也就是```back_of_house```，```back_of_house```又跟```serve_order```同级，所以也能调用成功


```rust
mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}

    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }

    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let mut meal = back_of_house::Breakfast::summer("Rye");
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please!", meal.toast);

    let order1 = back_of_house::Appetizer::Soup;
}
```

1. 结构体，只有加了pub的field才能是公共
2. 枚举声明为pub，变体自动都是公共的


```rust
// 绝对路径，以crate开头，全路径引入
use crate::front_of_house::hosting;

// 相对路径，以self开头，如果调用方跟被调用同一级可以用
use self::back_of_house::Breakfast;

// 引入标准库HashMap
use std::collections::HashMap;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();

    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

```rust
// 使用as关键字解决作用域同名类型问题
use std::fmt::Result;

use std::io::Result as IoResult
```

```rust
// 嵌套路径减少use语句
use std::cmp::Ordering;
use std::io;

use std::{cmp::Ordering, io};

use std::io;
use std::io::Write;

use std::io::{self, Write}


// 通配符，指定路径下所有公共条目都引入
use std::collections::*;

```

模块拆分为不同的文件，具体参考书本P172

### 第8章 通用集合类型

3个被广泛使用在Rust程序中的集合

* 动态数组(vector)可以让你连续存储任意多个值
* 字符串(string)是字符的集合
* 哈希映射(hash map)

```rust
// 创建动态数组
let v: Vec<i32> = Vec::new();

// 利用vec!宏创建动态数组
let v2 = vec![1, 2, 3];

// 添加元素
v2.push(5);
v2.push(6);

// 通过索引或get方法访问数组
let third: i32 = &v[2];
let four: i32 = v.get(3);
```

1. 动态数组索引值从零开始，数组元素类型相同，并连续存储的
2. 索引访问会直接返回元素引用，get会返回Option<T>。所以如果索引访问不存在的元素会导致程序触发panic

```rust
// for循环遍历并打印
let v = vec![1, 2, 3];
for i in &v {
    println!("{}", i);
}

for i in &mut v {
    *i += 50; // 解引用获取i绑定的值
}

```


```rust
// 动态数组使用枚举存储不同类型的值
enum SpreadSheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadSheetCell::Int(3),
    SpreadSheetCell::Float(1.00),
    SpreadSheetCell::Text(String::from("test")),
];

```

```rust
// 创建一个新的字符串
let mut s = String::new();

// 使用to_string方法基于字符串字面量创建String
let s = "hello world".to_string();

// from函数
let s = String::from("hello world");

// push_str接收字符串切片作为参数
s.push_str("!!!");

// push接收单个字符作为参数
s.push('1');


let s1 = String::from("hello");
let s2 = String::from(" world");
let s3 = s1 + &s2; // 注意，这里s1已经被移动且再也不能被使用

// 上面+号是拼接，但其实会调用一个add方法
// fn add(self, s: &str) -> String {}
```

总结一下add方法，s2用了```&```符号，实际上是将第二个字符串的引用与第一个字符串相加，但add参数表明，只能是String跟&str相加，```&s2```的类型是```&String```，为什么能匹配add方法的```&str```呢？因为编译器可以自动将```&String```类型的参数强制转换为```&str```，被称作解引用强制转换，将```&s2```转换成```&s2[..]```，所以add函数并不会取参数```s: &str```的所有权。 P187

```rust
let s1 = String::from("hello");
let s2 = String::from(" world");
let s3 = String::from(" !!!");

// 拼接多个，有点笨拙
let s = s1 + "-" + &s2 + "-" + &s3;

// 复杂字符串拼接，使用format!宏
let s = format!("{}-{}-{}", s1, s2, s3);

```

注意：String不能通过索引获取字符，需要通过字符串切片获取，或者for循环

```rust
// 打印每个字符
for c in "hello".chars() {
    println!("{}", c);
}

// 打印出字节值
for b in "hello".bytes() {
    println!("{}", b);
}
```


```rust
// 创建HashMap并插入数据
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 20);

// 访问存储的数据
let blue_team = String::from("Blue");
// 这里get会返回Some(&10)，因为get返回的是一个Option<T>
let blue_score = scores.get(&blue_team);

// for循环也能获取数据
for (key, value) in &scores {
    
}


// 使用动态数组创建HashMap
let teams = vec![String::from("Blue"), String::from("Yellow")];
let scores2 = vec![10, 20];

let scores_map: HashMap<_, _> = teams.iter().zip(scores2.iter()).collect();
```

对于实现了Copy trait的类型，它们的值会被简单地复制到HashMap。而String类型的所有权会转移给HashMap，如果只是将引入插入HashMap，这些引用所指向的值必须保证，在HashMap有效时自己也是有效的

```rust
// 使用entry函数来判断是否有值
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

### 第9章 错误处理

Rust中，错误分两大类：
* 可恢复错误
* 不可恢复错误

Rust没有异常机制，但提供了可恢复错误的类型Result<T, E>，以及在程序出现不可恢复错误时中止运行的panic!宏

```rust
// 直接调用panic!宏
panic!("panic test");

// 缓冲区溢出触发panic
let v = vec![1, 2, 3];
v[99];

```

```rust
// 通过将环境变量RUST_BACKTRACE设置为非0，获得回溯信息
RUST_BACKTRACE=1 cargo run
```


```rust
// 打开文件失败返回panic的输出
use std::fs::File;

let f: Result<_, _> = File::open("hello.txt");

let f = match f {
    Ok(file) => file,
    Err(error) => {
        panic!("There was a problem opening the file {:?}", error);
    },
};
```

```rust
// 用不同的方式处理不同的错误
use std::fs::File;
use std::io::ErrorKind;

let f: Result<_, _> = File::open("hello.txt");

let f = match f {
    Ok(file) => file,
    Err(error) => match error.kind() {
       ErrorKind::NotFound => match File::create("hello.txt") {
           Ok(fc) => fc,
           Err(e) => panic!("try to create file but there a problem opening the file {:?}", e),
       },
       other_error => panic!("There was a problem opening the file {:?}", other_error),
    },
};
```


```rust
// 触发panic!调用的快捷方式
use std::fs::File;
let f = File::open("hello.txt").unwrap();

let f = File::open("hello.txt").expect("Failed to open hello.txt");
```

```rust
// 从文件读取内容，复杂写法
// 主要体现传播错误，错误返回给调用者
fn read_string_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}

let s = read_string_from_file();
println!("read string is {:?}", s);

//打印结果: read string is Ok("88888")
```

```rust
// 一个使用?运算符将错误返回给调用者的函数
use std::io;
use std::io::Read;
use std::fs::File;

fn read_string_from_file() -> Result<String, io::Error> {
    // ?运算符，传播错误的快捷方式
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

let s = read_string_from_file();
println!("read string is {:?}", s);
```

match表达式与？运算符的一个区别：被？运算符所接收的错误值会隐式地被from函数处理，这个函数定义于标准库的From trait 中，用于在错误类型之间进行转换。当？运算符调用from函数时，它就开始尝试将传入的错误类型转换为当前函数的返回错误类型。当一个函数拥有不同的失败原因，却使用了统一的错误返回类型来同时进行表达时，这个功能会十分有用。只要每个错误类型都实现了转换为返回错误类型的from函数，？运算符就会自动帮我们处理所有的转换过程。

```rust
// ?运算符链式调用
fn read_string_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

```rust
// 使用fs::read_to_string方法读取文件
fn read_string_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

```rust
// 通过结构体自定义类型
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new (value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100");
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}

let g = Guess::new(100);
```

### 第10章 泛型、trait与生命周期

```rust
// 求列表最大值函数
fn get_largest_number(list: &[i32]) -> i32 {
    let mut largest = list[0];
    for &n in list.iter() {
        if n > largest {
            largest = n;
        }
    }
    largest
}

let number_list = vec![5,345,76,233,11];
let result = get_largest_number(&number_list);

println!("the largest number is {}", result);
```


```rust
// 用泛型改写，但这代码无法通过编译
fn get_largest<T>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

```rust
// 结构体应用泛型
// x和y为相同类型
struct Point<T> {
    x: T,
    y: T,
}

let integer = Point{x: 5, y: 5};
let float = Point{x: 5.1, y: 5.4};

// x和y为不同类型
struct Point<T, U> {
    x: T,
    y: U,
}
```

```rust
// 方法应用泛型
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

```rust
// 泛型方法跟指定类型方法同时定义
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

let integer = Point{x: 5, y: 5};
let float = Point{x: 5.0, y: 5.0};

let sqrt = float.distance_from_origin();

println!("sqrt is {}", sqrt);
```


##### trait：定义共享行为

* trait与接口的功能类似，但也不尽相同
* 每个实现了某个trait都要实现trait定义的方法
* 一个trait可以定义多个方法，每个方法签名占据单独一行并以分号结尾

```rust
// 为NewArticle跟Tweet类型实现Summary trait
pub struct NewArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

pub struct Tweet {
    pub username: String,
    pub reply: bool,
    pub content: String,
}
pub trait Summary {
    fn summarize(&self) -> String;
}

impl Summary for NewArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{} -> {}", self.username, self.content)
    }
}

let tweet = Tweet {
    username: String::from("test"),
    content: String::from("test123 test123"),
    reply: false,
};

println!("1 new tweet: {}", tweet.summarize());
```


```rust
// trait提供默认返回，并定义另一个需实现的方法
pub trait Summary {
    fn summarize(&self) -> String {
        format!("Read more from {} ...", self.summarize_author())
    }

    fn summarize_author(&self) -> String;
}

impl Summary for NewArticle {
    
    fn summarize_author(&self) -> String { 
        format!("@{}", self.author)
    }
}
```

```rust
// trait作为参数示例
// 这里强制两个参数都是相同类型
pub fn notify<T: Summary>(item1: T, item2: T) {}


// 跟上面示例一样，这里算是一种语法糖
pub fn notify(item1: impl Summary, item2: impl Summary) {}
```

```rust
// 通过+语法指定多个trait约束
fn some_function<T: Display + Clone, U: Clone + Debug> (t: T, u: U) -> i32{
    1
}

// 使用where从句来简化trait约束
fn some_function<T, U> (t: T, u: U) -> i32 
    where T: Display + Clone,
        U: Clone + Debug
{
    1
}
```

```rust
// 返回实现了trait的类型
fn return_summarizable() -> impl Summary {
    Tweet {
        username: String::from("test"),
        content: String::from("test123 test123"),
        reply: false,
    }
}
```


```rust
// 使用trait约束来修复之前的largest函数
fn get_largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

let number_list = vec![5, 345, 76, 233, 11];
let result = get_largest(&number_list);
println!("the largest number is {}", result);

let char_list = vec!['y', 'm', 'e', 'q'];
let char_result = get_largest(&char_list);
println!("the largest char is {}", char_result);
```

1. 使用大于运算符比较两个T类型的值，需要实现std::cmp::PartialOrd
2. 由于list中可能存在没有实现Copy这个trait的类型，所以加上Copy这个trait。i32跟char都已经实现了Copy trait


```rust
// 标准库满足Display trait约束的类型都实现了ToString trait
impl<T: Display> ToString for T {}

// 任何实现了Display trait类型调用ToString中的to_string方法都可以
// 整数实现了Display
let s = 3.to_string();
```

##### 使用生命周期来避免悬垂引用

P256

Rust编译器拥有一个*借用检查器*，用于比较不同的作用域并确定所有借用的合法性。

生命周期的标注：生命周期的参数名称必须以撇号(')开头，且通常使用全小写字符

```rust
&i32         // 引用
&'a i32      // 拥有显式生命周期的引用
&'a mut i32  // 拥有显式生命周期的可变引用
```

单个生命周期的标注本身并没有太多意义，标注之所以存在是为了向Rust描述多个泛型生命周期参数之间的关系。例如，假设我们编写了一个函数，这个函数的参数first是一个指向i32的引用，并且拥有生命周期'a。它的另一个参数second同样也是指向i32且拥有生命周期'a的引用。这样的标注就意味着：first和second的引用必须与这里的泛型生命周期存活一样长的时间。

```rust
fn longest<'a> (x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

从根本上说，生命周期语法就是用来关联一个函数中不同参数及返回值的生命周期的。一旦它们形成了某种联系，Rust就获得了足够的信息来支持保障内存安全的操作，并阻止那些可能会导致悬垂指针或其他违反内存安全的行为。


```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

let noval = String::from("Call me Jack. Some years ago...");

let first_sentence = noval.split('.')
                    .next()
                    .expect("Could not find a '.'");
let i = ImportantExcerpt {part: first_sentence};
```

1. 结构体存储引用必须为结构体定义的引用添加生命周期标注
2. 上面ImportantExcerpt的生命周期标注意味着ImportantExcerpt实例的存活时间不能超过存储在part字段引用的存活时间
3. part的引用是从noval过来的，所以ImportantExcerpt根据noval的作用域生效

在没有显式标注的情况下，编译器使用的3种规则来计算引用生命周期。这些规则不但对fn定义有效，也对impl代码块有效

1. 每一个引用参数都会拥有自己的生命周期参数。单参数拥有一个，双参数就拥有两个，以此类推
2. 当只存在一个输入生命周期参数时，这个生命周期会被赋予给所有输出生命周期参数
3. 当拥有多个输入生命周期参数，而其中一个是```&self```或```&mut self```时，self的生命周期会被赋予给所有的输出生命周期参数。

需要多看几次后面生命周期省略规则的代码示例


### 第11章 编写自动化测试

```rust
pub fn aad_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        println!("2 + 2");
        assert_eq!(result, 4);
    }

    #[test]
    #[should_panic]
    fn another() {
        panic!("Make this test fail!");
    }
    
    #[test]
    #[should_panic(expected="message")]
    fn another2() {
        panic!("Make this test fail!");
    }
    
    // 使用Result<T, E>编写测试
    #[test]
    fn result_test() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four!"))
        }
    }
}

```

```shell
# 单线程运行测试
cargo test -- --test-threads=1

// 默认test有println宏也不会打印，需要加上下面参数才行
cargo test -- --nocapture

// 运行单个测试或者多个匹配这个名字的测试方法
cargo test one_hundred

// 需要在#[test]后面加上#[ignore]方法才会被忽略测试
cargo test -- --ignored
```

在test模块上标注```#[cfg(test)]```可以让Rust只在执行cargo test名时编译和运行该部分测试代码，在cargo build时剔除它们

cargo对tests目录进行了特殊处理，不需要标注```#[cfg(test)]，它只会在执行cargo test命令时编译这个目录下的文件

```shell
// 指定测试文件
cargo test --test integration_test  
```

在test目录下面定义公共模块
```
// 错误
tests/common.rs
// 正确
tests/common/mod.rs
```

如果我们的项目是一个只有src/main.rs文件而没有src/ib.rs文件的二进制包，那么我们就无法在 tests目录中创建集成测试，也无法使用use语句将src/main.rs中定义的函数导入作用域。只有代码包（library crate）才可以将函数暴露给其他包来调用，而二进制包只被用于独立执行。
这就是Rust的二进制项目经常会把逻辑编写在src/lib.rs 文件中，而只在 src/main.rs 文件中进行简单调用的原因。这种组织结构使得集成测试可以将我们的项目视作一个代码包，并能够使用use访问包中的核心功能。只要我们能够保证核心功能一切正常，src/main.rs中的少量胶水代码就能够工作，无须进行测试


### 第12章 I/O项目：编写一个命令行程序

二进制程序进行关注点分离的指定到性原则：
* 将程序拆分为```main.rs```和```lib.rs```，并将实际的业务逻辑放入```lib.rs```
* 当命令行逻辑相对简单，留在main.rs中也无妨
* 当命令行解析逻辑开始变得复杂时，同样需要将它从main.rs提取至lib.rs中

```rust
// 通过结构体跟Result组织代码
use std::env;
use std::fs;
use std::process;
fn main() { 

    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("error ->: {}", err);
        process::exit(1);
    });

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.filename).expect("something wrong when read file");
    println!("contents is : {}", contents);
}

struct Config {
    query: String,
    filename: String
}

impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enouth args");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config{query, filename})
    }
}
```

使用```unwrap_or_else```可以让我们执行一些自定义的且不会产生panic!的错误处理策略。当Result的值时Ok时，这个方法的行为与unwrap相同：它会返回Ok中的值。但是，当值Err时，这个方法则会调用闭包中编写的代码，也就是定义出来并通过参数传入```unwrap_or_else```这个匿名函数

```rust
use std::env;
use std::fs;
use std::process;
use std::error::Error;
fn main() { 

    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("error ->: {}", err);
        process::exit(1);
    });

    if let Err(e) = run(config){
        println!("Application error ->: {}", e);
        process::exit(1);
    }
    
    //run(config);
}

fn run(config: Config) -> Result<(), Box<dyn Error>>{
    let contents = fs::read_to_string(config.filename)?;
    println!("contents is : {}", contents);

    Ok(())
}

// fn run(config: Config) {
//     let contents = fs::read_to_string(config.filename).expect("something wrong when read file");
//     println!("contents is : {}", contents);
// }

struct Config {
    query: String,
    filename: String
}

impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enouth args");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config{query, filename})
    }
}
```

* ```Box<dyn Error>```意味着函数会返回一个实现了Error trait的类型，并不需要指定具体的类型是什么
* 意味着可以在不同场景下返回不同的错误类型，dyn关键字代表dynamic的含义
* ?运算符代替expect,可以将错误值返回给函数调用者来进行处理
* run函数运行成功返回的是()，没有必要调用```unwrap_or_else```把这个必定时()的值取出来，只需要关心错误时的情况，所以这里用```if let```


最后把代码拆分成2块
```rust
// main.rs
use std::env;
use std::process;

use minigrep::Config;

fn main() { 

    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("error ->: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config){
        println!("Application error ->: {}", e);
        process::exit(1);
    }
    //run(config);
}

```

```rust
// lib.rs
use std::error::Error;
use std::fs;

pub fn run(config: Config) -> Result<(), Box<dyn Error>>{
    let contents = fs::read_to_string(config.filename)?;
    println!("contents is : {}", contents);

    Ok(())
}

// fn run(config: Config) {
//     let contents = fs::read_to_string(config.filename).expect("something wrong when read file");
//     println!("contents is : {}", contents);
// }
pub struct Config {
    query: String,
    filename: String
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enouth args");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config{query, filename})
    }
}
```

标准错误
```rust
if let Err(e) = minigrep::run(config){
    eprintln!("Application error ->: {}", e);
    process::exit(1);
}
```

### 第13章 函数式语言特性：迭代器与闭包

Rust中的闭包是一种可以存入变量或作为参数传递给其他函数的匿名函数

```rust
// 使用两种不同类型调用同一个需要类型推导的闭包(编译错误)
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

P363 健身计划代码，创建一个同时存放闭包及闭包返回值的结构体。这个结构体只会在需要获得结果值时运行闭包，并将首次运行闭包的结果缓存起来，这样余下的代码就不必再负责存储结果，而可以直接复用该结果。这种模式一般叫作 **记忆化**或**惰性求值**

```rust
struct Cacher<T> 
    where T: Fn(u32) -> u32 {
        calculation: T,
        value: Option<u32>,
    }

impl<T> Cacher<T>
    where T: Fn(u32) -> u32 {
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}

let mut expensive_result = Cacher::new(|num| {
    println!("cal slowly...");
    thread::sleep(Duration::from_secs(2));
    num
});
```

以上代码还有两个问题
1. 第一次使用后，Some会存储在self.value。之后无论调用value方法时传入什么值，都只会返回第一次的值。解决方法是让Cache存储一个哈希表而不是一个单一值
2. 只能接收一个获取u32类型参数并返回u32类型的值的闭包。如果想缓存其他类型，需要引入更多的泛型参数

闭包还有一项函数不具备的功能：可以捕获自己所在的上下文环境并访问自己被定义时的作用域中的变量

```rust
// 正确闭包中的x就是外面的变量x
let x = 4;

let equal_to_x = |z| z == x;

let y = 4;
assert!(equal_to_x(y));
```

```rust
// 错误
let x = 4;

fn equal_to_x(z: i32) -> bool {
    z == x
}

let y = 4;
assert!(equal_to_x(y));
```

闭包可以通过3种方式从环境中捕获值：获取所有权、可变借用、不可变借用
3种方式i分别编码在3种Fn系列的trait中
* FnOnce
* FnMut
* Fn

上面例子```equal_to_x```只需要读取x中的值，仅仅不可变地借用了x并实现了Fn trait
强制闭包获取环境中值的所有权，可以在参数列表前添加move关键字

```rust
let x  = vec![1, 2, 3];

let equal_to_x = move |z| z == x;

// 这里会报错，因为闭包已经拥有了x的所有权
println!("can't use x here: {:?}", x);

let y = vec![1, 2, 3];
assert!(equal_to_x(y));
```

##### 使用迭代器处理元素序列

迭代器是惰性的。意味着创建迭代器后，除非主动调用方法来消耗，否则不会产生任何实际效果

```rust
// for循环中使用迭代器
let x  = vec![1, 2, 3];

let x_iter = x.iter();

for val in x_iter {
   println!("val: {}", val); 
}
```

所有迭代器都实现了标准库中Iterator trait
```rust
// 手动调用迭代器的next方法
let x  = vec![1, 2, 3];

let mut x_iter = x.iter();

assert_eq!(x_iter.next(), Some(&1));
assert_eq!(x_iter.next(), Some(&2));
assert_eq!(x_iter.next(), Some(&3));
assert_eq!(x_iter.next(), None);
```

这里```x_iter```必须是可变的，因为调用next方法改变了迭代器内部用来记录序列位置的状态。换句话说，就是消耗或使用了迭代器，每次调用next都吃掉了一个元素。在刚才for循环不要求可变，是因为循环取得```x_iter```的所有权并在内部使得它可变

```rust
// sum方法得到迭代器中所有元素的总和
let x  = vec![1, 2, 3];

let x_iter = x.iter();

// sum方法会获取迭代器的所有权并反复调用next来遍历元素
let total: i32 = x_iter.sum();

assert_eq!(total, 6);
```

由于调用sum的过程获取了迭代器的所有权，所以该迭代器无法被后续代码使用

```rust
// 调用map方法创建新迭代器，接着调用collect方法将其消耗掉并得到一个动态数组
let x  = vec![1, 2, 3];

let x2: Vec<_> = x.iter().map(|x| x + 1).collect();

assert_eq!(x2, vec![2, 3, 4]);
```

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe>{
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}

let shoes = vec![
    Shoe { size: 10, style: String::from("aaaa")},
    Shoe { size: 30, style: String::from("bbbb")},
    Shoe { size: 10, style: String::from("cccc")},
];

let in_my_size = shoes_in_my_size(shoes, 10);


let shoes2 = vec![
    Shoe { size: 10, style: String::from("aaaa")},
    Shoe { size: 10, style: String::from("cccc")},
];

assert_eq!(in_my_size, shoes2);
```

```into_iter()```创建可以获取动态数组所有权的迭代器


P384 使用Iterator跟```std::env::Args```实现读取参数

```rust
// 用迭代器改写search函数
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}

pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines().filter(|line| line.contains(query))
        .collect()
}
```

### 第14章 进一步认识Cargo及```crates.io```

Cargo.toml文件中没有任何[profile.*]区域时，cargo针对每个配置都会有一套默认的配置，我们可以添加配置来覆盖默认配置

下面是dev与release的默认优化配置
```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

P399 通过```pub use```语句重新导出到顶层结构

P401 发布包

### 第15章 智能指针

指针，指代那些包含内存地址的变量
Rust中最常用的指针就是引用，用```&```符号表示，会借用它所指向的值。引用除了指向数据外没有任何其他功能，也没有任何额外的开销

智能指针则是一种数据结构，类似指针但拥有额外的元数据和附加功能

引用与智能指针差别：引用只借用数据的指针；而大多数智能指针本身就拥有它们指向的数据

String跟Vec<T>都属于智能指针，因为它们都拥有一片内存区域并允许用户对其进行操作

通常会使用结构体来实现智能指针，但区别于一般结构体的地方在于它们会实现 Deref与Drop这两个 trait。Deref trait使得智能指针结构体的实例拥有与引用一致的行为，它使你可以编写出能够同时用于引用和智能指针的代码。Drop trait 则使你可以自定义智能指针离开作用域时运行的代码。

标准库中最为常见的智能指针

:wq* Box<T>，可用于在堆上分配值
* Rc<T>，允许多重所有权的引用计数类型
* Ref<T>和RefMut<T>，可以通过RefCell<T>访问，是一种可以在运行时而不是编译时执行借用规则的类型

##### 使用Box<T>在堆上分配数据

装箱使我们可以将数据存储在堆上，并在栈中保留一个指向堆数据的指针

使用场景

* 当你拥有一个无法再编译时确定大小的类型，但又想要在一个要求固定尺寸的上下文环境中使用这个类型的值时
* 当你需要传递大量数据的所有权，但又不希望产生大量数据的复制行为时
* 当你希望拥有一个实现了执行trait的类型值，但又不关心具体的类型时

```rust
// 使用Box<T>的List实现递归
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::Cons;
use crate::List::Nil;

fn main() {
    println!("Hello, world!");

    let list = Cons(1, 
            Box::new(Cons(2, 
                Box::new(Cons(3, Box::new(Nil))))));

}
```
Cons为链接列表

```rust
let x = 5;
// 变量y存储了x的引用
let y = &x;
// 这里y设置为指向x值的装箱指针
let y = Box::new(x);


assert_eq!(5, x);
// 使用解引用跳转到它指向的值
assert_eq!(5, *y);

```

##### 定义自己的智能指针

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

use std::ops::Deref;

// 实现Deref trait来将类型视作引用
impl<T> Deref for MyBox<T> {
    type Target = T;
    
    // Deref trait要求实现deref方法，该方法会借用self并返回一个指向内部数据的引用
    // deref方法体填&self.0，意味着deref会返回一个指向值的引用，进而允许调用者通过*运算符访问值
    fn deref(&self) -> &T {
        &self.0
    }
}

let x = 5;
let y = MyBox::new(x);

assert_eq!(5, x);
assert_eq!(5, *y);

```

上面代码的```*y```会被rust隐式展开为```*(y.deref())```

```rust
// 解引用转换特性使MyBox<String>值的引用传入hello函数
fn hello(name: &str) {
    println!("Hello {}", name);
}

let m = MyBox::new(String::from("Rust"));

hello(&m);
```
上面的代码中将参数```&m```传入了hello函数，而```&m```正是一个指向```MyBox<String>```值的引用。因为我们在```MyBox<T>```实现了Deref trait，所以Rust可以通过调用deref来将```&MyBox<String>```转换为```&String```。因为标准库为String提供的 Deref实现会返回字符串切片（你可以在 Deref的API文档中看到这一信息），所以Rust可以继续调用deref来将```&String```转换为```&str```，并最终与hello函数的定义相匹配。

```rust
// 没有解引用转换，就必须这样编写代码
hello(&(*m)[...]);
```

代码中的```(*m)```首先将```MyBox<String>```进行解引用得到String，然后，通过```&```和```[..]```来获取包含整个String的字符串切片以便匹配hello函数的签名。缺少了解引用转换的代码会充斥着这类符号，从而变得更加难以阅、编写和理解。解引用转换使Rust可以为我们自动处理这些转换过程。

##### Drop trait在清了时运行代码

常常被用来释放文件、网络连接等资源，几乎每一种智能指针的实现都会用到这个trait。例如Box<T>通过自定义Drop来释放装箱指针指向的堆内存空间

```rust
struct CustomerSmartPointer {
    data: String,
}

impl Drop for CustomerSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomerSmartPointer with data `{}`", self.data);
    }
}

let c = CustomerSmartPointer {
    data: String::from("123"),
};
let c2 = CustomerSmartPointer {
    data: String::from("456"),
};
// c.drop(); // 错误
println!("end");
```

上面代码，会依次输出end, 456, 123因为c跟c2在代码结束才离开作用域调用Drop trait的drop方法，并且变量丢弃顺序与创建顺序相反

上面代码不能显式调用drop方法，因为显式调用跟离开作用域自动调用会导致重复释放错误

```rust
// 提前显式清理
let c = CustomerSmartPointer {
    data: String::from("123"),
};

println!("begin drop ======");

drop(c);
println!("end ======");
```

上面代码在离开作用域前显式调用```std::men::drop```函数来丢弃。该函数被放置在预导入模块，可以直接调用

##### 基于引用计数的智能指针Rc<T>

Rust提供一个名为Rc<T>的类型支持多重所有权，Rc是Reference counting(引用计数)的缩写。Rc<T>类型的实例会在内部维护一个用于记录值引用次数的计数器，从而确认这个值是否仍在使用。如果对一个值的引用次数为零，那么就意味着这个值可以被安全地清理掉

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

fn main() {
    println!("Hello, world!");

    use crate::List::Cons;
    use crate::List::Nil;

    println!("Hello, world!");

    let list = Cons(1, Box::new(Cons(3, Box::new(Nil))));
    // 1
    let b = Cons(3, Box::new(list));
    //2
    let c = Cons(3, Box::new(list));
}

```

在位置1，list列表在创建b列表时被移动至b中，也就是说b列表持有了list列表的所有权。随后位置2尝试移动到c就会编译错误

```rust
use std::rc::Rc;
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

fn main() {
    println!("Hello, world!");

    use crate::List::Cons;
    use crate::List::Nil;

    let list = Rc::new(Cons(1, Rc::new(Cons(3, Rc::new(Nil)))));
    println!("after creat a list.count = {}", Rc::strong_count(&list));
    let b = Cons(3, Rc::clone(&list));
    println!("after creat b list.count = {}", Rc::strong_count(&list));
    {
        let c = Cons(3, Rc::clone(&list));
        println!("after creat c list.count = {}", Rc::strong_count(&list));
    }
    println!("end list.count = {}", Rc::strong_count(&list));
}

```

上面代码打印可以看Rc的引用计数，说明克隆可以增加引用计数


##### RefCell<T>和内部可变性模式

内部可变性是Rust的设计模式之一，它允许你在只持有不可变的前提下对数据进行修改。为了能改变数据，内部可变性模式在他的数据结构中使用了unsafe(不安全)代码来绕过Rust正常的可变性和借用规则。

与Rc<T>不同，RefCell<T>类型代表了其持有数据的唯一所有权

借用规则：
* 在任何给定的时间里，要么只能拥有一个可变引用，要么只能拥有任意数量的不可变引用
* 引用总是有效的

RefCell<T>和Box<T>的区别:
对于使用一般引用和Box<T>的代码，Rust会在编译阶段强制代码遵守这些借用规则。而对于使用RefCell<T>的代码，Rust则只会在**运行时**检查，并在出现违反借用规则的情况下触发panic来提前中止程序

在**编译期**检查借用规则对于大多数场景而言都是最佳的选择，这也正是Rust将编译期检查作为默认行为的原因。

在运行时检查借用规则可以使我们实现某些特定的内存安全场景，即便这些场景无法通过编译期检查。并不是程序中所有的属性都能够通过分析代码来得出：经典例子是停机问题(Halting Problem)

在编译器无法理解代码，但开发者可以保证借用规则能够满足的情况下，RefCell<T>便有了它的用武之地

RefCell<T>只能用于单线程场景中。强行将它用于多线程环境中会产生编译错误

选择使用Box<T>, Rc<T>, 还是RefCell<T>的依据:
* Rc<T>允许一份数据有多个所有者，而Box<T>和RefCell<T>都只有一个所有者
* Box<T>允许在编译时检查可变或不可变借用，Rc<T>仅允许编译时检查的不可变借用，RefCell<T>允许运行时检查可变或不可变借用
* 由于RefCell<T>允许在运行时检查可变借用，所以即使RefCell<T>本身是不可变的，仍然能够更改其中存储的值

##### 内部可变性：可变地借用一个不可变的值

```rust
// 下面代码无法通过编译
let x = 5;
let y = &mut x;

```

然而在某些特定情况下，需要一个值在对外保持不可变性，同时能够在方法内部修改自身。除了这个值本身的方法，其余的代码则依然不能修改这个值。使用````RefCell<T>```就是获得这种内部可变性的一种方法

##### 内部可变性的应用场景：模拟对象

测试中的模拟对象(mock object)


```rust
// src/lib.rs
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: 'a + Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error, over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger.send("Warning, over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger.send("Warning, over 75% of your quota!");
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messenger: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messenger: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            // 1
            self.sent_messenger.push(String::from(message));
        }
    }

    #[test]
    fn sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();

        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messenger.len(), 1);
    }
}

```

上面代码在1处报错，由于send方法接收了self的不可变引用，所以无法修改MockeMessenger的内容来记录消息。也不能修改为```&mut self```，因为修改后签名与Messenger trait定义的send签名不符

```rust
// 用RefCell改写
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        // 1
        sent_messenger: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                // 2
                sent_messenger: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            //self.sent_messenger.push(String::from(message));
            // 3
            self.sent_messenger.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();

        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);
        // 4
        assert_eq!(mock_messenger.sent_messenger.borrow().len(), 1);
    }
}

```

在保持外部值不可变的前提下，使用RefCell<T>来修改内部存储的值

borrow和borrow_mut方法分别返回Ref<T>与RefMut<T>这两种智能指针，都实现了Deref，所以可以把它们当作一般的引用

RefCell<T>借用检查规则：在任何一个给定的时间里，只允许拥有多个不可变借用或一个可变借用

```rust
// 创建两个同时有效的可变借用
impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        //self.sent_messenger.push(String::from(message));
        //self.sent_messenger.borrow_mut().push(String::from(message));
        let mut one = self.sent_messenger.borrow_mut();
        let mut two = self.sent_messenger.borrow_mut();

        one.push(String::from(message));
        two.push(String::from(message));
    }
}
```

上面代码会引发panic，报错```already borrowed:BorrowMutError```



##### Rc<T>和RefCell<T>结合使用来实现一个拥有多重所有权的可变数据

```rust
use std::{cell::RefCell, rc::Rc};
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

fn main() {
    println!("Hello, world!");

    use crate::List::Cons;
    use crate::List::Nil;

    // 1
    let list = Rc::new(RefCell::new(5));

    // 2
    let a = Rc::new(Cons(Rc::clone(&list), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    // 3
    *list.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

首先创建一个Rc<RefCell<i32>实例，并将它暂时存入list变量中，以便之后可以直接访问。接着，使用含有list的Cons变体创建一个List类型的a变量。为了确保a和list同时持有内部值5的所有权，这里的代码还克隆了list，而不仅仅只是将list的所有权传递给a，或者让a借用list

为了让b和c能够同时指向a，将a封装到了Rc<T>中

创建完a、b、c这3个列表后，通过调用```borrow_mut```来将list指向的值增加10.注意，这里使用了自动解引用功能来将Rc<T>解引用为RefCell<T>。```borrow_mut```方法返回一个RefMut<T>智能指针，可以使用解引用运算符来修改其内部值

这种实现方法非常简单明了！通过使用 ```RefCell<T>```，我们拥有的List保持了表面上的不可变状态，并能够在必要时借由```RefCell<T>```提供的方法来修改其内部存储的数据。运行时的借用规则检查同样能够帮助我们避免数据竞争，在某些场景下为了必要的灵活性而牺牲一些运行时性能也是值得的。

标准库还提供了其他一些类型来实现内部可变性，例如与```RefCell<T>```十分类似的```Cell<T>```，但相比于前者通过借用来实现内部数据的读写，```Cell<T>```选择了通过复制来访问数据。另外还有在第16章会讨论到的```Mutex<T>```，它被用于实现跨线程情形下的内部可变性模式。请参考标准库文档来了解有关这些类型有哪些区别的更多信息。

##### 循环引用会造成内存泄露 P452

##### 使用Weak<T>代替Rc<T>来避免循环引用 P456


### 第16张 无畏并发

安全并且高效地处理并发编程是Rust的另一个主要目标。
并发编程（concurrent programming)与并行编程（paralld programming）这两种概念随着计算机设备的多核心化而变得越来越重要。前者允许程序中的不同部分相互独立地运行，而后者则允许程序中的不同部分同时执行

在大部分现代操作系统中，执行程序的代码会运行在进程（process)中，操作系统会同时管理多个进程。类似地，程序内部也可以拥有多个同时运行的独立部分，用来运行这些独立部分的就叫作线程（thread)。
由于多个线程可以同时运行，所以将程序中的计算操作拆分至多个线程可以提高性能。但这也增加了程序的复杂度，因为不同线程在执行过程中的具体顺序是无法确定的。这可能会导致一系列的问题，比如：

* 当多个线程以不一致的顺序访问数据或资源时产生的竞争状态(race
condition)。
* 当两个线程同时尝试获取对方持有的资源时产生的死锁(deadlock)，
它会导致这两个线程无法继续运行。
* 只会出现在特定情形下且难以稳定重现和修复的bug。

```rust
use std::thread;

use std::time::Duration;
// 用spawn创建新线程
thread::spawn(|| {
    for i in 1..10 {
        println!("number {} from the spawned thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
});


for i in 1..5 {
    println!("number {} from the main thread!", i);
    thread::sleep(Duration::from_millis(1));
}

```

main线程首先打印，新线程不管打印到那一个数字，只要main线程结束，新线程就结束
线程会交替执行，无法对他们的执行顺序做任何保证，执行顺序由操作系统的线程调度策略决定


```rust
use std::thread;

use std::time::Duration;

let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("number {} from the spawned thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
});

for i in 1..5 {
    println!("number {} from the main thread!", i);
    thread::sleep(Duration::from_millis(1));
}
// 阻塞main线程1
handle.join().unwrap();
```

```thread::spawn```的返回类型是一个自持有所有权的JoinHandle，调用它的join方法可以阻塞当前线程直到对应的新线程运行结束

也可以把join语句移到main线程前面，那就会等新线程执行完才开始执行main线程

##### 在线程中使用move闭包

move闭包常常被用来与```thread::spawn```函数配合使用，它允许你在某个线程中使用来自另一个线程的数据

在闭包的参数列表前使用move关键字来强制闭包从外部环境中捕获值的所有权。它可以跨线程地传递某些值的所有权。

```rust

use std::thread;

let v = vec![1, 2, 3];

let handle = thread::spawn(move || println!("vector {:?}", v));

handle.join().unwrap();

```

##### 使用消息传递在线程间转移数据

Rust标准库中实现了一个名为通道(channel)的编程概念，它可以被用来实现基于消息传递的并发机制。

```rust

use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("hello mpsc");
    tx.send(val).unwrap();
});

let received = rx.recv().unwrap();
println!("Got val: {}", received);

```

上面引入的包mpsc，英文是"multiple producer, single consumer"(多个生产者，单个消费者)的缩写

函数```mpsc::channel```返回一个含有发送端跟接收端的元组。

##### 发送多个值并观察接收者的等待过程

```rust

use std::sync::mpsc;
use std::thread;
use std::time::Duration;

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let vals = vec![
        String::from("hello mpsc 1"),
        String::from("hello mpsc 2"),
        String::from("hello mpsc 3"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_millis(1000));
    }
});

for received in rx {
    println!("Got val: {}", received);
}

```

上面代码并没有在主线程的for循环中执行暂停或延迟指令，表明主线程确实是在等待接收新线程中传递过来的值。


##### 通过克隆发送者创建多个生产者

```rust

use std::sync::mpsc;
use std::thread;
use std::time::Duration;

let (tx, rx) = mpsc::channel();

let tx2 = mpsc::Sender::clone(&tx);

thread::spawn(move || {
    let vals = vec![
        String::from("tx2 hello mpsc 1"),
        String::from("tx2 hello mpsc 2"),
        String::from("tx2 hello mpsc 3"),
    ];

    for val in vals {
        tx2.send(val).unwrap();
        thread::sleep(Duration::from_millis(1000));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("tx hello mpsc 1"),
        String::from("tx hello mpsc 2"),
        String::from("tx hello mpsc 3"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_millis(1000));
    }
});

for received in rx {
    println!("Got val: {}", received);
}

```


