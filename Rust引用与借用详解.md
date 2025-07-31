# Rust引用与借用详解

引用与借用是Rust所有权系统的核心组成部分，它们允许程序访问数据而不获取所有权，从而避免不必要的复制和移动操作。这套机制在编译时强制执行内存安全规则，防止数据竞争和悬垂指针。

## 引用基础概念

### 什么是引用？

- 引用是指向数据的指针，但不拥有数据
- 使用 `&` 符号创建不可变引用
- 使用 `&mut` 符号创建可变引用
- 引用有生命周期，不能超过被引用值的生命周期

rust

```
fn main() {
    let s = String::from("hello");
    let r = &s; // 不可变引用
    println!("{}", r);

    let mut s2 = String::from("world");
    let r2 = &mut s2; // 可变引用
    r2.push_str("!");
    println!("{}", r2);
}
```

## 引用类型详解

### 不可变引用（共享引用）

- 允许多个同时存在
- 不能修改被引用的数据
- 语法：`&T`

rust

```
fn calculate_length(s: &String) -> usize {
    s.len() // 可以读取但不能修改
}

fn main() {
    let s = String::from("hello");
    let len = calculate_length(&s);
    println!("字符串长度: {}", len);
}
```

### 可变引用（独占引用）

- 同一时间只能有一个
- 可以修改被引用的数据
- 语法：`&mut T`

rust

```
fn modify_string(s: &mut String) {
    s.push_str(", world!");
}

fn main() {
    let mut s = String::from("hello");
    modify_string(&mut s);
    println!("{}", s); // 输出 "hello, world!"
}
```

## 借用规则（核心原则）

Rust的借用检查器在编译时强制执行以下规则：

1. **共享不可变，可变不共享**
   - 同一作用域内，要么：
     - 多个不可变引用，或
     - 一个可变引用
   - 两者不能同时存在

rust

```
fn main() {
    let mut data = vec![1, 2, 3];

    let r1 = &data;      // 第一个不可变引用
    let r2 = &data;      // 第二个不可变引用，允许
    println!("{:?}, {:?}", r1, r2);

    let r3 = &mut data;  // 尝试创建可变引用
    // 错误！因为存在不可变引用时不能创建可变引用
    r3.push(4);
}
```

2. **引用必须始终有效**
   - 被引用的值不能比引用更早被销毁
   - 编译器通过生命周期确保这一点

rust

```
fn main() {
    let r;
    {
        let x = 5;
        r = &x; // 错误！x的生命周期小于r
    }
    println!("{}", r);
}
```

## 引用解引用操作

### 显式解引用

使用 `*` 运算符访问引用指向的值：

rust

```
fn main() {
    let mut x = 10;
    let r = &mut x;
    *r += 1; // 显式解引用并修改值
    println!("x = {}", x); // 输出 11
}
```

### 隐式解引用

`.` 运算符自动解引用：

rust

```
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 10, y: 20 };
    let r = &p;
    println!("x = {}, y = {}", r.x, r.y); // 自动解引用
}
```

## 引用与函数

### 函数参数引用

rust

```
fn print_length(s: &String) {
    println!("长度: {}", s.len());
}

fn main() {
    let s = String::from("hello");
    print_length(&s);
}
```

### 函数返回引用

需要明确指定生命周期：

rust

```
fn longest<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() > s2.len() { s1 } else { s2 }
}

fn main() {
    let s1 = String::from("hello");
    let s2 = "world";
    let result = longest(&s1, s2);
    println!("最长的字符串是: {}", result);
}
```

## 切片：特殊的引用类型

切片是对集合中连续元素的引用，不拥有数据：

rust

```
fn first_word(s: &str) -> &str {
    for (i, &item) in s.as_bytes().iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

fn main() {
    let s = String::from("hello world");
    let word = first_word(&s);
    println!("第一个单词: {}", word);
}
```

## 引用与模式匹配

### 引用模式

rust

```
fn main() {
    let point = &(10, 20);

    match point {
        &(x, y) => println!("坐标: ({}, {})", x, y),
    }
}
```

### ref 关键字

在模式匹配中避免移动值：

rust

```
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 10, y: 20 };

    match point {
        Point { ref x, ref y } => {
            println!("坐标: ({}, {})", x, y);
            // point 仍然有效，因为使用了ref
        }
    }
}
```

## 引用与并发安全

所有权系统天然防止数据竞争：

rust

```
use std::thread;

fn main() {
    let mut data = vec![1, 2, 3];

    // 错误！尝试在多个线程中共享可变引用
    thread::spawn(|| {
        data.push(4); // 捕获data
    });

    thread::spawn(|| {
        data.push(5); // 再次捕获data
    });
}
```

正确做法是使用线程安全类型：

rust

```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));

    for i in 0..3 {
        let data_clone = Arc::clone(&data);
        thread::spawn(move || {
            let mut data = data_clone.lock().unwrap();
            data.push(i + 4);
        });
    }

    thread::sleep(std::time::Duration::from_secs(1));
    println!("{:?}", data.lock().unwrap());
}
```

## 引用使用最佳实践

1. **优先使用不可变引用**
   
   - 除非需要修改数据，否则使用 `&T`
   - 提高代码可读性和安全性

2. **限制可变引用作用域**
   
   rust
   
   ```
   fn main() {
      let mut s = String::from("hello");
   
      {
          let r = &mut s;
          r.push_str(", world");
      } // r的作用域结束
   
      let r2 = &s; // 现在可以创建不可变引用
      println!("{}", r2);
   }
   ```

3. **使用数据结构避免借用冲突**
   
   rust
   
   ```
   struct Data {
      a: i32,
      b: i32,
   }
   
   fn main() {
      let mut data = Data { a: 1, b: 2 };
      let a_ref = &mut data.a;
      let b_ref = &mut data.b; // 允许！不同字段
   
      *a_ref += 1;
      *b_ref += 2;
   }
   ```

## 引用与C/C++指针的区别

| 特性   | Rust引用   | C/C++指针         |
| ---- | -------- | --------------- |
| 空值   | 永远不为空    | 可以为NULL/nullptr |
| 悬垂   | 编译时防止    | 运行时错误           |
| 别名   | 严格规则     | 无限制             |
| 算术   | 不支持      | 支持              |
| 转换   | 需要unsafe | 自由转换            |
| 生命周期 | 编译时检查    | 无检查             |

## All In All

Rust的引用与借用系统：

1. 通过编译时检查保证内存安全
2. 防止数据竞争和悬垂指针
3. 允许高效访问数据而不转移所有权
4. 支持不可变共享和可变独占两种模式
5. 与所有权系统协同工作，提供零成本抽象



![Rust引用与借用详解.png](C:\Users\ASUS\Downloads\Rust引用与借用详解.png)
