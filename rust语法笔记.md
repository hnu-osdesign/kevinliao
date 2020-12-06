# rust学习笔记  
<<<<<<< HEAD

## 属性

```rust
#[repr(align(n))]  指定结构体在内存中以n对齐
```



=======
## 方法  
1. 可以在迭代器上调用`collect`方法将其转换为一个集合  
2. unwrap()和expect（）方法，在Result为Err时调用panic！宏，expect可以指定错误信息。
>>>>>>> 635138f3faf6bb690069692b2a8297c335734d5b
## string  

### 字符串索引
1. Rust 的字符串不支持索引
2. 因为一个字符串字节值的索引并不总是对应一个有效的标量值。  
3. 可以使用 [] 和一个 range 来创建含特定字节的字符串 slice。  
```
let hello = "Здравствуйте";
let s = &hello[0..4]; 
```
### 遍历字符串


1. 调用chars方法会返回char类型的值。
```
for c in "这是个字符串".chars() {
println!("{}", c);
}
```
这些代码会打印出如下内容：  
```
这
是
个
字
符
串
```
2. bytes方法返回每一个原始字节。
```
for b in "这是个字符串".bytes() {
println!("{}", b);
}
```
这些代码会打印出组成 String 的字节：  
```
232
191
--snip
175
149
```

## hash map	


1. HashMap<K, V> 类型储存了一个键类型 K 对应一个值类型 V 的映射
2. `use std::collections::HashMap;`
3. 使用collect方法合并元组称为hashmap
4. string进入hashmap转移所有权 
5. `scores.entry(String::from("Blue")).or_insert(50);`没有则插入

## 泛型


1. 多个泛型参数：
```
struct Point<T, U> {
x: T,
y: U,
}
fn main() {
let both_integer = Point { x: 5, y: 10 };
let both_float = Point { x: 1.0, y: 4.0 };
let integer_and_float = Point { x: 5, y: 4.0 };
}
```

## trait 

### trait基本语法
1. trait在大括号后提供方法签名，以分号结尾,同一个trait可以又多个方法：
```
pub trait Summary {
	fn summarize(&self) -> String;
}
```
2. 为类型实现trait：
```
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
3. 调用方法:  
```
let tweet = Tweet {
	username: String::from("horse_ebooks"),
	content: String::from("of course, as you probably already know, people"),
	reply: false,
	retweet: false,
};
println!("1 new tweet: {}", tweet.summarize());
```
4. 默认实现，在trait中定义：  
```
pub trait Summary {
	fn summarize(&self) -> String {
	String::from("(Read more...)")
	}
}
```
默认实现中可以调用别的trait方法。

### trait作为参数
```
pub fn notify<T: Summary>(item: T) {
	println!("Breaking news! {}", item.summarize());
}
```
或者使用where从句
```
fn some_function<T, U>(t: T, u: U) -> i32
	where T: Display + Clone,
	U: Clone + Debug
{
```
### trait作为返回值
```
fn returns_summarizable() -> impl Summary {
	Tweet { username: String::from("horse_ebooks"), content: String::from("people"), reply: false, retweet: false,
	}
}
```
通过使用impl Summary作为返回值类型，我们指定了returns_summarizable函数返回某个实现了Summary trait的类型，但是不确定其具体的类型。

## 生命周期  
### 基本语法
生命周期参数名称必须以撇号（ '）开头。  
当在函数中使用生命周期注解时，这些注解出现在函数签名中，而不存在于函数体中的任何代码中。  
```
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```
我们通过生命周期参数告诉Rust的是： longest函数返回的引用的生命周期应该与传入参数的生命周期中较短那个保持一致.  
### 结构体定义中的生命周期注解
```
struct ImportantExcerpt<'a> {
	part: &'a str,
}
fn main() {
	let novel = String::from("Call me Ishmael. Some years ago...");
	let first_sentence = novel.split('.')
		.next()
		.expect("Could not find a '.'");
	let i = ImportantExcerpt { part: first_sentence };
}
```
这个注解意味着ImportantExcerpt的实例不能比其part字段中的引用存在的更久。
### 声明周期省略  
如果编译器检查完以下三条规则后仍然存在没有计算出生命周期的引用，编译器将会停止并生成错误。这些规则适用于fn定义，以及impl块。  
- 第一条规则是每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数： `fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数， `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
- 第二条规则是如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：` fn foo<'a>(x: &'a i32) -> &'a i32`。
- 第三条规则是如果方法有多个输入生命周期参数，不过其中之一因为方法的缘故为&self或&mut self，那么self的生命周期被赋给所有输出生命周期参数。
```
use std::fmt::Display;
fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
where T: Display
{
	println!("Announcement! {}", ann);
	if x.len() > y.len() {
		x
	} else {
		y
	}
}
```

# 闭包  
## 闭包的基本语法  



1. 我们希望能够在程序的一个位置指定某些代码，并只在程序的某处实际需要结果的时候执行这些代码。这正是闭包的用武之地！
```
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```
定义一个闭包并储存到变量 expensive_closure 中。闭包定义是 expensive_closure 赋值的`=`之后的部分。闭包的定义以一对竖线`（|）`开始，在竖线中指定闭包的参数；这个闭包有一个参数 num；如果有多于一个参数，可以使用逗号分隔，比如` |param1, param2|`。  
参数之后是存放闭包体的大括号 —— 如果闭包体只有一行则大括号是可以省略的。注意这个 let 语句意味着 expensive_closure 包含一个匿名函数的定义，不是调用匿名函数的返回值.  

1. 如果尝试对同一闭包使用不同类型则会得到类型错误。例如：  
```
let example_closure = |x| x;
let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

## 带有泛型和 Fn trait 的闭包

`Fn `系列 trait 由标准库提供。所有的闭包都实现了` trait Fn` 、` FnMut` 或 `FnOnce` 中的一个。
```
struct Cacher<T>
where T: Fn(u32) -> u32
{
	calculation: T,
	value: Option<u32>,
}
```
定义一个 Cacher 结构体来在 calculation 中存放闭包并在 value 中存放Option 值

## 闭包会捕获其环境
1. 他们可以捕获其环境并访问其被定义的作用域的变量
```
let x = 4;
let equal_to_x = |z| z == x;///编译能通过
let y = 4;
assert!(equal_to_x(y));
```
2. 三种捕获环境的方式  
	- `FnOnce` 消费从周围作用域捕获的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。  
	- `FnMut` 获取可变的借用值所以可以改变其环境  
	- `Fn` 从其环境获取不可变的借用值  
1. 如果你希望强制闭包获取其使用的环境值的所有权，可以在参数列表前使用 move 关键字。
这个技巧在将闭包传递给新线程以便将数据移动到新线程中时最为实用。
```
let x = vec![1, 2, 3];
let equal_to_x = move |z| z == x;
println!("can't use x here: {:?}", x);///不能编译，x所有权被移交。
let y = vec![1, 2, 3];
assert!(equal_to_x(y));
```

## 迭代器

### 基本语法
1. 迭代器（iterator）负责遍历序列中的每一项和决定序列何时结束的逻辑。在Rust中，迭代器是惰性的（lazy），这意味着在调用方法使用迭代器之前它都不会有效果。
```
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
for val in v1_iter {
	println!("Got: {}", val);
}
```
2. 迭代器都实现了一个叫做Iterator的定义于标准库的trait。  
next是Iterator实现者被要求定义的唯一方法。next一次返回迭代器中的一个项，封装在Some中，当迭代器结束时，它返回None。
```
fn iterator_demonstration() {
let v1 = vec![1, 2, 3];
let mut v1_iter = v1.iter();
assert_eq!(v1_iter.next(), Some(&1));
assert_eq!(v1_iter.next(), Some(&2));
assert_eq!(v1_iter.next(), Some(&3));
assert_eq!(v1_iter.next(), None);
}
```
3. iter方法生成一个不可变引用的迭代器。如果我们需要一个获取v1所有权并返回拥有所有权的迭代器，则可以调用into_iter而不是iter。类似的，如果我们希望迭代可变引用，则可以调用iter_mut而不是iter。

### 实现 Iterator trait来创建自定义迭代器

我们已经展示了可以通过在vector上调用`iter`、`into_iter`或`iter_mut`来创建一个迭代器。也可以用标准库中其他的集合类型创建迭代器，比如哈希map。另外，可以实现Iterator trait来创建任何我们希望的迭代器。正如之前提到的，**定义中唯一要求提供的方法就是next方法。**一旦定义了它，就可以使用所有其他由Iterator trait提供的拥有默认实现的方法来创建自定义迭代器了！
```
# struct Counter {
# 	count: u32,
#}
#
impl Iterator for Counter {
	type Item = u32;
	fn next(&mut self) -> Option<Self::Item> {
		self.count += 1;
		if self.count < 6 {
			Some(self.count)
		} else {
			None
		}
	}
}
```
通过定义next方法实现Iterator trait，我们现在就可以使用任何标准库定义的拥有默认实现的Iterator trait方法了，因为他们都使用了next方法的功能。
## 智能指针  
### 概述  
智能指针通常使用结构体实现。智能指针区别于常规结构体的显著特性在于其实现了 `Deref` 和 `Drop` trait。`Deref` trait 允许智能指针结构体实例表现的像引用一样，这样就可以编写既用于引用、又用于智能指针的代码。`Drop` trait 允许我们自定义当智能指针离开作用域时运行的代码。  
### Box <T>  
box 允许你将一个值放在堆上而不是栈上。，box 没有性能损失。同时也没有很多额外的功能。它们多用于如下场景：
- 当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候  
- 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候  
- 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候   
1. 在栈上分配List不确定空间，不能编译。
```
enum List {
    Cons(i32, List),
    Nil,
}
fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```
2. 在堆上分配不确定空间，Box作为一个指针大小usize返回到栈中，可以编译。
```
enum List {
    Cons(i32, Box<List>),
    Nil,
}
fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```  

### 通过 Deref trait 将智能指针当作常规引用处理  
为了实现 trait，需要提供 trait 所需的方法实现。要求实现名为 `deref` 的方法，其借用 self 并返回一个内部数据的引用。  
```
fn main() {
use std::ops::Deref;
struct MyBox<T>(T);
impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}
}
```
现在就可以完成MyBox的解引用了`*y`。
### 使用 Drop Trait 运行清理代码
指定在值离开作用域时应该执行的代码的方式是实现 `Drop` trait。`Drop` trait 要求实现一个叫做 drop 的方法，它获取一个 `self` 的**可变引用**。当实例离开作用域 Rust 会自动调用 `drop`，并调用我们指定的代码。变量以被创建时相反的顺序被丢弃，