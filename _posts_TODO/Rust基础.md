rust基础：

https://colobu.com/2020/03/05/A-half-hour-to-learn-Rust/



# 变量

### 变量声明

##### 使用 `let` 声明，和ts啥的意思都一样，表示变量不可变，类似于final


```rust
// 可以先声明后赋值
let a;
a = 1;

// 等价于
let a = 1;

// 也可以指定类型
let a: i32;
a = 1;

// 等价于
let a: i32 = 1;
```



##### 注意：先声明后赋值的话，在赋值前是不能使用的


```rust
// 在赋值前使用，会报错
let a;
foobar(a); // error: borrow of possibly-uninitialized variable: `a`
a = 1;

// 这样就ok了
let a;
a = 1;
foobar(a);
```



##### 如果想要一个可变变量，需要使用`let mut`

```rust
// 这样是错误的
let a = 1;
a = 2; // error: cannot assign twice to immutable variable

//这样就可以了
let mut a = 1;
a = 2;
```



##### 虽然 `let` 表示的变量不可变，但可以被隐藏(Shadowing)

```rust
// 这样是OK的
let a = 1;
let a = 2;

// 可以修改类型
let a: i32 = 1;
let a: (i32, f64, u8) = (500, 6.4, 1);
```

使用 `let ` 再隐藏和 `mut` 有什么区别？

1. 使用 `let` 能防止对一些不可变变量的误赋值
2. 隐藏实际是重新创建了新的变量，所以可以修改类型



##### 使用 `const` 来声明常量（constants）

常量类似于`static final` ，一般用于全局变量

```rust
// 常量必须声明类型
const AAA = 1; // error: missing type for `const` item

// 这样就OK了
const AAA: i32 = 1;

// 常量不能用动态函数来生成
const AAA: i32 = get_five(); // error: cannot call non-const fn `get_five` in constants

// 可以用常量表达式
const AAA: i32 = 60 * 60 * 3;

// 常量不允许隐藏
const AAA: i32 = 1;
const AAA: i32 = 2; // error: the name `AAA` is defined multiple times
```



##### 变量 `_` ， 以及以 `_` 开头的变量

下划线`_`是一个特殊的名字，和lua，python差不多，场景为：“我拿到你了，但我不需要用”。

为什么要加这个呢？我理解是为了一是为了可读性而搞的代码规范，二是为了减少编译器对未被使用的变量的告警，让开发者能更好的关注有效的告警。

以 `_` 开头的变量名也是同样的使用场景，未被使用的话同样会被编译器跳过。

下面是未使用变量的编译器告警：

```
   |
17 |     let a = 2;
   |         ^ help: if this is intentional, prefix it with an underscore: `_a`
```



### 数据类型

##### 整型

| 长度    | 有符号  | 无符号  |
| ------- | ------- | ------- |
| 8-bit   | `i8`    | `u8`    |
| 16-bit  | `i16`   | `u16`   |
| 32-bit  | `i32`   | `u32`   |
| 64-bit  | `i64`   | `u64`   |
| 128-bit | `i128`  | `u128`  |
| arch    | `isize` | `usize` |

`isize` 和 `usize` 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的，32 位架构上它们是 32 位的。



##### 不同进制，不同写法的整型

| 数字字面值                    | 例子          |
| ----------------------------- | ------------- |
| Decimal (十进制)              | `98_222`      |
| Hex (十六进制)                | `0xff`        |
| Octal (八进制)                | `0o77`        |
| Binary (二进制)               | `0b1111_0000` |
| Byte (单字节字符)(仅限于`u8`) | `b'A'`        |



##### 对付整型溢出的不同处理方法

1. 正常的溢出方式，不检查，最大值+1溢出为最小值。使用 `std::intrinsics::warpping_xxx` ， `xxx` 为操作，比如`add`。
2. 需要手动检查是否有溢出，方法一。使用 `u32::check_add` 等，这个会返回一个 `Optional`。
3. 需要手动检查是否有溢出，方法二。使用`u32::overflowing_add`, 这个会返回 `(u32, boolean)`。
4. 到达边界后，保持最大或者最小值，比如MAX + 10 的时候溢出了，还能返回 MAX。 使用 `u32::saturating_add`。



##### 浮点型

浮点型有两种， `f32` 和 `f64` ， rust 默认使用 `f64`, 因为在现代 CPU 中，它与 `f32` 速度几乎一样，不过精度更高。所有的浮点型都是有符号的。



##### 元组 tuple

```rust
let pair = ('a', 1);
pair.0; // `a`
pair.1; // 1

// 可以忽略类型注解，也可以加上
let pair: (char, i32) = ('a', 1);

// 可以用赋值方法对元组解构，这个用在获取返回tuple的函数的时候特别有用
let (some_char, some_int) = ('a', 1);
some_char; // 'a'
some_int; // 1

// 可以结合上面的_来使用，比如有个get_two的函数，返回(i32, i32)，而我们只需要用第一个，就可以
let (some_int, _) = get_two();
```



##### 数组初始化

```rust
// 省略类型和长度
let a = [1, 2, 3, 4, 5];
let first = a[0];

// 指定类型为i32，长度为5
let a: [i32; 5] = [1, 2, 3, 4, 5];

// 提供默认值的数组， 表示默认值为3， 长度为5
let a = [3; 5];
```



# 函数

##### 简单的函数示例

```rust
// 一个入参
fn another_function(x: i32) {
    println!("The value of x is: {x}");
}

// 两个不同类型入参
fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}

// 返回值，可以不用return
fn get_five1() -> i32 {
    println!("get_five");
    5
}

// 返回值，return
fn get_five2() -> i32 {
    println!("get_five");
    return 5;
}
```

##### 表达式

```rust
    let y = {
        let x = 3;
        x + 1
    };
```



# 控制流

### if表达式

##### 简单的if示例

```rust
    let number = 3;
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }

	// else if
    if number > 5 {
        println!("number is over 5");
    } else if number == 3 {
        println!("number is 3");
    } else {
        println!("number is not 3");
    }
```

##### 三元控制符

```rust
    let condition = true;
    let number = if condition { 5 } else { 6 };
```



### 循环

##### 最简单的循环 - `loop`

```rust
loop {
    println!("again");
}
```

##### 从 `loop` 中获得返回值

```rust
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
}

println!("{counter}");
```

##### 循环标签：内存循环可以直接跳出外层循环

```rust
'loop1: loop {
	loop {
        break 'loop1;
    }
}
```

##### `while` 循环

```rust
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }
```

##### `for` 循环

```rust
    let a = [10, 20, 30, 40, 50];
    for element in a {
        println!("the value is: {element}");
    }

	// 使用range，配合反转。
    // 下面这个会打印 3 2 1
    for number in (1..4).rev() {
        println!("{number}!");
    }
```

