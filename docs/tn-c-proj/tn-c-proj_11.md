# 11 文件查找器

在古代，我编写过最流行的 MS-DOS 工具之一是快速文件查找器。当然，它并不特别快。但给定一个文件名时，它可以在 PC 的硬盘上找到任何位置的文件。这个程序包含在我早期许多计算机书籍提供的配套软盘上。是的，软盘。

在今天的操作系统上，查找文件是一个大问题。Windows 和 Mac OS X 都提供了强大的文件查找工具，不仅可以通过名称，还可以通过日期、大小和内容来定位文件。Linux 命令提示符提供了自己的文件查找工具，这些工具与它们的图形界面工具一样强大（如果不是更强大）。对于一位初学者 C 程序员，或者任何想要提高他们的 C 语言技能的人来说，使用这些工具是有用的，但仅仅使用这些工具并不能提高你的编程技能。

搜索文件，以及可能对它们进行某些操作，依赖于第十章中介绍的目录探险工具。从这个基础出发，你可以通过以下方式扩展你的 C 语言知识：

+   审查其他文件查找工具

+   探索查找文本的方法

+   在目录树中定位文件

+   使用通配符匹配文件

+   查找文件名重复项

当我编写一个实用程序时，尤其是与已经可用的类似实用程序时，我会寻找改进。许多命令行工具都有一系列选项和功能。这些开关使命令变得强大，但超出了我的需求。我发现选项的丰富性令人不知所措。对我而言，构建一个更具体的实用程序版本会更好。虽然这样的程序可能没有像过去那些经验丰富的 C 程序员编写的程序那样强大，但它符合我的需求。通过编写自己的文件工具，你可以更多地了解 C 语言编程，并且你可以使用这个工具——并根据自己的工作流程进行定制。

## 11.1 大型文件搜索

我个人文件查找工具是基于对现有 Linux 文件查找工具的挫败感——特别是 *find* 和 *grep*。

这些命令没有问题，一些精心挑选的咒骂词就能解决。然而，我发现我无法记住命令格式和选项。在需要使用这些文件查找工具时，我总是不断查阅文档。我明白这种承认可能会让我被赶出社区计算机俱乐部。

*find* 命令功能强大。在 Linux 中，这种强大意味着有大量的选项，通常比字母表中的字母还要多——大小写字母。这种复杂性解释了为什么许多极客宁愿使用图形界面文件搜索工具，而不是终端窗口来查找丢失的文件。

这是 find 命令的看似简单的格式：

```
find path way-too-many-options
```

是的。很简单。

假设你想要定位一个名为 budget.csv 的文件，它位于你的主目录树中的某个位置。以下是你要使用的命令：

```
find ~ -name budget.csv -print
```

路径名是~，代表你的主目录。-name 开关用于标识要查找的文件，budget.csv。最后的开关，-print（大家容易忘记的），指示*find*命令将结果发送到标准输出。你可能认为输出应该是必要的默认选项，但*find*命令可以对找到的文件做更多的事情，而不仅仅是将它们的名称发送到标准输出。

*find*命令期望的输出可能单独出现在一行上，这是幸运的。更常见的情况是，你必须筛选出一系列错误和重复的匹配。最终找到所需的文件，并显示其路径：

```
/home/dang/documents/financial/budget.csv
```

是的，你可以创建一个别名来指向你经常使用的特定*find*工具格式。不，我不会就*find*命令有多强大和有用或为什么我不把它与美味的棒棒糖进行比较进行辩论。

另一个文件查找命令是*grep*，我特别使用它来定位包含特定文本片段的文件。实际上，我在写这本书的时候多次使用 grep 来在头文件中定位定义的常量。从/usr/include 目录，以下是查找各种头文件中 time_t 定义常量的命令：

```
grep -r "time_t" *
```

-r 开关指示*grep*递归地遍历目录。要查找的字符串是 time_t，而*通配符指示程序搜索所有文件名。

当执行这个命令时，会输出许多行文本，因为 time_t 定义的常量在多个头文件中被引用。即使这个技巧也没有找到我想要的特定定义，但它确实指明了正确的方向。

这些工具——*find*和*grep*（以及它的更好伴侣，*egrep*）——非常棒且功能强大。然而，我想要一个友好且易于使用的工具，无需经常检查*man*页面或参考厚重的命令行参考书籍。这就是为什么我编写了自己的版本，这些版本在本章中有所介绍。

有了你对 C 语言的知识，你可以轻松地编写满足你需求的特定文件查找工具，无论是复杂还是简单。然后，如果你忘记了任何选项，你只能怪自己。

## 11.2 文件查找器

我寻找文件的目标是输入这样的命令：

```
find thisfile.txt
```

工具深入当前目录树，逐个子目录地搜索，寻找特定的文件。如果找到，将输出完整的路径名——对我非常有用。添加使用通配符定位文件的能力，我就再也不需要使用*find*命令了——在特定格式下查找文件。

哦，对了——我想我的工具名字不能叫*find*，因为 Linux 中已经使用了这个名字。那叫*ff*，代表*Find File*怎么样？

### 11.2.1 编写文件查找器工具

第十章涵盖了目录探索的过程，使用递归的 *dir()* 函数来深入子目录。在这个函数的基础上构建是创建文件查找工具的完美选择。目标是扫描目录，并将找到的文件与用户提供的匹配文件名进行比较。

本章中介绍的 *Find File* 工具不使用第十章中的相同 *dir()* 函数。不，递归目录查找函数需要修改以定位特定文件，而不是所有文件。我已将函数重命名为 *find()*，因为我知道这个名字会激怒查找工具。

我的 *find()* 函数具有与第十章中的 *dir()* 相同的前两个参数。但如下一列表所示，这个更新的函数添加了一个第三个参数 match，以帮助寻找命名的文件。*dir()* 和 *find()* 之间的其他差异在列表中已注释说明。

列表 11.1 递归 *find()* 函数

```
void find(char *dirpath,char *parentpath,char *match)
{
    DIR *dp;
    struct dirent *entry;
    struct stat fs;
    char subdirpath[PATH_MAX];                        ❶

    dp = opendir(dirpath);
    if( dp==NULL )
    {
        fprintf(stderr,"Unable to read directory '%s'\n",
                dirpath
               );
        exit(1);
    }

    while( (entry=readdir(dp)) != NULL )
    {
        if( strcmp(entry->d_name,match)==0 )          ❷
        {
            printf("%s/%s\n",dirpath,match);          ❸
            count++;                                  ❹
        }

        stat(entry->d_name,&fs);
        if( S_ISDIR(fs.st_mode) )
        {
            if( strncmp( entry->d_name,".",1)==0 )    ❺
                continue;
            if( chdir(entry->d_name)==-1 )
            {
                fprintf(stderr,"Unable to change to %s\n",
                        entry->d_name
                       );
                exit(1);
            }

            getcwd(subdirpath,BUFSIZ);
            find(subdirpath,dirpath,match);           ❻
        }
    }

    closedir(dp);

    if( chdir(parentpath)==-1 )
    {
        if( parentpath==NULL )
            return;
        fprintf(stderr,"Parent directory lost\n");
        exit(1);
    }
}
```

❶ 使用 limits.h 中的最大路径大小值（参见文本中的讨论）。

❷ 对找到的文件名与传递的文件名进行比较

❸ 输出任何匹配的文件名

❹ 增加外部 count 变量

❺ 避免检查隐藏文件

❻ 再次进行递归调用，这次将传递的文件名作为第三个参数

除了列表 11.1 中提到的添加之外，我还使用了定义的 PATH_MAX 常量，这需要包含 limits.h 头文件。因为并非每个 C 库都实现了 PATH_MAX，所以需要一些预处理指令：

```
#ifndef PATH_MAX
#define PATH_MAX 256
#endif
```

PATH_MAX 的值因操作系统而异。例如，在 Windows 上可能是 260 字节，但在我使用的 Ubuntu Linux 版本中，它是 1024 字节。我见过高达 4096 字节的，所以 256 似乎是一个不会导致任何问题的好值。如果你想定义一个更高的值，请随意定义。

我的 *Find File* 工具也计算匹配的文件数。为了跟踪，我使用了定义在外的变量 count。我非常不愿意使用全局变量，但在这个情况下，让 count 成为外部变量是跟踪找到的文件的有效方法。否则，我可以在 *find()* 函数中将 count 包含为第四个参数，但作为一个递归函数，保持其值的一致性会引入各种混乱。

包含 *find()* 函数的源代码文件名为 findfile01.c，其中 *main()* 函数在以下列表中显示。*main()* 函数的职责是从命令行获取文件名，检索当前路径，调用 *find()* 函数，然后报告结果。*main()* 函数如下所示。

列表 11.2 findfile01.c 中的 *main()* 函数

```
int main(int argc, char *argv[])
{
    char current[PATH_MAX];

    if( argc<2 )                            ❶
    {
        fprintf(stderr,"Format: ff filename\n");
        exit(1);
    }

    getcwd(current,PATH_MAX);
    if( chdir(current)==-1 )
    {
        fprintf(stderr,"Unable to access directory %s\n",
                current
               );
            exit(1);
    }

    count = 0;                              ❷
    printf("Searching for '%s'\n",argv[1]);
    find(current,NULL,argv[1]);             ❸
    printf(" Found %d match",count);        ❹
    if( count!=1 )                          ❺
        printf("es");
    putchar('\n');
    return(0);
}
```

❶ 需要一个命令行参数。

❷ 初始化外部 *int* 变量 count，用于跟踪找到的文件数量

❸ 调用函数，指定文件名参数为第三个参数

❹ 报告结果

❺ 对于除了 1 以外的任何 count 值，神奇地添加“es”

*find()* 和 *main()* 都包含在源代码文件 findfile01.c 中，该文件可在本书的在线仓库中找到。我已经将源代码构建到名为 *ff* 的程序文件中。以下是一些示例运行：

```
$ ff a.out
Searching for 'a.out'
/Users/Dan/code/a.out
/Users/Dan/code/communications/a.out
/Users/Dan/code/communications/networking/a.out
/Users/Dan/Tiny C Projects/code/08_unicode/a.out
/Users/Dan/Tiny C Projects/code/11_filefind/a.out
 Found 5 matches
```

*查找文件* 工具会定位我主目录树中的所有 a.out 文件：

```
$ ff hello
Searching for 'hello'
 Found 0 matches
```

在前面的例子中，该工具没有找到任何名为 hello 的文件：

```
$ ff *.c
Searching for 'finddupe01.c'
/Users/Dan/Tiny C Projects/code/11_filefind/finddupe01.c
 Found 1 match
```

该工具尝试定位当前目录中所有具有 .c 扩展名的文件。而不是返回所有文件，你只能看到第一个匹配的文件报告：finddupe01.c。这里的问题是代码没有识别通配符；它只找到特定的文件名。

要匹配具有通配符的文件，你必须了解一个称为 glob 的概念。与同名的 1958 年恐怖电影《 Blob》的主角不同，了解 glob 不会让你丧命。

### 11.2.2 理解 glob

glob 可以是来自外太空的粘稠物质，但在计算机世界中，它简称为 *glob*al。具体来说，*glob* 是一种使用通配符指定或匹配文件名的方法。我知道的大多数人都更喜欢说“通配符”而不是“glob”。但在编程领域，术语是 *glob*，过程是 *globbing*，而 globbing 的人被称为 globbers。值得关注的 C 库函数是 *glob()*。

作为复习，文件名通配符如下：

+   ? 匹配单个字符

+   * 匹配多个字符的组

在 Windows 中，通配符匹配会自动进行。但在 Linux 环境中，必须激活通配符功能才能展开通配符。如果不这样做，*和? 通配符将被字面地解释，这并不是大多数用户所期望的。

要确保 globbing 是激活的，请输入 **set -o** 命令。在输出中，*noglob* 选项应设置为 *off*：

```
noglob   off
```

如果你看到选项是开启的，请使用此命令：

set +o noglob

当通配符激活时，shell 会展开 ? 和 * 通配符以匹配文件。在上一个部分中，提供的输入是 *.c。然而，程序只处理了一个名为 finddupe01.c 的文件。文件名是匹配的，但它不是目录中唯一的 *.c 文件名。这是怎么回事？

下一个列表中的代码有助于你理解在命令提示符中输入通配符时 globbing 的工作方式。从 glob01.c 生成的程序遍历所有输入的命令行选项，除了第一个项目，即程序文件名。

列表 11.3 glob01.c 的源代码

```
#include <stdio.h>

int main(int argc, char *argv[])
{
    int x;

    if( argc>1 )                    ❶
    {
        for( x=1; x<argc; x++ )     ❷
            printf("%s\n",argv[x]);
    }

    return(0);
}
```

❶ 如果在提示符下只输入程序名，无需烦恼。

❷ 遍历所有参数

这里是使用 glob01.c 创建的程序的一个示例运行，该程序被命名为 a.out：

```
$ ./a.out this that the other
this
that
the
other
```

该程序忠实地回显所有命令行选项。现在尝试运行相同的程序，但指定一个通配符：

```
$ ./a.out *.c
finddupe01.c
finddupe02.c
finddupe03.c
finddupe04.c
finddupe05.c
findfile01.c

findfile02.c
glob01.c
glob02.c
```

*.c 通配符（globby 事物）由 shell 展开，将当前目录中每个匹配的文件名作为命令行参数传递给程序。而不是提供一个单一的参数 *.c，而是提供了多个参数。

使用通配符匹配的问题在于，你的程序实际上并不知道是否提供了多个命令行参数，或者输入了一个通配符并进行了展开。此外，由于通配符参数被转换成多个匹配的文件，你无法知道指定了哪个通配符。或许存在某种方法，因为我知道一些能够以惊人方式执行通配符匹配的实用程序，但我还没有发现这种魔法的本质。

而不是感到困惑，你可以依赖 *glob()* 函数为你执行模式匹配。以下是 *man* 页面的格式：

```
int glob(const char *pattern, int flags, int (*errfunc) (const char *epath, int eerrno), glob_t *pglob);
```

函数有四个参数：

+   const char *pattern 是一个路径名通配符模式，用于匹配。

+   int flags 是用于自定义函数行为的选项，通常是一系列逻辑 OR 一起定义的常量。

+   int (*errfunc) 是一个错误处理函数的名称（及其两个参数），这是必要的，因为 *glob()* 函数可能有些古怪。指定 NULL 以使用默认的错误处理器。

+   glob_t *pglob 是一个包含匹配文件详细信息的结构。两个有用的成员是 gl_pathc，它列出了匹配文件的数量，以及 gl_pathv，它作为指向当前目录中匹配文件名的指针列表的基址。

*glob()* 函数在成功时返回零。其他返回值包括定义的常量，你可以测试这些常量以确定函数是否出错或未能找到任何匹配的文件。

关于 *glob()* 函数的更多精彩细节可以在 *man* 页面中找到。请特别注意 flags 参数，因为它容易引发各种问题。

你必须在源代码中包含 glob.h 头文件，以使编译器知道 *glob()* 函数。

在下一个列表中，glob02.c 的源代码使用 *glob()* 函数在当前目录中搜索匹配的文件。用户被提示输入。输入字符串被清除任何换行符。调用 glob() 函数处理输入，搜索与指定的任何通配符匹配的文件名。最后，一个 *while* 循环输出匹配的文件名。

列表 11.4 glob02.c 的源代码

```
#include <stdio.h>
#include <stdlib.h>
#include <glob.h>
#include <limits.h>                                                  ❶

#ifndef PATH_MAX                                                     ❷
#define PATH_MAX 256
#endif

int main()
{
    char filename[PATH_MAX];
    char *r;
    int g;                                                           ❸
    glob_t gstruct;                                                  ❹
    char **found;                                                    ❺

    printf("Filename or wildcard: ");                                ❻
    r = fgets(filename,PATH_MAX,stdin);
    if( r==NULL )
        exit(1);
    while( *r!='\0' )
    {
        if( *r=='\n' )
        {
            *r = '\0';
            break;
        }
        r++;
    }

    g = glob(filename, GLOB_ERR , NULL, &gstruct);                   ❼
    if( g!=0 )                                                       ❽
    {
        if( g==GLOB_NOMATCH )
            fprintf(stderr,"No matches for '%s'\n",filename);
        else
            fprintf(stderr,"Some kinda glob error\n");
        exit(1);
    }

    printf("Found %zu filename matches\n",[CA]gstruct.gl_pathc);     ❾
    found = gstruct.gl_pathv;                                        ❿
    while(*found)                                                    ⓫
    {
        printf("%s\n",*found);                                       ⓬
        found++;                                                     ⓭
    }

    return(0);
}
```

❶ 对于 PATH_MAX 的定义——如果可用

❷ 如果 PATH_MAX 没有定义，则创建它

❸ *glob()* 的返回值

❹ 在 *glob()* 函数中指定的结构

❺ 一个指向匹配文件名列表的双指针

❻ 提示输入文件名通配符；这段代码是为了验证输入并删除换行符。

❼ 对 *glob()* 函数的调用，大多数情况下默认，除了 GLOB_ERR 标志

❽ 检查错误，特别是没有匹配的文件名

❾ 使用结构成员 gl_pathc 输出匹配项；占位符 %zu 用于 *size_t* 值。

❿ gl_pathv 成员是指针列表的基址，被分配给找到的双指针。

⓫ 当 *found* 引用的字符串不为 NULL 时循环

⓬ 输出匹配的文件名

⓭ 增加找到的指针以引用列表中的下一个项目

记住，通配符输入必须由用户提供，因为程序不会将通配符输入解释为命令行参数。以下是一个示例运行：

```
Filename or wildcard: find*
Found 7 filename matches
finddupe01.c
finddupe02.c
finddupe03.c
finddupe04.c
finddupe05.c
findfile01.c
findfile02.c
```

程序成功找到了所有以 find 开头的文件。现在可以将源代码中使用的技术整合到*查找文件*实用程序中，以便在搜索中使用通配符。

### 11.2.3 使用通配符查找文件

需要对查找文件实用程序进行一些修改，以便利用通配符。为了辅助*glob()*函数，现在必须在提示符下输入匹配的文件名，类似于前一小节中的 glob02.c 程序。然后必须将*glob()*函数集成到*find()*函数中，以帮助搜索子目录中的匹配文件名。

对*main()*函数的修改可以在源代码文件 findfile02.c 中找到，该文件可在在线仓库中找到。这些更新反映了从 glob02.c 源代码文件中添加的语句，主要是接受和确认有关通配符的输入。其余的修改如下所示，其中*glob()*函数被集成到*find()*函数中。在这个版本的代码中，字符串参数 match 可以是特定文件名或包含通配符的文件名。

列表 11.5 源代码文件 findfile02.c 中的*find()*函数

```
void find(char *dirpath,char *parentpath,char *match)
{
    DIR *dp;
    struct dirent *entry;
    struct stat fs;
    char subdirpath[PATH_MAX];
    int g;
    glob_t gstruct;
    char **found;
    dp = opendir(dirpath);
    if( dp==NULL )
    {
        fprintf(stderr,"Unable to read directory '%s'\n",
                dirpath
               );
        exit(1);
    }

    g = glob(match, GLOB_ERR, NULL, &gstruct);    ❶
    if( g==0 )                                    ❷
    {
        found = gstruct.gl_pathv;
        while(*found)
        {
            printf("%s/%s\n",dirpath,*found);
            found++;
            count++;
        }
    }

    while( (entry=readdir(dp)) != NULL )          ❸
    {
        stat(entry->d_name,&fs);
        if( S_ISDIR(fs.st_mode) )                 ❹
        {
            if( strncmp( entry->d_name,".",1)==0 )
                continue;
            if( chdir(entry->d_name)==-1 )
            {
                fprintf(stderr,"Unable to change to %s\n",
                        entry->d_name
                       );
                exit(1);
            }

            getcwd(subdirpath,BUFSIZ);
            find(subdirpath,dirpath,match);
        }
    }

    closedir(dp);

    if( chdir(parentpath)==-1 )
    {
        if( parentpath==NULL )
            return;
        fprintf(stderr,"Parent directory lost\n");
        exit(1);
    }
}
```

❶ 使用*glob()*在目录中查找匹配的文件

❷ 成功后，输出找到的文件（这里而不是下面）

❸ 此循环仍然必要，以查找和探索子目录。

❹ 只在这里查找目录文件；匹配的文件已经输出。

在其最终版本中，*查找文件*实用程序（源代码文件 findfile02.c）会提示输入，可以是特定文件或通配符。当前目录及其所有子目录中的所有文件都会被搜索，并报告结果：

```
$ ff
Filename or wildcard: *.c
Searching for '*.c'
/Users/Dan/code/0424a.c
/Users/Dan/code/0424b.c
...
/Users/Dan/Tiny C Projects/code/11_filefind/sto/unique04.c
/Users/Dan/Tiny C Projects/code/11_filefind/sto/unique05.c
/Users/Dan/Tiny C Projects/code/11_filefind/sto/unique06.c
 Found 192 matches
```

在这里，*查找文件*实用程序在我的主目录中找到了 192 个 C 源代码文件。

```
$ ff
Filename or wildcard: *deposit*
Searching for '*deposit*'
/Users/Dan/Documents/bank deposit.docx
 Found 1 match
```

在这里显示的示例运行中，*查找文件*实用程序找到了我的银行存款文件。程序中包含的*glob()*函数允许有效地使用通配符。尽管当输入完整名称时，程序仍然可以定位特定文件：

```
$ ff
Filename or wildcard: ch03.docx
Searching for 'ch03.docx'
/Users/Dan/Documents/Word/text/ch03.docx
 Found 1 match
```

正如我之前写的，我经常使用这个实用程序，因为它简单，并且能生成我想要的结果。我不希望继续修改这个实用程序，这可能会最终导致我重新发明*find*程序。不，相反，查找文件的概念可以进一步扩展到定位重复文件。

## 11.3 重复文件查找器

我最喜欢的 MS-DOS 共享软件实用程序之一是*finddupe*。我在 Windows 中找不到类似的东西（尽管我没有积极寻找）。该实用程序的版本仍然可在 Windows 的命令行界面中使用。它不仅通过名称，还通过内容查找重复文件。*finddupe*是清理和组织大容量存储设备上文件的便捷工具。

我从未费心编写自己的*finddupe*实用程序，主要是因为现有的工具很棒。即便如此，我经常思考这个过程：程序不仅必须扫描所有目录，还必须记录文件名。从记录的文件名列表中，每个都必须与其他列表中的文件进行比较，以查看是否存在相同的名称。一想到比较文件内容的过程，我就感到不寒而栗。

尽管如此，这个话题仍然吸引了我：你是如何扫描子目录中的文件，然后检查是否找到任何重复的名称的？

创建*Find Dupe*实用程序的过程大量借鉴了第十章中介绍的子目录扫描工具，以及本章前面使用过的工具。但其余的代码——记录和扫描保存的文件列表——是新的领域：必须创建一个文件列表。该列表必须被扫描以查找重复项，然后输出重复项及其路径名。

### 11.3.1 构建文件列表

就像任何编程项目一样，我尝试了多次才成功地构建了一个包含子目录中找到的文件的列表。很明显，我需要某种结构来保存文件信息：名称、路径等等。但是，我是创建一个动态数组（分配指针）的结构，使用链表，使结构数组外部化，还是放弃并成为一名乳牛农场主呢？

对我来说，将任何变量外部化都是最后的手段。有时这是唯一的选择，但永远不应该是因为它容易做到而选择。正如本章前面所展示的，有时这是必要的，因为实现变量的其他方式很笨拙。特别是与递归一起使用时，外部变量可以解开一些否则处于圣诞树灯水平上的结。

剩下的两个选项是传递一个动态分配的列表或使用链表。我编写了几次代码，其中将动态分配的结构列表传递给递归函数。它失败了，这是很容易理解的，因为指针在递归中可能会丢失。因此，我唯一剩下的选择是创建一个链表。

链表结构必须有一个成员，指向列表中的下一个项目，即*节点*。这个成员成为存储找到的文件名及其路径的结构的一部分。以下是它的定义：

```
struct finfo {
  int index;
  char name[BUFSIZ];
  char path[PATH_MAX];
  struct finfo *next;
};
```

我最初将这个结构命名为 fileinfo。我本想保留这个名称，但这本书的边距只有那么宽，我不喜欢源代码的换行。所以，我决定使用 finfo。这个结构包含四个成员：

+   index，它记录找到的文件数量（避免使用外部变量）

+   name，包含找到的文件名

+   path，包含文件的完整路径

+   next，它引用链表中的下一个节点，或 NULL 表示列表的末尾

这个结构必须声明为外部，以便代码中的所有函数都理解其定义。

我对这个程序的第一次构建仅仅是看看它是否工作：即链表是否被正确分配、填充，并且从递归函数中返回。在下一部分中，你可以看到 *main()* 函数。它为链表分配第一个节点。这个结构必须为空；是递归函数 *find()* 构建了链表。*main()* 函数获取递归函数的起始目录。完成后，一个 *while* 循环输出链表引用的文件名。

列表 11.6 从 finddupe01.c 的 *main()* 函数

```
int main()
{
    char startdir[PATH_MAX];
    struct finfo *first,*current;                  ❶

    first = malloc( sizeof(struct finfo) * 1 );    ❷
    if( first==NULL )                              ❸
    {
        fprintf(stderr,"Unable to allocate memory\n");
        exit(1);
    }

    first->index = 0;                              ❹
    strcpy(first->name,"");                        ❹
    strcpy(first->path,"");                        ❹
    first->next = NULL;                            ❹

    getcwd(startdir,PATH_MAX);                     ❺
    if( chdir(startdir)==-1 )
    {
        fprintf(stderr,"Unable to access directory %s\n",
                startdir
               );
            exit(1);
    }

    find(startdir,NULL,first);                     ❻

    current = first;                               ❼
    while( current )                               ❽
    {
        if( current->index > 0 )                   ❾
            printf("%d:%s/%s\n",                   ❿
                    current->index,
                    current->path,
                    current->name
                   );
        current = current->next;                   ⓫
    }

    return(0);
}
```

❶ 需要一个指针用于基础（第一个）和检查列表中的项（当前）。

❷ 分配基础指针

❸ 确认指针已分配

❹ 将第一个节点填充为空值

❺ 获取 *find()* 函数调用的当前目录

❻ 调用递归函数

❼ 将当前指针设置为列表的开始

❽ 当当前指针不为 NULL 时循环

❾ 跳过列表中的第一个项目，零

❿ 输出索引值、路径名和文件名

⓫ 引用列表中的下一个项目

*while* 循环跳过了链表中的第一个节点，即空项。我可以通过将当前初始化语句替换为以下内容来避免 if( current->index > 0) 文本（前面已展示）：

```
current = first->next;
```

我现在才想到这个变化，所以你不会在源代码文件中找到它。无论如何，链表中的第一个节点被跳过了。

我为 *Find Dupe* 代码编写的 *find()* 函数是基于本章前面介绍的 *Find File* 工具中的 *find()* 函数。*find()* 函数的第三个参数被替换为链表当前节点的指针。该函数的任务是在当前目录中找到文件时创建新节点，并填充它们的结构。

下一部分显示了 *Find Dupe* 实用程序的 *find()* 函数，该函数在列表 11.6 中展示的 *main()* 函数中被调用。当在当前目录中找到文件时，该函数在列表中为新节点分配存储空间。这是函数的唯一添加。

列表 11.7 从 finddupe01.c 的 *find()* 函数

```
void find( char *dirpath, char *parentpath, struct finfo *f)
{
    DIR *dp;
    struct dirent *entry;
    struct stat fs;
    char subdirpath[PATH_MAX];
    int i;

    dp = opendir(dirpath);                                   ❶
    if( dp==NULL )
    {
        fprintf(stderr,"Unable to read directory '%s'\n",
                dirpath
               );
        exit(1);
        /* will free memory as it exits */
    }

    while( (entry=readdir(dp)) != NULL )
    {
        stat(entry->d_name,&fs);
        if( S_ISDIR(fs.st_mode) )                            ❷
        {
            if( strncmp( entry->d_name,".",1)==0 )
                continue;
            if( chdir(entry->d_name)==-1 )
            {
                fprintf(stderr,"Unable to change to %s\n",
                        entry->d_name
                       );
                exit(1);
            }
            getcwd(subdirpath,BUFSIZ);
            find(subdirpath,dirpath,f);
        }
        else                                                 ❸
        {
            f->next = malloc( sizeof(struct finfo) * 1);     ❹
            if( f->next == NULL )
            {
                fprintf(stderr,
                        "Unable to allocate new structure\n");
                exit(1);
            }

            i = f->index;                                    ❺
            f = f->next;                                     ❻
            f->index = i+1;                                  ❼
            strcpy(f->name,entry->d_name);                   ❽
            strcpy(f->path,dirpath);                         ❾
            f->next = NULL;                                  ❿
        }
    }

    closedir(dp);

    if( chdir(parentpath)==-1 )
    {
        if( parentpath==NULL )
            return;
        fprintf(stderr,"Parent directory lost\n");
        exit(1);
    }
}
```

❶ 获取当前目录——未改变

❷ 测试子目录和递归

❸ 如果不是子目录，则保存文件信息

❹ 分配链表中的下一个节点（并进行错误检查）

❺ 保存当前索引值

❻ 引用新分配的节点

❼ 更新索引值

❽ 保存文件名

❾ 保存路径名

❿ 初始化下一个指针；函数的其余部分与前面的示例相同。

*find()* 函数根据目录条目是子目录还是文件做出简单的决定。当找到子目录时，函数会递归调用。否则，在链表中分配一个新的节点，并记录文件条目信息。

finddupe01.c 的完整源代码可以在在线仓库中找到。以下是我在工作目录中样本运行的输出：

```
1:/Users/Dan/code/11_filefind/.finddupe01.c.swp
2:/Users/Dan/code/11_filefind/a.out
3:/Users/Dan/code/11_filefind/finddupe01.c
4:/Users/Dan/code/11_filefind/finddupe02.c
5:/Users/Dan/code/11_filefind/finddupe03.c
6:/Users/Dan/code/11_filefind/finddupe04.c
7:/Users/Dan/code/11_filefind/finddupe05.c
8:/Users/Dan/code/11_filefind/findfile01.c
9:/Users/Dan/code/11_filefind/findfile02.c
10:/Users/Dan/code/11_filefind/glob01.c
11:/Users/Dan/code/11_filefind/glob02.c
12:/Users/Dan/code/11_filefind/sto/findword01.c
13:/Users/Dan/code/11_filefind/sto/findword02.c
```

输出能够记录当前目录以及 sto 子目录中的文件。然而，当我切换到父目录（代码）并再次运行程序时，输出没有变化。它应该变化：我的代码目录在各个子目录中有超过 100 个文件。那么为什么输出没有变化呢？

我在猫的帮助下思考这个错误，试图找到解决方案。经过几声喵喵叫后，我想到：问题在于递归函数，这应该是我首先应该注意的线索。

当 *find()* 函数返回，或“展开”时，使用的是指针 f 的先前值，而不是递归调用中分配的新值。每次函数切换到父目录时，链表中创建的结构就会丢失，因为指针 f 被重置为传递给函数的原始值。唉。

幸运的是，解决方案很简单：返回指针 f。

更新代码只需要三个更改和一个添加。首先，*find()* 函数的数据类型必须从 void 更改为 struct finfo*：

```
struct finfo *find( char *dirpath, char *parentpath, struct finfo *f)
```

第二，递归调用必须捕获函数的返回值：

```
f = find(subdirpath,dirpath,f);
```

实际上，这个更改更新了指针 f，以反映递归调用后的新值。

第三，在 *chdir()* 函数的错误检查中的 *return* 语句必须指定变量 f 的值：

```
return(f);
```

最后，*find()* 函数必须在末尾有一个语句来返回指针 f 的值：

```
return(f);
```

这些更新可以在在线仓库的源代码文件 finddupe02.c 中找到。

构建代码后，程序能够准确扫描子目录，并且在返回父目录时保留链表。输出完整且准确：通过链表记录了当前目录及其子目录中找到的所有文件。

### 11.3.2 定位重复项

在 *Find Dupe* 程序中创建链表的目的是为了找到重复项。在某个时刻，必须扫描列表，并确定哪些文件名是重复的，以及重复项在哪些目录中找到。

我想到了几种完成这个任务的方法。其中大多数涉及创建第二个文件名列表。但我不想一个接一个地构建列表。相反，我在 finfo 结构中添加了一个新的成员，repeat，如下所示更新结构定义：

```
struct finfo {
  int index;
  int repeat;         ❶
  char name[BUFSIZ];
  char path[PATH_MAX];
  struct finfo *next;
};
```

❶ 新成员用于跟踪重复的文件名

repeat 成员跟踪一个名称重复的次数。它在 *find()* 函数中初始化为 1，因为每个节点被创建。毕竟，每个找到的文件名至少存在一次。

为了跟踪重复的文件名，在创建列表后扫描列表时，repeat 成员会增加。在 *main()* 函数中，嵌套循环就像冒泡排序一样工作。它按顺序将列表中的每个节点与列表中的其余节点进行比较。

为了执行第二次扫描，我需要在*main()*函数中声明另一个*struct* finfo 变量。这个变量*scan，除了*first 和*current 外，还用于扫描链表：

```
struct finfo *first,*current,*scan;
```

在输出列表的*while*循环之前添加嵌套的*while*循环。此嵌套循环使用*current 指针处理整个链表。*scan 指针在内层*while*循环中使用，以比较 current->name 成员与后续的 name 成员。当找到匹配项时，具有重复名称的文件的 current->repeat 结构成员递增，如下所示。

列表 11.8 finddupe03.c 中*main()*函数中添加的嵌套*while*循环

```
current = first;
while( current )                                          ❶
{
    if( current->index > 0 )                              ❷
    {
        scan = current->next;                             ❸
        while( scan )                                     ❹
        {
            if( strcmp(current->name,scan->name)==0 )     ❺
            {
                current->repeat++;                        ❻
            }
            scan = scan->next;                            ❼
        }
    }
    current = current->next;                              ❽
}
```

❶ 循环遍历列表，直到 current 的值为 NULL

❷ 跳过第一个空条目

❸ 获取下一个条目的地址，扫描从这里开始

❹ 循环直到扫描引用列表中的最后一个（NULL）节点

❺ 比较文件名

❻ 如果名称相同，则递增当前条目的重复计数器

❼ 继续扫描

❽ 继续在整个列表中递增，比较每个节点与列表中的其余部分

这些嵌套循环更新包含相同文件名的结构的 repeat 成员。它们后面跟着现有的输出列表的*while*循环。该循环中的*printf()*语句更新为输出重复值：

```
printf("%d:%s/%s (%d)\n",
        current->index,
        current->path,
        current->name,
        current->repeat
      );
```

所有这些更改都可在 finddupe03.c 源代码文件中找到，该文件可在在线仓库中获取。输出尚未显示重复文件。对*Find Dupe*系列源代码文件的这一增量改进仅输出相同的完整文件列表，但文件路径字符串末尾显示了重复次数：

```
163:/Users/Dan/code/sto/secret01.c (1)
```

*Find Dupe*程序的下一个更新可在源代码文件 finddupe04.c 中找到。从在线仓库获取此源代码文件，并在编辑器中显示它。随着文本的进行，回顾我将进行的两个改进。

首先，在*main()*函数顶部声明一个新的*int*变量 found，这是我更喜欢设置变量声明的地方：

```
int found = 0;
```

当在嵌套的*while*循环中发现重复的文件名时，found 的值重置为 1：

```
if( strcmp(current->name,scan->name)==0 )
{
    current->repeat++;
    found = 1;
}
```

找到的值不需要累积；它实际上是一个布尔变量。当它保持在 0 时，没有找到重复的文件名，然后执行以下语句：

```
if( !found )
{
    puts("No duplicates found");
    return(1);
}
```

输出“未找到重复项”的消息，然后程序以返回值 1 退出。

当 found 的值重置为 1 时，检测到重复的文件名。在*main()*函数中的第二个*while*循环继续处理列表。此循环更新为捕获并输出重复项，如下所示。同样，*scan 变量在嵌套的*while*循环中使用，但这次用于输出重复的文件名及其路径名。

列表 11.9 finddupe04.c 中*main()*函数中的第二个嵌套*while*循环

```
current = first;                                     ❶
while( current )
{
    if( current->index > 0 )
    {
        if( current->repeat > 1 )                    ❷
        {
            printf("%d duplicates found of %s:\n",   ❸
                    current->repeat,
                    current->name
                  );
            printf(" %s/%s\n",                       ❹
                    current->path,
                    current->name
                  );
            scan = current->next;                    ❺
            while( scan )
            {
                if( strcmp(scan->name,current->name)==0 )
                {
                    printf(" %s/%s\n",
                            scan->path,
                            scan->name
                          );
                }
                scan = scan->next;
            }
        }
    }
    current = current->next;
}
```

❶ 如前所述，遍历整个文件列表

❷ 寻找重复计数大于 1 的项

❸ 输出给定文件名的重复数量

❹ 输出当前文件名及其路径

❺ 开始嵌套循环以输出匹配文件名的名称和路径

对 finddupe04.c 代码的这次更新输出了一个更短的列表，只显示那些重复的文件名，并列出所有重复的名称及其路径。

例如，在我的编程树中，我看到程序输出中有五个 a.out 文件的重复：

```
5 duplicates found of a.out:
 /Users/Dan/code/a.out
 /Users/Dan/code/communications/a.out
 /Users/Dan/code/communications/networking/a.out
 /Users/Dan/Tiny C Projects/code/08_unicode/a.out
 /Users/Dan/Tiny C Projects/code/11_filefind/a.out
```

问题在于，重复项也显示了重复。因此，对于 a.out 文件的多次出现，输出继续如下：

```
4 duplicates found of a.out:
 /Users/Dan/code/communications/a.out
 /Users/Dan/code/communications/networking/a.out
 /Users/Dan/Tiny C Projects/code/08_unicode/a.out
 /Users/Dan/Tiny C Projects/code/11_filefind/a.out
3 duplicates found of a.out:
 /Users/Dan/code/communications/networking/a.out
 /Users/Dan/Tiny C Projects/code/08_unicode/a.out
 /Users/Dan/Tiny C Projects/code/11_filefind/a.out
```

这种输出效率低下，因为它反复列出重复项。原因是链表中重复的文件名结构中的重复成员大于 1。因为这个值在第一个重复的文件名输出时不会改变，所以代码捕获了所有重复项。

这个问题让我很沮丧，因为我既不想创建另一个结构成员，也不想回到康复中心。我的目标是避免在一个已经复杂且嵌套的*while*循环中产生异常。

我对这个问题思考了一段时间，但最终灵感来了，一个一行解决方案呈现在眼前：

```
scan->repeat = 0;
```

这条语句被添加到第二个嵌套*while*循环中，如列表 11.9 所示。它在检测到匹配的文件名之后出现：

```
while( scan )
{
    if( strcmp(scan->name,current->name)==0 )
    {
        printf(" %s/%s\n",
                scan->path,
                scan->name
               );
        scan->repeat = 0;
    }
    scan = scan->next;
}
```

在嵌套*while*循环中，在输出重复的文件名之后，其重复值被重置为 0。这种更改防止了重复的文件名在输出中再次出现。这种更改在源代码文件 finddupe05.c 中可用。

*Find Dupe*程序已完成：它扫描当前目录结构，列出匹配的文件名，显示所有重复项的完整路径名。

就像所有代码一样，*Find Dupe*系列可以进一步改进。例如，可以将文件大小添加到 finfo 结构中。可以输出文件名匹配和文件大小匹配。你也可以全力以赴，尝试匹配文件内容。所需系统的基本框架已在现有代码中提供。剩下的只是你改进它的愿望和必要的时间来预见那些注定会在过程中发生的不期而遇的事情。
