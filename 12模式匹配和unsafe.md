# Rust

## 模式匹配

> **模式**:模式是Rust中的一种特殊的语法,用于匹配复杂的简单类型的结构

* 将模式与匹配表达式和其它构造结合使用,可以更好的控制程序的控制流
* 模式由以下元素(或一些组合)组成:
  * 字面值
  * 解构的数组,enum,struct和tuple
  * 变量
  * 通配符
  * 占位符
* 想要使用模式,需要将其与某个值进行比较
  * 如果模式匹配,姐可以在代码中使用这个值的相应部分

> match的ARM(分支)

```rs
match value{
  pattern =>expression,
  pattern =>expression,
  pattern =>expression,
}
```

* match表单时要求详尽所有的可能性
* 一个特殊的模式匹配:`_`
  * 他会匹配到任何东西,不会绑定变量
  * 通常用于match最后一个arm;或者省略某些值的时候

> 条件if let表达式

* `if let`表达式主要作为一种间短的方式来等价的代替只有一个匹配项
* if let可选的可以拥有`else`,包括`else if`,`else if let`
* 但是`if let`不会检查穷尽性

> while let条件循环

* 只要模式继续满足匹配条件,那他允许while循环一直运行

```rs
let mut stack = Vec::new();
stack.push(1);
stack.push(2);
stack.push(3);
while let Some(top) = stack.pop() {
  println!("{}", top);
}
```

>for 循环

* for循环时Rust中最常见的循环
* for循环中,模式就是紧随for关键字后的值

```rs
let v = vec![1, 2, 3, 4, 5];
for (index,value) in v.iter().enumerate() {
  println!("{} is at index {}", value, index);
}
```

>let语句

* let语句也是一种模式`let pattern = expression`

```rs
let (x,y,z) =(1,2,3);
```

>函数参数:也是一种模式

```rs
fn print_coordinates(&(x,y):&(i32,i32)){
  println!("current location{},{}",x,y);
}
```

### 可辨驳性

* 模式的两种形式:`可辨驳的`,`无可辩驳的`
* 能匹配任何可能传递的值的:`可辨驳的`
  * 例如:`let x=5;`
* 对于某些可能的值,无法进行匹配的模式:可辩驳的
  * 例如:`if let Some(x)=a_value`
* 函数参数,let语句,for循环只接受`无可辩驳的`模式
* lf let和while let接收`可辨驳的`和`无可辩驳的`

```rs
let a:Option<i32> =Some(4);
//由于let是不可辩驳的,Some是可辨驳的
//let Some(x) =a;
if let Some(x) = a{}
```

### 匹配值语法

>匹配字面量

```rs
let x = 1;
match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("other"),
}
```

>匹配命名变量:命名的变量是可以匹配任何值的无可辩驳模式

```rs
let x= Some(5);
let y= 10;
match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {:?}", y),
    _ => println!("Default case, x = {:?}", x),
}
```

>多重模式:在match中,使用`|`语法(或的意思),可以匹配多种模式

```rs
let x = 1;
match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

>使用`..=`来匹配范围的值

```rs
let x =5;
match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}

let x = 'c';
match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

> 解构以分解值

```rs
let p = Point{ x: 0, y: 7 };
let Point{ x: a, y: b} = p;
//如果解构的名称相同
let Point{ x, y} = p;
```

* 模式匹配

```rs
match p {
  //没有赋值,变量值随意
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y } => println!("On neither axis: ({}, {})", x, y),
}
```

>解构enum

```rs
fn main(){
    let msg = Message::ChangeColor(0, 160, 255);
    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x, y
            )
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r, g, b
            )
        }
    }
}
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

>解构嵌套的struct和enum

```rs
enum Color {
  Rgb(i32, i32, i32),
  Hsv(i32, i32, i32),
}
enum Message {
  Quit,
  Move { x: i32, y: i32 },
  Write(String),
  ChangeColor(Color),
}
fn main() {
  let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));
  match msg {
    Message::ChangeColor(color) => match color {
      Color::Rgb(r, g, b) => println!("Change the color to red {}, green {}, and blue {}", r, g, b),
      Color::Hsv(h, s, v) => println!(
        "Change the color to hue {}, saturation {}, and value {}",
        h, s, v
      ),
    },
    _ => (),
  }
}
```

>解构struct和tuple

```rs
fn main(){
  let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
}

struct Point {
  x: i32,
  y: i32,
}
```

> 在模式种忽略值

1. `_`

   ```rs
   fn foo(_:i32,y:i32){
     println!("this y={}",y);
   }
   ```

2. `_`配合其它模式.忽略值的某一部分

   ```rs
   let mut setting_value = Some(5);
   let new_setting_value = Some(10);
   //模式匹配只需要这两个值属于Some类型的就可以
   match (setting_value, new_setting_value) {
     (Some(_), Some(_)) => {
       println!("Can't overwrite an existing customized value");
     }
     _ => {
       setting_value = new_setting_value;
     }
   }
   ```

3. 使用以`_`开头的名称,忽略未使用的变量
   * 创建临时变量不使用,可以使用`_`来创建

   ```rs
   let _x=5;
   //这个会有警告
   let y=5;
   
   let s =Some(String::from("hrllo"));
   //如果使用_不会转移所有权,访问s依然有效
   //如果使用_s会发生所有权转移
   if let Some(_) = s {
     println!("found a String");
   }

   println!("{:?}",s);
   ```

4. `..`忽略值的剩余部分

```rs
let origin = Point { x: 0, y: 0, z: 0 };
match origin {
    Point { x, .. } => println!("x is {}", x),
  }
}
struct Point {
  x: i32,
  y: i32,
  z: i32,
}

//tuple
let numbers = (2, 4, 8, 16, 32);
match numbers {
  (first, .., last) => println!("Some numbers: {}, {}", first, last),
}
```

>使用match守卫来提供额外的条件

* match守卫就是match arm模式后额外的if条件,想要匹配该条件也必须满足
* match守卫使用与比单独模式更复杂的场景

```rs
let num = Some(4);
match num {
  Some(x) if x < 5 => println!("less than five: {}", x),
  Some(x) => println!("{}", x),
  None => (),
}
```

> @绑定

* @符号可以让我们创建一个变量,该变量可以在测试某个值是否与模式匹配的同时保存该值

```rs
let mes = Mess::Hello { id: 5 };
match mes {
  Mess::Hello {
    //id的值需要在3~7范围内,并且使用@将值存在id_variable中
    id: id_variable @ 3..=7,
  } => println!("id is {}", id_variable),
  Mess::Hello { id: 10..=12 } => println!("id is 10 or 11 or 12"),
  Mess::Hello { id } => println!("id is {}", id),
}
enum Mess {
  Hello { id: i32 },
}
```

## unsafe

* rust中没有强制内存安全的保证:`unsafe Rust`.和普通的rust一样,不过有一些别的能力
* unsafe存在的原因:静态分析是保守的.
  * 使用unsafe:必须直到自己在做什么,并且承担相应的风险

>unsafe模式:使用`unsafe`关键字切换到unsafe Rust,开启一个块,里面放着unsafe代码

* unsafe可执行的四个动作
   1. 解引用原始指针
   2. 调用unsafe函数或者方法
   3. 访问或者修改可变的静态变量
   4. 实现unsafe trait
* 注意
  * unsafe并没有关闭借用检查或者停用其他安全检查
  * 任何内存安全相关的错误必须留在unsafe块里
  * 尽可能隔离unsafe代码,最好将其封装在安全的抽象里,提供安全的API

>解引用原始指针

* 原始指针.
  * 可变的:`*mut T`
  * 不可变的:`*const T`.意味着指针在解引用后不能直接对其进行赋值
  * 注意:这里的*不是解引用符号,他是类型名的一部分
* 与引用不同,原始指针:
  * 允许通过同时具有不可变和可变指针或多个指向同一位置的可变指针来忽略借用规则
  * 无法保证能指向合理的内存
  * 允许为null
  * 不实现任何自动清理
* 放弃保证的安全,换取更好的性能/与其他语言或者硬件接口的能力

```rs
let mut num = 5;
let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
//如果不放在unsafe中,那么这个程序会报错,这是不安全的
unsafe {
  print!("{}", *r1);
  print!("{}", *r2);
}
let address = 0x012345usize;
let r = address as *const i32;
unsafe {
  //出现非法的访问,后果由自己承担
  println!("{}", *r);
}
```

>调用unsafe函数或者方法

* unsafe函数或者方法:在定义前加上了unsafe关键字
  * 调用前需要手动满足一些条件(看文档),因为rust无法对这些条件进行验证

```rs
unsafe {
  dangerous();
}
unsafe fn dangerous() {}
```

>创建unsafe代码的安全抽象

* 函数包含unsafe代码并不意味着需要将整个函数标记为unsafe
* 将unsafe代码包裹在安全函数中是一个常见的抽象

```rs
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  let len = slice.len();
  let ptr = slice.as_mut_ptr();
  assert!(mid <= len);

  unsafe {
    (
      slice::from_raw_parts_mut(ptr, mid),
      slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid),
    )
  }
}
```

>使用extern函数调用外部代码(任何在extern块里的都是不安全的)

* extern关键字:简化创建和使用外部函数接口(FFI)的过程
* 外部函数接口(FFI):它与匈奴一种编程语言定义函数,并且让其它编程语言能调用这些函数

```rs
//c指的是application binary interface,遵循c语言的abi
extern "C"{
  fn abs(input: i32) -> i32;
}
```

* 应用二进制接口(application binary interface):定义函数在汇编层的调用方式

* 从其他语言调用Rust
  * 可以使用extern创建接口,其它语言通过他们可以调用rust函数
  * 在fn前面添加关键字,并指定ABI
  * 还需要添加`#[no_mangle]`注解,避免rust在编译时改变他们的名称

```rs
#[no_mangle]
pub extern "C" fn call_from_c() {
  println!("Just called a Rust function from C!");
}
```

>访问或者修改一个可变静态变量

* rust支持全局变量,但是因为所有权机制可能产生一些问题,例如数据竞争
* 在rust中,全局变量叫做静态(static)变量

```rs
static HELLO_WORLD: &str = "Hello, world!";
```

* 静态变量与常量类似,必须全是大写
* 必须标注类型
* 静态变量只能存储在`'static`生命周期的引用,无需显示标注
* 访问不可变的静态变量是安全的

* 常量和不可变静态变量的区别
  1. 静态变量:有固定的内存地址,使用他的值总会访问同样的数据
  2. 常量:允许使用它们的时候对数据进行复制
  3. 静态变量是可以改变的,访问和修改静态可变变量是不安全的(unsafe)的

```rs
static mut counter: u32 = 0;
//如果是多线程可能产生数据竞争
fn add_to_count(inc: u32) {
  unsafe {
    counter += inc;
  }
}
```

>实现不安全的trait

* 当某一个trait中存在至少一个方法拥有编译器无法校验的不安全因素,就称这个trait是不安全的
* 声明unsafe trait:在定义前加unsafe关键字.并且该trait只能在unsafe代码块中实现

```rs
unsafe trait Foo{}

unsafe impl Foo for i32{}
fn main(){}
```

>何时使用unsafe海马

* 编译器无法保证内存安全的时候
* 通过显示标记unsafe,可以轻松定位

## 高级Trait

>在trait定义中使用关联类型是Trait中的类型占位符,它可以用于trait的方法

* 在签名中可以定义出包含某些类型的Trait,而在实现前无需直到这些类型是什么

```rs
pub trait Iterator{
  //Item就是占位符,每次迭代的值替换Item
  type Item;
  fn next(&mut self) -> Option<Self::Item>;
}
```

>关联类型和泛型的区别

1. 泛型:每次实现trait是标注类型.可以为一个类型多次实现某个trait(不同泛型参数)
2. 关联类型:无需标注类型.无法为单个类型多次实现某个trait

>默认泛型参数和运算符重载

* 可以使用泛型参数时为泛型指定一个默认的具体类型
* 语法:<PlaceholderType = ConcreteType>
* rust不允许创建自己的运算符以及重载任意的运算符
* 但可以通过实现`std::ops`中列出的那些trait来重载一部分的运算符

```rs
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
  x: i32,
  y: i32,
}

//如果不为Rhs指定具体的类型,那么Rhs就是self(Point)
impl Add for Point {
  type Output = Point;
  fn add(self, other: Point) -> Point {
    Point {
      x: self.x + other.x,
      y: self.y + other.y,
    }
  }
}
```

>默认泛型参数的主要使用场景

* 扩展一个类型而不破坏现有的代码
* 允许在大部分用户都不需要的特定场景下进行自定义

>安全限定语法(Fully Qualified Synax),如何调用同名方法

```rs
fn main() {
  let person = Human;
  //调用Human自身的方法
  person.fly();
  Pilot::fly(&person);
  Wizard::fly(&person);
}
trait Pilot {
  fn fly(&self);
}
trait Wizard {
  fn fly(&self);
}
struct Human;
impl Pilot for Human {
  fn fly(&self) {
    println!("This is your captain speaking.");
  }
}
impl Wizard for Human {
  fn fly(&self) {
    println!("Up!");
  }
}
impl Human {
  fn fly(&self) {
    println!("*waving arms furiously*");
  }
}
```

* 完全限定语法:<Type as Trait>::function(receiver_if_method,netx_arg,...);
  * 可以在任何调用函数或者方法的地方使用
  * 允许忽略那些从其他上下文能推导出的部分
  * 当rust无法区分你期望用哪个具体实现的的时候才需要使用这种语法

```rs
fn main(){
  println!("A body of type {:?}", Dog::baby_name());
  println!("A body of type {:?}", <Dog as Animal>::baby_name());
}

trait Animal {
  fn baby_name() -> String;
}
struct Dog;
impl Dog {
  fn baby_name() -> String {
    String::from("Spot")
  }
}
impl Animal for Dog {
  fn baby_name() -> String {
    String::from("puppy")
  }
}
```

>使用`supertrait`来要求trait附带其它trait的功能

* 需要在一个trait中使用其它的trait功能
  * 需要被依赖的trait也被实现
  * 那个被间接依赖的trait就是当前trait的supertrait

```rs
//:后面是他依赖的trait
trait Outline: fmt::Display{
  fn outline_print(&self) {
    let output = self.to_string();
    let len = output.len();
    println!("{}", "*".repeat(len + 4));
    println!("*{}*", " ".repeat(len + 2));
    println!("* {} *", output);
    println!("*{}*", " ".repeat(len + 2));
    println!("{}", "*".repeat(len + 4));
  }
}

struct Point2 {
  x: i32,
  y: i32,
}
//Point2需要实现这两个trait
impl Outline for Point2 {}

impl fmt::Display for Point2{
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "({}, {})", self.x, self.y)
  }
}
```

>使用newType模式在外部类型上实现外部trait

* 孤儿规则:只有当trait或者类型定义在本地包的时候,才能为该类型实现这个trait
* 可以通过`newtype`模式来绕过这一个规则
  * 利用tuple struct(元组结构体)创建一个新的类型

```rs
struct Wrapper(Vec<String>);

//由于Vec和Display都在外部包中,无法直接为Vec实现Display
impl fmt::Display for Wrapper {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "[{}]", self.0.join(", "))
  }
}
fn main(){
  let w = Wrapper(vec![String::from("hello"), String::from("world")]);
  println!("w = {}", w);
}
```

## 高级类型

>使用`newtype`模式实现类型安全和抽象

* newType模式可以用来静态的保证各值之间不会混肴表明值的单位
* 为类型的某些细节提供抽象能力
* 通过轻量级的封装隐藏内部的细节

>类型别名创建类型同义词

* rust为现有的类型产生另外的名称(同义词).并不是一个独立的类型,使用type关键字

```rs
type kilometers = i32;
```

> Never类型

* 名为`!`的特殊类型:他没有任何值(空类型).充当不反魂的函数中的返回类型
* 不返回值的函数也称为发送函数(diverging function)

>`Sized Trait`:动态大小

* rust中需要在编译时确定为一个特定类型的值的分配空间
* 动态大小的类型(Dynamically Sized Types,DST)的概念
  * 编写代码时使用只有在运行时才能确定大小的值
* str是动态大小的类型(注意不是`&str`):只有在运行时才能确定字符串的长度
  
  ```rs
  //无法工作
  let s1:str = "hello";
  let s1:str = "hello world";
  ```

* 使用`&str`,存str的地址和长度

* rust使用动态大小的类型的通用方式
  * 附带一些额外的元数据来存储动态信息的大小
  * 使用动态大小类型是总把他的值放在某种指针后面

>trait:另外一种动态大小的类型

* 每一个trait都是一个动态大小的类型,可以通过名称对其进行引用
* 为了将trait用作trait对象,必须将他放置在某种指针之后
  * 例如:`&dyn Trait`后者`Box<dyn Trait>(Rc<dyn Trait>)`之后

> `Sized trait`

* 为了处理动态大小的类型,rust提供了一个Sized trait来确定一个类型的大小在编译时是否已知
  * 编译时可计算出大小的类型会自动实现这个trait
  * rust会为每一个泛型函数隐式的添加Sized约束

```rs
fn generic<T>(t:T){}
//默认会隐式的转化为下面这个函数
fn generic<T:Sized>(t:T){}
```

* 默认情况下,泛型函数只能被用于编译时已经知道代销的类型,可以通过特殊的语法解除这一限制

* **?Sized**:不确定性

```rs
//T的大小可能不是确定的,使用引用,只能用于Sized,这个trait
fn generic<T:?Sized>(t:&T){}
```

## 高级函数和闭包

>可以加个你函数传递给其它函数,函数在传递的过程中被强制转换成fn类型(函数指针)

```rs
fn add_one(x: i32) -> i32 {
  x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
  f(arg) + f(arg)
}

fn main() {
  let answer = do_twice(add_one, 5);
  println!("The answer is: {}", answer);
}
```

>函数指针与闭包的不同

* fn是一个类型,不是一个trait
  * 可以直接指定fn为参数类型,不用声明一个以`fn trait`为约束的泛型参数
* 函数指针实现了全部3种闭包trait(Fn,FnMut,FnOnce)
  * 总是可以把函数指针用作参数传递给一个接收闭包的函数
  * 所以倾向于搭配闭包trait的泛型来编写函数:可以同时接收闭包和普通函数
* 某些情景下,只想接收fn而不接收闭包
  * 与外部不支持闭包的代码交互:C函数

* enum的变体也实现了闭包的trait

```rs
enum Status{
  Value(u32),
  Stop,
}
let v = Status::Value(10);
let list_of_statuses:Vec<Status> =(0u32..20).map(Status::Value).collect();
```

>返回闭包

* 闭包使用trait进行表达,无法在函数中直接返回一个闭包,可以将一个实现了该trait的具体类型作为返回值

```rs
fn return_closure() -> Box<dyn Fn(i32) -> i32> {
  let num = 5;
  Box::new(move |x| x + num)
}
```

## 宏macro

* 宏在Rust里指的是一组相关特性的集合称谓
  * 使用`macro_rules!`(将弃用)构建的声明宏(declarative macro)
  * 3种过程宏
     1. 自定义`#[derive]`宏,用于struct和enum,可以为其指定随derive属性添加的代码
     2. 类似属性的宏,在任何条目上添加自定义属性
     3. 类似函数的宏,看起来像函数调用,对于指定为参数的token进行操作

>函数与宏的差别

1. 本质上,宏是用来编写可以生成其它代码的代码(元编程)
2. 函数在定义签名是,必须声明参数的个数和类型,宏可以处理可变的参数
3. 编译器会在解释代码前展开宏
4. 红的定义比函数复杂的多,难以阅读,理解和维护
5. 在某个文件调用宏,必须提前定义宏或将宏引入当前作用域
6. 函数可以在任何位置定义在任何位置使用

>基于属性来生成代码的过程宏

* 这种形式更像函数(某种形式的过程)一样
  * 接收并操作输入的rust代码
  * 生成另外一些Rust代码作为结果
* 三种过程宏
  1. 自定义派生
  2. 属性宏
  3. 函数宏
* 创建过程宏:宏定义不许单独放在他们自己的包中,并使用特殊的包类型

>自定义derive宏

略

>类似属性的宏

* 属性宏与自定义derive宏类似
  * 允许创建新的属性
  * 但不是为`derive`属性生成代码
* 属性宏更加灵活
  * derive只能用于struct和enum
  * 属性宏可以用于任意条目,比如函数

>类似函数的宏

* 函数宏定义类似于函数调用的宏,但是比普通函数更加灵活
* 函数宏可以接收`TokenStream`作为参数
* 与另外两个过程宏一样,在嘀咕一种使用rust代码来操作TokenStream
