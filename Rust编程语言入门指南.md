# Rust编程语言入门指南

## 1. Rust简介

Rust是一种系统编程语言，专注于**性能、安全和并发**。它结合了：

- C/C++级别的性能控制
- 现代类型系统防止常见错误
- 无垃圾收集器的内存安全
- 零成本抽象

**核心特点**：

- **所有权系统**：编译时保证内存安全
- **借用检查器**：防止数据竞争
- **模式匹配**：强大的控制流工具
- **零成本抽象**：高级特性不引入运行时开销

## 2. 安装与环境

**<mark>attention：具体操作请参考RUST官网</mark>**[Rust 程序设计语言](https://www.rust-lang.org/zh-CN/)

### 安装Rust

bash

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs  | sh
```

### 验证

```
rustc --version
```



### 配置VScode

![](C:\Users\ASUS\AppData\Roaming\marktext\images\2025-07-31-11-27-15-image.png)

### 创建第一个项目

bash

```
cargo new hello_world
cd hello_world
```

查看生成的`src/main.rs`：

rust

```
fn main() {
    println!("Hello, world!");
}
```

运行程序：

bash

```
cargo run
```

## 3. 基本概念

### 变量与可变性

rust

```
let x = 5;          // 不可变变量
let mut y = 10;      // 可变变量
y = 15;
```

### 数据类型

| 类型       | 示例                      | 说明        |
| -------- | ----------------------- | --------- |
| `i32`    | `42`                    | 32位有符号整数  |
| `f64`    | `3.14`                  | 64位浮点数    |
| `bool`   | `true`                  | 布尔值       |
| `char`   | `'A'`                   | Unicode字符 |
| `&str`   | `"hello"`               | 字符串切片     |
| `String` | `String::from("world")` | 可增长字符串    |

### 函数

rust

```
fn add(a: i32, b: i32) -> i32 {
    a + b  // 注意：无分号表示返回值
}
```

## 4. 所有权系统

### 所有权规则

1. 每个值有唯一所有者
2. 值离开作用域时自动释放
3. 所有权可转移（移动）

### 示例

rust

```
let s1 = String::from("hello");
let s2 = s1;  // s1的所有权移动到s2
// println!("{}", s1); // 错误！s1不再有效
```

## 5. 引用与借用

### 不可变引用

rust

```
fn calculate_length(s: &String) -> usize {
    s.len()
}
```

### 可变引用

rust

```
fn change(s: &mut String) {
    s.push_str(", world!");
}
```

### 借用规则

1. 同一时间只能有一个可变引用或多个不可变引用
2. 引用不能超过被引用值的生命周期

## 6. 结构体与方法

### 定义结构体

rust

```
struct Rectangle {
    width: u32,
    height: u32,
}
```

### 实现方法

rust

```
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

## 7. 枚举与模式匹配

### 定义枚举

rust

```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
}
```

### 模式匹配

rust

```
match msg {
    Message::Quit => println!("退出"),
    Message::Move { x, y } => println!("移动到 ({}, {})", x, y),
    Message::Write(text) => println!("文本: {}", text),
}
```

## 8. 错误处理

### Result类型

rust

```
use std::fs::File;

fn main() -> std::io::Result<()> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

### 错误传播

rust

```
fn read_file() -> Result<String, std::io::Error> {
    let mut s = String::new();
    std::fs::File::open("file.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

## 9. 模块系统

### 创建模块

rust

```
mod sound {
    pub fn play() {
        println!("播放声音!");
    }
}

fn main() {
    sound::play();
}
```

### 使用外部crate

在`Cargo.toml`中添加：

toml

```
[dependencies]
rand = "0.8.5"
```

在代码中使用：

rust

```
use rand::Rng;

fn main() {
    let num = rand::thread_rng().gen_range(1..101);
    println!("随机数: {}", num);
}
```

## 10. 并发编程

### 线程创建

rust

```
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("线程: {}", i);
        }
    });

    handle.join().unwrap();
}
```

### 通道通信

rust

```
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        tx.send("消息").unwrap();
    });

    println!("收到: {}", rx.recv().unwrap());
}
```

## 学习资源推荐

1. Rust圣经中文翻译（非官方）[关于本书 - Rust语言圣经(Rust Course)](https://course.rs/about-book.html)
2. 《Programming Rust, 2nd Edition》
3. Rustlings练习
4. Rust Playground

> "Rust将系统编程的安全性与现代语言的开发体验完美结合。" - Rust核心开发者

![Rust入门指南.png](C:\Users\ASUS\Downloads\Rust入门指南.png)
