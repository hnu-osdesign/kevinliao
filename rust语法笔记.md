# rust学习笔记  
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

1. trait在大括号后提供方法签名，以分号结尾,同一个trait可以又多个方法：
```
pub trait Summary {
	fn summarize(&self) -> String;
}
```
2. 为类型实现trait：
```
# pub trait Summary {
# 	fn summarize(&self) -> String;
# }
#
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


### trait作为返回值


#闭包

我们希望能够在程序的一个位置指定某些代码，并只在程序的某处实际需要结果的时候执行这些代码。这正是闭包的用武之地！

```
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

定义一个闭包并储存到变量 expensive_closure 中  

闭包定义是 expensive_closure 赋值的`=`之后的部分。闭包的定义以一对竖线`（|）`开始，在竖线中指定闭包的参数；这个闭包有一个参数 num；如果有多于一个参数，可以使用逗号分隔，比如` |param1, param2|`。
参数之后是存放闭包体的大括号 —— 如果闭包体只有一行则大括号是可以省略的。  

注意这个 let 语句意味着 expensive_closure 包含一个匿名函数的定义，不是调用匿名函数的返回值.

如果尝试对同一闭包使用不同类型则会得到类型错误