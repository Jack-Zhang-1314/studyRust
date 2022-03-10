# rust基础

## Rust的代码组织

> 代码组织的主要包括

* 哪些细节可以暴露,哪些细节是私有的
* 作用域的哪些名称是有效的
* ...

> 模块系统

* `Package(包)`:Cargo的特性,构建,测试,共享crate
* `Crate(单元包)`:一个模块树,它可以产生一个library或可执行文件
* `Module(模块)`,use:让你控制代码的组织,作用域,私有路径
* `Path(路径)`:为struct,function或者module等项命名的方式

### Package和Crate

* Crate类型
  * binary:二进制类型的Crate
  * library:库类型的Crate
* Crate Root
  * 源代码文件
  * Rust编译从这里开,开始组成你的Crate的根Module
* 一个Package
  * 包含一个`Cargo.toml`,描述了如何构建这些Crates
  * 只能包含0-1个`library crate`
  * 可以包含任意数量的`binary crate`
  * 必须至少包含一个crate(library或binary)

>例如`cargo new my-project`:就是创建一个`binary crate`

* **src/main.rs**:binary crate 的 crate root
  * crate名与package名相同,例如上面的my-project

* **src/lib.rs**:library crate 的 crate root
  * package包含一个library crate
  * crate名与package名相同,例如上面的my-project

* Cargo把crate root文件交给rustc来构建library或binary

>Crate将相关功能组合到一个作用域内,便于在项目中进行共享,防止冲突

* 例如rand crate,访问它的功能需要通过他的名字:rand

### 定义module来控制作用域和私有性

* Module:
  * 在一个crate内,将代码进行分组
  * 增加可读性,易于复用
  * 控制项目(item)的私有性.public,private
* 建立module
  * mod关键字
  * 可嵌套
  * 可包含其它项(struct,enum,常量,trait,函数等)的定义

```rs
//lib.rs
mod front_of_house {
  mod hosting {
    fn add_to_waitlist() {}
    fn seat_at_table() {}
  }

  mod serving {
    fn take_order() {}
    fn serve_order() {}
    fn take_payment() {}
  }
}
```

* `src/main.rs`,`src/lib.rs`叫做crate roots.这两个文件(任意一个)的内容形成了crate的模块,位于整块树的根部

### Path

> 在Rust中的模块中找到某个条目,需要使用路径

* 路径的两种形式:
  * 绝对路径:从`crate root`开始,使用`crate`名字或者字面值`crate`
  * 相对对镜,从当前模块开始,使用self,super或者当前模块的标识符
* 路径至少有一个标识符组成,标识符之间使用`::`

> 私有边界

* 模块不仅可以组织代码,还可以定义私有边界
* 如果想把函数或者struct等设为私有,可以将他放到某个模块中
* Rust中所有的条目(函数,方法,struct,enum,模块,常量)默认是私有的
* 父级模块里可以使用所有祖先模块中的条目

```rs
mod front_of_house {
  pub mod hosting {
    pub fn add_to_waitlist() {}
  }
}

pub fn eat_at_retaurant() {
  crate::front_of_house::hosting::add_to_waitlist();
  front_of_house::hosting::add_to_waitlist();
}
```

* `front_of_house`和`eat_at_retaurant`都是文件的根级,无论是私有的还是公有的,都可以互相调用

> super关键字

* **super**:用来访问父级模块路径中的内容,类似文件系统中的`..`

```rs
fn serve_order() {}
mod back_of_house {
  fn fix_incorrect_order() {
    cook_order();
    //相对路径
    super::serve_order();
    //绝对路径
    crate::serve_order();
  }
  fn cook_order() {}
}
```

> pub struct

* pub放在struct前:
  * struct是公共的
  * struct的字段默认是私有的
* struct的字段需要单独设置pub来变成公有的

```rs
mod back_of_house {
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
}

pub fn eat_at_restaurant() {
  let mut meal = back_of_house::Breakfast::summer("Rye");
  meal.toast = String::from("wheat");
  println!("I'd like {} toast please", meal.toast);
  //私有的无法访问
  //meal.seasonal_fruit = String::from("peaches");
}
```

> pub enum

* pub放在enum前,enum是公共的,其中所有的变体也是公共的

```rs
mod back_of_house {
  pub enum Appetizer {
    Soup,
    Salad,
  }
}
```

### use

* 可以使用use关键字将路径导入到作用域中
  * 仍需要遵循私有性规则
  * 使用use指定绝对路径

```rs
mod first_of_house {
  pub mod hosting {
    pub fn add_to_waitlist() {}
  }
}

//相当于在跟模块下定义了hosting
use crate::first_of_house::hosting;

pub fn eat_at_restaurant() {
  hosting::add_to_waitlist();
}
```

* 使用use指定相对路径

```rs
use first_of_house::hosting;
```

> 使用use

1. 函数:将函数的父级模块引入作用域(指定到父级)
2. struct,enum,其它:指定完整的路径(指定到本身)

   ```rs
   use std::collections::HashMap;
   fn main() {
     let mut map = HashMap::new();
     map.insert(1, 2);
   }
   ```

3. 同名条目:指定到父级

  ```rs
  use std::fmt;
  use std::io;
  fn f1() -> fmt::Result {}
  fn f2() -> io::Result {}
  fn main() {}
  ```

* 或者可以使用`as`重命名

```rs
use std::fmt::Result;
use std::io::Result as IoResult;

fn f1() -> Result {}
fn f2() -> IoResult {}

fn main() {}
```

>使用pub use重新导出名称

* 使用use将路径(名称)导入到作用域内后,该名称在此作用域内是私有的
  * 将条目引入作用域
  * 该条目可以被外部代码引入到他们的作用域

```rs
mod front_of_house {
  pub mod hosting {
    pub fn add_to_waitList() {}
  }
}

//对于当前模块是可见的,对于外部模块是私有的
//use crate::front_of_house::hosting;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
  hosting::add_to_waitList();
}
```

>使用外部包(package)

1. `Cargo.toml`添加依赖的包
2. use将特定条目引入作用域.
   * 例如`rand`的引入
3. 标准库(std0)也会被当作外部包
   * 不需要修改`Cargo.toml`来包含std
   * 需要使用use将std中的特定条目引入到当前作用域

>使用嵌套枯井清理大量的use语句

* 如果使用同一个包或者模块下的多个条目
* 可以使用嵌套路径在同意函内将上述条目进行引入
  * `路径相同的部分::{路径差异的部分}`
* 如果两个use路径知意是另一个的子路径,使用`self`

```rs
use std::{cmp::Ordering,io};

//引入io本身和io下的Write
use std::io::{self, Write};
```

>通配符*,使用`*`,可以将路径中所欲的公共条目都引入作用域(谨慎,测试或者与导入(predule))

```rs
use std::colloections::*;
```

### 将模块内容移到其它文件

* 模块定义时,如果模块名后边是`;`,而不是代码块,rust会从域模块同名的文件中加载内容
  * 模块树的结构不会变
* 随着模块的变大,该技术可以让你把模块的内容移到其他文件夹

```rs
//front_of_house/hosting.rs
pub fn add_to_waitlist(){
    println!("add_to_waitlist");
}

//front_of_house.rs
pub mod hosting;

//lib.rs
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_resturant(){
  hosting::add_to_waitlist();
}
```
