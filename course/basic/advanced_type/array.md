---
outline: deep
---

# 数组

数组是日常敲代码使用相当频繁的类型之一，在 zig 中，数组的分配和 C 类似，均是在内存中连续分配且固定数量的相同类型元素。

因此数组有以下三点特性：

- 长度固定
- 元素必须有相同的类型
- 依次线性排列

## 创建数组

在 zig 中，你可以使用以下的方法，来声明并定义一个数组：

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    const message = [5]u8{ 'h', 'e', 'l', 'l', 'o' };
    // const message = [_]u8{ 'h', 'e', 'l', 'l', 'o' };
    print("{s}\n", .{message}); // hello
    print("{s}\n", .{message[0]}); // h
}
```

以上代码展示了定义一个字面量数组的方式，其中你可以选择指明数组的大小或者使用 `_` 代替。使用 `_` 时，zig 会尝试自动计算数组的长度。

数组元素是连续放置的，故我们可以使用下标来访问数组的元素，下标索引从 `0` 开始！

关于[越界问题](https://ziglang.org/documentation/0.11.0/#Index-out-of-Bounds)，zig 在编译期和运行时均有完整的越界保护和完善的堆栈错误跟踪。

### 多维数组

多维数组（矩阵）实际上就是嵌套数组，我们很容易就可以创建一个多维数组出来：

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    const matrix_4x4 = [4][4]f32{
        [_]f32{ 1.0, 0.0, 0.0, 0.0 },
        [_]f32{ 0.0, 1.0, 0.0, 1.0 },
        [_]f32{ 0.0, 0.0, 1.0, 0.0 },
        [_]f32{ 0.0, 0.0, 0.0, 1.0 },
    };

    for (matrix_4x4, 0..) |arr_val, arr_index| {
        for (arr_val, 0..) |val, index| {
            print("元素{}-{}是: {}\n", .{ arr_index, index, val });
        }
    }
}
```

在以上的示例中，我们使用了 [for](/basic/process_control/loop) 循环，来进行矩阵的打印，关于循环我们放在后面再聊。

## 哨兵数组

> 很抱歉，这里的名字是根据官方的文档直接翻译过来的，原文档应该是 ([Sentinel-Terminated Arrays](https://ziglang.org/documentation/0.11.0/#toc-Sentinel-Terminated-Arrays)) 。

我们使用语法 `[N:x]T` 来描述一个元素为类型 `T`，长度为 `N` 的数组，在它对应 `N` 的索引处的值应该是 `x`。前面的说法可能比较复杂，换种说法，就是这个语法表示数组的长度索引处的元素应该是 `x`，具体可以看下面的示例：

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    const array = [_:0]u8{ 1, 2, 3, 4 };
    print("数组长度为: {}\n", .{array.len}); // 4
    print("数组最后一个元素值: {}\n", .{array[array.len - 1]}); // 4
    print("哨兵值为: {}\n", .{array[array.len]}); // 0
}
```

:::info 🅿️ 提示

注意：只有在使用哨兵时，数组才会有索引为数组长度的元素！

:::

## 操作

### 乘法

可以使用 `**` 对数组做乘法操作，运算符左侧是数组，右侧是倍数，进行矩阵的叠加。

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    const small = [3]i8{ 1, 2, 3 };
    const big: [9]i8 = small ** 3;
    print("{any}\n", .{big});// [9]i8{ 1, 2, 3, 1, 2, 3, 1, 2, 3 }
}
```

### 串联

数组之间可以使用 `++` 进行串联操作（编译期），只要两个数组类型（长度、元素类型）相同，它们就可以串联！

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    const part_one = [_]i32{ 1, 2, 3, 4 };
    const part_two = [_]i32{ 5, 6, 7, 8 };
    const all_of_it = part_one ++ part_two;// [_]i32{ 1, 2, 3, 4, 5, 6, 7, 8 }

    _ = all_of_it;
}
```

## 字符串

我们放在这里讲解字符串，为什么呢？

> 在 zig 中，字符串是没有独立的类型的，它就是 `u8` 的数组类型，而且是哨兵字符为 0(`null`) 的哨兵数组！

zig 会将字符串假定为 UTF-8 编码，这是由于 zig 的源文件本身就是 UTF-8 编码的，任何的非 ASCII 字节均会被作为 UTF-8 字符看待。编译器还不会对字节进行修改，因此如果想把非 UTF-8 字节放入字符串中，可以使用转义 `\xNN`。

Unicode 码点字面量类型是 `comptime_int`，所有的转义字符均可以在字符串和 Unicode 码点中使用。

为了方便处理 UTF-8 和 Unicode ，zig的标准库 `std.unicode` 中实现了相关的函数来处理它们。

可以参照以下示例：

```zig
const print = @import("std").debug.print;
const mem = @import("std").mem; // 用于比较字节

pub fn main() void {
    const bytes = "hello";
    print("{}\n", .{@TypeOf(bytes)});                   // *const [5:0]u8
    print("{d}\n", .{bytes.len});                       // 5
    print("{c}\n", .{bytes[1]});                        // 'e'
    print("{d}\n", .{bytes[5]});                        // 0
    print("{}\n", .{'e' == '\x65'});                    // true
    print("{d}\n", .{'\u{1f4a9}'});                     // 128169
    print("{d}\n", .{'💯'});                            // 128175
    print("{u}\n", .{'⚡'});
    print("{}\n", .{mem.eql(u8, "hello", "h\x65llo")});      // true
    print("{}\n", .{mem.eql(u8, "💯", "\xf0\x9f\x92\xaf")}); // true
    const invalid_utf8 = "\xff\xfe";      // 非UTF-8 字符串可以使用\xNN.
    print("0x{x}\n", .{invalid_utf8[1]}); // 索引它们会返回独立的字节
    print("0x{x}\n", .{"💯"[1]});
}
```

### 转义字符

| 转义字符     | 含义                                           |
| ------------ | ---------------------------------------------- |
| `\n`         | 换行                                           |
| `\r`         | 回车                                           |
| `\t`         | 制表符 Tab                                     |
| `\\`         | 反斜杠 \                                       |
| `\'`         | 单引号                                         |
| `\"`         | 双引号                                         |
| `\xNN`       | 十六进制八位字节值，2 位                       |
| `\u{NNNNNN}` | 十六进制 Unicode 码点 UTF-8 编码，1 位或者多位 |

### 多行字符串

如果要使用多行字符串，可以使用 `\\`，多行字符串没有转义，最后一行行尾的换行符号不会包含在字符串中。示例如下：

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    const hello_world_in_c =
        \\#include <stdio.h>
        \\
        \\int main(int argc, char **argv) {
        \\    printf("hello world\n");
        \\    return 0;
        \\}
    ;
    print("{s}\n", .{hello_world_in_c});
}
```

## 奇技淫巧

受益于 zig 自身的语言特性，我们可以实现某些其他语言所不具备的方式来操作数组。

### 使用函数初始化数组

可以使用函数来初始化数组，函数要求返回一个数组的元素或者一个数组。

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    var array = [_]i32{make(3)} ** 10;
    print("{any}\n", .{array});
}

fn make(x: i32) i32 {
    return x + 1;
}
```

### 编译期初始化数组

通过编译期来初始化数组，以此来抵消运行时的开销！

```zig
const print = @import("std").debug.print;

pub fn main() !void {
    var fancy_array = init: {
        var initial_value: [10]usize = undefined;
        for (&initial_value, 0..) |*pt, i| {
            pt.* = i;
        }
        break :init initial_value;
    };
    print("{any}\n", .{fancy_array});
}
```

这个示例中，我们使用了编译期的功能，来帮助我们实现这个数组的初始化，同时还利用了 `blocks` 和 `break` 的性质，关于这个我们会在 [循环](/basic/process_control/loop) 讲解！
