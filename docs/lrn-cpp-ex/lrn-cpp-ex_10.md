# 附录。更多资源

本附录包含每章提到的资源列表，以便于查阅。

## 第一章

+   国际标准化组织（ISO）的第 21 工作组（WG21）的一些详细信息可在 [`isocpp.org/std`](https://isocpp.org/std) 找到。

+   IncludeCpp 群组有一个 Discord 服务器，并在会议上经常设立摊位（[`www.includecpp.org/).`](https://www.includecpp.org/)

+   ISOCpp 网站有一个常见问题解答部分（[`isocpp.org/wiki/faq`](https://isocpp.org/wiki/faq)），提供了关于一些最近的 C++ 变更和宏观问题的概述。

+   C++ Insights ([`cppinsights.io/`](https://cppinsights.io/)) 将代码转换，使我们能够看到编译器为我们所做的工作。

+   Matt Godbolt 的 Compiler Explorer ([`godbolt.org/`](https://godbolt.org/)) 支持大量不同的编译器，使我们能够看到每个编译器的行为，而无需安装它们。

+   CppReference 列出了每个新特性的编译器支持列表（[`en.cppreference.com/w/cpp/compiler_support`](https://en.cppreference.com/w/cpp/compiler_support)）。

## 第二章

+   在 [`isocpp.org/get-started`](https://isocpp.org/get-started) 可以找到一份免费的 C++ 编译器列表。

+   我们使用了 `std::format`，但如果你使用的编译器还不支持该格式，你可能需要使用 `ftm` 库（[`fmt.dev/latest/index.html`](https://fmt.dev/latest/index.html)）。

+   我们使用了 `{}` 来初始化变量，但初始化是一个很大的话题，可能会变得复杂。Nicolai Josuttis 的演讲“C++ 初始化的噩梦”对此进行了详细阐述（见 [`www.youtube.com/watch?v=7DTlWPgX6zs`](https://www.youtube.com/watch?v=7DTlWPgX6zs)）。

+   在 Herb Sutter 的“每周问题大师”中，Herb Sutter 告诉我们几乎总是使用 `auto`（[`herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/`](https://herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/)）。

+   Jason Turner 在 C++ Weekly 中讨论了 `emplace` 与 `push_back` 的优缺点（[`www.youtube.com/watch?v=jKS9dSHkAZY`](https://www.youtube.com/watch?v=jKS9dSHkAZY)）。

+   Thomas Becker 在 2013 年就博客中讨论了右值引用（[`thbecker.net/articles/rvalue_references/section_01.html`](http://thbecker.net/articles/rvalue_references/section_01.html)）。

+   你可以直接在 Godbolt 上使用 `fmt` 库进行实验，请访问 [`godbolt.org/z/Eq5763.`](https://godbolt.org/z/Eq5763)

+   在他们的书籍《C++ 编程标准：101 条规则、指南和最佳实践》（Addison-Wesley Professional, 2004）中，Herb Sutter 和 Andrei Alexandrescu 建议优先使用算法调用而不是手写循环。

+   如果你的编译器不完全支持 ranges，你可以在 [`godbolt.org/z/YrnsTGbfx.`](https://godbolt.org/z/YrnsTGbfx) 进行实验。

+   我们简要地回顾了核心指南，其中建议我们不应通过使用`unsigned`来尝试避免负值（[`isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-nonnegative`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-nonnegative)）。

## 第三章

+   我们使用`random_device`为随机数生成器设置种子（[`en.cppreference.com/w/cpp/numeric/random/random_device`](https://en.cppreference.com/w/cpp/numeric/random/random_device)）

+   Angelika Langer 和 Klaus Kreft 合著了一本名为《Standard C++ IOStreams and Locales: Advanced Programmer’s Guide and Reference》（Addison-Wesley Professional, 2000）的书。

+   将 lambda 复制到`std::function`可能效率不高。Scott Meyers 在他的书《Effective Modern C++》（O'Reilly Media, Incorporated, 2014）中的“第 5 项：优先使用 auto 而非显式类型声明”中提供了全部细节。

+   我们提到了一个建议，即引入`std::function_ref`作为`std::function`的替代方案，以克服性能问题（[`www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0792r10.html`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0792r10.html)）。Open Standards 小组在 WG21 目录中收集了与 C++相关的各种建议和论文。

## 第四章

+   Howard Hinnant 的 2019 年“Meeting C++”演讲为`std::chrono`的设计提供了背景信息（[`www.youtube.com/watch?v=adSAN282YIw`](https://www.youtube.com/watch?v=adSAN282YIw)）。

+   整数值语法的根源在于 Ward Cunningham 的 CHECKS 模式语言（[`c2.com/ppr/checks.html`](http://c2.com/ppr/checks.html)），并由 Martin Fowler 的 Quantity 模式进一步探讨（[`martinfowler.com/eaaDev/Quantity.html`](https://martinfowler.com/eaaDev/Quantity.html)）。

+   如果你的编译器不完全支持 C++20 的`chrono`功能，克隆 Howard Hinannt 的日期库（[`github.com/HowardHinnant/date`](https://github.com/HowardHinnant/date)）以包含其`date.h`，并使用其定义（例如，`using` `date::operator<<;`）。

+   ISOCpp 的核心指南 SF.7 告诉我们不要在头文件的全局作用域中编写`using namespace`（[`isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive)）。

+   Jason Turner 的 C++ Weekly 第 34 期介绍了阅读汇编语言的基础（[`www.youtube.com/watch?v=my39Gpt6bvY`](https://www.youtube.com/watch?v=my39Gpt6bvY)）。

+   Rainer Grimm 的网站提供了编译和使用日期库的说明（[`www.modernescpp.com/index.php/calendar-and-time-zone-in-c-20-time-zones`](https://www.modernescpp.com/index.php/calendar-and-time-zone-in-c-20-time-zones)），你可能需要它来处理时区，正如 Howard Hinnant 的 GitHub 页面（[`howardhinnant.github.io/date/tz.html#Installation`](https://howardhinnant.github.io/date/tz.html#Installation)）所做的那样。

+   霍华德·欣南特为`chrono`编写了一系列示例和食谱([`github.com/HowardHinnant/date/wiki/Examples-and-Recipes`](https://github.com/HowardHinnant/date/wiki/Examples-and-Recipes))。

## 第五章

+   有关 ISO 交付成果的详细信息，包括 C++技术规范（TS），请参阅[`www.iso.org/deliverables-all.html`](https://www.iso.org/deliverables-all.html)。

+   关于编译时或静态反射的 TS，请参阅[`www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/n4856.pdf`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/n4856.pdf)。它正在路上，您可以通过[`developercommunity.visualstudio.com/t/implement-the-c-reflection-ts/826632`](https://developercommunity.visualstudio.com/t/implement-the-c-reflection-ts/826632)在 Visual Studio 中进行投票。

+   关于创建`variant`、`optional`或`any`变量的额外细节，请参阅[`www.cppstories.com/2018/07/in-place-cpp17/`](https://www.cppstories.com/2018/07/in-place-cpp17/)。

+   斯坦·拉瓦维杰解释说，当你需要一个随机数时，你不应该调用`rand()`，更不用说使用`rand()` `%` `100` ([`learn.microsoft.com/en-us/events/goingnative-2013/rand-considered-harmful`](https://learn.microsoft.com/en-us/events/goingnative-2013/rand-considered-harmful))。

## 第六章

+   关于隐藏的回顾，请参阅斯科特·梅耶斯（Scott Meyers）的《Effective C++》一书中的“项目 33：避免隐藏继承名称”。

+   当我们不使用声明或定义六个成员函数中的任何一个时，我们遵循了零规则([`isocpp.github.io/CppCoreGuidelines/CppCore Guidelines#Rc-zero`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero)`)。

+   关于公共、受保护和私有访问修饰符的行为的提醒，请参阅 CppReference ([`en.cppreference.com/w/cpp/language/access`](https://en.cppreference.com/w/cpp/language/access))。

+   简单快速多媒体库（SFML）在多媒体方面相对容易使用([`www.sfml-dev.org/index.php`](https://www.sfml-dev.org/index.php))。

+   零规则意味着避免用户声明任何特殊成员函数，核心指南甚至告诉我们如果可能的话避免定义默认值([`isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero`](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero))。

+   彼得·索默拉德建议如果定义了析构函数，则删除移动赋值操作。他称这种模式为 DesDeMovA：*析构* => *删除* *移动* *赋值* (参见[`www.youtube.com/watch?v=fs4lIN3_IlA`](https://www.youtube.com/watch?v=fs4lIN3_IlA)中的演讲或[`github.com/boostcon/cppnow_presentations_2019/blob/master/lightning_talks/Rule_of_DesDeMovA__Peter_Sommerlad__cppnow_05062019.pdf`](https://github.com/boostcon/cppnow_presentations_2019/blob/master/lightning_talks/Rule_of_DesDeMovA__Peter_Sommerlad__cppnow_05062019.pdf)中的概述)。

+   Howard Hinnant 举办了一场名为“关于移动语义你想要知道的一切”的演讲 ([`www.youtube.com/watch?v=vLinb2fgkHk`](https://www.youtube.com/watch?v=vLinb2fgkHk))，[`howardhinnant.github.io/classdecl.html`](http://howardhinnant.github.io/classdecl.html) 提供了一个简洁的概述。

+   有一个提议要扩展 C++11 的随机数生成器（见 [`wg21.link/P1932`](https://wg21.link/P1932)），它提供了关于当前引擎限制的更多详细信息。

+   想要了解更多关于 `std::weak_ptr` 的细节，请参阅 [`www.modernescpp.com/index.php/std-weak-ptr`](https://www.modernescpp.com/index.php/std-weak-ptr)。

## 第七章

+   Tim van Deurzen 在 2019 年的 Meeting C++ 上就结构化绑定进行了闪电演讲（见 [`www.youtube.com/watch?v=YC_TMAbHyQU)`](https://www.youtube.com/watch?v=YC_TMAbHyQU)）。

+   想要了解更多关于 `std::string_view` 的细节，请参阅 [`www.modernescpp.com/index.php/c-17-avoid-copying-with-std-string-view`](https://www.modernescpp.com/index.php/c-17-avoid-copying-with-std-string-view)。

+   Nico Josuttis 的著作 *《标准库，第二版》*（Addison-Wesley Professional，2005）是一本优秀的参考书，可以了解更多关于容器和算法等内容。

+   Donald Knuth 的著作 *《计算机程序设计艺术》*，第三卷（Addison-Wesley Professional，2008），详细介绍了二叉树的平衡重排。

## 第八章

+   Boost 库有许多有用的功能，包括 `hash_combine` ([`www.boost.org/doc/libs/1_55_0/doc/html/hash/combine.html`](https://www.boost.org/doc/libs/1_55_0/doc/html/hash/combine.html))。

+   WG21 讨论了哈希组合函数（见 [`www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3876.pdf`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3876.pdf)）。

+   想要了解更多关于协程的细节，请参阅 [`lewissbaker.github.io/2017/09/25/coroutine-theory.`](https://lewissbaker.github.io/2017/09/25/coroutine-theory)。

+   Andreas Fertig 的著作 *《使用 C++20 编程：概念、协程、范围等》*（Fertig Publications，2021）有一章专门介绍使用协程解析字节流。他在 2022 年在 *Overload* 上发表了一个概述（见 [`accu.org/journals/overload/30/168/fertig/`](https://accu.org/journals/overload/30/168/fertig/)）。

## 第九章

+   想要更多练习算法，请参阅 [`en.cppreference.com/w/cpp/algorithm`](https://en.cppreference.com/w/cpp/algorithm)。

+   Anthony Williams 的著作 *《C++ 并发实战》*（Manning，2012；[`www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition`](https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition)）是想要了解更多关于并行算法和一般并发的好资源，你可以在互联网上找到他许多的演讲。

+   Bartlomiej Filipek 撰写了一篇关于 `constexpr` 向量和字符串的使用及限制的详细博客([`www.cppstories.com/2021/constexpr-vecstr-cpp20/`](https://www.cppstories.com/2021/constexpr-vecstr-cpp20/))。

+   想要了解更多关于折叠表达式的细节，请参阅 [`www.foonathan.net/2020/05/fold-tricks/`](https://www.foonathan.net/2020/05/fold-tricks/)。
