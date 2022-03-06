# rust

## 泛型

>泛型事具体类型后者其它属性的抽象代替.可以提高代码的复用能力

* 在编写的代码不是最终的代码,而是一种模板,里面有一种`占位符`
* 编译器在编译时将`占位符`替换为具体类型

* 函数中泛型

```rs
fn largest<T>(list: &[T]) -> T {}
```

* struct中泛型

```rs
struct Point<T>{
  x:T,
  y:T,
}
```

* 枚举中使用泛型

```rs
enum Option<T>{
  Some(T),
  None,
}

enum Result<T,E>{
  Ok(T),
  Err(E),
}
```

* 在方法中使用泛型.(也可以实现泛型)
  * 在方法中使用泛型和结构体中的泛型可以不同

```rs
impl<T> Point<T> {
  fn x(&self) -> &T {
    &self.x
  }
}
```

## Trait

>Trait可以告诉编译器具有哪些并且可以与其他类型共享的功能

* `Trait`:抽象的定义共享行为
* `Trait bounds(约束)`:泛型类型参数指定为实现特定行为的类型
* Trait与其他语言的接口`interface`类型,但有些区别

### define Trait

>Trait 的定义:把方法签名放在一起,来定义实现某种目的所必须的一组行为

* 关键字trait.只有方法签名,没有具体实现
* trait可以有多个方法:每个方法签名占一行,以`;`结尾
* 实现改trait的类型必须提供具体的方法实现

```rs
pub trait Summary {
  fn summarize(&self) -> String;
}
```

> 在类型上实现trait

* 与类型实现方法类似
  1. `impl xxx for Tweet{...}`
  2. 在impl块中,需要对Trait里的方法签名进行具体实现

```rs
pub trait Summary {
  fn summarize(&self) -> String;
}

pub struct NewsArticle {
  pub headline: String,
  pub location: String,
  pub author: String,
  pub content: String,
}

impl Summary for NewsArticle {
  fn summarize(&self) -> String {
    format!("{}, by {} ({})", self.headline, self.author, self.location)
  }
}
```

>实现trait的约束

* 可以在某个类型上实现某个trait的前提条件是
  * 这个类型或者这个trait是在本地crate里定义的
* 无法为外部不类型来实现外部的trait:
  * 这个限制是程序属性的一部分(一致性).**孤儿规则**"父类型不存在.此规则确保其他人的代码不能破坏你的代码,反之亦然.
  * 如果没有这个规则,两个trait就可以为同一类型实现同一个trait,rust就不知道应该实现哪一个

### 默认实现

>默认实现的方法可以调用所在trait内其它的方法,包括未默认实现的,但是在impl块中,必须实现每一个无默认实现的trait内方法,不然会疯狂报错

* 注意:无法从方法的重写实现里面调用默认的实现

```rs
pub trait Summary {
  fn summarize(&self) -> String{
    String::from("默认实现")
  }
}
```

### Trait作为参数

* 使用`impl ...`作为类型

```rs
pub fn notify(item: impl Summary){
  println!("Breaking news! {}", item.summarize());
}
```

* 使用`Trait bounds(约束)`的方式.实际上impl的语法是Trait bounds(约束)的语法糖

```rs
pub fn notify<T:Summary>(item: T){
  println!("Breaking news! {}", item.summarize());
}
```

>使用`+`指定多个Trait bounds(约束)

```rs
pub fn notify1(item: impl Summary+Display) {
  println!("Breaking news! {}", item.summarize());
}
//or
pub fn notify<T: Summary+Display>(item: T) {
  println!("Breaking news! {}", item.summarize());
}
```

* 使用方法签名的后面实现where语句

```rs
pub fn notify2<T, U>(a: T, b: U) -> String
where
  T: Summary + Display,
  U: Clone + Debug,
{
  format!("Breaking news! {}", a.summarize())
}
```

### Trait作为返回类型

```rs
pub fn notify1(s:&str) ->impl Summary {
  ...
}
```

* 注意:`impl Trait`只能返回确定的同一种类型,返回可能不同的类型的代码会报错

* 找到最大值的一个写法

```rs
fn largest<T: PartialOrd + Clone>(list: &[T]) -> &T {
  let mut largest = &list[0];
  for item in list.iter() {
    if item > largest {
      //std::cmp::PartialOrd
      largest = item;
    }
  }
  largest
}
```

### Trait Bound有条件的实现方法

* 在使用泛型类型参数的impl块上使用Trait bound,我们可以有条件的未实现特定trait的类型来实现方法

```rs
struct Pair<T>{
  x:T,
  y:T,
}

impl<T> Pair<T>{
  fn new(x:T,y:T)->self{
    Self{x,y}
  }
}

impl <T:Display+PartialOrd> Pair<T>{
  fn cmp_display(&self){
    if self.x>self.y {
      println!("the largest number is x={}",self.x);
    }else{
      println!("the largest number is y={}",self.y);

    }
  }
}
```

* 也可以未实现了其它Trait的任意类型有田间的实现某个Trait
* 为满足Trait Bound的所有类型上实现Trait叫做覆盖实现(blanket implementations)
