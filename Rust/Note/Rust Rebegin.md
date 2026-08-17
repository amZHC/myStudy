# Rust Rebegin

第二次开始学习Rust，希望能有所收获。

## 学习笔记

### Rust结构

crate
library crate  (1)
binary crate  (n)

### 猜数游戏

```rust
use rand::Rng;  //引入的crate为rand 0.8.5
use std::{cmp::Ordering, io};

fn main() {
    println!("Guess the number");
    let secret_number: u32 = rand::thread_rng().gen_range(1..=100); //生成 1 <= secret_number <= 100
    // println!("The secret number is: {}", secret_number);
    loop {
        println!("Please enter your guess.");
        let mut guess: String = String::new();
        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
        println!("Your guess is: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small"),
            Ordering::Greater => println!("Too big"),
            Ordering::Equal => {
                println!("You win");
                break;
            },
        }
    }
}
```

### 变量与数据类型

#### 常量

1. 使用const声明
2. 不可使用mut
3. 必须标注类型
4. 可以在任意作用域声明
5. 仅可以使用常量表达式赋值
6. 不支持变量遮蔽，也就是不能重复定义

```rust
fn main()
{
    const PI:f64 = 3.1415926;
}
```

#### 变量遮蔽 Shadowing

1. 可以用之前变量相同的名字声明一个新变量:第一个变量被第二个变量“遮蔽”了
2. 就是创建了一个新变量，只不过名字相同

```rust
fn main(){
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("The value of x in the inner scope is : {x}"); // 12
    }
    println!("The value of x  is : {x}"); //6
}
```

#### 数据类型

##### 标量类型 Scalar

表示一个单一的值

1. 整数 Integer
   
   <img title="" src="Rust Rebegin.assets/image-20250519160136971.png" alt="image-20250519160136971" >
   
   <img title="" src="Rust Rebegin.assets/image-20250519160308738.png" alt="image-20250519160308738" >

2. 浮点 Floating Point
   
    a. f32:4字节

    b. f64(默认):8字节

    c. 都有符号(signed)

3. 布尔 Boolean
   
    a. 两个值:true,false

    b. 1字节

4. 字符 Character
   
    a. 4字节

    b. 表示一个Unicode标量值(Unicode Scalar Value)

##### 复合类型 Compound

###### Tuple与Array

可以将多个值组合在一个类型

1. 元组 Tuple
   
    a. 固定长度

    b. 可包含不同类型的数据

2. 数组 Array
   
    a. 固定长度

    b. 元素类型相同

   ```rust
   fn main() {
       let tup1 = (100, 'A', false);
       let tup2: (i32, f64, u8) = (500, 6.4, 1);
       println!("tup1.0: {}", tup1.0);
       println!("tup2.0: {}", tup2.0);
   
       let (x, y, z) = tup2;
       println!("x: {}, y: {},z: {}", x, y, z);
   
       let arr1 = [1, 2, 3, 4, 5];
       let arr2: [i32; 5] = [6, 7, 8, 9, 10];
       let arr3: [i32; 5] = [3; 5];
   
       println!("arr1[0]: {}", arr1[0]);
       println!("arr2[0]: {}", arr2[0]);
       println!("arr3[0]: {}", arr3[0]);
   }
   
   fn main() {
       let one = [1, 2, 3];
   
       let two: [u8; 3] = [4, 5, 6];
       let blank1 = [0; 3];
       let blank2: [u8; 3] = [1; 3];
   
       let arrays: [[u8; 3]; 4] = [one, two, blank1, blank2];
   
       for a in &arrays {
           print!("{:?}", a);
   
           for n in a.iter() {
               print!("\t{} + 10 = {}", n, n + 10);
           }
   
           let mut sum = 0;
   
           for i in 0..a.len() {
               sum += a[i];
           }
           println!("\t({:?} = {})", a, sum);
       }
   ```

```
fn main() {
    //元组 tuple
    let m_tuple = (1,2,3,4);
    println!("By tuple: {}",m_tuple.0);

    //数组
    let m_array:[i32;5] = [1,2,3,4,5];
    println!("By Array: {}", m_array[4]);

    //get Array length
    println!("The length of Array: {}",m_array.len());

    //遍历数组
    for item in m_array.iter()
    {
        println!("foreach m_array: {}", item);
    }

}

// 值传递，原数组在可变情况下在方法内修改也不会影响原数组
// 此时mut修饰的是array数组在方法内是可变的
fn foreachArray(mut array:[i32;5])
{
    for item in array.iter()
    {
        println!("Array content: {}", item);
    }
    array[0] = 10;
}


// 地址引用
// 此时方法内修改array, 会影响到外部变量
fn change_arr(array:&mut [i32;5])
{

}
```

##### 字符串

```rust
//1.
//内置字符串字面量 &str，
//在没款 std::str，本质是字符串切片
fn main()
{
    let s1 = "this is &str";
    println!("s1 is {}", s1);
}

//2.
//标准库中的一个公开pub结构体。字符串对象String。
//在堆上分配，可以在运行时提供字符串的值以及相应的操作方法。
//String::new()  创建一个新的字符串，静态的
//String::from() 从具体的字符串字面量创建字符串对象
fn main()
{
    let s1 = String::new();
    println!("s1 is {}", s1);

    let s2 = String::from("study for money");
    println!("s2 is {}", s2);

    let mut s3 = String::new();
    s3 = String::from("this is not s3");
    s3.push_str(" , I am liying");
    s3.push('.');
    println!("s3 been changed to {}",s3);


    //字符串对象和字面量相互转换
    let s4 = "this is &str to String".to_string();

    let s5 = s3.as_str();
}
```

遍历字符串

```rust
fn main() {
    let str: String = String::from("China");
    let bytes: &[u8] = str.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        println!("{} {}", i, item as char);
    }
}
```

##### 枚举

```rust
fn main()
{
    let eenum = Week::Friday;

    println!("{:?}", eenum);
    
    // Option
    let price = 250;
    println!("{:?}",get_diconnect(price));

    // 作为match判断
    let today = Week::Sunday;
    what_today(today);
}

#[derive(Debug)]
// enum 枚举
enum Week{
    Monday,
    Tuesday,
    Wednessday,
    Thursday,
    Friday,
    Saturday,
    Sunday
}

/*
        //带数据类型的枚举
        enum 枚举名称{
            varient1(数据类型1),
            varient2(数据类型2)，
            ...
        }
     */
enum StudyRoadMap{
    Name(String),
    Age(i32)
}

    /*
        //Option基于enum，经常用于函数的返回值，可以有返回值，也可以没有返回值
        enum Option<T>{
            Some(T),  //用于返回一个值
            None      // 返回null，用None表示
        }

     */

fn get_diconnect(price:i32) ->Option<bool>
{
    if(price > 100){
        Some(true)
    }else {
        None
    }
}

// 作为match的判断值
fn what_today(day:Week){
    match day {
        Week::Monday => {
            println!("Today is Moday");
        },
        Week::Tuesday => {
            println!("2");
        },
        Week::Wednessday => {
            println!("3");
        },
        Week::Thursday => {
            println!("4");
        },
        Week::Friday => {
            println!("5");
        },
        Week::Saturday => {
            println!("6");
        },
        Week::Sunday => {
            println!("7");
        }
    }
}

```

#### 集合

vector



#### 函数

##### 函数实现

命名和变量的命名规范：snake case

1. 所有字母都小写

2. 单词之间用“_”连接

3. 必须声明参数的类型
   
   ```rust
   fn main(){
       println!("Hello World!");
   
       another_function();
       print_label_message(5，a);
   }
   
   fn another_function(){
       println!("This is another_function")
   }
   
   fn print_label_message(value: i32, unit_label: char){
       println!("Value is {},unit_label is {}",value,unit_label);
   }
   ```
   
   函数体由一系列语句组成，可由表达式结尾

##### 函数的返回值

1. 可以使用return返回

2. 函数最后一个表达式的值（不可以加;作为结尾）

```rust
fn is_larger(num1:i32, num2:i32) ->bool
{
    if num1 > num2 { true } else {false}
}
//或者
//fn is_larger(num1:i32, num2:i32) ->bool
//{
//    if num1 > num2 { return true; } else { return false; }
//}

fn main()
{
    let a:i32 = 32;
    let b:i32 = 23;

    println!("{:?}", is_larger(a, b));
}
```

#### 语句与表达式

**Rust中不支持+ +(自增)和- -(自减)运算符**

语句(Statements): 执行某些指令的操作，不返回值
 表达式(Expressions): 计算并返回一个结果值

```rust
fn main(){
 let x = five();
 println!("x is {}",x);
}

fn five() -> i32 {
 5
}
```

#### 控制流(Control Flow)

##### IF表达式

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4 or 3 or 2");
    }
}
```

```rust
fn main(){
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("number is {}",number); // 5    
}
```

##### loop循环

```rust
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;  //跳出循环且带返回值
        }
    };

    println!("result is {}", result); // result is 20
}
```

loop循环添加标签，在循环嵌套时可以在内层循环里直接跳出外层循环。

```rust
fn main() {
    let mut x = 1;
    let mut y = 1;
    'label1: loop { //label1是loop循环的标签
        loop {
            if x > 9 {
                break 'label1;  //直接跳出label1循环
            }
            if y <= x {
                print!("{} * {} = {}  ", y, x, x * y);
                y += 1;
            } else {
                break;
            }
        }
        print!("\n");
        y = 1;
        x += 1;
    }
}
```

##### for循环

```rust
// eg1 
fn main() {
    for i in 1..=5 {
        println!("{}", i);
    }
    let arr = [1,2,3,4,5];
    for x in arr {
        println!("arr {}",x)
    }
}

//eg2 结合集合遍历
fn main()
{
    let studyList = vec![
        "I love Cpp",
        "Rust is my extern",
        "I want to be a program master"
    ];

    for con in studyList.iter()
    {
        match con {
            &"I love Cpp" => println!("I found that"),
            _ => println!("not that")
        }
    }

}

//eg3 结合集合知识和借用相关知识
fn main()
{
    let studyList = vec![
        "I love Cpp",
        "Rust is my extern",
        "I want to be a program master"
    ];

    //iter()每次迭代是借用集合中的一个元素。元素本身不会改变，循环之后可以使用
    for con in studyList.iter()
    {
        match con {
            &"I love Cpp" => println!("I found that"),
            _ => println!("not that")
        }
    }


    let studyList2 = vec![
        "I love Cpp",
        "Rust is my extern",
        "I want to be a program master"
    ];

    //into_iter()会消耗集合，每次迭代，集合中的数据本身被提供，
    //一旦集合被消耗完了，之后集合再无法使用，因为元素都被move了
    for con in studyList2.into_iter()
    {
        match con {
            "I love Cpp" => println!("I found that"),
            _ => println!("not that")
        }
    }

    let mut studyList3 = vec![
        "I love Cpp",
        "Rust is my extern",
        "I want to be a program master"
    ];

    //iter_mut() 可变借用集合中的内容，不会消耗集合中的元素
    for con in studyList3.iter_mut()
    {
        //通过match 返回内容
        *con = match con {
            &mut "I love Cpp" => {"I love Rust"},
            _ => *con,
        };

        println!("after changed : {:?}",con);
    }

    for con2 in studyList3.iter() {
        println!("studyList3 :{}",con2);
    }
}
```

##### while循环

```rust
fn main()
{
     let mut num = 2;

    while num < 20 {
        println!("num: {}", num);
        num *= 2;
    }
}
```

### 模式匹配

#### match

match类似其他语言的switch，但match必须穷尽所有可能，

```rust
fn main() {
    let x = 10;
    match x {
        1 => println!("This is 1"),
        10 => println!("This is 10"),
        _ => println!("I don't know")   //_表示default
    }
}

//使用match复制
enum IpAddr {
    Ipv4,
    Ipv6,
}
fn main() {
    let ip1 = IpAddr::Ipv6;
    let ip_str = match ip1 {
        IpAddr::Ipv4 => "127.0.0.1",
        _ => "::1",
    };
    println!("{}", ip_str);
}
```

#### 模式绑定

从模式中取出绑定的值

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nikel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nikel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
fn main() {
    let t = Coin::Quarter(UsState::Alaska);
    value_in_cents(t);
}
```

```rust
enum Action {
    Say(String),
    MoveTo(i32, i32),
    ChangeColorRGB(u16, u16, u16),
}

fn main() {
    let actions = [
        Action::Say("Hello Rust".to_string()),
        Action::MoveTo(1, 2),
        Action::ChangeColorRGB(255, 255, 0),
    ];

    for action in actions {
        match action {
            Action::Say(s) => {
                println!("he say {}", s);
            }
            Action::MoveTo(x, y) => {
                println!("Then move to x:{} , y:{}", x, y);
            }
            Action::ChangeColorRGB(R, G, B) => {
                println!("Change to color to RGB({},{},{})", R, G, B);
            }
        }
    }
}
```

#### 穷尽匹配

`match` 的匹配必须穷尽所有情况。

#### _ 通配符

除了目标情况，其他用_代替

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

除了`_`通配符，用一个变量来承载其他情况也是可以的。

```rust
#[derive(Debug)]
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        other => println!("other direction: {:?}", other),
    };
}
```

#### if let

**你只要匹配一个条件，且忽略其他条件时就用 `if let` ，否则都用 `match`**。

```rust
let v = Some(3u8);
match v {
    Some(3) => println!("three"),
    _ => (),
}
//这两段代码的作用相同
if let Some(3) = v {
    println!("three");
}
```

#### matches!宏

将一个表达式跟模式进行匹配，然后返回匹配的结果 `true` or `false`。

```rust
#[derive(Debug)]
enum MyEnum {
    Foo,
    Bar,
}
fn main() {
    let v = vec![MyEnum::Foo, MyEnum::Bar, MyEnum::Foo];
    // 对 v 进行过滤，只保留类型是 MyEnum::Foo 的元素
    let mut v2 = v.iter().filter(|x| matches!(x, MyEnum::Foo));
    let mut temp = v2.next();
    while !matches!(temp, None) {
        println!("{:?}", temp);
        temp = v2.next();
    }
    // println!("{:?}", v2.next());
    // println!("{:?}", v2.next());
    // println!("{:?}", v2.next());

    //更多例子
    let foo = 'f';
    assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));

    let bar = Some(4);
    assert!(matches!(bar, Some(x) if x > 2));
}
```

#### Option\<T>

**一个变量要么有值：`Some(T)`, 要么为空：`None`**。

因为 `Option`，`Some`，`None` 都包含在 `prelude` 中，因此你可以直接通过名称来使用它们，而无需以 `Option::Some` 这种形式去使用，总之，千万不要因为调用路径变短了，就忘记 `Some` 和 `None` 也是 `Option` 底下的枚举成员！

```rust
//定义
enum Option<T> {
    None,
    Some(T),
}

//例子
fn option_plus_int(a: Option<i32>, b: i32) -> Option<i32> {
    match a {
        None => None,
        Some(i) => Some(i + b),
    }
}
fn main() {
    let mut x = Some(5);
    let y = 12;
    x = option_plus_int(x, y);
    println!("new x is {:?}",x);
}
//使用if let取出Some(T)中的T
fn main() {
    let x = Some(52);
    let mut x2: i32 = 0;
    if let Some(y) = x {
        x2 = y;
    }
    println!("{}", x2);
}
```

结构嵌套的结构体和枚举

```rust
enum Color {
    RGB(i32, i32, i32),
    HSV(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::HSV(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::RGB(r, g, b)) => {
            println!("Change Color to red: {} , blur: {} , green: {}", r, g, b)
        }
        Message::ChangeColor(Color::HSV(h, s, v)) => {
            println!("Change Color to hue: {} , saturation: {} , value: {}", h, s, v)
        }
        _ => println!("I don't know how to do"),
    }
}
```

### 结构体

```rust

fn main()
{
    let s = Study{
        name:String::from("this is myStruct"),
        target:String::from("I want do more with Rust"),
        append: 2
    };

    println!("{:?}",s);

    let s2 = getInstance(String::from("this fn"), String::from("How to say it"), 3);

    // 单元结构体 是一个类型，有且只有一个值()


    // 元组结构体
    let pair = (String::from("Cpp To Rust"), 1);

    println!("pair 包含 {:?} and {}", pair.0, pair.1);

    // 解构元组结构体
    let (pair_string, pair_i32) = pair;
    println!("pair 包含 {:?} and {}", pair_string, pair_i32);
}

#[derive(Debug)]
// C语言风格结构体
struct Study{
    name:String,
    target:String,
    append:i32
}

// 函数 用于创建一个结构体
// 不属于任何部分，作用范围内唯一命名
fn getInstance(name:String, target:String, append:i32) -> Study{
    return Study { name, target, append };
}

// implement 实现
// 方法 基于结构体   
//因为基于结构体，所以可以重名
impl Study {
    fn show_my_name(&self){
        println!("{}",self.name);
    }

    // 静态方法
    fn get_instance(name:String, target:String, append:i32) -> Study{
        return Study { name, target, append };
    }
}


```

#### 定义方法

```rust
//结构体的方法
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
    // 这种方法往往用于初始化当前结构体的实例
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }
    // Circle的方法，&self表示借用当前的Circle结构体
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

模块

```rust
mod my {
    pub struct Rectangle {
        width: u32,         //该参数是私有的
        pub height: u32,    //该参数是公开可见的
    }
    impl Rectangle {
        pub fn new(width: u32, height: u32) -> Self {
            Rectangle { width, height }
        }
        pub fn width(&self) -> u32 {
            self.width
        }
        pub fn height(&self) -> u32 {
            self.height
        }
    }
}

use crate::my::Rectangle;
fn main() {
    let rec1 = Rectangle::new(12, 55);

    println!("width: {}", rec1.width());
    println!("height: {}", rec1.height());
    println!("height: {}", rec1.height);
}
```

#### 为枚举实现方法

```rust
#[allow(unused)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        if let Message::Write(str) = &self {
            println!("He yall {}", str);
        }
    }
}

fn main() {
    let m = Message::Write(String::from("Hello"));

    m.call();
}
```

### 泛型和特征

#### 泛型Generics

T是**泛型参数**，可以是任何类型，使用效果类似C语言中的多态，但在实现某些功能，如相加时，不是所有T都能相加，需要使用std::ops::Add<Output = T>对T进行限制。

```rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
    a + b
}
```

**显式地指定泛型的类型参数**

```rust
use std::fmt::Display;

fn create_and_print<T>() where T: From<i32> + Display {
    let a: T = 100.into(); // 创建了类型为 T 的变量 a，它的初始值由 100 转换而来
    println!("a is: {}", a);
}

fn main() {
    create_and_print::<i64>();  //在此处显式指定参数类型为 i64
}
```

**在结构体中使用泛型**

```rust
/// eg1
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let intP = Point { x: 5, y: 7 };
    let floatP = Point { x: 1.2, y: 4.5 };
    // let mixP = Point {x: 1,y: 2.4};  //这个类型会报错，在该结构体中x和y只能是同一种类型
}


/// eg2
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let intP = Point { x: 5, y: 7 };
    let floatP = Point { x: 1.2, y: 4.5 };
    let mixP = Point { x: 1, y: 2.4 }; //此时x和y可以是不同类型
}
```

**方法中使用泛型**

```rust
/// eg1
struct Point<T> {
    x: T,
    y: T,
}
impl<T> Point<T> {
    fn x(&self) -> T {
        &self.x 
    }
}

/*使用泛型参数前，依然需要提前声明：impl<T>，只有提前声明了，我们才能在Point<T>中使用它，
这样 Rust 就知道 Point 的尖括号中的类型是泛型而不是具体类型。需要注意的是，这里的 Point<T> 不再是泛型声明，
而是一个完整的结构体类型，因为我们定义的结构体就是 Point<T> 而不再是 Point。 */

fn main() {
    let intP = Point { x: 5, y: 7 };
    let floatP = Point { x: 1.2, y: 4.5 };
    let mixP = Point { x: 1, y: 2.4 }; //此时x和y可以是不同类型
}


struct Point<T, U> {
    x: T,
    y: U,
}

/// eg2
impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

**为具体的泛型类型实现方法**

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

 **Const泛型**

```rust
//定义一个[T; N]的数组，中 T 是一个基于类型的泛型参数，这个和之前讲的泛型没有区别，而重点在于 N 这个泛型参数，它是一个基于值的泛型参数！因为它用来替代的是数组的长度。
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

#### 特性trait

来自教程，此段代码存在所有权移动的问题

```rust
use std::fmt::{Display, Formatter};

fn main()
{
    let m_book:Book = Book { 
        name: String::from("bookName"), 
        id: 2, 
        author: String::from("niCai"),
    };

    //m_book.Show();

    // 此时m_book的所有权会移动到show2中
    show2(m_book);
}

// 结构体
struct Book{
    name:String,
    id:u32,
    author:String,
}

// trait 特质
// 类似Java中的接口或者cpp中的抽象基类
// ShowBook 要求实现Show方法
trait ShowBook {
    fn Show(&self);
}

// 为Book实现 ShowBook的trait
// &self是方法的接收者，表示不可变借用
impl ShowBook for Book{
    // 此处具体实现
    fn Show(&self) {
        println!("Book name: {}", self.name);
    }
}

// 泛型用作函数参数，泛型参数约束
// T:Display 表示参数T必须实现Display trait
// 只有实现了Display才能作为参数传入
// 这是Rust中的 trait bound语法
fn show2<T:Display>(t:T){
    println!("show2 : {}",t);
}

// Display trait实现,来自教程
// 
/*
原代码的问题
Display应该格式化输出到f，而不是打印到控制台
虽然编译能通过，但不符合Display的语义
impl Display for Book {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        println!("impl Display for Book :  Book's ID: {}", self.id);
        let res = Result::Ok(());
        res
    }
}
*/

// deepseek认为正确的实现方法
impl Display for Book {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        // 应该写入f，而不是打印到控制台
        write!(f, "Book: {} (ID: {}) by {}", self.name, self.id, self.author)
    }
}
```

deepseek推荐的实现方法

```rust
use std::fmt::{Display, Formatter};

struct Book {
    name: String,
    id: u32,
    author: String,
}

trait ShowBook {
    fn show(&self);  // Rust约定使用snake_case命名
}

impl ShowBook for Book {
    fn show(&self) {
        println!("Book name: {}", self.name);
    }
}

impl Display for Book {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        write!(f, "{} (ID: {}) by {}", self.name, self.id, self.author)
    }
}

fn show2<T: Display>(t: T) {
    println!("show2 : {}", t);
}

fn main() {
    let m_book = Book { 
        name: String::from("bookName"), 
        id: 2, 
        author: String::from("niCai"),
    };
    
    m_book.show();  // 调用ShowBook trait的方法
    show2(m_book);  // 调用Display trait的方法
}
```

**关键要点总结**

1. **Trait是Rust实现多态的主要方式**
2. **Display trait用于格式化输出**（与`{}`配合）
3. **Debug trait用于调试输出**（与`{:?}`配合）
4. **Trait bound限制泛型类型必须实现特定trait**
5. **实现trait时要遵循其语义**（如`fmt`应该写入`Formatter`）





### 所有权

所有权只发生在堆类型的数据上，栈数据不具备该特性，默认为值传递。

#### 借用





#### 引用

![image-20250603144321496](Rust Rebegin.assets/image-20250603144321496-17489330046781.png)

#### 解引用

![image-20250603144844358](Rust Rebegin.assets/image-20250603144844358-17489333267603.png)

![image-20250603145330109](Rust Rebegin.assets/image-20250603145330109-17489336120715.png)

```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("Rust");
    greet_rust(&m1, &m2);
    println!("{} {}",m1, m2);
}

fn greet_rust(g1: &String, g2: &String) {
    println!("{} {}",g1,g2)
}
```

![image-20250603150403934](Rust Rebegin.assets/image-20250603150403934-17489342453227.png)

![image-20250604110125560](Rust Rebegin.assets/image-20250604110125560-17490060875801.png)

```rust
//eg1
fn main() {
    let x = 0;  
    let mut x_ref = &x; //此处mut可变表示x_ref可以借用其他变量，x本身是不可变的，所以x_ref也无法修改x的值0
}

//eg2

fn main() {
    let mut v: Vec<i32> = vec![1, 2, 3];
    let x = &v[2];

    v.push(4);      //这一句会报错，此时v处于被借用状态，v不具备写（W）权限
    //变量在被借用时，本身只有读（R）的权限，没有写（W）和所有权（O）

    println!("{}", x);
}
```

可变引用提供对数据“唯一的”且“非拥有的”访问
不可变引用（共享引用）：只读的
可变引用（独占引用）：在不移动数据的情况下，临时提供可变访问

![image-20250605104715186](Rust Rebegin.assets/image-20250605104715186-17490916363611.png)

**流动权限F** ：在表达式使用输入引用或返回输出引用时需要。
F权限在函数体内不会变化。
如果一个引用被允许在特定表达式中使用（即流动），那么它就具有流动权限。

```rust
fn main()
{

}
```

#### 修复所有权常见错误

```rust
//eg1
fn main() {
    let value = return_a_string();
    println!("{}", value);
}

fn return_a_string() -> &'static str {
    "Hello world" //这个字符串数据的声明周期从程序开始到程序结束
}

//eg2
use std::rc::Rc;
fn main() {
    let value = return_a_string();
    println!("{}", value);
}

fn return_a_string() -> Rc<String> {
    let s = Rc::new(String::from("Hello wprld"));
    Rc::clone(&s)
}

//eg3
fn main() {
    let mut s = String::from("Hello");
    return_a_string(&mut s);
    println!("{}", s);
}

fn return_a_string(output: &mut String) {
    output.replace_range(.., "Hello world");
}
```

#### 函数演示

```rust
//基础数据类型
//直接传参，与所有权没关系
fn change_num_1(mut num:i32)
{
    num *= 2;
    println!("after changed num : {}",num);
}


//引用   *为解引用符号，类似c中的解指针
fn change_num_2(num:&i32)
{
    //*num *= 2;  //不允许，因为是不可变引用
    println!("here can't change the num");
}

//可变借用
fn change_num_3(num:&mut i32)
{
    *num *= 2;
    println!("I can change it, num: {}", num);
}

//上述内容也可以说为值传递和引用传递
fn main()
{
    let mut m_num:i32 = 6;

    change_num_1(m_num);
    println!("after change_num_1 : {}",m_num);

    change_num_2(&m_num);
    println!("after change_num_2 : {}",m_num);

    change_num_3(&mut m_num);
    println!("after change_num_3 : {}",m_num);
}

//------------------------------------------------------------------
// 复合类型
// 下面的写法会报错，内容为最后又一行println!()中没有name的所有权
// 因为按照值传递的方法，符合类型name实际会将所有权传递给fn中的name_fn，mian中将不再拥有name的所有权
fn show_name(name_fn:String) {
    println!("in fn: {}",name_fn);
}

fn main()
{
    let name = String::from("Rust owner");

    show_name(name);

    println!("in main: {}", name);
}
//报错内容：
/*
PS D:\Rust\Projects\wgpu_gui_app\src> cargo run
   Compiling slint_demo v0.1.0 (D:\Rust\Projects\wgpu_gui_app)
error[E0382]: borrow of moved value: `name`
  --> src\main.rs:11:29
   |
 7 |     let name = String::from("Rust owner");
   |         ---- move occurs because `name` has type `String`, which does not implement the `Copy` trait
 8 |
 9 |     show_name(name);
   |               ---- value moved here
10 |
11 |     println!("in main: {}", name);
   |                             ^^^^ value borrowed here after move
   |
note: consider changing this parameter type in function `show_name` to borrow instead if owning the value isn't necessary
  --> src\main.rs:1:19
   |
 1 | fn show_name(name:String) {
   |    ---------      ^^^^^^ this parameter takes ownership of the value
   |    |
   |    in this function
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
   |
 9 |     show_name(name.clone());
   |                   ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `slint_demo` (bin "slint_demo") due to 1 previous error
*/

//修复方法：明确使用引用传递
fn show_name(name_fn:&String) {
    println!("in fn: {}",name_fn);
}

fn main()
{
    let name = String::from("Rust owner");

    show_name(&name);

    println!("in main: {}", name);
}
```



### IO





### 元编程

#### 函数宏（Declarative Macros）- macro_rules!





#### 过程宏（Procedural Macros）

##### 派生宏（Derive Macros）

###### 基本概念

```rust
#[derive(Debug)]  // 这是一个派生宏
struct MyStruct { ... }
```

- 派生宏是**过程宏**的一种

- 在**编译时**自动为结构体/枚举生成Trait的实现代码

- 属于**代码生成**技术

###### 工作原理

- 编译器读取结构体定义

- 派生宏分析结构体的字段

- 自动生成`impl Debug for MyStruct`的代码

###### 常用派生宏列表

| 派生宏                  | 生成的Trait | 用途     |
| ----------------------- | ----------- | -------- |
| `#[derive(Debug)]`      | Debug       | 调试输出 |
| `#[derive(Clone)]`      | Clone       | 显式复制 |
| `#[derive(Copy)]`       | Copy        | 隐式复制 |
| `#[derive(PartialEq)]`  | PartialEq   | 相等比较 |
| `#[derive(Eq)]`         | Eq          | 完全相等 |
| `#[derive(Hash)]`       | Hash        | 哈希计算 |
| `#[derive(Default)]`    | Default     | 默认值   |
| `#[derive(PartialOrd)]` | PartialOrd  | 偏序比较 |
| `#[derive(Ord)]`        | Ord         | 全序比较 |

###### 与手动实现对比

```rust
// 使用派生宏（推荐）
#[derive(Debug)]
struct Book { id: u32 }

// 手动实现（不推荐，代码冗长）
impl Debug for Book {
    fn fmt(&self, f: &mut Formatter<'_>) -> fmt::Result {
        write!(f, "Book {{ id: {} }}", self.id)
    }
}
```



###### 自定义派生宏（进阶）

```rust
// 使用第三方库创建自定义派生宏
use proc_macro::TokenStream;

#[proc_macro_derive(MyTrait)]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    // 生成代码的逻辑
}
```



##### 属性宏（Attribute Macros）



##### 函数式宏（Function-like Macros）



## 项目

### 单文件项目

#### 读取二进制文件，追加所有内容到文件末尾

```rust
// 内容复制
use std::env;
use std::fs::File;
use std::io::{Read, Write, Seek, SeekFrom};
use std::process;

fn main() {
    // 收集命令行参数
    let args: Vec<String> = env::args().collect();
    if args.len() < 2 {
        eprintln!("用法: {} <文件名> [重复次数]", args[0]);
        eprintln!("示例: {} data.dat 100", args[0]);
        process::exit(1);
    }

    let file_name = &args[1];
    let repeat_count: usize = args.get(2)
        .and_then(|s| s.parse().ok())
        .unwrap_or(100);  // 默认重复 100 次

    // 1. 读取原文件的全部内容
    let content = match File::open(file_name) {
        Ok(mut file) => {
            let mut buffer = Vec::new();
            if let Err(e) = file.read_to_end(&mut buffer) {
                eprintln!("读取文件 '{}' 失败: {}", file_name, e);
                process::exit(1);
            }
            buffer
        }
        Err(e) => {
            eprintln!("无法打开文件 '{}': {}", file_name, e);
            process::exit(1);
        }
    };

    // 如果文件为空，直接退出（避免无意义操作）
    if content.is_empty() {
        eprintln!("文件为空，无需追加。");
        return;
    }

    // 2. 以追加模式打开同一个文件
    let mut file = match File::options()
        .write(true)
        .append(true)
        .open(file_name)
    {
        Ok(f) => f,
        Err(e) => {
            eprintln!("无法以追加模式打开文件 '{}': {}", file_name, e);
            process::exit(1);
        }
    };

    // 3. 将内容重复写入文件尾
    for i in 0..repeat_count {
        if let Err(e) = file.write_all(&content) {
            eprintln!("第 {} 次写入失败: {}", i + 1, e);
            process::exit(1);
        }
        // 可选：每次写入后刷盘，但批量 flush 更高效，在循环结束后统一 flush
    }

    // 4. 确保所有数据写入磁盘
    if let Err(e) = file.flush() {
        eprintln!("刷新缓冲区失败: {}", e);
        process::exit(1);
    }

    println!("成功将文件内容重复追加 {} 次。", repeat_count);
}
```







## 抄书

### 面向加薪学习(http://www.go-edu.cn/)

#### 第1章 为什么要学习Rust

Rust 是一种快速、高并发、安全且具有授权性的编程语言，最初由 Graydon Hoare 于 2006  年创造和发布。现在它是一种开源语言，主要由 Mozilla 团队和许多开源社区成员共同维护和开发。它的目标是 C 和  C++占主导地位的系统编程领域。

**Rust 的目标**

Rust 语言瞄准的是工业系统的霸者 - C++ 语言。

**Rust 的优势**

Rust 是一门编译语言，因此它的效率可以媲美 C 或 C++ 语言。

由于没有 GC(垃圾回收机制)，所以是安全度极高的语言。

**Rust 可以做什么？**

1. 可以使用 Rust 编写操作系统、游戏引擎和许多性能关键型应用程序。
2. 可以使用它构建高性能的 Web 应用程序、网络服务，类型安全的数据库对象关系映射（Object Relational Mapping，ORM）库，还可以将程序编译成 WebAssembly 在 Web 浏览器上运行。
3. Rust 还在为嵌入式平台构建安全性优先的实时应用程序方面获得了相当大的关注，例如 Arm 基于 Cortex-M 的微控制器，目前该领域主要由 C 语言主导。Rust 因其广泛的适用性在多个领域都表现良好。

#### 第 2 章 Rust 环境搭建

在 Mac 环境下，利用 brew 包管理，十分方便。

1. brew upgrade

2. brew install rustup-init

3. 进入/opt/homebrew/Cellar/rustup-init/1.24.3/bin

4. 运行 rustup-init
   
   ```cmd
   1) Proceed with installation (default) //默认安装
   2) Customize installation //自定义安装
   3) Cancel installation    //取消安装
   
   我选的是第 1 个。
   ```

5. 屏幕出现成功字样 **Rust is installed now. Great!**

6. 最后执行这句，让环境变量生效 source $HOME/.cargo/env

7. 再打开终端
   
   ```cmd
   rustc -V
   屏幕输出  rustc 1.59.0
   ```

   出现上面的提示，证明你的 Rust 环境安装好了。

#### 第3章 hello_rust

第一个 Rust 程序。

1. 先创建一个目录 hello_rust。

2. 可以使用趁手的编辑（jetbrains 系列可以安装 rust 插件，或者是 vscode 的 rust 插件都可以）。

3. 创建 rust 源代码文件，它是用 **.rs** 作为扩展名的。

4. 在 hello.rs 中，输入
   
   ```rust
   fn main(){
       println!("Hello Rust!")
   }
   ```
   
   > fn - Rust 语言使用 `fn` 关键字定义函数。
   > 
   > `main()` 函数是一个预定义的主函数，充当 Rust 程序的入口点，每个语言都会有自己的 `main()` 函数。
   > 
   > println!() 是 Rust 语言中的一个 **预定义的宏**。它用于将传递给它的参数输出到 **标准输出**。
   
   注：Rust 语言中的 **宏** 都会以 **感叹号 ( `!` )** 结尾。以后看到以 `!` 结尾的类似函数调用，都是 **宏调用**。Rust 提供了一个功能非常强大的 **宏** 体系，通过这些 **宏**，我们可以很方便的进行 **元编程**。Rust 中的 **宏** 有点类似于 **函数**。可以将 **宏** 理解为 **函数的加强版**。

5. 使用 **rustc hello.rs**,编译出一个以 hello 为名字的二进制的可执行文件

6. 运行 ./hello,屏幕输出 Hello Rust!

#### 第 4 章 Rust 的数据类型

类型，我们先说一下现实中的菜系吧，
鲁菜、川菜、粤菜 、苏菜 、闽菜 、浙菜 、徽菜 、湘菜，都有不同的口味，当说到哪一个体系的菜的时候，你会知道它的特点，并且适合哪些人去吃。

那说回到计算机，数据类型也是一样的，就是存储和运算，并且要检查和保证这个数据在这个类型中是有效的。

Rust 是一个静态的严格数据类型的语言。每个值都有唯一的数据类型，要么是整型，要么是浮点型等等。

Rust 语言在赋值时并不强制要求指定变量的数据类型，Rust 编译器可以根据分配给它的值自动推断变量的数据类型。

**声明变量**

Rust 语言使用 `let` 关键字来声明和定义一个变量。

```
let 变量名=值
```

先粗略带过变量的声明，后面的章节我们会详细介绍。

```rust
fn main(){
    let food = "清蒸螃蟹";  // string 字符串类型
    let price = 366;       // float 类型
    let checked = true;   // boolean 类型

    println!("food is:{}", food); //输出 food is:清蒸螃蟹
    println!("price is:{}", price);//输出 price is:366
    println!("checked is :{}", checked);//输出 checked is :true
}
```

上面的代码中，我们并没有为每一个变量指定它们的数据类型。Rust 编译器会自动从 **等号 =** 右边的值中推断出该变量的类型。例如 Rust 会自动将 **双引号** 阔起来的数据推断为 **字符串**，把没有小数点的数字自动推断为 **整型**。把 `true` 或 `false` 值推断为 **布尔类型**。

**基本数据类型**

Rust 语言中有四种标量数据类型：

- 整型
- 浮点型
- 布尔类型
- 字符类型

#### 第 5 章 整型

最大值 std::u128::MAX，它的值是 340282366920938463463374607431768211455

最小值 std::i128::MIN，它的值是 -170141183460469231731687303715884105728

整数可以分为 **有符号整型** 和 **无符号整型**

- 有符号整型，英文 `signed`，既可以存储正数，也可以存储负数。
- 无符号整型，因为 `unsigned`，只能存储正数。

| 大小      | 有符号   | 无符号   |
| ------- | ----- | ----- |
| 8 bit   | i8    | u8    |
| 16 bit  | i16   | u16   |
| 32 bit  | i32   | u32   |
| 64 bit  | i64   | u64   |
| 128 bit | i128  | u128  |
| Arch    | isize | usize |

按照存储空间来说，整型可以进一步划分为 `1字节`、`2字节`、`4字节`、`8字节`、`16字节`。

1 字节 = 8 位，每一位能只能存储二进制 0 或 1，因此每一个字节能够存储的最大数字是 256，而最小数字则是 -127。

整型的长度还可以是 `arch`。`arch` 是由 CPU 构架决定的大小的整型类型。大小为 `arch` 的整数在 `x86` 机器上为 `32` 位，在 `x64` 机器上为 `64` 位。

**i32** 是默认的整型。

```rust
let price = 100;    // i32 默认
let price2:u32 = 200;
let price3:i32 = -300;
let price4:isize = 400;
let price5:usize = 500;

println!("price is {}", price);
//输出 price is 100

println!("price2 is {} and price3 is {}", price3, price2);
//输出 price2 is -300 and price3 is 200

println!("price4 is {} and price5 is {}", price4, price5);
//输出 price4 is 400 and price5 is 500
```

如果类型和值不匹配，编译不会通过，并且报错。

```rust
let price6:i32=66.66
编译器会提示：mismatched types [E0308] expected `i32`, found `f64`
```

**整型取值范围**

**有符号整型** 能够存储的最小值为 `-(2^(n-1)`，能够存储的最大值为 `2^(n-1) -1`。

**无符号整型** 能够存储的最小值为 `0`，能够存储的最大值为 `2^n - 1`。

其中 `n` 是指数据类型的大小。（上面表格里的第一列）

整型 `i8` ，能够存储的最小值为 `-(2^(8-1)) = -128`。最大值为 `(2^(8-1)-1) = 127`。

**整型溢出**

我们已经计算了 i8 的最大值是 127。我给一个更大的数值会如何呢？

```rust
let price7:i8=192;
println!("price7 is {}", price7);

报错如下：
16 |     let price7:i8=192;
 |                   ^^^
 |
 = note: `#[deny(overflowing_literals)]` on by default
 = note: the literal `192` does not fit into the type `i8` whose range is `-128..=127`
 = help: consider using the type `u8` instead
```

**很明确的告诉你超出了 i8 的范围**

#### 第 6 章 浮点型

按照存储大小，把浮点型划分为 `f32` 和 `f64`。其中 `f64` 是默认的浮点类型。

- `f32` 又称为 **单精度浮点型**。
- `f64` 又称为 **双精度浮点型**，它是 Rust 默认的浮点类型.

Rust 中不能将 `0.0` 赋值给任意一个整型，也不能将 `0` 赋值给任意一个浮点型。

```rust
let price8:f64 = 99;
报错：mismatched types [E0308] expected `f64`, found `i32`
let price9 = 18.00;        // 默认是 f64
let price10:f32 = 8.88;
let price11:f64 = 168.125;  // 双精度浮点型

println!("price9 {}", price9); //输出 price9 18
println!("price10 {}", price10);//输出 price10 8.88
println!("price11 {}", price11);//输出 price11 168.125
```

**_下划线**

当数字很大的时候，Rust 可以用 **(_下划线) ** ，来让数字变得可读性更好。

```rust
let price12 =1_000_000;
println!("price12 {}", price12); //输出 price12 1000000

let price13 =1_000_000.666_123;
println!("price13 {}", price13);//输出 price13 1000000.666123
```

#### 第7章 布尔类型

Rust 使用 bool 关键字来声明一个 布尔类型 的变量。
布尔类型 取值是 true 或 false 。

```rust
let checked:bool = true;
println!("checked {}", checked);//输出 checked true
```

配置vscode进行rust debug,在.vscode/launch.json中复制如下:

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
         {
      "type": "lldb",
      "request": "launch",
      "name": "Custom launch",
      "program": "${workspaceRoot}/bool07/target/debug/bool07",
      "cwd": "${workspaceRoot}",
    },
    ]
}
```

#### 第 8 章 字符类型

1. **字符(char)** ，就是字符串的基本组成部分，也就是单个字符或字。
2. **Rust 使用 UTF-8 作为底层的编码** ，而不是常见的使用 ASCII 作为底层编码。
3. Rust 中的 **字符数据类型** 包含了 **数字**、**字母**、**Unicode** 和 **其它特殊字符**。

```rust
let c = 'R';
println!("c {}", c);//输出 c R
```

#### 第 9 章 定义变量

在计算机里硬件有内存条，当通电以后，会被加载到操作系统中，我们可以认为这是一个大酒店，但是它内部是要有房间的，每个房间都有自己的位置，有自己的名称，这样管理的好处是方便，直接，所以计算机也采用了这样的模式。

**声明变量**

```rust
let 变量名 = 值;           // 不指定变量类型
let 变量名:数据类型 = 值;   // 指定变量类型
```

**变量** 就是给 **某一块内存地址** 起名字。比如: s=内存地址 1、s2=内存地址 2 。

前面说过 **变量是有数据类型的**，所以 **内存存储的数据也是有数据类型的**。

**变量的命名规范**

1. 可以包含 **字母**、**数字** 和 **下划线** 。
2. 变量名必须以 **字母** 或 **下划线** 开头。不能以 **数字** 开头。
3. 变量名是 **区分大小** 写的。也就是大写的 Study 和小写的 study 是两个不同的变量。

```rust
fn main() {
    let Study = "";
    print!("{}",study)
}

报错如下：
    print!("{}",study)
    ^^^^^ help: a local variable with a similar name exists (notice the capitalization): `Study`
```

**let 关键字-不可变变量**

Rust 语言中使用 `let` 声明的变量，在第一次赋值之后，是不可变更不可重新赋值的，变成了 **只读** 状态。默认情况下，Rust 语言中的变量是不可变的。

```rust
let price = 188;
price=288;
print!("{}",price);

编译器报错提示：Cannot assign twice to immutable variable [E0384]
```

**mut 关键字-可变变量**

Rust 语言提供了 `mut` 关键字表示 **可变的**。 在变量名的前面加上 `mut` 关键字告诉编译器这个变量是可以重新赋值的。

```rust
let mut 变量名 = 值;
let mut 变量名:数据类型 = 值;
```

修改上面的例子

```rust
let mut price = 188;
price=288;
print!("{}",price); //输出 288
```

#### 第10章 常量

**常量** 就是那些值不能被改变的变量。定义后，再也没有任何方法可以改变常量的值。

- const：不可改变的值（通常使用这种）。
- static：具有 ‘static 生命周期的，可以是可变的变量（须使用 static mut 关键字）。

> 有个特例就是 “string” 字面量。它可以不经改动就被赋给一个 static 变量，因为它 的类型标记：&’static str 就包含了所要求的生命周期 ‘static。其他的引用类型都 必须特地声明，使之拥有’static 生命周期。

**常量定义**

```
const 常量名称:数据类型=值;
```

1. 使用 const 关键字定义常量。
2. 定义常量时必须指定数据类型。
3. 常量名称的命名规则和之前变量的命名规则一样，但常量名称一般都是 **大写字母**。

```rust
fn main() {
    const PI:f64=3.1415926;
    println!("{}",PI);//输出 3.1415926
}
```

**常量** 可以在任意作用域里定义，包括全局作用域。也就是可以在任何地方定义。**常量** 只是一个符号，会在 **编译时** 替换为具体的值。

**变量的隐藏**

Rust 语言中允许重复定义一个相同变量名的变量。规则是 **后面定义的变量会隐藏** 前面定义的同名变量。

```rust
let name ="《Go语言极简一本通》";
let name="《从0到Go语言微服务架构师》";
println!("{}",name); //输出 《从0到Go语言微服务架构师》
```

我们定义了两个同名的变量 `name`，第一次赋值 `《Go语言极简一本通》` ，第二次 赋值为`《从0到Go语言微服务架构师》`。第二个 `name` 隐藏了第一次定义的变量。

**隐藏变量后并且改变了数据类型**

下面的例子：

```rust
let price=199;
let price="299";
println!("{}",price);//输出 299
```

**同名常量**

Rust 中，**常量不能被隐藏，也不能被重复定义**。

```rust
const DISCOUNT:f64=0.8;
const DISCOUNT:f64=0.6;

编辑器报错
A value named `DISCOUNT` has already been defined in this block [E0428]
```

**static**

```rust
static BOOK: &'static str = "《Go语言极简一本通》";
```

#### 第11章 字符串

Rust 语言提供了两种字符串

- Rust 核心内置的数据类型`&str`，字符串字面量 。
- Rust 标准库中的一个 **公开 `pub`** 结构体。字符串对象 `String`。

**字面量&str**

字符串字面量的核心代码可以在模块 `std::str` 中找到。Rust 中的字符串字面量被称之为 **字符串切片**。因为它的底层实现是 **切片**。

```rust
let lesson="Go语言微服务架构核心22讲";
```

字面量 `&str` 就是在 **编译时** 就知道其值的字符串类型，它也是字符的集合，被硬编码赋值给一个变量。

字符串字面量模式是 **静态** 的，所以，字符串字面量从创建时开始会一直保存到程序结束。

**字符串对象**

字符串对象并不是 Rust 核心内置的数据类型，它只是标准库中的一个 **公开 `pub`** 的结构体。

```rust
pub struct String
```

**字符串对象是使用 `UTF-8` 作为底层数据编码格式,长度可变的集合**。

字符串对象在 **堆 `heap`** 中分配，可以在运行时提供字符串值以及相应的操作方法。

**新建字符串对象**

```rust
String::new() //创建一个新的空字符串,它是静态方法。
String::from() //从具体的字符串字面量创建字符串对象。
```

例子

```rust
let s1 = String::new();
println!("s1:{},s1-len:{}",s1,s1.len());
//输出 s1:,s1-len:0

let s2 = String::from("面向加薪学习");
println!("s2:{},s2-len:{}",s2,s2.len());
//输出 s2:面向加薪学习,s2-len:18
```

**字符串对象常用方法**

**new()**

创建一个新的字符串对象

**push_str()方法**

再字符串末尾追加字符串。

```rust
let mut s3 = String::new();
s3.push_str("Go语言极简一本通");
println!("{}",s3); //输出 Go语言极简一本通
```

**push()方法**

是在原字符上追加字符，而不是返回一个新的字符串

```rust
s3.push('O');
s3.push('K');
println!("{}",s3);//输出 Go语言极简一本通OK
```

**replace()**

指定字符串子串替换成另一个字符串

```rust
let s4 = String::from("面向加薪学习");
let result = s4.replace("面向加薪学习","www.go-edu.cn");
println!("{}",result);//输出 www.go-edu.cn
```

**len() 获取长度**

返回字符串中的 **总字节数**。该方法会统计包括 **制表符 `\t`\**、\**空格 ``\**、\**回车 `\r`\**、\**换行 `\n`** 和回车换行 `\r\n` 等等。

```rust
let s5 = String::from("面向加薪学习 www.go-edu.cn\n");
println!("length is {}",s5.len());//输出 33
```

**to_string()**

将字符串转换为字符串对象，方便以后可以有更多的操作。

```rust
let s6 = "从0到Go语言微服务架构师".to_string();
println!("{}",s6);//输出 从0到Go语言微服务架构师
```

**as_str()**

返回一个字符串对象的 **字符串** 字面量。

```rust
fn show_name(name:&str){
    println!("充电科目:{}",name);
}

let s7 = String::from("Go语言微服务架构核心22讲");
show_name(s7.as_str()); //输出 充电科目:Go语言微服务架构核心22讲
```

**trim()**

去除字符串头尾的空白符。空白符是指 **制表符 `\t`\**、\**空格 ``\**、\**回车 `\r`\**、\**换行 `\n`** 和回车换行 `\r\n` 等等。

```rust
let s8 = " \tGo语言极简一本通\tGo语言微服务架构核心22讲 \r\n从0到Go语言微服务架构师\r\n     ";
println!("length is {}",s8.len());//输出 length is 103
println!("length is {}",s8.trim().len());//输出 length is 94
println!("s8:{}",s8);
//输出
s8:     Go语言极简一本通        Go语言微服务架构核心22讲
从0到Go语言微服务架构师
```

**split()**

将字符串根据某些指定的 **字符串子串** 分割，返回分割后的字符串子串组成的切片上的迭代器。

```rust
let s9 = "Go语言极简一本通、Go语言微服务架构核心22讲、从0到Go语言微服务架构师";
for item in s9.split('、'){
   println!("充电科目: {}",item);
}
//输出
//充电科目: Go语言极简一本通
//充电科目: Go语言微服务架构核心22讲
//充电科目: 从0到Go语言微服务架构师
```

**chars()**

将一个字符串打散为所有字符组成的数组

```rust
let s10 = "从0到Go语言微服务架构师";
for c in s10.chars(){
   println!("字符: {}",c);
}
//输出
字符: 从
字符: 0
字符: 到
字符: G
字符: o
字符: 语
字符: 言
字符: 微
字符: 服
字符: 务
字符: 架
字符: 构
字符: 师
```

**字符串拼接 +**

**+** 的内部实现是重写了 `add()` 方法。

```rust
add(self,&str)->String {
}
```

返回新的字符串对象。

```rust
let s11="Go语言极简一本通".to_string();
let s12 = " 欢喜".to_string();
let result2=s11 + &s12;
println!("{}",result2);
//输出 Go语言极简一本通 欢喜
```

#### 第12章 运算符

Rust 语言支持以下四种运算符

- 算术运算符
- 位运算符
- 关系运算符
- 逻辑运算符

算术运算符

| 名称  | 运算符 |
| --- | --- |
| 加   | +   |
| 减   | -   |
| 乘   | *   |
| 除   | /   |
| 求余  | %   |

> **注：Rust 语言不支持自增自减运算符 `++` 和 `--`**

**关系运算符**

| 名称   | 运算符 | 说明                                 |
| ---- | --- | ---------------------------------- |
| 大于   | >   | 如果左操作数大于右操作数则返回 true 否则返回 false    |
| 小于   | <   | 如果左操作数小于于右操作数则返回 true 否则返回 false   |
| 大于等于 | >=  | 如果左操作数大于或等于右操作数则返回 true 否则返回 false |
| 小于等于 | <=  | 如果左操作数小于或等于右操作数则返回 true 否则返回 false |
| 等于   | ==  | 如果左操作数等于右操作数则返回 true 否则返回 false    |
| 不等于  | !=  | 如果左操作数不等于右操作数则返回 true 否则返回 false   |

**逻辑运算符**

| 名称  | 运算符  | 说明                                 |
| --- | ---- | ---------------------------------- |
| 逻辑与 | &&   | 两边的条件表达式都为真则返回 true 否则返回 false     |
| 逻辑或 | \|\| | 两边的条件表达式只要有一个为真则返回 true 否则返回 false |
| 逻辑非 | !    | 如果表达式为真则返回 false 否则返回 true         |

**位运算**

| 名字  | 运算符 | 说明                       |
| --- | --- | ------------------------ |
| 位与  | &   | 相同位都是 1 则返回 1 否则返回 0     |
| 位或  | \|  | 相同位只要有一个是 1 则返回 1 否则返回 0 |
| 异或  | ^   | 相同位不相同则返回 1 否则返回 0       |
| 位非  | !   | 把位中的 1 换成 0 ， 0 换成 1     |
| 左移  | <<  | 操作数中的所有位向左移动指定位数，右边的位补 0 |
| 右移  | >>  | 操作数中的所有位向右移动指定位数，左边的位补 0 |

#### 第13章 条件判断

| 条件判断语句                 | 说明                                      |
| ---------------------- | --------------------------------------- |
| `if` 语句                | `if` 语句用于模拟现实生活中的 **如果…就…**             |
| `if...else` 语句         | `if...else` 语句用于模拟 **如果…就…否则…**         |
| `else...if` 和嵌套`if` 语句 | 嵌套`if` 语句用于模拟 **如果…就…如果…就…**            |
| `match` 语句             | `match` 语句用于模拟现实生活中的 **老师点名** 或 **银行叫** |

**if 语句**

```rust
if 条件表达式 {
    // 条件表达式为true时要执行的逻辑
}

let total:f32=666.00;
if total>500.00{
   println!("打8折,{}",total*0.8)
}
//输出 打8折,532.8
```

**if …else 语句**

```rust
if 条件表达式 {
   // 如果 条件表达式 为真则执行这里的代码
} else {
   // 如果 条件表达式 为假则执行这里的代码
}

let total:f32=166.00;
if total>500.00{
   println!("打8折,{}",total*0.8)
}else{
   println!("无折扣优惠,{}",total)
}
输出 无折扣优惠,166
```

**if…else if… else 语句**

```rust
if 条件表达式1 {
   // 当 条件表达式1 为 true 时要执行的语句
} else if 条件表达式2 {
   // 当 条件表达式2 为 true 时要执行的语句
} else {
   // 如果 条件表达式1 和 条件表达式2 都为 false 时要执行的语句
}

let total:f32=366.00;
if total>200.00 && total<500.00{
   println!("打9折,{}",total*0.9)
}else if total>500.00{
   println!("打8折,{}",total*0.9)
} else{
   println!("无折扣优惠,{}",total)
}
//输出 打9折,329.4
```

**match 语句**

Rust 中的 `match` 语句有返回值，它把 **匹配值** 后执行的最后一条语句的结果当作返回值。

语法

```rust
match variable_expression {
   constant_expr1 => {
      // 语句;
   },
   constant_expr2 => {
      // 语句;
   },
   _ => {
      // 默认
      // 其它语句
   }
};

let code = "10010";
let choose = match code {
   "10010" => "联通",
   "10086" => "移动",
   _ => "Unknown"
};
println!("选择 {}", choose);
//输出 选择 联通


let code = "80010";
let choose = match code {
   "10010" => "联通",
   "10086" => "移动",
   _ => "Unknown"
};
println!("选择 {}", choose);
//输出 选择 Unknown
```

#### 第14章 循环

现实中的循环很多，比如我们在学校操场里跑步，一圈一圈的跑。在计算机中，**循环** 其实就是一种重复，在满足指定的条件下，重复的做某些事情。

Rust 语言中也有三种表示 **循环** 的语句：

- `loop` 语句。一种重复执行且永远不会结束的循环。
- `while` 语句。一种在某些条件为真的情况下就会永远执行下去的循环。
- `for` 语句。一种有确定次数的循环。

**for 循环**

```rust
for 临时变量 in 左区间..右区间 {
   // 执行业务逻辑
}
```

**左区间..右区间**，这是一个左闭右开的区间，1..5，那就只包含 1,2,3,4

```rust
for num in 1..5{
   println!("num is {}", num);
}
//输出
num is 1
num is 2
num is 3
num is 4
```

可以使用 a..=b 表示两端都包含在内的范围。

```rust
for num in 1..=5 {
   println!("num is {}", num);
}
输出
num is 1
num is 2
num is 3
num is 4
num is 5
```

**for 与迭代器**

iter - 在每次迭代中借用集合中的一个元素。这样集合本身不会被改变，循环之后仍可以使用。

```rust
 let studyList = vec![
        "《Go语言极简一本通》",
        "Go语言微服务架构核心22讲",
        "从0到Go语言微服务架构师",
    ];
    for name in studyList.iter() {
        match name {
            &"从0到Go语言微服务架构师" => println!("恭喜你进阶到第三阶段-{}!", name),
            _ => println!("学习: {}", name),
        }
    }
//输出
学习: 《Go语言极简一本通》
学习: Go语言微服务架构核心22讲
恭喜你进阶到第三阶段-从0到Go语言微服务架构师!
```

into_iter - 会消耗集合。在每次迭代中，集合中的数据本身会被提供。一旦集合被消耗了，之后就无法再使用了，因为它已经在循环中被 “移除”（move）了。

```rust
 let studyList2 = vec![
        "《Go语言极简一本通》",
        "Go语言微服务架构核心22讲",
        "从0到Go语言微服务架构师",
    ];
    for name in studyList2.into_iter() {
        match name {
            "从0到Go语言微服务架构师" => println!("恭喜你进阶到第三阶段-{}!", name),
            _ => println!("学习: {}", name),
        }
    }
输出
学习: 《Go语言极简一本通》
学习: Go语言微服务架构核心22讲
恭喜你进阶到第三阶段-从0到Go语言微服务架构师!
```

iter_mut - 可变地（mutably）借用集合中的每个元素，从而允许集合被就地修改。
就是停止本次执行剩下的语句，直接进入下一个循环。

```rust
let mut studyList3 = vec![
        "《Go语言极简一本通》",
        "Go语言微服务架构核心22讲",
        "从0到Go语言微服务架构师",
    ];
    for name in studyList3.iter_mut() {
        *name = match name {
            &mut "从0到Go语言微服务架构师" => {
                "恭喜你进阶到第三阶段---从0到Go语言微服务架构师"
            }
            _ => *name,
        }
    }
    println!("studyList3: {:?}", studyList3);
    输出
    studyList3: ["《Go语言极简一本通》", "Go语言微服务架构核心22讲", "恭喜你进阶到第三阶段---从0到Go语言微服务架构师"]
```

**while 循环**

```rust
while ( 条件表达式 ) {
    // 执行业务逻辑
}
```

上面的条件表达式为真，就会执行 while 循环。

```rust
 let mut num = 1;
 while num < 20{
    println!("num is {}",num);
    num= num*2;
}
//输出
num is 1
num is 2
num is 4
num is 8
num is 16
```

**loop 循环+break**

```rust
loop {
    // 执行业务逻辑
}
break; 中断的意思，就是跳出loop循环

let mut num = 1;
loop {
   if num > 20{
      break;
   }
   println!("num is {}",num);
   num= num*3;
}
//输出
num is 1
num is 3
num is 9
```

#### 第15章 函数

函数是一组一起执行一个任务的语句块。每个 Rust 程序都至少有一个函数，即主函数 `main()`。划分的标准是每个函数执行一个单一的任务。这也是软件设计中经常说的 单一职责。这会让你的代码可读性更好。

**函数的定义**

定义函数时必须以 `fn` 关键字开头，`fn` 关键字是 `function` 的缩写。

函数名称的命名规则和变量的命名规则一致。

```rust
fn 函数名称([参数:数据类型]) -> 返回值 {
   // 函数代码
}
```

- 参数用于将值传递给函数内部的语句。参数是可选的。
- 一个 “不” 返回值的函数。实际上会返回一个单元类型 `()`。当函数返回 `()` 时，函数签名可以省略返回类型。

> 函数（function）使用 fn 关键字来声明。函数的参数需要标注类型，就和变量一样，如果函数返回一个值，返回类型必须在箭头 -> 之后指定。函数最后的表达式将作为返回值。也可以在函数内使用 return 语句来提前返一个值，甚至可以在循环或 if 内部使用。

```rust
fn hello(){
    println!("Hello, rust!");
}
```

**函数调用**

函数需要调用才会被执行，否则就是没用的，多余的代码。

语法

```rust
fn 函数名称([参数:数据类型]) 返回值{
    //函数体
}
```

如果函数定义没有参数，那么参数是可以省略的。

```rust
fn main() {
    hello();
}

//输出 Hello, rust!
```

在 main()函数中调用 hello()函数。

**函数返回值**

函数在代码执行完成后，除了将控制权还给调用者之外，还可以携带值给它的调用者。函数可以返回值给它的调用者。称为 **函数返回值**。

Rust 语言的返回值定义语法，在 **小括号后面使用 箭头 ( `->` ) 加上数据类型** 来定义的。

**有 return**

```rust
fn 函数名称() -> 返回类型 {
   return 返回值;
}
```

**没有 return**

如果函数代码中没有使用 `return` 关键字，那么函数会默认使用最后一条语句的执行结果作为返回值。

```rust
fn 函数名称() -> 返回类型 {
   // 业务逻辑
   返回值 // 没有分号则表示返回值
}
```

**注:最后一条语句的执行结果,必须和函数定义时的返回数据类型一样，不然会编译会出错** 。

```rust
fn get_name() -> String {
    return String::from("Go语言微服务架构核心22讲");
}

fn get_name2() -> String {
    String::from("从0到Go语言微服务架构师")
}

fn main() {
    println!("r1:{}", get_name()); //输出 r1:Go语言微服务架构核心22讲
    println!("r2:{}", get_name2());//输出 r2:从0到Go语言微服务架构师
}
```

**函数参数**

**函数参数** 是一种将外部变量和值带给函数内部代码的一种机制。函数定义时指定的参数名叫做 **形参**。同时把调用函数时传递给函数的值叫做 **实参**。传递的 **实参** 数量与 **形参** 数量和类型必须相同。

**参数-值传递**

**值传递** 是把传递的变量的值传递给函数的 **形参**，所以，函数体外的变量值和函数参数是各自保存了相同的值，互不影响。因此函数内部修改函数参数的值并不会影响外部变量的值。

```rust
fn double_price(mut price:i32){
    price=price*2;
    println!("内部的price是{}",price)
}

fn main() {
    let mut price=99;
    double_price(price); //输出 内部的price是198
    println!("外部的price是{}",price); //输出 外部的price是99
}
```

**参数-引用传递**

值传递变量导致重新创建一个变量。但引用传递则不会，引用传递把当前变量的内存位置传递给函数。传递的变量和函数参数都共同指向了同一个内存位置。引用传递在参数类型的前面加上 `&` 符号。

```rust
fn 函数名称(参数: &数据类型) {
   // 执行逻辑代码
}

fn double_price2(price:&mut i32){
    *price=*price*2;
    println!("内部的price是{}",price)
}

fn main() {
    let mut price=88;
    double_price2(&mut price); //输出 内部的price是176
    println!("外部的price是{}",price);//输出 外部的price是176
}
```

**星号（`\*`）** 用于访问变量 `price` 指向的内存位置上存储的变量的值。这种操作也称为 **解引用**。 因此 **星号（`\*`）** 也称为 **解引用操作符**。

**复合类型传参**

对于复合类型，比如字符串，如果按照普通的方法传递给函数后，那么该变量将不可再访问。

```rust
fn show_name(name:String){
    println!("充电科目 :{}",name);
}

fn main() {
    let name:String = String::from("从0到Go语言微服务架构师");
    show_name(name);
    println!("调用show_name函数后: {}",name);
}

报错如下
error[E0382]: borrow of moved value: `name`
let name:String = String::from("从0到Go语言微服务架构师");
  |---- move occurs because `name` has type `String`, which does not implement the `Copy` trait
  |show_name(name);
  |  ---- value moved here
  |println!("调用show_name函数后: {}",name);
  |  ^^^^ value borrowed here after move
```

#### 第18章 所有权

因为变量要负责释放它们拥有的资源，所以**资源只能拥有一个所有者**。这也防止了资源的重复释放。注意并非所有变量都拥有资源（例如引用）。

在进行赋值（let a = b）或通过值来传递函数参数（foo(a)）的时候，资源的所有权（ownership）会发生转移。按照 Rust 的规范，这被称为资源的移动（move）。

在移动资源之后，原来的所有者不能再被使用，这可避免悬挂指针（dangling pointer）的产生。

内存分为两大类：**栈(stack)** 和 **堆(heap)**

**栈**

它是一种 **后进先出** 的机制，类似我们日常的落盘子，只能一个一个向上方，然后从最上面拿一个盘子。一个变量要放到栈上，那么它的大小在编译时就要明确。i32 
类型的变量，它就占用 4 个字节。Rust 
中可以放到栈上的数据类型，他们的大小都是固定的。如果是字符串，在运行时才会赋值的变量，在编译期的时候大小是未知或不确定的。所以字符串类型存储在**堆**上。

**堆**

用于编译时大小未知或不确定的，只有运行时才能确定的数据。在**堆**上存储一些动态类型的数据。**堆**是不受系统管理的，是用户自己管理的，也增加了内存溢出的风险。

**所有权**

所有权就是值一个东西归属谁。Rust 中一个变量对应一个值，变量就称为这个值得**所有者**。

let name="从0到Go语言微服务架构师";  

这句话的意思就是，”从 0 到 Go 语言微服务架构师” 这个值所在内存块由变量 name 所有。

Rust 中，只能由一个所有者，不允许两个同时指向同一块内存区域。变量必须指向不同的内存区域。

**转让所有权**

类似我们人类把一个东西送人或丢弃。

以下几种方式转让所有权：

1. 把一个变量赋值给另一个变量。

     

```rust
fn main() {  
 // 栈分配的整型  
 let a = 88;  
 // 将 `a` *复制*到 `b`——不存在资源移动  
 let b = a;  
 // 两个值各自都可以使用  
 println!("a {}, and b {}", a, b);

let v1 = vec!["Go语言极简一本通","Go语言微服务架构核心22讲","从0到Go语言微服务架构师"];  
let v2 =v1;  
println!("{:?}",v1); 
```

- v1 拥有堆上数据的所有权。（每次只能有一个变量对堆上数据有所有权）

- v2=v1 v2 拥有了堆上数据的所有权。

- v1 已经没有对数据的所有权了，所以再使用 v1 会报错。

- 如果 Rust 检查到 2 个变量同时拥有堆上内存的所有权。会报错如上。
2. 把变量传递给函数参数。

```rust
fn show(v:Vec<&str>){  
    println!("v {:?}",v)  
}  
fn main() {  
    let studyList = vec!["Go语言极简一本通","Go语言微服务架构核心22讲","从0到Go语言微服务架构师"];  
    //studyList 拥有堆上数据管理权  
    let studyList2 = studyList;  
    // studyList 将所有权转义给了 studyList2  
    show(studyList2);  
    // studyList2 将所有权转让给参数 v,studyList2 不再可用。  
    println!("studyList2 {:?}",studyList2);  
    //studyList2 已经不可用。  
}  

error[E0382]: borrow of moved value: `studyList2`  
| let studyList2 = studyList; // studyList 将所有权转义给了 studyList2  
| ---------- move occurs because `studyList2` has type `Vec<&str>`, which does not implement the `Copy` trait  
| show(studyList2); // studyList2 将所有权转让给参数 v,studyList2 不再可用。  
| ---------- value moved here  
| println!("studyList2 {:?}",studyList2);//studyList2 已经不可用。  
| ^^^^^^^^^^ value borrowed here after move  
```



3. 函数中的返回值。

```rust
fn show2(v:Vec<&str>) -> Vec<&str>{  
    println!("v {:?}",v);  
    return v;  
}  

fn main() {  
    let studyList3 = vec!["Go语言极简一本通","Go语言微服务架构核心22讲","从0到Go语言微服务架构师"];  
    let studyList4 = studyList3;  
    let result = show2(studyList4);  
    println!("result {:?}",result);  
    //输出 result ["Go语言极简一本通", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]  
}  
```

**基础数据类型与所有权**

所有权只会发生在堆上分配的数据，基础数据类型(整型，浮点型，布尔，字符)存储在栈上，所以没有所有权的概念。基础类型可以认为是值拷贝，在内存上另外的地方，存储和复制来的数据，然后让新的变量指向它。

let a = 88;  
let b = a;  
println!("a {}, and b {}", a, b);  

**总结**
赋值并不是唯一涉及移动的操作。值在作为参数传递或从函数返回时也会被移动。



#### 第19章 Borrowing 借用所有权

生活中，我们对工具有所有权，但是也不妨碍我们可以把工具借给别人甚至租用给别人，别人用完了，要还给你的。

Rust 中，**Borrowing（借用）**，就是一个函数中的变量传递给另外一个函数作为参数暂时使用。也会要求函数参数离开自己作用域的时候将**所有权** 还给当初传递给它的变量（好借好还，再借不难嘛!）。

```
&变量名  //要把参数定义的时候这样定义。
```

例子如下:

```rust
fn show(v:&Vec<&str>){
    println!("v:{:?}",v)
}
fn main() {
    let studyList = vec!["Go语言极简一本通","Go语言微服务架构核心22讲","从0到Go语言微服务架构师"];
    let studyList2 =studyList;
    show2(&studyList2);
    println!("studyList2:{:?}",studyList2); //我们看到，函数show使用完v2后，我们仍然可以继续使用
}
//输出
v:["Go语言极简一本通", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
v2:["Go语言极简一本通", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
```

**可变的借用**

上面我们的例子可以说都是只读的，我们看下面:

```rust
fn show2(v:&Vec<&str>){
    v[0]="第一个充电项目已完成";
    println!("v:{:?}",v)
}
报错如下:
error[E0596]: cannot borrow `*v` as mutable, as it is behind a `&` reference
| fn show2(v:&Vec<&str>){
| ---------- help: consider changing this to be a mutable reference: `&mut Vec<&str>`
| v[0]="第一个充电项目已完成";
| ^ `v` is a `&` reference, so the data it refers to cannot be borrowed as mutable
```

报错的原因：我们的这个借用不可以是可变的。那么 Rust 中，如果想要让一个变量是可变的，只能在前面加上 **mut** 关键字。

修改如下:

```rust
fn show2(v:&mut Vec<&str>){
    v[0]="第一个充电项目已完成";
    println!("v:{:?}",v)
}
fn main() {
 let mut studyList3 = vec!["Go语言极简一本通","Go语言微服务架构核心22讲","从0到Go语言微服务架构师"];
 println!("studyList3:{:?}",studyList3);
 show2(&mut studyList3);
 println!("调用后，studyList3:{:?}",studyList3);
 }
 //输出
studyList3:["Go语言极简一本通", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
v:["第一阶段学习已完成", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
调用后，studyList3:["第一阶段学习已完成", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
```

如果我们要在**Borrowing（借用）**的时候改变其中的值：

1. 变量要用**mut**关键字。
2. 函数参数为可变的要用 **&mut** 关键字。
3. 传递参数的时候，也要用 **&mut** 关键字。



#### 第20章 Slice(切片)

切片是只向一段连续内存的指针。在 Rust 中，连续内存够区间存储的数据结构：数组(array)、字符串(string)、向量(vector)。切片可以和它们一起使用，切片也使用数字索引访问数据。下标索引从**0**开始。slice 可以指向数组的一部分，越界的下标会引发致命错误（panic）。

**切片**是运行时才能确定的，并不像数组那样编译时就已经确定了。

**切片的定义**

```rust
let 切片值 = &变量[起始位置..结束位置]
```

1. [起始位置..结束位置]，这是一个左闭右开的区间。
2. **起始位置**最小值是**0**。
3. **结束位置**是数组、向量、字符串的长度。

```rust
fn main() {
   let mut v = Vec::new();
   v.push("Go语言极简一本通");
   v.push("Go语言微服务架构核心22讲");
   v.push("从0到Go语言微服务架构师");
   println!("len:{:?}",v.len());
   let s1=&v[1..3];
   println!("s1:{:?}",s1);
}
//输出
len:3
s1:["Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
```

**切片当参数**

切片通过引用的方式传递给函数。

```rust
fn show_slice(s:&[&str]){
   println!("show_slice函数内:{:?}",s);
}
show_slice(s1);//把上面的s1传递给函数show_slice
//输出 show_slice函数内:["Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
```

**可变切片**

如果我们声明的原数据是可变的，同时定义切片也有**&mut**关键字，就可以更改切片元素来更改元数据了。

```rust
fn modify_slice(s: &mut [&str]) {
   s[0] = "这个阶段已学习完毕";
   println!("modify_slice:{:?}", s);
}
let mut v2 = Vec::new();
v2.push("Go语言极简一本通");
v2.push("Go语言微服务架构核心22讲");
v2.push("从0到Go语言微服务架构师");
println!("modify_slice之前:{:?}", v2);

modify_slice(&mut v2[1..3]);
println!("modify_slice之后:{:?}", v2);

//输出
modify_slice之前:["Go语言极简一本通", "Go语言微服务架构核心22讲", "从0到Go语言微服务架构师"]
modify_slice:["这个阶段已学习完毕", "从0到Go语言微服务架构师"]
modify_slice之后:["Go语言极简一本通", "这个阶段已学习完毕", "从0到Go语言微服务架构师"]
```



#### 第21章 结构体(Struct)

**结构体（ `struct` ）**可以由各种不同类型组成。使用 struct 关键字来创建。**struct** 是 **structure** 的缩写。结构体可以作为另一个结构体的字段。结构体是可以嵌套的。

- 元组结构体（tuple struct），事实上就是具名元组而已。

  ```tust
  struct Pair(String, i32);
  ```

- 经典的 C 语言风格结构体（C struct）。

  ```rust
  struct 结构体名称 {
      ...
  }
  ```

- 单元结构体（unit struct），不带字段，在泛型中很有用。

  ```rust
  struct Unit;
  ```

**定义结构体**

```rust
struct 结构体名称 {
    字段1:数据类型,
    字段2:数据类型,
    ...
}
```

**创建结构体实例**

```rust
let 实例名称 = 结构体名称{
    field1:value1,
    field2:value2
    ...
};
```

结构体初始化，其实就是对 **结构体中的各个元素进行赋值**。

```rust
#[derive(Debug)]
struct Study {
    name: String,
    target: String,
    spend: i32,
}

fn main() {
    let s = Study {
        name: String::from("从0到Go语言微服务架构师"),
        target: String::from("全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。"),
        spend: 3,
    };
    println!("{:?}", s);
    //输出 Study { name: "从0到Go语言微服务架构师", target: "全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。", spend: 3 }
}
```

**访问实例属性**

```rust
实例名称.属性

println!("{}",s.name);//输出 从0到Go语言微服务架构师
```

**修改结构体实例**

结构体实例默认是**不可修改**的，如果想修改结构体实例，声明时使用**mut**关键字。

```rust
let mut s2 = Study {
        name: String::from("从0到Go语言微服务架构师"),
        target: String::from("全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。"),
        spend: 3,
    };
 s2.spend=7;
 println!("{:?}",s2);//输出 Study { name: "从0到Go语言微服务架构师", target: "全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。", spend: 7 }
```

**结构体做参数**

```rust
fn show(s: Study) {
    println!(
        "name is :{} target is {} spend is{}",
        s.name, s.target, s.spend
    );
}
```

**结构体作为返回值**

```rust
fn get_instance(name: String, target: String, spend: i32) -> Study {
    return Study {
        name,
        target,
        spend,
    };
}

let s3 = get_instance(
    String::from("Go语言极简一本通"),
    String::from("学习Go语言语法，并且完成一个单体服务"),
    5,
);
println!("{:?}", s3);
//输出 Study { name: "Go语言极简一本通", target: "学习Go语言语法，并且完成一个单体服务", spend: 5 }
```

**方法**

方法（method）是依附于对象的函数。这些方法通过关键字 self 来访问对象中的数据和其他。方法在 impl 代码块中定义。

> 与函数的区别
>
> 函数：可以直接调用，同一个程序不能出现 2 个相同的函数签名的函数，应为函数不归属任何 owner。
>
> 方法：归属某一个 owner，不同的 owner 可以有相同的方法签名。

```rust
impl 结构体{
    fn 方法名(&self,参数列表) 返回值 {
        //方法体
    }
}
```

impl 是 implement 的缩写。意思是 “实现”的意思。

self 是“自己”的意思，&self 表示当前结构体的实例。 &self 也是结构体普通方法固定的第一个参数，其他参数可选。

结构体方法的作用域仅限于结构体内部。

```rust
impl Study {
    fn get_spend(&self) -> i32 {
        return self.spend;
    }
}

println!("spend {}", s3.get_spend());
//输出 spend 5
```

**结构体静态方法**

```rust
fn 方法名(参数: 数据类型,...) -> 返回值类型 {
      // 方法体
   }

调用方式
结构体名称::方法名(参数列表)
impl Study {
    ...
    fn get_instance_another(name: String, target: String, spend: i32) -> Study {
        return Study {
            name,
            target,
            spend,
        };
    }
}

let s4 = Study::get_instance_another(
    String::from("Go语言极简一本通"),
    String::from("学习Go语言语法，并且完成一个单体服务"),
    5,
);
```

单元结构体
unit type 是一个类型，有且仅有一个值：()，单元类型()类似 c/c++/java 语言中的  void。当一个函数并不需要返回值的时候，rust 则返回()。但语法层面上，void  仅仅只是一个类型，该类型没有任何值；而单元类型()既是一个类型，同时又是该类型的值。

实例化一个元组结构体

```rust
let pair = (String::from("从0到Go语言微服务架构师"), 1);
```

访问元组结构体的字段

```rust
println!("pair 包含 {:?} and {:?}", pair.0, pair.1);
```

解构一个元组结构体

```rust
let (study, spend) = pair;
println!("pair contains {:?} and {:?}", study, spend);
```



#### 第22章 Enum(枚举)

枚举 **enum** 关键字允许创建一个从数个不同取值中选其一的枚举类型（enumeration）。任何一个在 struct 中合法的取值在 enum 中也合法。

在日常生活中很常见。比如：1 年有 12 个月，1 周有 7 天。

**枚举的定义**

```rust
enum 枚举名称{
 variant1,
 variant2,
 ...
}
```

**使用枚举**

```rust
枚举名称::variant

#[derive(Debug)]
enum RoadMap {
    Go语言极简一本通,
    Go语言微服务架构核心22讲,
    从0到Go语言微服务架构师,
}

//这是Go语言学习3个阶段

fn main() {
   let level = RoadMap::从0到Go语言微服务架构师;
    println!("level---{:?}",level);
}
```

`#[derive(Debug)]` 注解的作用，就是让 `派生自`Debug`。

**Option 枚举**

```rust
enum Option<T> {
   Some(T),      // 用于返回一个值
   None          // 用于返回 null,用None来代替。
}
```

`Option` 枚举经常用在函数中的`返回值`，它可以表示有返回值，也可以用于表示没有返回值。如果有返回值。可以使用返回 Some(data)，如果函数没有返回值，可以返回 None。

```rust
fn getDiscount(price: i32) -> Option<bool> {
    if price > 100 {
        Some(true)
    } else {
        None
    }
}

let p=666;  //输出 Some(true)
// let p=6;//输出 None
let result = getDiscount(p);
println!("{:?}",result)
```

**match 语句**

判断一个枚举变量的值，唯一操作符就是 match。

```rust
fn print_road_map(r:RoadMap){
    match r {
        RoadMap::Go语言极简一本通=>{
            println!("Go语言极简一本通");
        },
        RoadMap::Go语言微服务架构核心22讲=>{
            println!("Go语言微服务架构核心22讲");
        },
        RoadMap::从0到Go语言微服务架构师=>{
            println!("从0到Go语言微服务架构师");
        }
    }
}
print_road_map(RoadMap::Go语言极简一本通);//输出 Go语言极简一本通
print_road_map(RoadMap::Go语言微服务架构核心22讲);//输出 Go语言微服务架构核心22讲
print_road_map(RoadMap::从0到Go语言微服务架构师);//输出 从0到Go语言微服务架构师
```

**带数据类型的枚举**

```rust
enum 枚举名称{
    variant1(数据类型1),
    variant2(数据类型2),
    ...
}
#[derive(Debug)]
enum StudyRoadMap{
    Name(String),
}

let level3 = StudyRoadMap::Name(String::from("从0到Go语言微服务架构师"));
match level3 {
    StudyRoadMap::Name(val)=>{
        println!("{:?}",val);
    }
}

//输出 "从0到Go语言微服务架构师"
```



#### 第23章 集合 Collections

Rust 语言标准库提供了通用的数据结构的实现。包括 **向量 （`Vector`）**、**哈希表（ `HashMap` ）**、**哈希集合（ `HashSet` ）** 。

**向量 （`Vector`）**

Rust 在标准库中定义了结构体 `Vec` 用于表示一个向量。向量和数组很相似，只是数组长度是编译时就确定了，定义后就不能改变了，那要么改数组，让他支持可变长度，显然 Rust 没有这么做，它用**向量**这个数据结构，也是在内存中开辟一段连续的内存空间来存储元素。

特点：

- 向量中的元素都是相同类型元素的集合。
- 长度可变，运行时可以增加和减少。
- 使用索引查找元素。（索引从 0 开始）
- 添加元素时，添加到向量尾部。
- 向量的内存在堆上，长度可动态变化。

**创建向量**

1. `new()` 静态方法用于创建一个结构体 Vec 的实例。

```rust
let mut 向量的变量名称 = Vec::new();
```

1. `vec!()` 宏来简化向量的创建。

```rust
let 向量的变量名称 = vec![val1,val2,...]
```

向量的使用方法

| 方法       | 说明                        |
| ---------- | --------------------------- |
| new()      | 创建一个空的向量的实例      |
| push()     | 将某个值 T 添加到向量的末尾 |
| remove()   | 删除并返回指定的下标元素。  |
| contains() | 判断向量是否包含某个值      |
| len()      | 返回向量中的元素个数        |

```rust
let mut v = Vec::new();//调用 Vec 结构的 new() 静态方法来创建向量。
v.push("Go语言极简一本通");       //通过push方法添加元素数据。并且追加到向量尾部。
v.push("Go语言微服务核心架构22讲");
v.push("从0到Go语言微服务架构师");
println!("{:?}",v);
println!("len :{}",v.len()); // 通过len方法获取向量中的元素个数。


let mut v2 = vec!["Go语言极简一本通","Go语言微服务核心架构22讲","从0到Go语言微服务架构师"];
// 通过vect!宏创建向量时，向量的数据类型由第一个元素自动推断出来。
println!("{:?}",v2);

let x=v2.remove(0);
// remove()方法移除并返回向量中指定的下标索引处的元素，将其后面的所有元素移到向左移动一位。
println!("{}",x); //输出 Go语言极简一本通
println!("{:?}",v2);//输出 ["Go语言微服务核心架构22讲", "从0到Go语言微服务架构师"]

//contains() 用于判断向量是否包含某个值。如果值在向量中存在则返回 true，否则返回 false。
if v.contains(&"从0到Go语言微服务架构师"){
   println!("找到了")
}

//访问向量中的某个元素,使用索引
let y = v[0];
println!("{}",y); //输出 Go语言极简一本通

//遍历向量
for item in v {
   println!("充电项目: {}", item);
}
//输出
充电项目: Go语言极简一本通
充电项目: Go语言微服务核心架构22讲
充电项目: 从0到Go语言微服务架构师
```

**哈希表 HashMap**

HashMap 就是**键值对**集合。键是不能重复的，值是可以重复的。

使用 `HashMap` 结构体之前需要显式导入 `std::collections` 模块。

**创建 HashMap**

使用 new()方法来创建。

```rust
let mut 变量名称 = HashMap::new();
```

这个哈希表只有当我们添加了元素之后才能正常使用。因为现在还没指定的数据类型。

| 方法         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| insert()     | 插入/更新一个键值对到哈希表中，如果数据已经存在则返回旧值，如果不存在则返回 None |
| len()        | 返回哈希表中键值对的个数                                     |
| get()        | 根据键从哈希表中获取相应的值                                 |
| iter()       | 返回哈希表键值对的无序迭代器，迭代器元素类型为 (&’a K, &’a V) |
| contains_key | 如果哈希表中存在指定的键则返回 true 否则返回 false           |
| remove()     | 从哈希表中删除并返回指定的键值对                             |

```rust
use std::collections::HashMap;

let mut process = HashMap::new();
process.insert("Go语言极简一本通", 1);
process.insert("Go语言微服务核心架构22讲", 2);
process.insert("从0到Go语言微服务架构师", 3);

println!("{:?}", process);
//输出 {"Go语言极简一本通": 1, "Go语言微服务核心架构22讲": 2, "从0到Go语言微服务架构师": 3}
println!("len {}",menu.len());
//输出  3


// get() 方法用于根据键从哈希表中获取相应的值。
match process.get(&"从0到Go语言微服务架构师"){
   Some(v)=>{
      println!("HashMap v:{}", v);
   }
   None=>{
      println!("找不到");
  }
}
//输出 HashMap v:3

//迭代哈希表 iter()
for (k, v) in process.iter() {
   println!("k: {} v: {}", k, v);
}
//输出
k: Go语言微服务核心架构22讲 v: 2
k: 从0到Go语言微服务架构师 v: 3
k: Go语言极简一本通 v: 1

// contains_key() 方法用于判断哈希表中是否包含指定的 键值对。如果包含指定的键，那么会返回相应的值的引用，否则返回 None。
if process.contains_key(&"Go语言极简一本通") {
   println!("找到了");
}
//输出 找到了

// remove() 用于从哈希表中删除指定的键值对。如果键值对存在则返回删除的键值对，返回的数据格式为 (&'a K, &'a V)。如果键值对不存在则返回 None

let x=process.remove(&"Go语言极简一本通");
println!("{:?}",x);
println!("{:?}",process);
//输出
Some(1)
{"Go语言微服务核心架构22讲": 2, "从0到Go语言微服务架构师": 3}
```

**哈希集合 HashSet**

Hashset 是相同数据类型的集合，它是没有重复值的。如果集合中已经存在相同的值，则会插入失败。

**创建 Hashset**

```rust
let mut 变量名称 = HashSet::new();
```

常用方法如下

| 方法         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| insert()     | 插入一个值到集合中 如果集合已经存在值则插入失败              |
| len()        | 返回集合中的元素个数                                         |
| get()        | 根据指定的值获取集合中相应值的一个引用                       |
| iter()       | 返回集合中所有元素组成的无序迭代器 迭代器元素的类型为 `&'a T` |
| contains_key | 判断集合是否包含指定的值                                     |
| remove()     | 从结合中删除指定的值                                         |

insert() 用于插入一个值到集合中。

```rust
let mut studySet = HashSet::new();
studySet.insert("Go语言极简一本通");
studySet.insert("Go语言微服务核心架构22讲");
studySet.insert("从0到Go语言微服务架构师");
println!("{:?}", studySet);
//输出 {"从0到Go语言微服务架构师", "Go语言微服务核心架构22讲", "Go语言极简一本通"}

studySet.insert("从0到Go语言微服务架构师");
println!("{:?}", studySet);
//输出 {"从0到Go语言微服务架构师", "Go语言微服务核心架构22讲", "Go语言极简一本通"}
```

len() 方法集合中元素的个数。

```rust
println!("len:{}",studySet.len());//输出 len:3
```

`iter()` 方法用于返回集合中所有元素组成的无序迭代器。

```rust
for item in studySet.iter(){
    println!("hashSet-充电项目: {}", item);
}
//输出
hashSet-充电项目: Go语言极简一本通
hashSet-充电项目: Go语言微服务核心架构22讲
hashSet-充电项目: 从0到Go语言微服务架构师
```

`get()` 方法用于获取集合中指定值的一个引用。

```rust
match studySet.get("从0到Go语言微服务架构师") {
    None => {
        println!("没找到");
    }
    Some(data) => {
        println!("studySet中找到:{}",data);
    }
}
//输出 studySet中找到:从0到Go语言微服务架构师
```

`contains()` 方法用于判断集合是否包含指定的值。

```rust
if studySet.contains("Go语言微服务核心架构22讲"){
    println!("包含 Go语言微服务核心架构22讲")
}
//输出 包含 Go语言微服务核心架构22讲
```

`remove()` 方法用于从集合中删除指定的值。如果该值在集合中，则返回 `true`，如果不存在则返回 `false`。

```rust
studySet.remove("Go语言极简一本通");
println!("{:?}",studySet);//输出 {"Go语言微服务核心架构22讲", "从0到Go语言微服务架构师"}

studySet.remove("golang");
println!("{:?}",studySet);//输出 {"Go语言微服务核心架构22讲", "从0到Go语言微服务架构师"}
```











