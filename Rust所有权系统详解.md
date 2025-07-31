# Rust所有权系统详解

所有权系统是Rust最核心的特性，它通过编译时检查而非运行时垃圾回收来保证内存安全。这套系统解决了传统系统编程语言中常见的内存管理问题，如空指针解引用、数据竞争和内存泄漏等。

## 所有权三原则

1. **每个值都有唯一的所有者**
   
   - 值在创建时绑定到变量，该变量成为其所有者
   - 当所有者离开作用域时，值被自动销毁

2. **同一时间只能有一个所有者**
   
   - 赋值操作会转移所有权（移动）
   - 原始所有者不能再访问该值

3. **引用必须遵守借用规则**
   
   - 不可变引用：允许多个同时存在
   - 可变引用：同一时间只能有一个，且不能与不可变引用共存

## 所有权转移（移动）

rust

```
fn main() {
    let s1 = String::from("hello"); // s1拥有字符串
    let s2 = s1; // 所有权转移到s2

    // println!("{}", s1); // 错误！s1不再有效
    println!("{}", s2); // 正确
} // s2离开作用域，内存自动释放
```

## 克隆（深拷贝）

rust

```
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone(); // 显式深拷贝

    println!("s1 = {}, s2 = {}", s1, s2); // 两者都有效
}
```

## 栈数据复制（Copy trait）

基本类型实现了Copy trait，赋值时自动复制而非移动：

rust

```
fn main() {
    let x = 5;
    let y = x; // 复制值，x仍有效

    println!("x = {}, y = {}", x, y);
}
```

实现Copy的类型：

- 所有整数类型（i32, u64等）
- 布尔类型（bool）
- 浮点类型（f32, f64）
- 字符类型（char）
- 仅包含Copy类型的元组（如(i32, char)）

## 函数与所有权

函数参数和返回值涉及所有权转移：

rust

```
fn take_ownership(s: String) { // s获得所有权
    println!("{}", s);
} // s被销毁

fn make_ownership() -> String {
    let s = String::from("hello");
    s // 所有权转移给调用者
}

fn main() {
    let s1 = String::from("hello");
    take_ownership(s1); // s1的所有权转移

    let s2 = make_ownership(); // 获得新字符串的所有权
}
```

## 引用与借用

### 不可变引用（共享引用）

rust

```
fn calculate_length(s: &String) -> usize {
    s.len() // 借用s，不获取所有权
}

fn main() {
    let s = String::from("hello");
    let len = calculate_length(&s);
    println!("'{}'的长度是{}", s, len); // s仍有效
}
```

### 可变引用（独占引用）

rust

```
fn change(s: &mut String) {
    s.push_str(", world!");
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s); // 输出"hello, world!"
}
```

### 借用规则

1. **共享不可变，可变不共享**
   
   - 同一作用域内，要么：
     - 多个不可变引用，或
     - 一个可变引用
   - 两者不能同时存在

2. **引用必须始终有效**
   
   - 被引用的值不能比引用更早被销毁

## 切片类型与所有权

切片是对集合中连续元素的引用，不拥有数据：

rust

```
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

fn main() {
    let s = String::from("hello world");
    let word = first_word(&s);
    // s.clear(); // 错误！同时存在可变和不可变引用
    println!("第一个单词: {}", word);
}
```

## 所有权系统优势

1. **内存安全**
   
   - 无悬垂指针
   - 无空指针解引用
   - 无数据竞争

2. **高效性**
   
   - 无运行时垃圾回收开销
   - 编译时检查，零运行时成本

3. **明确资源管理**
   
   - 资源获取即初始化（RAII）模式
   - 自动资源释放

## 常见所有权模式

### 返回所有权

rust

```
fn take_and_give_back(s: String) -> String {
    s // 返回所有权
}
```

### 使用元组返回多个值

rust

```
fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length) // 返回s的所有权
}
```

### 结构体字段所有权

rust

```
struct User {
    username: String, // 结构体拥有字段的所有权
    email: String,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,    // 所有权转移
        username, // 所有权转移
    }
}
```

## 所有权与并发安全

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

正确做法是使用Arc和Mutex：

rust

```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));

    let data_clone = Arc::clone(&data);
    thread::spawn(move || {
        let mut data = data_clone.lock().unwrap();
        data.push(4);
    });

    let data_clone = Arc::clone(&data);
    thread::spawn(move || {
        let mut data = data_clone.lock().unwrap();
        data.push(5);
    });
}
```

# All In All

Rust的所有权系统通过以下机制保证内存安全：

1. **明确的所有权关系**：每个值有唯一所有者
2. **移动语义**：赋值操作转移所有权
3. **借用检查**：严格控制引用规则
4. **生命周期**：确保引用始终有效



![](C:\Users\ASUS\AppData\Roaming\marktext\images\2025-07-31-10-55-49-image.png)
