# 附录 B. `printf()` 格式说明符细节

`printf()` 函数支持的格式几乎构成了它们自己的语言。 虽然不是详尽的列表，本附录详细描述了我在本书中使用的所有选项。 我还描述了这些选项如何与不同类型的输出配合工作，即使我没有使用某个组合。 就像编程中的许多内容一样，自己尝试一下是很有用的，以便了解各部分如何配合。

代码示例包括一个简单的 C 程序，演示了更流行的标志、宽度、精度和类型组合。 您可以按原样编译和运行 *popular_formats.c*，或者您可以编辑它以调整某些行并测试自己的组合。

如果您想更多了解您可以在 `printf()` 中指定的事项，包括非标准和特定于实现的选项，我建议查看专门介绍[此主题](https://oreil.ly/Adirl)的维基百科页面。

# 说明符语法

我在本书中使用的说明符包含三个可选元素和一个必需的类型，排列如下：

```cpp
% flag(s) width . precision type
```

再次强调，并不需要标志（或标志）、宽度和精度。

## 说明符类型

当使用 `printf()` 时，打印给定值的解释取决于您使用的类型说明符。 例如，值 65 使用 `%c`（字符）会打印为字母“A”，但使用 `%x`（十六进制整数）会打印为“41”。 表 B-1 概述了本书中使用的类型，尽管这不是详尽的列表。

表 B-1\. `printf()` 的格式说明符类型

| 说明符 | 类型 | 描述 |
| --- | --- | --- |
| `%c` | `char` | 打印单个字符 |
| `%d` | `char, int, short, long` | 以十进制（基数 10）打印有符号整数值 |
| `%f` | `float, double` | 打印浮点数值 |
| `%i` | `char, int, short` | 以十进制（与 `%d` 相同的输出）打印整数值 |
| `%o` | `int, short, long` | 以八进制（基数 8）打印整数值 |
| `%p` | 地址 | 打印指针（作为十六进制地址） |
| `%s` | `char[]` | 将字符串（char 数组）打印为文本 |
| `%u` | `unsigned (char, int, short)` | 以十进制打印无符号整数值 |
| `%x` | `char, int, short, long` | 以十六进制（基数 16）打印整数值 |

`%i` 和 `%u` 整数类型可以使用长度修饰符。 `l` 或 `ll`（例如 `%li` 或 `%llu`）告诉 `printf()` 期望 `long` 或 `long long` 长度的参数。 对于浮点类型，可以使用 `L`（例如 `%Ld`）来指示 `long double` 参数。

## 说明符标志

您指定的每种类型都可以使用一个或多个标志进行修改。 表 B-2 列出了说明符标志。 并非所有标志对每种类型都有效，所有标志都是可选的。

表 B-2\. `printf()` 的格式说明符标志

| 说明符 | 描述 |
| --- | --- |
| `-` | 在其字段内左对齐输出 |
| `+` | 强制在正数值前加上加号（`+`）前缀 |
| `(space)` | 强制在正数值前加空格前缀（与完全没有前缀相对应）。 |
| `0` | 在数值左侧用 0 填充（如果它们没有填满指定宽度的字段）。 |
| `#` | 在与 `o`、`x` 或 `X` 类型一起使用时打印前缀（`0`、`0x` 或 `0X`）。 |

当你有数值、列式输出时，更常见使用这些标志，尽管像“-”这样的标志也可以用在字符串上。

## 宽度和精度

对于任何说明符，你可以为输出字段提供一个最小宽度。（最小限定符意味着对于大于给定宽度的值不会发生截断。）默认是右对齐输出，但可以通过在表 B-2 中提到的`-`标志进行更改。

你也可以提供一个精度，这会影响输出的最大宽度。对于浮点类型，它指定小数点右侧的数字位数。对于字符串，它会截断过长的值。对于整数类型，它会被忽略。

# 常见格式

若要查看一些更常见或流行的格式的实际应用，请看一下[*appB/popular_formats.c*](https://oreil.ly/R2vNI)。这只是一个大批量的 `printf()` 调用，但它包含了使用不同格式说明符的各种示例。我在这里不会列出源代码，但输出可供快速参考：

```cpp
appB$ gcc popular_formats.c
appB$ ./a.out
char Examples
  Simple char:             %c       |y|
  In a 9-char field:       %9c      |        y|
  Left, 9-char field:      %-9c     |y        |
  The percent sign:        %%       |%|

int Examples
  Simple int:              %i       |76|
  Simple decimal int:      %d       |76|
  Simple octal int:        %o       |114|
  Prefixed octal int:      %#o      |0114|
  Simple hexadecimal int:  %x       |4c|
  Uppercase hexadecimal:   %X       |4C|
  Prefixed hexadecimal:    %#x      |0x4c|
  Prefixed uppercase:      %#X      |0X4C|
  In a 9-column field:     %9i      |       76|
  Left, 9-column field:    %-9i     |76       |
  Zeros, 9-column field:   %09i     |000000076|
  With plus prefix:        %+i      |+76|
    negative value:                 |-12|
  With space prefix:       % i      | 76|
     negative value:                |-12|
  Huge number:             %llu     |28054511505742|
  (Ignored) precision:     %1.1d    |76|

float Examples
  Simple float:            %f       |216.289993|
  2 decimal places:        %.2f     |216.29|
  1 decimal place:         %.1f     |216.3|
  No decimal places:       %.0f     |216|
  In a 12-column field:    %12f     |  216.289993|
  2 decimals, 12 columns:  %12.2f   |      216.29|
  Left, 12-column field:   %-12.2f  |216.29      |

string (char[]) Examples
  Simple string:           %s       |Ada Lovelace|
  In a 20-column field:    %20s     |        Ada Lovelace|
  Left, 20-column field:   %-20s    |Ada Lovelace        |
  6-column field:          %6s      |Ada Lovelace|
  6-columns, truncated:    %6.6s    |Ada Lo|

And last but not least, a blank line (\n):
```

Wikipedia 页面上关于[printf 格式字符串](https://oreil.ly/xvtiC)有一个详尽的选项概述。
