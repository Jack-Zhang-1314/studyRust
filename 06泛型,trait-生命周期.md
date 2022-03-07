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

## 生命周期

* Rust的每个引用都有自己的生命周期
* 生命周期:引用会保持有效的作用域
* 大多数情况:生命周期是隐式的,可被推断的
* 当引用的生命周期可能以不同的方式相互关联时:手动标注生命周期

>生命周期存在的目的:避免悬垂引用

```rs
{
  //在rust中不允许有null值
  let r;
  //let b = r;
  {
    let x= 5;
    r = &x;
  }
  //error
  println!("r: {}", r);
}
```

* r在作用域引用了x.不过当x脱离了这个作用域时,会自动销毁x变量.此时再次引用会报错

* **借用检查器**:rust编译器的借用检查器,比较作用域来判断所有的借用是否合法
  * 只要x的生命周期不小于r的生命周期就可以引用

```rs
let x = 5;
let r = &x;
println!("r: {}", r);
```

### 生命周期的标注

* 生命周期的标注不会改变引用的生命周期长度
* 当指定了泛型生命周期的参数,函数可以接收带有任何生命周期的应用
* 生命周期的标注:描述了多个引用的生命周期之间的关系,但不影响生命周期

>生命周期标注语法

* 生命周期参数:使用`'`开头.通常全小写,并且非常短,很多人会使用`'a`
* 标注位置:在应用的`&`的符号后,使用空格将标注和应用类型分开
  * `&i32`:引用
  * `&'a i32`:显示生命周期的引用
  * `&'a mut i32`:显示生命周期的不可变引用
* 单个生命周期的引用是没有意义的

> 函数签名中的生命周期标注

* 泛型生命周期参数声明在:函数名-和参数列表之间的<>里

  ```rs
  fn main() {
    let string1 = String::from("hello");
    let string2 = "world";
    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
  }
  
  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
      x
    } else {
      y
    }
  }
  ```

* 生命周期`'a`的实际声明周期是:x和y两个生命周期较小的那个

### 深入理解生命周期

* 指定生命周期参数的方式依赖于函数所做的事情
  * 从函数返回的引用时,返回类型的生命周期参数需要域其中一个参数的生命周期匹配
  * 如果返回的引用没有指向任何参数,那么只能在引用函数内创建的值
    * 这就是悬垂引用:该值在函数结束的时候就走出了作用域

```rs
//这里的生命周期就是x的生命周期
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

* `longest`将引用传给result,但是longest这块内存已经被清理.指针指向就会有问题
  * 最好的方式就是返回值而不是引用

  ```rs
  fn main() {
    let string1 = String::from("hello");
    let string2 = "world";
    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
  }
  
  //fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  //  let result = String::from("abc");
  //  //error
  //  result.as_str()
  //}

  fn longest<'a>(x: &'a str, y: &'a str) -> String {
    let result = String::from("abc");
    //所有权移交给函数的调用者
    result
  }
  ```

> Struct定义生命周期标注

* struct中包括自持有的类型.引用,需要在每一个引用上添加生命周期标注

```rs
struct ImportantExcerpt<'a> {
    part: &'a str,
}
fn main(){
  let novel =String::from("Call me Ishmael. Some years ago...");
  let first_sentence = novel.split('.').next().expect("Could not find a '.'");

  let i =ImportantExcerpt{
    part:first_sentence,
  };
}
```

>生命周期省略规则

* 在Rust中引用分析中所编入的模式称为生命周期省略规则
  * 这些规则无需开发者来遵守.他们是由特殊的情况,由编译器考虑
* 生命周期省略规则不会提供完整的推断
  * 如果应用规则后,引用的生命周期任然模糊不清->编译错误
  * 那么就需要添加生命周期标注,表明引用之间的相互关系

>输入生命周期/输出生命周期

* 生命周期在函数/方法的参数:输入生命周期
* 生命周期在函数/方法的返回值:输出生命周期

>生命周期省略的三个规则

* 编译器使用3个规则在没有显示标注生命周期的情况下,来确定引用的生命周期.这些规则适用于fn定义和impl块
* 1适用于输入生命周期,2和3适用于输出生命周期

1. 每个引用类型的参数都有自己的生命周期
2. 如果只有一个输入生命周期的参数,那么该生命周期赋给所有的输出生命周期参数
3. 如果有多个输入生命周期参数,但是其中一个是`&self`或者`&mut self`(方法),那么self的生命周期会被赋给所欲的输出生命周期的参数

* 如果当编译器使用了所有的规则之后任然无法判断返回值的生命周期就会报错

### 方法中定义生命周期标注

* 在struct上使用生命周期实现方法,语法和泛型参数的语法一样
* 在哪使用生命周期参数依赖于,生命周期的参数和字段,**方法的参数**和**返回值**有关
* struct字段的生命周期名:在`impl`后声明,`struct`名后使用,这些生命周期是struct类型的一部分
* impl块内的方法签名中:
  * 引用必须绑定与struct字段引用的生命周期,或者引用是独立的.
  * 生命周期省略规则经常使方法中的生命周期标注不是必须的

```rs
impl<'a> ImportantExcerpt<'a>{
  fn level(&self) -> i32 {
    3
  } 
  fn announce_and_return_part(&self, announcement: &str) -> &str {
    println!("Attention please: {}", announcement);
    self.part
  }
}
```

### 静态生命周期

> `'static`是一个特殊的生命周期:整个程序的执行时间

* 所有的字符串字面值都拥有`'static`生命周期
  * `let s:&'static str = "abc"`
* 如果使用`'static`生命周期要三思.是否需要引用在程序整个生命周期都存活

```rs
fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
where
  T: Display,
{
  println!("Announcement! {}", ann);
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```
