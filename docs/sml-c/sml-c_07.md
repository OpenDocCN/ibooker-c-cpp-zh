# 第七章 图书馆

C 语言的一大优点是其编译代码中的最小装饰。对于一些更现代的语言如 Java，有一个经典的笑话是关于“Hello, World”程序的大小。我们在“创建 C 语言的‘Hello, World’”中的第一个程序在我的 Linux 机器上占用了略多于 16Kb 的空间，没有进行任何优化。而在同一系统上使用 Java 来实现相同的输出，需要数十兆字节的空间，并且需要更多的工作量来构建。这并不是一个完全公平的比较，因为 Java 的 Hello 应用程序需要整个 Java 运行时嵌入到可执行文件中，但这也正是重点所在：对于特定系统，C 语言可以轻松创建精简的代码。

当我们处理小东西如“Hello, World”或者过去章节中的大多数示例时，这种简易性非常棒。但当我们准备跳入微控制器和 Arduino 的世界时，我们开始担心是否要重新创建一些相当乏味的问题的解决方案。例如，我们编写了一些自己的函数来比较字符串。我们编写了一个更复杂的程序来编码 base64 内容。这些都很有趣，但我们是否总是需要从头开始做这种工作呢？

幸运的是，对于这个问题的答案是：不需要。C 语言支持使用*库*的概念，可以快速友好地扩展其功能，而不会影响最终可执行文件的精简性。库是一组代码的集合，可以导入到您的项目中以添加新功能，比如处理字符串或与无线网络通信。但使用库的关键在于，您只需要添加包含您需要功能的那一个库。相比之下，Java 的 Hello 应用程序可能会包含整个图形界面和网络连接的支持，尽管这些功能在终端窗口中只是打印文本时不会被使用。

例如，使用 Arduino 时，您会发现为大多数流行的传感器如温度组件或光电阻器以及输出如 LED 和 LCD 显示器提供了库。您不需要编写自己的设备驱动程序来使用电子纸或更改 RGB LED 的颜色。您可以加载一个库，并专注于在电子纸上显示什么，而不必担心如何实现。

# C 标准库

在书中我们到达目前位置的路上已经使用了几个库。即使是我们的第一个程序也需要*stdio.h*头文件来访问`printf()`函数。而我们在第六章关于指针的最新工作需要*stdlib.h*头文件中的`malloc()`函数。我们不需要做太多事情就能访问到这些功能。事实上，我们只需在程序顶部写上一个`#include`语句，然后就可以开始工作了！

这些函数之所以如此易于集成，是因为它们属于 C 标准库。每个 C 编译器或开发环境都会提供此库。它在不同平台上的打包方式可能有所不同（例如包含或排除数学函数），但你始终可以指望它的整体内容可以被包含进来。我无法涵盖库中的所有内容，但我确实想要强调一些有用的函数和提供它们的头文件。在 “将其放在一起” 中，我还将介绍在哪里查找处理更广泛功能的其他库。

## stdio.h

从一开始我们就一直在使用 *stdio.h* 头文件。我们已经使用了两个最有用的函数（对我们的目的而言）：`printf()` 和 `scanf()`。该头文件中的其他函数主要围绕文件访问展开。在接下来的章节中，我们将使用的微控制器有时会有文件系统，但我们将编写的程序类型不需要该特定功能。不过，如果你确实想在桌面或高性能微控制器上处理文件，这个头文件是一个很好的起点！

## stdlib.h

我们还看到了 *stdlib.h* 中的几个函数，即 `malloc()` 和 `free()`。但是这个头文件还有一些更有用的技巧值得一提。

### atoi()

在 “命令行参数和 main()” 中，我给了你一个将字符串转换为数字的练习。 “额外积分”注释提到使用 *stdlib.h* 来访问 C 标准的转换函数：`atoi()`。还有另外两个转换函数用于其他基本类型：`atol()` 转换为 `long` 值，而 `atof()` 转换为浮点类型，但与函数名称中的最后一个字母相反，`atof()` 返回一个 `double` 值。（如果需要，你可以将其转换为低精度的 `float` 类型。）

那个额外练习的解决方案，[*ch07/sum2.c*](https://oreil.ly/x8J8O)，突显了如果包含必要的头文件，转换就有多么简单：

```cpp
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
  int total = 0;
  for (int i = 1; i < argc; i++) {
    total += atoi(argv[i]);
  }
  printf("The sum of these %d numbers is %d\n", argc - 1, total);
}
```

非常简单！当然，这也是使用库函数的希望所在。你*可以*自己编写这个转换代码，但如果你能找到一个合适的库函数来使用，你可以节省大量时间（和相当数量的调试）。

###### 警告

在使用这些函数时要小心一些。它们在遇到非数字字符时会停止解析字符串。例如，如果尝试将单词“one”转换为数字，解析将立即停止，并且 `atoi()`（或其他函数）会返回 0 而没有任何错误。如果 0 可能是字符串中的一个合法值，你在调用它们之前需要添加自己的有效性检查。

### rand() 和 srand()

随机值在许多情况下都起到了有趣的作用。想要改变 LED 灯的颜色吗？想要洗牌一副虚拟的扑克牌？需要模拟潜在的通信延迟吗？随机数来拯救！

`rand()`函数返回一个 0 到一个常量（技术上是一个*宏*；更多关于这些内容详见“特殊值”），即`RAND_MAX`之间的伪随机数。我说“伪随机”是因为你得到的“随机”数是算法的产物。¹

相关的函数`srand()`可以用来*种子*伪随机数生成算法。种子值是算法在跳跃产生各种值之前的起始点。你可以使用`srand()`来每次程序运行时提供新的值——例如当前时间戳——或者你可以使用种子来产生已知的数列。这可能看起来是一个奇怪的需求，但在测试中会很有用。

让我们试用这两个函数来感受它们的用法。看看[*ch07/random.c*](https://oreil.ly/sst4C)：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
  printf("RAND_MAX: %d\n", RAND_MAX);
  unsigned int r1 = rand();
  printf("First random number: %d\n", r1);
  srand(5);
  printf("Second random number: %d\n", rand());
  srand(time(NULL));
  printf("Third random number: %d\n", rand());
  unsigned int pin = rand() % 9000 + 1000;
  printf("Random four digit number: %d\n", pin);
}
```

让我们编译并运行它来看看输出：

```cpp
ch07$ gcc random.c
ch07$ ./a.out
RAND_MAX: 2147483647
First random number: 1804289383
Second random number: 590011675
Third random number: 1205842387
Random four digit number: 7783

ch07$ ./a.out
RAND_MAX: 2147483647
First random number: 1804289383
Second random number: 590011675
Third random number: 612877372
Random four digit number: 5454
```

在我的系统上，`rand()`返回的最大值是 2147483647。我们生成的第一个数应该在 0 到 2147483647 之间，确实如此。我们生成的第二个数将在相同的范围内，但在我们为`srand()`提供一个新的种子值后，希望它与`r1`不同，结果确实如此。

但是看看我们第二次运行的输出中的前两个“随机”数。它们完全一样！几乎不随机。正如我所提到的，`rand()`是一个伪随机生成器。如果你从未调用过`srand()`，生成算法的默认种子是 1。但是，如果你使用像 5 这样的常量调用它，情况也不会好转。它会生成不同的数列，但每次运行程序时都会是相同的“不同”数列。

因此，为了获得不同的伪随机数，你需要提供一个在每次运行程序时都会改变的种子。最常见的技巧是像我一样包含另一个头文件*time.h*（见“time.h”）并引入当前时间戳（自 1970 年 1 月 1 日以来的秒数）。只要我们不在一秒内两次启动程序，我们将在每次运行时得到新的序列。你可以在上述两次运行中看到，这种种子效果良好，因为第三个数字在它们之间确实是不同的。

使用了更好的种子²之后，对`rand()`的后续调用应该在每次执行时看起来都是随机的。我们可以通过生成的最终随机数的 PIN 来看到这种好处。使用了一种获取范围内随机数的流行技巧来限制 PIN。你可以使用取余运算符来确保你得到一个适当限制的范围，并且加上一个基值。为了 PIN 正好有四位数，我们使用了基数 1000 和范围 9000（从 0 到 8999 包括）。

### 退出()

我要突出的*stdlib.h*中的最后一个函数是`exit()`函数。在“返回值和 main()”中，我们看过使用`return`语句来结束程序，并可选地从`main()`函数返回一个值，以向操作系统提供一些状态信息。

此外还有一个单独的`exit()`函数，它接受一个`int`参数，用于与`main()`方法中的`return`语句相同的退出码值。使用`exit()`和从`main()`返回的区别在于，`exit()`可以从任何函数调用，并立即退出应用程序。例如，我们可以编写一个“确认”函数，询问用户是否确定要退出。如果他们回答`*y*`，那么我们可以在那一点上使用`exit()`，而不是返回一些特殊值给`main()`，然后使用`return`。查看[*ch07/areyousure.c*](https://oreil.ly/W5lIr)：

```cpp
#include <stdio.h>
#include <stdlib.h>

void confirm() {
  char answer;
  printf("Are you sure you want to exit? (y/n) ");
  scanf("%c", &answer);
  if (answer == 'y' || answer == 'Y') {
    printf("Bye\n\n");
    exit(0);
    printf("This will never be printed.\n");
  }
}

int main() {
  printf("In main... let's try exiting.\n");
  confirm();
  printf("Glad you decided not to leave.\n");
}

```

下面是两次运行的输出：

```cpp
ch07$ gcc areyousure.c
ch07$ ./a.out
In main... let's try exiting.
Are you sure you want to exit? (y/n) y
Bye

ch07$ ./a.out
In main... let's try exiting.
Are you sure you want to exit? (y/n) n
Glad you decided not to leave.
```

注意当我们使用`exit()`时，我们不会返回到`main()`函数，甚至不会完成我们`confirm()`函数本身中的代码。我们真的退出程序并向操作系统提供退出码。

顺便说一下，在`main()`内部，无论您使用`return`还是`exit()`，大部分情况下都不会有太大差别，尽管前者更为“礼貌”。（例如，如果使用`return`，则从完成`main()`函数时的任何清理仍将运行。如果使用`exit()`，则会跳过同样的清理。）还值得注意的是，只要像我们一直在做的那样到达`main()`体的末尾是完成程序的一种很好且流行的方式，当然，在没有错误时完成程序时，这种方式会更好。

## `string.h`

字符串如此常见且如此有用，甚至有自己的头文件。*string.h*头文件可以添加到任何需要比简单存储和打印字符串更多地比较或操作字符串的程序中。这个头文件描述了比我们这里有时间涵盖的更多函数，但我们想要突出显示一些重要的实用程序在表 7-1 中。

表 7-1\. 有用的字符串函数

| 函数 | 描述 |
| --- | --- |
| `strlen(char *s)` | 计算字符串的长度（不包括最后的空字符） |
| `strcmp(char *s1, char *s2)` | 比较两个字符串。如果 s1 < s2，则返回-1，如果 s1 == s2，则返回 0，如果 s1 > s2，则返回 1 |
| `strncmp(char *s1, char *s2, int n)` | 比较 s1 和 s2 的最多 n 个字节（结果类似于 strcmp） |
| `strcpy(char *dest, char *src)` | 将 src 复制到 dest |
| `strncpy(char *dest, char *src, int n)` | 将 src 的最多 n 个字节复制到 dest |
| `strcat(char *dest, char *src)` | 将 src 追加到 dest |
| `strncat(char *dest, char *src, int n)` | 将最多 n 个字节的 src 附加到 dest |

我们可以在一个简单的程序中演示所有这些功能，[*ch07/fullname.c*](https://oreil.ly/dzycy)，通过请求用户提供其全名的各个部分（安全地！）在最后拼接起来。如果我们发现在与丹尼斯·里奇交互时，我们会感谢他编写了 C 语言。

```cpp
#include <stdio.h>
#include <string.h>

int main() {
  char first[20];
  char middle[20];
  char last[20];
  char full[60];
  char spacer[2] = " ";

  printf("Please enter your first name: ");
  scanf("%s", first);
  printf("Please enter your middle name or initial: ");
  scanf("%s", middle);
  printf("Please enter your last name: ");
  scanf("%s", last);

  // First, assemble the full name
  strncpy(full, first, 20);
  strncat(full, spacer, 40);
  strncat(full, middle, 39);
  strncat(full, spacer, 20);
  strncat(full, last, 19);

  printf("Well hello, %s!\n", full);

  int dennislen = 17;  // length of "Dennis M. Ritchie"
  if (strlen(full) == dennislen &&
      strncmp("Dennis M. Ritchie", full, dennislen) == 0)
  {
    printf("Thanks for writing C!\n");
  }
}
```

以下是一个示例运行：

```cpp
ch07$ gcc fullname.c
ch07$ ./a.out
Please enter your first name: Alice
Please enter your middle name or initial: B.
Please enter your last name: Toklas
Well hello, Alice B. Toklas!
```

让自己尝试这个程序。如果您输入了丹尼斯的姓名（包括他的中间名后面的句点：“M.”），您是否像预期的那样收到感谢消息？

###### 警告

将在`strncat()`中要连接的最大字符数设置为目标中剩余的最大字符数，而不是源字符串的长度。您的编译器可能会因此错误而发出“指定的边界 X 等于源长度”的警告消息。当然，X 将是您在调用`strncat()`时指定的边界。这只是一个警告，你可能确实有与你的源长度完全相同的长度。但如果您看到这个警告，请再次检查，确保您没有意外使用源长度。

作为另一个例子，我们可以重新审视默认值的概念和从“初始化字符串”中覆盖数组。您可以延迟字符数组的初始化，直到您知道用户做了什么。我们可以声明但不初始化一个字符串，用`scanf()`使用它，然后如果用户没有给出良好的替代值，我们可以返回到默认值。

让我们尝试一个关于未来某个惊人应用程序背景颜色的问题。我们可能假设一个黑色背景的暗色主题。我们可以提示用户提供一个不同的值，或者如果他们想保留默认值，他们可以简单地按回车键。这里是[*ch07/background.c*](https://oreil.ly/a89Jd)：

```cpp
#include <stdio.h>
#include <string.h>

int main() {
  char background[20];                                     ![1](img/1.png)
  printf("Enter a background color or return for the default: ");
  scanf("%[^\n]s", background);                            ![2](img/2.png)
  if (strlen(background) == 0) {                           ![3](img/3.png)
    strcpy(background, "black");
  }
  printf("The background color is now %s.\n", background); ![4](img/4.png)
}
```

![1](img/#co_libraries_CO1-1)

声明一个具有足够容量的字符串，但不设置任何内容。

![2](img/#co_libraries_CO1-2)

从用户处获取输入并将其存储在我们的数组中。

![3](img/#co_libraries_CO1-3)

如果在提示用户后数组为空，请存储我们的默认值。

![4](img/#co_libraries_CO1-4)

展示最终值，可以是用户提供的，也可以是默认值。

这里是一些示例运行，包括保留黑色背景的示例：

```cpp
ch07$ gcc background.c
ch07$ ./a.out
Enter a background color or return for the default: blue
The background color is now blue.
ch07$ ./a.out
Enter a background color or return for the default: white
The background color is now white.
ch07$ ./a.out
Enter a background color or return for the default:
The background color is now black.
```

如果您邀请用户提供一个值，请记住在数组中分配足够的空间以容纳用户可能提供的任何响应。如果您不能信任您的用户，`scanf()`有另一个技巧可以部署。就像在`printf()`中的格式说明符一样，您可以在`scanf()`中的任何输入字段中添加一个宽度。对于我们之前的例子，我们可以改变为显式的限制为 19（为最终的`'\0'`字符节省空间）：

```cpp
  scanf("%19[^\n]s", background);
```

非常简单。它看起来很密集，但对于可能无法为冗长用户提供大量额外空间的有限设备来说，这是一个不错的选择。

## math.h

*math.h*头文件声明了多个有用的函数，用于执行各种算术和三角函数计算。表 7-2 包含了几个比较流行的函数。所有这些函数都返回`double`值。

表 7-2\. *math.h*中的便利函数

| 函数 | 描述 |
| --- | --- |
| 三角函数 |
| `cos(double rad)` | 余弦 |
| `sin(double rad)` | 正弦 |
| `atan(double rad)` | 反正切 |
| `atan2(double y, double x)` | 双参数反正切（正 X 轴与点（x，y）之间的角度） |
| 根和指数 |
| `exp(double x)` | *e*^x |
| `log(double x)` | x 的自然对数（以 *e* 为底） |
| `log10(double x)` | x 的常用对数（以 10 为底） |
| `pow(double x, double y)` | x^y |
| `sqrt(double x)` | x 的平方根 |
| 四舍五入 |
| `ceil(double x)` | ceiling 函数，返回比 x 大的最小整数 |
| `floor(double x)` | floor 函数，返回比 x 小的最大整数 |
| 符号 |
| `fabs(double x)`^(a) | 返回 x 的绝对值 |
| ^(a) 奇怪的是，整数类型的绝对值函数`abs()`在*stdlib.h*中声明。 |

为了任何需要`int`或`long`答案的情况，你只需进行类型转换。例如，我们可以编写一个简单的程序（[*ch07/rounding.c*](https://oreil.ly/rEMTv)）来对多个整数求平均，并将结果四舍五入为最接近的`int`值，像这样：

```cpp
#include <stdio.h>
#include <math.h>

int main() {
  int grades[6] = { 86, 97, 77, 76, 85, 90 };
  int total = 0;
  int average;

  for (int g = 0; g < 6; g++) {
    total += grades[g];
  }
  printf("Raw average: %0.2f\n", total / 6.0);
  average = (int)floor(total / 6.0 + 0.5);
  printf("Rounded average: %d\n", average);
}
```

由于我们可能需要帮助编译器使用这个库，让我们看一下编译命令：

```cpp
gcc rounding.c -lm
```

同样，*math.h*声明了 C 标准库中的函数，但这些函数不一定实现在同一位置。包含大多数我们讨论的函数的二进制文件是*libc*（或 GNU 版本的*glibc*）。然而，在许多系统上，数学函数位于单独的二进制文件*libm*中，需要添加尾随的 **`-lm`** 标志来确保编译器知道要链接数学库。

你的系统可能不同。尝试不使用 **`-lm`** 选项进行编译不会有任何损害，以查看系统是否自动包含*libm*（或已经在*libc*中包含了所有函数）。如果尝试编译而不使用该标志，并且没有收到任何错误消息，那就表示一切正常！如果确实需要该库标志，则会看到类似以下的信息：

```cpp
ch07$ gcc rounding.c
/usr/bin/ld: /tmp/ccP1MUC7.o: in function `main':
rounding.c:(.text+0xaf): undefined reference to `floor'
collect2: error: ld returned 1 exit status
```

自己试试（根据需要使用或不使用库标志）。你应该得到 85 作为答案。如果经常需要进行四舍五入，可以编写自己的函数来简化操作，避免在代码中添加稍微繁琐的`floor()`调用和类型转换前加 0.5 值的麻烦事务。

## time.h

这个头文件为你提供了一些工具，帮助确定和显示时间。它使用两种类型的存储来处理日期和时间：一个简单的时间戳（使用类型别名 `time_t`，代表自 1970 年 1 月 1 日以来的秒数，UTC 时间）和一个更详细的结构体 `struct tm`，其定义如下：

```cpp
struct tm {
  int tm_sec;   // seconds (0 - 60; allow for leap second)
  int tm_min;   // minutes (0 - 59)
  int tm_hour;  // hours (0 - 23)
  int tm_mday;  // day of month (1 - 31)
  int tm_mon;   // month (0 - 11; WARNING! NOT 1 - 12)
  int tm_year;  // year (since 1900)
  int tm_wday;  // day of week (0 - 6)
  int tm_yday;  // day of year (0 - 365)
  int tm_isdst; // Daylight Saving Time flag
                // This flag can be in one of three states:
                // -1 == unavailable, 0 == standard time, 1 == DST.
}
```

我不会使用这个漂亮的结构来处理所有分离的字段，但如果你正在处理日期和时间，比如在日历应用程序中可能会用到，它可能会很有用。我会不时使用时间戳，正如我们在“rand() and srand()” 中已经看到的，用于向 `srand()` 函数提供一个变化的种子。表 7-3 展示了一些处理这些简单值的函数：

表 7-3\. 时间戳处理

| 功能 | 描述 |
| --- | --- |
| `char *ctime(time_t *t)` | 返回本地时间的字符串 |
| `struct tm *localtime(time_t *t)` | 将时间戳展开为详细结构 |
| `time_t mktime(struct tm *t)` | 将结构体转换为时间戳 |
| `time_t time(time_t *t)` | 返回当前时间作为时间戳 |

最后一个函数 `time()` 的定义可能看起来有点奇怪。它既接受又返回一个 `time_t` 指针。你可以用 `NULL` 值或有效的 `time_t` 类型变量的指针来调用 `time()`。如果使用 `NULL`，则简单地返回当前时间。如果提供指针，则返回当前时间，并更新指向的变量为当前时间。我们在处理随机数时只需要 `NULL` 选项，但你会碰到一些使用这种模式的实用函数。如果你在处理堆内存时，这可能会很有用。

## ctype.h

许多处理用户输入的情况都要求你验证输入是否符合某种期望的类型或值。例如，邮政编码应为五位数字，美国州的缩写应为两个大写字母。*ctype.h* 头文件声明了几个检查单个字符的便利函数。它还有两个辅助函数，用于大小写转换。表 7-4 强调了几个这样的函数。

表 7-4\. 使用 *ctype.h* 处理字符

| 功能 | 描述 |
| --- | --- |
| 测试 |
| `isalnum(int c)` | 判断 `c` 是否为数字字符或字母 |
| `isalpha(int c)` | 判断 `c` 是否为字母 |
| `isdigit(int c)` | 判断 `c` 是否为十进制数字 |
| `isxdigit(int c)` | 判断 `c` 是否为十六进制数字（不区分大小写） |
| `islower(int c)` | 判断 `c` 是否为小写字母 |
| `isupper(int c)` | 判断 `c` 是否为大写字母 |
| `isspace(int c)` | 判断 `c` 是否为空格、制表符、换行符、回车符、垂直制表符或换页符 |
| 转换 |
| `tolower(int c)` | 返回 `c` 的小写版本 |
| `toupper(int c)` | 返回 `c` 的大写版本 |

不要忘记你的布尔运算符！你可以轻松地扩展这些测试来询问诸如“不是空格”的问题，使用`!`运算符：

```cpp
  if (!isspace(answer)) {
    // Not a blank character, so go ahead
    ...
  }
```

###### 注意

与需要`int`的地方获取`double`结果的数学函数一样，`ctype.h`中的转换函数返回`int`，但你可以根据需要轻松地将其强制转换为`char`。

# 组装

让我们引入一些新的头文件，并使用前几章的一些主题来制作一个更加完整的示例。我们将创建一个结构来存储一个简单银行账户的信息。我们可以使用新的`string.h`工具向每个账户添加一个名字字段。而且我们将使用`math.h`函数来计算账户余额上的一个样本复利支付。这个示例将需要以下的包含文件：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
```

准备好我们的头文件后，让我们深入进入并启动程序本身。

## 填充字符串

让我们通过创建包含一个字符串“name”字段的账户类型来开始我们的示例。新的`struct`非常简单：

```cpp
struct account {
  char name[50];
  double balance;
};
```

现在我们可以使用我们的字符串函数在创建结构之后用实际内容填充`name`。我们还可以使用`malloc()`在函数中创建该结构并返回我们账户的地址。以下是新函数，为了可读性省略了一些安全检查：

```cpp
struct account *create(char *name, double initial) {
  struct account *acct = malloc(sizeof(struct account));
  strncpy(acct->name, name, 49);
  acct->balance = initial;
  return acct;
}
```

注意我选择在这里使用`strncpy()`。我的想法是我不能保证传入的`name`参数能够适应。当然，由于我写了整个程序，我当然能保证这个细节，但这不是重点。我想要确保如果我允许用户输入，比如通过提示用户输入详细信息，我的`create()`函数有一些安全措施。

让我们继续创建一个函数来打印我们的账户详细信息。希望这段代码看起来从我们在第六章的工作中很熟悉。我们还可以开始我们的`main()`函数来尝试我们到目前为止编写的一切：

```cpp
void print(struct account *a) {
  printf("Account: %s\n", a->name);
  printf("Balance: $%.2f\n", a->balance);
}

int main() {
  struct account *checking;
  checking = create("Bank of Earth (checking)", 200.0);
  print(checking);
  free(checking);
}
```

让我们编译并运行[*ch07/account1.c*](https://oreil.ly/i8vvr)。这是我们的输出：

```cpp
ch07$ gcc account1.c
ch07$ ./a.out
Account: Bank of Earth (checking)
Balance: $200.00
```

好极了！到目前为止，一切顺利。接下来是计算利息支付的问题。

## 找到我们的利息

使用`math.h`库中的`pow()`函数，我们可以在一个表达式中计算每月复利。我知道这个公式是在高中教的，但每次真正需要用到时，我还是得上网查一下。然后我们将更新`main()`以添加一年（5%利率）的利息到我们的账户，并再次打印出详细信息。以下是来自[*ch07/account2.c*](https://oreil.ly/EAOwS)的新部分：

```cpp
void add_interest(struct account *acct, double rate, int months) {
  // Put our current balance in a local var for easier use
  double principal = acct->balance;
  // Convert our annual rate to a monthly percentage value
  rate /= 1200;
  // Use the interest formula to calculate our new balance
  acct->balance = principal * pow(1 + rate, months);
}

int main() {
  struct account *checking;
  checking = create("Bank of Earth (checking)", 200.0);
  print(checking);

  add_interest(checking, 5.0, 12);
  print(checking);

  free(checking);
}
```

看起来非常不错！让我们编译并运行*account2.c*。如果你的系统在编译时需要**`-lm`**数学库标志，请确保在编译时添加它：

```cpp
ch07$ gcc account2.c -lm
ch07$ ./a.out
Account: Bank of Earth (checking)
Balance: $200.00
Account: Bank of Earth (checking)
Balance: $210.23
```

一切都运行正常！尽管你现在是在我修复了写作过程中的各种小错误之后阅读最终输出的代码。例如，我在`strncpy()`中颠倒了源和目标字符串的顺序。第一次就把所有事情做对是很罕见的。编译器通常会告诉你哪里出错了。你只需回到编辑器中修复它。熟悉错误——以及修复它们！——是我鼓励你输入这些示例的原因之一。实际编写代码是提高编程能力的最佳方法之一。

# 查找新的库

在这里，有比我在此提及的更多可供使用的库。事实上，在 C 标准库中，就有更多的函数；你可以通过在线查阅[GNU C 库](https://oreil.ly/58fLM)的文档深入挖掘。

除了标准库之外，还有其他库可以帮助你的项目。对于那些需要专门库的情况，你最好的选择是在线搜索。例如，如果你想直接与 USB 连接的设备交互，可以搜索“C USB 库”，最终可能找到很棒的*libusb*，它来自[*https://libusb.info*](https://libusb.info)。

你也可以找到一些流行库的列表，但这些列表的质量和维护情况各不相同。遗憾的是，没有像某些语言那样的“所有 C 语言事物”的中央存储库。我的建议是浏览搜索结果，寻找指向像[GitHub](https://github.com)或[gnu.org](https://gnu.org)这样的信誉网站的链接。并且不要害怕仅仅阅读库的源代码。如果你看到任何引起注意的地方，要多加关注。大多数情况下，你将得到你所期望的结果，但在使用在线资源时，始终保持一点谨慎是明智的。

# 下一步

帮助你提升编写代码能力当然是本书的目标之一。我们在本章节中涵盖了一些更常见和流行的库（以及我们必须包含的头文件，以便使用它们的函数）。当然还有更多的库在那里等着你！希望你能看到库和头文件如何与你自己的代码互动。

接下来我们将处理微控制器，并且在此过程中开始着眼于编写*更紧凑*的代码。良好的代码并不一定是进行优化工作的前提条件，但它确实有帮助。在继续前进之前，可以回顾一些过去章节的例子。试着做出一些改变。试着破坏一些东西。试着修复你破坏的东西。每一次成功的编译都应该成为你程序员生涯中的一个里程碑。

¹ 该算法是确定性的，虽然对大多数开发者来说这没问题，但它并非真正的随机。

² 我没有足够的空间来介绍好的生成器，但是在网上搜索“C 随机生成器”将为您带来一些有趣的选项。有更好的算法，比如 Blum Blum Shub 或者 Mersenne Twister，但您也可以找到硬件相关的生成器，它们可能更好。
