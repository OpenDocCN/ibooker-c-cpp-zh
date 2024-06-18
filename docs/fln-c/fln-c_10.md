# 第九章：逃离`#ifdef`地狱

C 语言广泛应用于需要高性能或接近硬件编程的系统中。与接近硬件编程相关的是处理硬件变体的必要性。除了硬件变体，一些系统在代码中支持多个操作系统或处理多个产品变体。解决这些问题的常用方法是使用 C 预处理器的`#ifdef`语句来区分代码中的变体。C 预处理器提供了这种功能，但使用它需要按照良好的结构化方式使用。

然而，这也揭示了 C 预处理器及其`#ifdef`语句的弱点。C 预处理器不支持强制其使用规则的方法。这很遗憾，因为它很容易被滥用。通过添加另一个`#ifdef`，很容易为代码添加另一个硬件变体或另一个可选功能。此外，`#ifdef`语句很容易被滥用来添加仅影响单个变体的快速错误修复。这使得不同变体的代码更加多样化，并导致需要为每个变体单独修复代码的情况越来越多。

在这种非结构化和临时的方式中使用`#ifdef`语句是通往地狱的必定路径。代码变得难以阅读和维护，所有开发人员都应该避免。本章介绍了几种摆脱这种情况或完全避免的方法。

本章详细介绍了如何在 C 代码中实现操作系统变体或硬件变体。它讨论了五种处理代码变体的模式以及如何组织或甚至摆脱`#ifdef`语句。这些模式可以视为组织此类代码的入门或重构非结构化`#ifdef`代码的指南。

图 9-1 展示了逃离`#ifdef`噩梦的方式，表 9-1 提供了本章讨论的模式的简要总结。

![逃离`#ifdef`噩梦的方法](img/fluc_0901.png)

###### 图 9-1\. 逃离`#ifdef`地狱的方式

表格 9-1\. 逃离`#ifdef`地狱的模式

|  | 模式名称 | 摘要 |
| --- | --- | --- |
|  | 避免变体 | 在每个平台上使用不同的函数使得代码更难读写。程序员需要最初理解、正确使用和测试这些多个函数，以实现跨多个平台的单一功能。因此，应使用在所有平台上都可用的标准化函数。如果没有标准化函数，则考虑不实现此功能。 |
|  | 隔离原语 | 使用`#ifdef`语句组织代码变体使得代码难以阅读。很难跟踪程序流程，因为为多个平台多次实现了代码。因此，隔离您的代码变体。在您的实现文件中，将处理变体的代码放入单独的函数，并从主程序逻辑调用这些函数，这样主程序逻辑只包含与平台无关的代码。 |
|  | 原子原语 | 包含变体的函数仍然难以理解，因为所有复杂的`#ifdef`代码仅放入该函数以便在主程序中摆脱它。因此，将您的原语设为原子。每个函数仅处理一种变体类型。如果处理多种类型的变体，例如操作系统变体和硬件变体，则为每种类型单独创建函数。 |
|  | 抽象层 | 你希望在代码库中的多个地方使用处理平台变体的功能，但不希望复制该功能的代码。因此，为每个需要特定于平台的功能提供一个 API。在头文件中只定义与平台无关的函数，并将所有特定于平台的`#ifdef`代码放入实现文件中。调用您函数的人只需包含您的头文件，而不需要包含任何特定于平台的文件。 |
|  | 拆分变体实现 | 仍然包含`#ifdef`语句的特定于平台的实现以区分代码变体。这使得难以看到并选择应为哪个平台构建的代码部分。因此，将每个变体实现放入单独的实现文件中，并选择每个文件为哪个平台编译。 |

# 运行示例

假设你想要实现将一些文本写入文件的功能，该文件将存储在新创建的目录中，具体取决于配置标志，可能是当前目录或主目录。为了使事情更复杂，您的代码应该在 Windows 系统和 Linux 系统上运行。

你的第一次尝试是创建一个包含所有配置和操作系统代码的实现文件。为了做到这一点，该文件包含许多`#ifdef`语句来区分代码变体：

```cpp
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#ifdef __unix__
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <unistd.h>
#elif defined _WIN32
  #include <windows.h>
#endif

int main()
{
  char dirname[50];
  char filename[60];
  char* my_data = "Write this data to the file";
  #ifdef __unix__
    #ifdef STORE_IN_HOME_DIR
      sprintf(dirname, "%s%s", getenv("HOME"), "/newdir/");
      sprintf(filename, "%s%s", dirname, "newfile");
    #elif defined STORE_IN_CWD
      strcpy(dirname, "newdir");
      strcpy(filename, "newdir/newfile");
    #endif
    mkdir(dirname,S_IRWXU);
    int fd = open (filename, O_RDWR | O_CREAT, 0666);
    write(fd, my_data, strlen(my_data));
    close(fd);
  #elif defined _WIN32
    #ifdef STORE_IN_HOME_DIR
      sprintf(dirname, "%s%s%s", getenv("HOMEDRIVE"), getenv("HOMEPATH"),
              "\\newdir\\");
      sprintf(filename, "%s%s", dirname, "newfile");
    #elif defined STORE_IN_CWD
      strcpy(dirname, "newdir");
      strcpy(filename, "newdir\\newfile");
    #endif
    CreateDirectory (dirname, NULL);
    HANDLE hFile = CreateFile(filename, GENERIC_WRITE, 0, NULL,
                              CREATE_NEW, FILE_ATTRIBUTE_NORMAL, NULL);
    WriteFile(hFile, my_data, strlen(my_data), NULL, NULL);
    CloseHandle(hFile);
  #endif
  return 0;
}
```

这种代码是混乱的。程序逻辑完全重复。这不是操作系统无关的代码；相反，它只是将两个不同的操作系统特定实现放入一个文件中。特别是，不同操作系统和不同目录创建位置的正交代码变体使代码变得丑陋，因为它们导致嵌套的`#ifdef`语句，这些语句非常难以理解。在阅读代码时，你必须不断地在这些行之间跳跃。你必须跳过其他`#ifdef`分支中的代码，以便跟随程序逻辑。这种重复的程序逻辑促使程序员只在当前工作的代码变体中修复错误或添加新功能。这导致代码片段和变体的行为分离，使代码难以维护。

从哪里开始？如何整理这一混乱局面？作为第一步，如果可能的话，你可以使用标准化函数以避免变体。

# 避免变体

## 上下文

当你编写可在多个操作系统平台或多个硬件平台上使用的可移植代码时，你可能会调用一些函数，这些函数在一个平台上可用，但在另一个平台上的语法和语义可能不完全相同。因此，你需要实现不同的代码变体——每个平台一个。现在你在不同平台上有不同的代码片段，并且在代码中使用`#ifdef`语句区分这些变体。

## 问题

**为每个平台使用不同的函数使得代码更难阅读和编写。程序员需要首先理解、正确使用和测试这些多个函数，才能在多个平台上实现单一功能。**

很多时候，我们的目标是在所有平台上实现行为完全相同的功能，但是当使用依赖于平台的函数时，这一目标变得更加困难，并且可能需要编写额外的代码。这是因为函数的语法和语义在各平台间可能略有不同。

对多个平台使用多个函数使得代码更难编写、阅读和理解。用`#ifdef`语句区分不同函数使得代码变得更长，并要求读者在多个行之间跳跃以查明单个`#ifdef`分支的代码功能。

对于你需要编写的任何代码片段，你可以问自己这样一个问题：这是否值得努力？如果所需功能并不重要，并且如果特定于平台的函数使得实现和支持该功能非常困难，那么有可能选择根本不提供该功能。

## 解决方案

**使用在所有平台上都可用的标准化函数。如果没有标准化函数可用，则考虑不实现该功能。**

可以使用的良好的标准化函数示例包括 C 标准库函数和 POSIX 函数。考虑要支持的平台，并检查这些标准化函数在所有平台上是否可用。如果可能的话，应该使用这些标准化函数，而不是更特定的依赖平台的函数，如下面代码所示：

*调用者的代码*

```cpp
#include <standardizedApi.h>

int main()
{
  /* just a single function is called instead of multiple via
 ifdef distinguished functions */
  somePosixFunction();
  return 0;
}
```

*标准化的 API*

```cpp
  /* this function is available on all operating systems
 that adhere to the POSIX standard */
  somePosixFunction();
```

再次强调，如果没有您想要的标准化函数，那么您可能不应该实现所请求的功能。如果仅有依赖于平台的函数可用于您要实现的功能，则可能不值得进行实现、测试和维护工作。

然而，在某些情况下，即使没有标准化的函数可用，您也必须在产品中提供功能。这意味着您必须在不同平台上使用不同的函数，或者甚至在一个平台上实现已在另一个平台上可用的功能。为了以结构化的方式实现这一点，对于您的代码变体，请使用隔离原语，并将其隐藏在一个抽象层后面。

例如，为了避免变体，您可以使用 C 标准库文件访问函数如 `fopen`，而不是使用操作系统特定的函数如 Linux 的 `open` 或 Windows 的 `CreateFile` 函数。另一个例子，您可以使用 C 标准库的时间函数。避免使用操作系统特定的时间函数，如 Windows 的 `GetLocalTime` 和 Linux 的 `localtime_r`，而应该使用 *time.h* 中的标准化 `localtime` 函数。

## 结果

代码写起来和读起来都很简单，因为可以为多个平台使用同一段代码。在编写代码时，程序员不必理解不同平台的不同函数，而在阅读代码时也不必在 `#ifdef` 分支之间跳转。

由于相同的代码片段在所有平台上都被使用，功能不会有所不同。但标准化的函数可能不是在每个平台上实现所需功能的最有效或高性能方式。一些平台可能提供其他平台特定的函数，例如使用该平台上的专用硬件来实现更高性能。这些优势可能无法被标准化函数利用。

## 已知的用途

下面的例子展示了这种模式的应用：

+   VIM 文本编辑器的代码使用操作系统独立的函数 `fopen`、`fwrite`、`fread` 和 `fclose` 来访问文件。

+   OpenSSL 代码将当前的本地时间写入其日志消息中。为此，它使用操作系统独立函数 `localtime` 将当前的 UTC 时间转换为本地时间。

+   OpenSSL 函数 `BIO_lookup_ex` 查找要连接的节点和服务。此函数在 Windows 和 Linux 上编译，并使用操作系统独立函数 `htons` 将值转换为网络字节顺序。

## 应用于运行示例

对于访问文件功能，您处于幸运的位置，因为现在有可用的操作系统无关函数。现在您有以下代码：

```cpp
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#ifdef __unix__
  #include <sys/stat.h>
#elif defined _WIN32
  #include <windows.h>
#endif

int main()
{
  char dirname[50];
  char filename[60];
  char* my_data = "Write this data to the file";
  #ifdef __unix__
    #ifdef STORE_IN_HOME_DIR
      sprintf(dirname, "%s%s", getenv("HOME"), "/newdir/");
      sprintf(filename, "%s%s", dirname, "newfile");
    #elif defined STORE_IN_CWD
      strcpy(dirname, "newdir");
      strcpy(filename, "newdir/newfile");
    #endif
    mkdir(dirname,S_IRWXU);
  #elif defined _WIN32
    #ifdef STORE_IN_HOME_DIR
      sprintf(dirname, "%s%s%s", getenv("HOMEDRIVE"), getenv("HOMEPATH"),
              "\\newdir\\");
      sprintf(filename, "%s%s", dirname, "newfile");
    #elif defined STORE_IN_CWD
      strcpy(dirname, "newdir");
      strcpy(filename, "newdir\\newfile");
    #endif
    CreateDirectory(dirname, NULL);
  #endif
  FILE* f = fopen(filename, "w+"); ![1](img/1.png)
  fwrite(my_data, 1, strlen(my_data), f);
  fclose(f);
  return 0;
}
```

![1](img/#co_escaping__ifdef_hell_CO1-1)

函数`fopen`、`fwrite`和`fclose`属于 C 标准库的一部分，可以在 Windows 和 Linux 上使用。

该代码中的标准化文件相关函数调用已经简化了事情。现在不再需要为 Windows 和 Linux 分别提供独立的文件访问调用，而是有了一个通用代码。通用代码确保这些调用在两个操作系统上执行相同的功能，并且不存在在 bug 修复或添加功能后两种不同实现运行不同的危险。

然而，由于您的代码仍然被`#ifdef`所主导，因此阅读起来非常困难。因此，请确保您的主程序逻辑不会被代码变体混淆。使用隔离的原语将代码变体与主程序逻辑分开。

# 隔离的原语

## 背景

您的代码调用特定于平台的函数。您为不同平台编写了不同的代码片段，并使用`#ifdef`语句区分代码变体。您不能简单地避免变体，因为没有可用的标准化函数可以在所有平台上以统一的方式提供所需功能。

## 问题

**使用`#ifdef`语句组织代码变体会使代码变得难以阅读。程序流程非常难以跟踪，因为为多个平台实现了多次。**

在尝试理解代码时，通常只关注一个平台，但`#ifdef`强制您在代码行之间跳转，以找到感兴趣的代码变体。

`#ifdef`语句还使代码难以维护。这些语句促使程序员仅修复他们感兴趣的平台上的代码，并因危险破坏其他代码而不敢触碰其他代码。但仅为一个平台修复错误或引入新功能意味着代码在其他平台上的行为会分歧。另一种选择——以不同方式在所有平台上修复此类错误——需要在所有平台上测试代码。

使用许多代码变体测试代码很困难。每个新的`#ifdef`语句都会使测试工作量加倍，因为必须测试所有可能的组合。更糟糕的是，每个这样的语句都会使可以构建和必须测试的二进制文件数量加倍。这带来了一个物流问题，因为构建时间增加，向测试部门和客户提供的二进制文件数量也增加。

## 解决方案

**隔离您的代码变体。在实现文件中，将处理变体的代码放入单独的函数中，并从主程序逻辑中调用这些函数，这样主程序逻辑只包含与平台无关的代码。**

每个函数应该只包含程序逻辑或仅处理变体。不能有函数同时包含两者。因此，函数中要么根本没有`#ifdef`语句，要么有带有单个依赖于变体的函数调用的`#ifdef`分支。这样的变体可以是构建配置中打开或关闭的软件功能，也可以是平台变体，如下面的代码所示：

```cpp
void handlePlatformVariants()
{
  #ifdef PLATFORM_A
    /* call function of platform A */
  #elif defined PLATFORM_B ![1](img/1.png)
    /* call function of platform B */
  #endif
}

int main()
{
  /* program logic goes here */
  handlePlatformVariants();
  /* program logic continues */
}
```

![1](img/#co_escaping__ifdef_hell_CO2-1)

类似于`else if`语句，互斥的变体可以使用`#elif`表达得很好。

每个`#ifdef`分支每次调用一个函数应该使得能够找到处理变体的函数的良好抽象粒度。通常，粒度正好在可用的特定于平台或特性的函数包装级别上。

如果处理变体的函数仍然复杂并包含`#ifdef`级联（嵌套的`#ifdef`语句），则有助于确保仅有原子变体。

## 结果

现在可以轻松跟踪主程序逻辑，因为代码变体已与其分离。在阅读主代码时，不再需要在行之间跳跃以查明代码在特定平台上的作用。

要确定代码在特定平台上的作用，必须查看实现此变体的调用函数。将该代码放在单独调用的函数中具有优势，因为可以从文件中的其他位置调用它，从而避免了代码重复。如果其他实现文件中也需要此功能，则必须实现抽象层。

不应在处理变体的函数中引入任何程序逻辑，因此更容易精确定位不在所有平台上发生的错误，因为可以轻松识别代码中平台行为不同的位置。

由于主程序逻辑与变体实现明确分离，代码重复不再是问题。不再有重复程序逻辑的诱惑，因此不会意外地仅在这些复制中的一个中进行错误修复。

## 已知用途

以下示例展示了此模式的应用：

+   VIM 文本编辑器的代码隔离了将数据转换为网络字节顺序的函数`htonl2`。VIM 的程序逻辑在实现文件中将`htonl2`定义为宏。该宏根据平台的字节序编译方式不同。

+   OpenSSL 函数`BIO_ADDR_make`将套接字信息复制到内部`struct`中。该函数使用`#ifdef`语句处理特定于操作系统和特性的变体，区分 Linux/Windows 和 IPv4/IPv6。该函数将这些变体与主程序逻辑隔离开来。

+   GNUplot 的函数`load_rcfile`从初始化文件中读取数据，并将操作系统特定的文件访问操作与其余代码隔离开来。

## 应用于运行示例

现在你有了孤立的基元，你的主程序逻辑变得更容易阅读，不需要读者在各种变体之间跳来跳去了：

```cpp
void getDirectoryName(char* dirname)
{
  #ifdef __unix__
    #ifdef STORE_IN_HOME_DIR
      sprintf(dirname, "%s%s", getenv("HOME"), "/newdir/");
    #elif defined STORE_IN_CWD
      strcpy(dirname, "newdir/");
    #endif
  #elif defined _WIN32
    #ifdef STORE_IN_HOME_DIR
      sprintf(dirname, "%s%s%s", getenv("HOMEDRIVE"), getenv("HOMEPATH"),
              "\\newdir\\");
    #elif defined STORE_IN_CWD
      strcpy(dirname, "newdir\\");
    #endif
  #endif
}

void createNewDirectory(char* dirname)
{
  #ifdef __unix__
    mkdir(dirname,S_IRWXU);
  #elif defined _WIN32
    CreateDirectory (dirname, NULL);
  #endif
}

int main()
{
  char dirname[50];
  char filename[60];
  char* my_data = "Write this data to the file";
  getDirectoryName(dirname);
  createNewDirectory(dirname);
  sprintf(filename, "%s%s", dirname, "newfile");
  FILE* f = fopen(filename, "w+");
  fwrite(my_data, 1, strlen(my_data), f);
  fclose(f);
  return 0;
}
```

现在代码变体已经很好地隔离开来了。`main`函数的程序逻辑非常容易阅读和理解，没有变体的干扰。然而，新函数`getDirectoryName`仍然被`#ifdef`主导，不容易理解。只有原子基元可能会有所帮助。

# 原子基元

## 上下文

你在代码中用`#ifdef`语句实现了变体，并将这些变体放入了单独的函数中，以处理这些变体。这些基元将变体从主程序流中分离出来，使得主程序结构清晰，易于理解。

## 问题

**包含变体并被主程序调用的函数仍然很难理解，因为所有复杂的`#ifdef`代码只是为了在主程序中摆脱它。**

处理所有种类的变体在一个函数中变得困难，一旦有许多不同的变体需要处理。例如，如果单个函数使用`#ifdef`语句区分不同的硬件类型和操作系统，则添加新的操作系统变体会变得困难，因为必须为所有硬件变体添加。每种变体不能再在一个地方处理；相反，随着不同变体数量的增加，工作量也会成倍增加。这是一个问题。在代码中添加新的变体应该是简单的。

## 解决方案

**确保你的基元是原子的。每个函数只处理一种变体。如果你处理多种变体，例如操作系统变体和硬件变体，那么应该为每种情况编写独立的函数。**

让其中一个函数调用另一个已经抽象化了一种变体的函数。如果你抽象了一个平台依赖和一个特性依赖，那么让处理特性的函数调用处理平台的函数，因为通常你会在所有平台上提供这些特性。因此，平台相关函数应该是最原子化的函数，如下面的代码所示：

```cpp
void handleHardwareOfFeatureX()
{
  #ifdef HARDWARE_A
   /* call function for feature X on hardware A */
  #elif defined HARDWARE_B || defined HARDWARE_C
   /* call function for feature X on hardware B and C */
  #endif
}

void handleHardwareOfFeatureY()
{
  #ifdef HARDWARE_A
   /* call function for feature Y on hardware A */
  #elif defined HARDWARE_B
   /* call function for feature Y on hardware B */
  #elif defined HARDWARE_C
   /* call function for feature Y on hardware C */
  #endif
}

void callFeature()
{
  #ifdef FEATURE_X
    handleHardwareOfFeatureX();
  #elif defined FEATURE_Y
    handleHardwareOfFeatureY();
  #endif
}
```

如果有一个函数明显需要在多种变体之间提供功能，并处理所有这些变体，那么函数的范围可能有问题。也许函数过于通用或者功能不单一。按照“函数分割”模式建议，将函数进行拆分。

在包含程序逻辑的主代码中调用原子基元。如果你希望在具有明确定义接口的其他实现文件中使用原子基元，则应该使用抽象层。

## 结果

现在每个函数只处理一种变体。这使得每个函数易于理解，因为不再存在`#ifdef`语句的级联。每个函数现在只抽象一种变体，只做这一件事。因此，函数遵循单一责任原则。

没有`#ifdef`级联使得程序员在一个函数中处理额外的变体不那么诱人，因为开始一个`#ifdef`级联比扩展现有级联的可能性小。

通过独立的功能，每种变体都可以轻松地扩展为另一种变体。为了实现这一点，在一个函数中只需要添加一个`#ifdef`分支即可，而处理其他种类变体的函数则不需要改动。

## 已知用途

下面的示例展示了此模式的应用：

+   OpenSSL 实现文件*threads_pthread.c*包含了线程处理的函数。有用于抽象操作系统的独立函数，以及用于抽象 pthread 是否可用的独立函数。

+   SQLite 的代码包含用于抽象特定操作系统文件访问（例如`fileStat`函数）的函数。代码用其他独立函数抽象与文件访问相关的编译时特性。

+   Linux 函数`boot_jump_linux`调用另一个函数，根据处理器体系结构执行不同的引导操作，通过该函数中的`#ifdef`语句选择配置资源（USB、网络等）进行清理。

## 应用于运行示例

现在，对于函数确定目录路径，使用原子原语，你有以下代码：

```cpp
void getHomeDirectory(char* dirname)
{
  #ifdef __unix__
    sprintf(dirname, "%s%s", getenv("HOME"), "/newdir/");
  #elif defined _WIN32
    sprintf(dirname, "%s%s%s", getenv("HOMEDRIVE"), getenv("HOMEPATH"),
            "\\newdir\\");
  #endif
}

void getWorkingDirectory(char* dirname)
{
  #ifdef __unix__
    strcpy(dirname, "newdir/");
  #elif defined _WIN32
    strcpy(dirname, "newdir\\");
  #endif
}

void getDirectoryName(char* dirname)
{
  #ifdef STORE_IN_HOME_DIR
    getHomeDirectory(dirname);
  #elif defined STORE_IN_CWD
    getWorkingDirectory(dirname);
  #endif
}
```

现在代码变体被非常好地隔离。为了获取目录名，现在没有一个复杂的函数有许多`#ifdef`，而是有几个只有一个`#ifdef`的函数。这使得理解代码变得更容易，因为现在每个函数只执行一件事情，而不是用`#ifdef`级联区分几种变体。

函数现在非常简单易读，但你的实现文件仍然很长。此外，一个实现文件既包含主程序逻辑，又包含区分变体的代码，这使得并行开发或在程序逻辑旁边独立测试变体代码几乎不可能。

为了改进，将实现文件分成依赖于变体和独立于变体的文件。为此，创建一个抽象层。

# 抽象层

## 上下文

在你的代码中，你有用`#ifdef`语句区分的平台变体。你可能有用于分离变体和确保具有原子原语的隔离原语，以确保在你的程序逻辑中分离变体。

## 问题

**您希望在代码库中的多个位置使用处理平台变体的功能，但不希望复制该功能的代码。**

您的调用者可能习惯于直接使用特定于平台的函数，但您不再希望如此，因为每个调用者都必须自行实现平台变体。通常情况下，调用者不应该处理平台变体。在调用者的代码中，不需要了解不同平台的实现细节，调用者也不需要使用任何 `#ifdef` 语句或包含任何特定于平台的头文件。

您甚至在考虑与不负责平台无关代码的不同程序员分开开发和测试平台相关代码。

您希望能够在不要求调用此代码的调用者关心此更改的情况下稍后更改特定于平台的代码。如果平台相关代码的程序员为一个平台修复错误或添加额外的平台，则不应要求调用者的代码进行更改。

## 解决方案

**为每个需要特定于平台代码的功能提供一个 API。在头文件中仅定义平台无关函数，并将所有特定于平台的 `#ifdef` 代码放入实现文件中。您的函数的调用者仅包含您的头文件，不必包含任何特定于平台的文件。**

尝试为抽象层设计稳定的 API，因为以后更改 API 将需要更改调用者的代码，有时这是不可能的。然而，设计稳定的 API 非常困难。对于平台抽象，请试着查看不同的平台，甚至是您尚未支持的平台。在了解它们的工作方式和差异之后，您可以创建一个 API 来为这些平台的特性进行抽象。这样，即使在为不同平台添加支持时，您也不需要后续更改 API。

确保彻底记录 API。添加注释到每个函数以描述其功能。此外，如果在整个代码库中未明确定义的话，请描述这些函数支持的平台。

下面的代码展示了一个简单的抽象层：

*caller.c*

```cpp
#include "someFeature.h"

int main()
{
  someFeature();
  return 0;
}
```

*someFeature.h*

```cpp
/* Provides generic access to someFeature.
 Supported on platform A and platform B. */
void someFeature();
```

*someFeature.c*

```cpp
void someFeature()
{
  #ifdef PLATFORM_A
    performFeaturePlatformA();
  #elif defined PLATFORM_B
    performFeaturePlatformB();
  #endif
}
```

## 结果

抽象特性可以从代码的任何地方使用，而不仅仅是从单个实现文件中。换句话说，现在您有了调用者和被调用者的明确角色。被调用者必须处理平台变体，而调用者可以是平台无关的。

这种设置的好处是调用者无需处理特定于平台的代码。调用者只需包含提供的头文件，而无需包含任何特定于平台的头文件。缺点是调用者不能直接再使用所有特定于平台的函数。如果调用者习惯于这些函数，则可能不满意使用抽象功能，并可能发现使用或功能上不理想。

现在，平台特定的代码可以分开开发，甚至可以单独进行测试。现在测试工作量是可管理的，即使有多个平台，因为您可以模拟硬件特定代码，以便为平台独立代码编写简单的测试。

当为所有平台特定函数构建这种 API 时，这些函数和 API 的总和构成了代码库的平台抽象层。通过平台抽象层，非常清楚哪些代码是平台相关的，哪些是平台独立的。平台抽象层还清楚地表明，为了支持额外的平台，必须触及代码的哪些部分。

## 已知用途

以下示例展示了此模式的应用：

+   大多数在多个平台上运行的大规模代码都有硬件抽象层。例如，诺基亚的 Maemo 平台有这样一个抽象层，用于抽象加载的实际设备驱动程序。

+   lighttpd web 服务器的`sock_addr_inet_pton`函数将 IP 地址从文本转换为二进制形式。该实现使用`#ifdef`语句区分 IPv4 和 IPv6 的代码变体。API 的调用者看不到这种区别。

+   gzip 数据压缩程序的`getprogname`函数返回调用程序的名称。获取该名称的方法取决于操作系统，并通过实现中的`#ifdef`语句进行区分。调用者无需关心函数在哪个操作系统上调用。

+   硬件抽象用于本科论文“开源实时以太网堆栈的硬件抽象—设计、实现和评估”中描述的 Time-Triggered Ethernet 协议。硬件抽象层包含用于访问中断和定时器的函数。这些函数标记为`inline`以避免性能损失。

## 应用于运行示例

现在您有了一个更简化的代码片段。每个函数仅执行一个操作，并且您隐藏了关于不同变体的实现细节在 API 背后：

*directoryNames.h*

```cpp
/* Copies the path to a new directory with name "newdir"
 located in the user's home directory into "dirname".
 Works on Linux and Windows. */
void getHomeDirectory(char* dirname);

/* Copies the path to a new directory with name "newdir"
 located in the current working directory into "dirname".
 Works on Linux and Windows. */
void getWorkingDirectory(char* dirname);
```

*directoryNames.c*

```cpp
#include "directoryNames.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void getHomeDirectory(char* dirname)
{
  #ifdef __unix__
    sprintf(dirname, "%s%s", getenv("HOME"), "/newdir/");
  #elif defined _WIN32
    sprintf(dirname, "%s%s%s", getenv("HOMEDRIVE"), getenv("HOMEPATH"),
            "\\newdir\\");
  #endif
}

void getWorkingDirectory(char* dirname)
{
  #ifdef __unix__
    strcpy(dirname, "newdir/");
  #elif defined _WIN32
    strcpy(dirname, "newdir\\");
  #endif
}
```

*directorySelection.h*

```cpp
/* Copies the path to a new directory with name "newdir" into "dirname".
 The directory is located in the user's home directory, if STORE_IN_HOME_DIR
 is set or it is located in the current working directory, if STORE_IN_CWD
 is set. */
void getDirectoryName(char* dirname);
```

*directorySelection.c*

```cpp
#include "directorySelection.h"
#include "directoryNames.h"

void getDirectoryName(char* dirname)
{
  #ifdef STORE_IN_HOME_DIR
    getHomeDirectory(dirname);
  #elif defined STORE_IN_CWD
    getWorkingDirectory(dirname);
  #endif
}
```

*directoryHandling.h*

```cpp
/* Creates a new directory of the provided name ("dirname").
 Works on Linux and Windows. */
void createNewDirectory(char* dirname);
```

*directoryHandling.c*

```cpp
#include "directoryHandling.h"
#ifdef __unix__
  #include <sys/stat.h>
#elif defined _WIN32
  #include <windows.h>
#endif

void createNewDirectory(char* dirname)
{
  #ifdef __unix__
    mkdir(dirname,S_IRWXU);
  #elif defined _WIN32
    CreateDirectory (dirname, NULL);
  #endif
}
```

*main.c*

```cpp
#include <stdio.h>
#include <string.h>
#include "directorySelection.h"
#include "directoryHandling.h"

int main()
{
  char dirname[50];
  char filename[60];
  char* my_data = "Write this data to the file";
  getDirectoryName(dirname);
  createNewDirectory(dirname);
  sprintf(filename, "%s%s", dirname, "newfile");
  FILE* f = fopen(filename, "w+");
  fwrite(my_data, 1, strlen(my_data), f);
  fclose(f);
  return 0;
}
```

你的主程序逻辑文件最终完全独立于操作系统；特定于操作系统的头文件甚至没有包含在这里。使用抽象层分离实现文件使文件更易于理解，并且使得可以在代码的其他部分重用这些函数。此外，开发、维护和测试可以分为依赖于平台和独立于平台的代码。

如果你在抽象层后面有隔离的基元，并且根据它们抽象的种类进行了组织，那么你最终会得到一个硬件抽象层或操作系统抽象层。现在你有了比以前更多的代码文件，特别是处理不同变体的文件，你可能需要考虑将它们组织成软件模块目录。

现在使用抽象层 API 的代码非常干净，但在该 API 下面的实现仍然包含不同变体的`#ifdef`代码。这种做法的缺点是，如果需要支持额外的操作系统，这些实现必须进行修改并且会增长。为了避免在添加另一个变体时修改现有实现文件，可以分离变体实现。

# 分离变体实现

## 上下文

你将平台变体隐藏在一个抽象层后面。在特定于平台的实现中，你使用`#ifdef`语句区分代码变体。

## 问题

**特定于平台的实现仍然包含`#ifdef`语句来区分代码变体。这使得很难看到并选择应该为哪个平台构建代码的哪一部分。**

因为将不同平台的代码放在单个文件中，所以无法按文件选择特定于平台的代码。然而，像 Make 这样的工具通常负责通过 Makefile 选择应该编译哪些文件，以生成不同平台的变体。

当从高层次观点看代码时，不可能看出哪些部分是特定于平台的，哪些不是，但在将代码移植到另一个平台时，能够迅速看出哪些代码需要修改是非常理想的。

开闭原则表明，为了引入新特性（或移植到新平台），不应该必须修改现有代码。代码应该对这些修改开放。然而，通过`#ifdef`语句区分平台变体要求在引入新平台时必须修改现有实现，因为必须在现有函数中放置另一个`#ifdef`分支。

## 解决方案

**将每个变体实现放入单独的实现文件中，并根据需要为每个文件选择要为哪个平台编译。**

同一平台的相关函数仍然可以放在同一个文件中。例如，可以有一个文件收集 Windows 上所有 socket 处理函数的函数，以及一个类似的文件在 Linux 上也这样做。

对于每个平台单独的文件，可以使用 `#ifdef` 语句确定在特定平台上编译哪些代码是可以接受的。例如，*someFeatureWindows.c* 文件可以在整个文件中使用类似于 Include Guards 的 `#ifdef _WIN32` 语句：

*someFeature.h*

```cpp
/* Provides generic access to someFeature.
 Supported on platform A and platform B. */
  someFeature();
```

*someFeatureWindows.c*

```cpp
#ifdef _WIN32
  someFeature()
  {
    performWindowsFeature();
  }
#endif
```

*someFeatureLinux.c*

```cpp
#ifdef __unix__
  someFeature()
  {
    performLinuxFeature();
  }
#endif
```

除了在整个文件中使用 `#ifdef` 语句之外，还可以使用其他独立于平台的机制，例如 Make，在文件层面上决定在特定平台上编译哪些代码。如果你的 IDE 能帮助生成 Makefile，那么这种替代方案可能更为舒适，但要注意，当更改 IDE 时，可能需要重新配置在新 IDE 中要在哪些平台上编译哪些文件。

对于每个平台单独的文件，问题就来了，应该把这些文件放在哪里以及如何命名：

+   一种选择是将每个软件模块的平台特定文件放在一起，并以一种清晰表明它们覆盖哪个平台的方式命名（例如 *fileHandlingWindows.c*）。这种软件模块目录的优势在于软件模块的实现位于同一位置。

+   另一种选择是将代码库中所有平台特定的文件放入一个目录，并为每个平台创建一个子目录。这样做的好处是，一个平台的所有文件都在同一个地方，你可以更轻松地配置你的 IDE 来决定在哪个平台上编译哪些文件。

## 影响

现在，可以在代码中完全不使用任何 `#ifdef` 语句，而是通过工具（如 Make）在文件层面上区分不同的变体。

在每个实现文件中，现在只有一种代码变体，因此在阅读代码时不需要在不同的 `#ifdef` 分支之间跳转。这样阅读和理解代码会更加容易。

当在一个平台上修复 bug 时，无需触碰其他平台的文件。当移植到新平台时，只需添加新文件，无需修改现有文件或现有代码。

容易识别哪部分代码依赖于特定平台，以及哪些代码必须添加以便在新平台上进行移植。要么所有平台特定的文件在一个目录中，要么文件命名方式明确表明它们是依赖于特定平台的。

然而，将每个变体放入单独的文件中会创建许多新文件。你拥有的文件越多，构建过程就变得越复杂，代码编译时间就越长。你需要考虑如何组织文件结构，例如使用软件模块目录。

## 已知用法

以下示例展示了此模式的应用：

+   书籍《编写可移植代码：开发多平台软件入门》（Brian Hook 著，No Starch Press，2005 年）中介绍的简单音频库使用单独的实现文件为 Linux 和 OS X 提供访问线程和互斥锁的功能。实现文件使用 `#ifdef` 语句确保仅编译适合该平台的正确代码。

+   Apache Web 服务器的多处理模块负责处理对 Web 服务器的访问，为 Windows 和 Linux 分别实现了单独的实现文件。实现文件使用 `#ifdef` 语句确保仅编译适合该平台的正确代码。

+   U-Boot bootloader 的代码将其支持的每个硬件平台的源代码放入单独的目录中。每个目录包含文件 *cpu.c*，其中包含重置 CPU 的函数。一个 Makefile 决定应该编译哪个目录（以及哪个 *cpu.c* 文件）——这些文件中没有 `#ifdef` 语句。U-Boot 的主程序逻辑调用该函数来重置 CPU，在这一点上不必关心硬件平台的细节。

## 应用于运行示例

分离变体实现后，您将得到以下用于创建目录并向文件写入数据的功能的最终代码：

*directoryNames.h*

```cpp
/* Copies the path to a new directory with name "newdir"
 located in the user's home directory into "dirname".
 Works on Linux and Windows. */
void getHomeDirectory(char* dirname);

/* Copies the path to a new directory with name "newdir"
 located in the current working directory into "dirname".
 Works on Linux and Windows. */
void getWorkingDirectory(char* dirname);
```

*directoryNamesLinux.c*

```cpp
#ifdef __unix__
  #include "directoryNames.h"
  #include <string.h>
  #include <stdio.h>
  #include <stdlib.h>

  void getHomeDirectory(char* dirname)
  {
    sprintf(dirname, "%s%s", getenv("HOME"), "/newdir/");
  }

  void getWorkingDirectory(char* dirname)
  {
    strcpy(dirname, "newdir/");
  }
#endif
```

*directoryNamesWindows.c*

```cpp
#ifdef _WIN32
  #include "directoryNames.h"
  #include <string.h>
  #include <stdio.h>
  #include <windows.h>

  void getHomeDirectory(char* dirname)
  {
    sprintf(dirname, "%s%s%s", getenv("HOMEDRIVE"), getenv("HOMEPATH"),
            "\\newdir\\");
  }

  void getWorkingDirectory(char* dirname)
  {
    strcpy(dirname, "newdir\\");
  }
#endif
```

*directorySelection.h*

```cpp
/* Copies the path to a new directory with name "newdir" into "dirname".
 The directory is located in the user's home directory, if STORE_IN_HOME_DIR
 is set or it is located in the current working directory, if STORE_IN_CWD
 is set. */
void getDirectoryName(char* dirname);
```

*directorySelectionHomeDir.c*

```cpp
#ifdef STORE_IN_HOME_DIR
  #include "directorySelection.h"
  #include "directoryNames.h"

  void getDirectoryName(char* dirname)
  {
    getHomeDirectory(dirname);
  }
#endif
```

*directorySelectionWorkingDir.c*

```cpp
#ifdef STORE_IN_CWD
  #include "directorySelection.h"
  #include "directoryNames.h"

  void getDirectoryName(char* dirname)
  {
    return getWorkingDirectory(dirname);
  }
#endif
```

*directoryHandling.h*

```cpp
/* Creates a new directory of the provided name ("dirname").
 Works on Linux and Windows. */
void createNewDirectory(char* dirname);
```

*directoryHandlingLinux.c*

```cpp
#ifdef __unix__
  #include <sys/stat.h>

  void createNewDirectory(char* dirname)
  {
    mkdir(dirname,S_IRWXU);
  }
#endif
```

*directoryHandlingWindows.c*

```cpp
#ifdef _WIN32
  #include <windows.h>

  void createNewDirectory(char* dirname)
  {
    CreateDirectory(dirname, NULL);
  }
#endif
```

*main.c*

```cpp
#include "directorySelection.h"
#include "directoryHandling.h"
#include <string.h>
#include <stdio.h>

int main()
{
  char dirname[50];
  char filename[60];
  char* my_data = "Write this data to the file";
  getDirectoryName(dirname);
  createNewDirectory(dirname);
  sprintf(filename, "%s%s", dirname, "newfile");
  FILE* f = fopen(filename, "w+");
  fwrite(my_data, 1, strlen(my_data), f);
  fclose(f);
  return 0;
}
```

此代码中仍然存在 `#ifdef` 语句。每个实现文件都有一个大的 `#ifdef`，以确保为每个平台和变体编译正确的代码。或者，决定应编译哪些文件可以放入 Makefile 中。这样可以摆脱 `#ifdef`，但您只需使用另一种机制来选择变体。决定使用哪种机制并不那么重要。如本章所述，更重要的是隔离和抽象变体。

使用其他机制处理变体时，代码文件看起来更清晰，但复杂性仍然存在。将复杂性放入 Makefile 中是个好主意，因为 Makefile 的目的是决定要构建哪些文件。在其他情况下，最好使用 `#ifdef` 语句。例如，如果正在构建特定于操作系统的代码，可能会使用专有的 Windows IDE 和 Linux IDE 来决定要构建哪些文件。在这种情况下，在代码中使用 `#ifdef` 语句的解决方案要干净得多；通过 `#ifdef` 语句只需配置一次要为哪个操作系统构建哪些文件，并且在切换到另一个 IDE 时无需更改。

运行示例的最终代码非常清晰地展示了如何逐步改进具有特定操作系统变体或其他变体的代码。与第一个代码示例相比，这段最终代码可读性强，可以轻松扩展以增加额外功能或移植到其他操作系统，而无需触及现有代码中的任何部分。

# 摘要

本章介绍了如何处理 C 代码中的变体，如硬件或操作系统变体，以及如何组织和摆脱`#ifdef`语句。

避免变体模式建议使用标准化函数而不是自行实现的变体。每当适用时，都应该应用这种模式，因为它一举解决了代码变体的问题。然而，并非总是有标准化函数可用，在这种情况下，程序员必须实现自己的函数来抽象变体。作为一种起步，孤立的原始元素建议将变体放入单独的函数中，而原子原始元素建议在这些函数中仅处理一种类型的变体。抽象层进一步采取了隐藏原始元素实现在 API 后面的附加步骤。分割变体实现建议将每个变体放入单独的实现文件中。

将这些模式作为编程词汇的一部分，C 程序员可以获得一个工具箱和逐步指导，以便处理 C 代码中的各种变体，从而组织代码并摆脱`#ifdef`地狱。

对于经验丰富的程序员来说，有些模式可能看起来像是显而易见的解决方案，这是一件好事。模式的一个任务是教育人们如何做正确的事情；一旦他们知道如何做正确的事情，模式就不再必要，因为人们会直观地按照模式建议的方式去做。

# 进一步阅读

如果你已经准备好继续学习，这里有一些资源可以帮助你进一步了解平台和变体抽象。

+   由 Brian Hook（No Starch Press，2005 年）撰写的书籍《编写可移植代码：多平台软件开发入门》描述了如何在 C 语言中编写可移植代码。该书涵盖了操作系统变体和硬件变体，并通过针对特定情况的建议，如处理字节顺序、数据类型大小或行分隔符标记，来提供指导。

+   文章[“#ifdef 被认为有害”](https://oreil.ly/eZ2CW)由 Henry Spencer 和 Geoff Collyer 撰写，是最早对使用`#ifdef`语句持怀疑态度的文章之一。该文章详细阐述了在结构不清晰地使用它们时可能出现的问题，并提供了替代方案。

+   文章[“编写可移植代码”](https://oreil.ly/XkTbj)由 Didier Malenfant 描述了如何构建可移植代码以及应该将哪些功能放置在抽象层下面。

# 展望

现在你已经掌握了更多的模式。接下来，你将学习如何应用这些模式，以及前几章的模式。下一章将涵盖更大的代码示例，展示所有这些模式的应用。
