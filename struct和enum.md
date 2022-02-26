# rust基础

## struct

> 结构体,与元组类似,也可以把多个类型组合到一起,作为新的类型.区别在于,他的每个元素都有自己的名字

* 实例化struct,只需要给每个字段赋值就可以了

```rs
  struct Point {
    x: i32,
    y: i32,
  }
  let point = Point { x: 0, y: 0 };
  println!("point.x = {},point.y={}", point.x, point.y);
```

>* 如果对应字段值与对应变量名相同时可以使用简写
>* 同时值也可以通过解构的方式

```rs
let x = 20;
let y = 30;
let Point { x: px, y: py } = Point { x, y };
println!("px = {},py={}", px, py);
```

>rust中还有一个语法糖,当想基于某一个struct实例来创建一个新实例的时候,可以使用struct更新语法

```rs
let use1 = User {
  email: String::from("2022@com"),
  username: String::from("zhangsan"),
  active: true,
  sign_in_count:100
};
let use2 = User{
  email: String::from("1999@com"),
  username: String::from("lisi"),
  ..use1
};
```

* 这样没有使用use1更新的会和use1不同,使用use1更新的字段会和use1相同

### tuple struct

>类似于tuple struct两者的混合.区别在于他们的成员没有名字

```rs
struct Color (i32, i32, i32);
let black = Color(0,0,0)
```

### Unit-Like Struct(没有任何字段)

* 可以定义没有任何字段的struct,叫做Unit-Like Struct(与(),单元类型类似)
* 适用于需要在某个类型上实现某个`trait`,但是在里面有没有任何想要存储的数据

### struct数据的所有权

```rs
struct User {
  email: String,
  username: String,
  active: bool,
  sign_in_count: u64,
}
```

* 这里的字段使用了String而不是&str
  * 该struct实例拥有其所有的数据
  * 只要struct实例是有效的,那么里面的字段数据也是有效的
* struct里面也可以存放引用,但是需要使用生命周期
  * 生命周期保证只要struct的实例是有效的,那么里面的引用也是有效的
  * 如果struct里面存储引用,而不使用生命周期,就会报错

>打印struct结构体

* `println!`的生成办法
  * `std::fmt::Display`
  * `std::fmt::Debug`
* `#[derive(Debug)]`:自定义的类型可以派生于Debug
* 调式模式的格式化字符串
  * `{:?}`
  * `{:#?}`

### struct方法

* 方法和函数雷系,fn关键字,名称,参数,返回值
* 方法和函数的不同之处
  * 方法是在struct(或enum,trait对象)的上下文中定义
  * 第一个参数是self,表示方法被调用的struct实例

>定义方法

1. 在impl块离定义方法
2. 方法的第一个参数可以是`&self`,也可以获得其所有权或课变借用,和其它参数一样

```rs
impl Rectangle {
  fn area(self: &Rectangle) -> u32 {
    self.width * self.height
  }
}
fn main(){
    let rect = Rectangle {
    width: 30,
    height: 50,
  };
  println!("{}", rect.area());
}
```

> 方法调用的运算符

1. rust会自动引用或者解引用(在调用方法时就会发生)
2. 在调用方法时,Rust会根据情况自动添加&,&mut或*,以便object可以匹配方法的签名
   * 下面两行代码效果等价
   * `p1.distance(&p2)`
   * `(&p1).distance(&p2)`

>方法参数

```rs
impl Rectangle {
  ...
  fn can_hold(&self, other: &Rectangle) -> bool {
    self.width > other.width && self.height > other.height
  }
}
...
let other = Rectangle {
  width: 60,
  height: 60,
};
println!("{}", other.can_hold(&rect));
```

>可以在impl块里定义不把self作为第一个参数的函数,他们叫关联函数

* 例如`String::from()`
* 关联函数通常用作构造器

```rs
impl Rectangle {
    fn square(size: u32) -> Rectangle {
    Rectangle {
      width: size,
      height: size,
    }
  }
}
...
let s = Rectangle::square(20);
```

* `::`符号:
  * 关联函数
  * 模块创建的命名空间

* 并且每个struct允许多个impl块

```rs
impl Rectangle{}
impl Rectangle{}
impl Rectangle{}
```

## 枚举enum

>rust中tuple,struct,tuple struct代表的是多个类型**与**的关系,那么enum更像是多个类型**或**的关系

* 结构体中可以使用枚举的类型

```rs
enum IpAddrKind {
  V4,
  V6,
}
struct IpAddr {
  kind: IpAddrKind,
  address: String,
}
```

* 将数据附加到枚举的变体中
  * 不需要额外使用struct
  * 每个变体可以拥有不同的类型以及关联的数据量

```rs
enum IpAddrKind {
  V4(u8, u8, u8, u8),
  V6(String),
}

fn main() {
  let home = IpAddrKind::V4(127, 0, 0, 1);
  let loopback = IpAddrKind::V6(String::from("::1"));
}
```

1. 枚举的变体都位于标识符的命名空间下
2. enum的变体,若附带有值,还有在函数的命名空间下生成一个函数,该函数的参数是对应的枚举变体附带的值(括号里的值),然后回根据西安舒,返回一个枚举变体,墅就是一个枚举变体函数(自动生成的)
3. 有利于库的设计维护与编写代码

> 在枚举的变体中可以嵌入任意类型的数据,无论是字符串,结构体,甚至另外一种枚举类型

```rs
enum Message{
  Quit,
  Move {x:i32, y:i32},
  Write(String),
  ChangeColor(i32, i32, i32),
}
fn main(){
  let q = Message::Quit;
  let m = Message::Move {x:10, y:20};
  let w = Message:: Write(String::from("hello"));
  let c = Message::ChangeColor(10, 20, 30);
}
```

> 和struct一样,也可以为枚举定义方法

```rs
impl Message {
  fn call(&self) {
    print!("{:?}", self);
  }
}
q.call();
...
```

### Option枚举

>rust中没有null:null的问题就是当你尝试使用非null那样使用null值的时候,就会引起某种错误

* `Option<T>`:(null的概念)因某种原因变为无效或者缺失的值.
* 标准库中的定义

```rs
enum Option<T>{
  Some(T),
  None
}
```

* 都包含在Prelude(预导入模块),可以直接使用(不需要`::`)

```rs
let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None;
```

> Option<T>和T是不同的类型,不可以吧Option<T>直接当作T
>
>若想使用Option<T>中的T,必须将他转换为T

```rs
let y: Option<i32> = None;
let x= 7;
//let w =y+x;//这样使用会报错
```

## match

>允许一个值与一系列模式进行匹配,并执行匹配成功的模式对应的代码

* 模式可以实字面值,变量名,通配符...

```rs
#[derive(Debug)]
enum UsState {
  Alabama,
  Alaska,
}
enum Coin {
  Penny,
  Nickel,
  Dime,
  Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
  match coin {
    Coin::Penny => {
      println!("Lucky penny!");
      1
    }
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quarter(state) => {
      println!("{:?}", state);
      25
    }
  }
}

fn main() {
  let c = Coin::Quarter(UsState::Alaska);
  println!("{}", value_in_cents(c));
}
```

>匹配Option枚举

```rs
fn plus_one(x:Option<i32>)->Option<i32>{
  match x{
    None=>None,
    Some(i)=>Some(i+1),
  }
}
```

* 在rust中必须穷举所有枚举
  * 如果不想要穷举所有模式,使用`_`替代
  * 下划线需要写到匹配的最后面

```rs
fn main(){
  let v=0_u32;
  match v{
    1=>println!("one"),
    _=>println!("other"),
  }
}
```

### if let 模式匹配

>`if let`只关心一种匹配模式而忽略其它匹配的情况(放弃了穷举的可能)

* `if let`匹配模式还可以搭配else使用

```rs
let v = Some(0_u8);
match v {
  Some(1) => println!("one"),
  _ => println!("other"),
}
if let Some(1) = v {
  println!("one");
}else {
    println!("other");
}
```
