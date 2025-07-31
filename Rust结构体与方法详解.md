# Rust结构体与方法详解

结构体是Rust中创建自定义数据类型的基础工具，而方法是与这些类型关联的函数。它们共同构成了Rust面向对象编程的核心，提供了封装数据和行为的强大机制。

## 结构体基础

### 结构体定义

结构体使用`struct`关键字定义，包含命名字段：

rust

```
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

### 结构体实例化

rust

```
let user1 = User {
    email: String::from("user@example.com"),
    username: String::from("user123"),
    active: true,
    sign_in_count: 1,
};
```

### 字段访问

rust

```
println!("用户名: {}", user1.username);
user1.sign_in_count += 1; // 需要mut关键字
```

## 结构体类型

### 命名字段结构体

最常见的形式，每个字段都有名称：

rust

```
struct Point {
    x: f64,
    y: f64,
}
```

### 元组结构体

字段没有名称，只有类型：

rust

```
struct Color(u8, u8, u8);
let white = Color(255, 255, 255);
```

### 单元结构体

没有字段的结构体：

rust

```
struct Empty;
let e = Empty;
```

## 方法定义

### 基本方法

使用`impl`块为结构体定义方法：

rust

```
impl Rectangle {
    // 关联函数（类似静态方法）
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }

    // 实例方法
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // 可变方法
    fn resize(&mut self, width: u32, height: u32) {
        self.width = width;
        self.height = height;
    }
}
```

### 方法调用

rust

```
let mut rect = Rectangle::new(30, 50);
println!("面积: {}", rect.area());
rect.resize(40, 60);
```

## 方法参数详解

### self的不同形式

| 形式          | 含义    | 所有权   |
| ----------- | ----- | ----- |
| `self`      | 获取所有权 | 转移所有权 |
| `&self`     | 不可变借用 | 只读访问  |
| `&mut self` | 可变借用  | 可修改访问 |

### 示例对比

rust

```
impl Rectangle {
    // 消耗结构体实例
    fn consume(self) -> u32 {
        self.width * self.height
    }

    // 只读访问
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // 修改结构体
    fn double_size(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
}
```

## 关联函数

### 构造函数模式

rust

```
impl User {
    fn new(email: String, username: String) -> User {
        User {
            email,
            username,
            active: true,
            sign_in_count: 0,
        }
    }
}
```

### 使用示例

rust

```
let user = User::new(
    String::from("new@example.com"),
    String::from("newuser")
);
```

## 泛型结构体

### 定义泛型结构体

rust

```
struct Pair<T> {
    first: T,
    second: T,
}
```

### 泛型方法

rust

```
impl<T> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Pair { first, second }
    }
}
```

### 特定类型实现

rust

```
impl Pair<f32> {
    fn distance(&self) -> f32 {
        (self.first - self.second).abs()
    }
}
```

## 派生特征

### 自动派生常见trait

rust

```
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

### 可派生trait列表

| Trait        | 功能    | 要求               |
| ------------ | ----- | ---------------- |
| `Debug`      | 格式化输出 | 所有字段实现Debug      |
| `Clone`      | 创建深拷贝 | 所有字段实现Clone      |
| `Copy`       | 按位复制  | 所有字段实现Copy       |
| `PartialEq`  | 相等比较  | 所有字段实现PartialEq  |
| `Eq`         | 完全相等  | 所有字段实现Eq         |
| `PartialOrd` | 比较排序  | 所有字段实现PartialOrd |
| `Ord`        | 完全排序  | 所有字段实现Ord        |
| `Hash`       | 哈希计算  | 所有字段实现Hash       |
| `Default`    | 默认值   | 所有字段实现Default    |

## 高级结构体特性

### 结构体更新语法

rust

```
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotheruser"),
    ..user1 // 复制其余字段
};
```

### 字段初始化简写

rust

```
fn build_user(email: String, username: String) -> User {
    User {
        email,    // 字段名与变量名相同时可简写
        username, // 同上
        active: true,
        sign_in_count: 1,
    }
}
```

## 内部可变性模式

### Cell的使用

rust

```
use std::cell::Cell;

struct Counter {
    count: Cell<u32>,
}

impl Counter {
    fn increment(&self) {
        let current = self.count.get();
        self.count.set(current + 1);
    }
}
```

### RefCell的使用

rust

```
use std::cell::RefCell;

struct Logger {
    log: RefCell<Vec<String>>,
}

impl Logger {
    fn add_entry(&self, message: &str) {
        self.log.borrow_mut().push(message.to_string());
    }
}
```

## 结构体与所有权

### 字段所有权

rust

```
struct User {
    username: String, // User拥有username的所有权
    email: String,    // User拥有email的所有权
}
```

### 包含引用

需要指定生命周期：

rust

```
struct Excerpt<'a> {
    part: &'a str,
}
```

## 结构体最佳实践

1. **优先使用命名字段结构体**
   
   - 提高代码可读性
   - 明确字段含义

2. **保持结构体小巧**
   
   - 理想情况下不超过5-7个字段
   - 复杂数据拆分为多个结构体

3. **合理使用方法**
   
   - 将与类型紧密相关的操作定义为方法
   - 通用操作定义为独立函数

4. **明智使用泛型**
   
   - 在需要多种类型支持时使用
   - 避免过度泛型化导致代码复杂

## 结构体内存布局

### 示例分析

rust

```
struct Point {
    x: f64,
    y: f64,
}
```

内存布局：

```
+------+------+
|  x   |  y   |
| 8字节 | 8字节 |+------+------+
```

### 优化布局

rust

```
struct OptimizedPoint {
    x: i32, // 4字节
    y: i32, // 4字节
}
```

内存占用减少50%，提高缓存效率



# All In All

Rust的结构体与方法提供了：

1. **数据封装**：将相关数据组织在一起
2. **行为关联**：通过方法定义类型相关操作
3. **类型安全**：编译时检查字段访问
4. **内存效率**：控制内存布局和分配
5. **灵活性**：支持泛型和各种设计模式



![Rust结构体与方法详解.png](C:\Users\ASUS\Downloads\Rust结构体与方法详解.png)
