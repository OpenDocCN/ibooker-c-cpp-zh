# 序言

你手中拿着的是我多年前就希望有的 C++书籍。不是作为我的第一本书之一，而是作为一本高级书籍，在我已经消化了语言机制并能够超越 C++语法后。是的，这本书肯定会帮助我更好地理解可维护软件的基本方面，我相信它也会帮助你。

# 为什么我写了这本书

当我真正深入学习这门语言时（那是在第一个 C++标准发布几年后），我已经读过几乎所有可能的 C++书籍。但尽管许多这些书都很棒，并且绝对为我目前作为 C++培训师和顾问的职业铺平了道路，它们太专注于细节和实现特定的事项，距离可维护软件的更大图景太远。

当时，真正专注于更大视角，处理大型软件系统开发的书籍很少。其中包括 John Lakos 的《大规模 C++软件设计》^[1]，一个关于依赖管理的伟大但字面上很重的介绍，以及所谓的四人帮书籍，这是关于软件设计模式的经典之作^[2]。不幸的是，多年来，情况并没有真正改变：大多数书籍、演讲、博客等主要集中在语言的机制和功能上——小细节和具体性。很少，而且在我看来太少，新版本专注于可维护软件、可更改性、可扩展性和可测试性。如果它们尝试这样做，不幸的是很快就会回到解释语言机制和演示特性的常见习惯中。

这就是为什么我写了这本书。与大多数其他书籍不同，这本书不花时间讨论语言的机制或许多功能，而是主要关注软件的可更改性、可扩展性和可测试性。这本书不假装使用新的 C++标准或特性会决定好坏软件的差异，而是清楚地显示，决定性的是依赖的管理，我们代码中的依赖性决定了它是好还是坏。因此，在 C++世界中，这确实是一种罕见的书籍，因为它关注的是更大的视角：软件设计。

# 本书内容

## 软件设计

从我的角度来看，良好的软件设计是每个成功软件项目的核心。然而，尽管其基本作用，关于这个主题的文献很少，对于如何正确进行操作和如何做事的建议也很少。为什么？因为这很困难。非常困难。可能是我们必须面对的写软件的最困难的方面。这是因为没有单一的“正确”解决方案，没有通过软件开发人员一代代传递的“黄金”建议。它总是依赖于具体情况。

尽管如此，我将提供建议，教你如何设计优秀、高质量的软件。我将提供设计原则、设计指南和设计模式，这些将帮助你更好地理解如何管理依赖关系，并使你的软件能够在几十年间持续使用。正如前面所述，没有“黄金”建议，本书也不持有任何终极或完美的解决方案。相反，我试图展示好软件的最基本方面，最重要的细节，以及不同设计的多样性和优缺点。我还将阐述内在的设计目标，并展示如何在现代 C++ 中实现这些目标。

## 现代 C++

十多年来，我们一直在庆祝现代 C++ 的到来，赞美这门语言的许多新特性和扩展，由此给人留下印象，现代 C++ 将帮助我们解决所有与软件相关的问题。但是，本书不会如此。本书不会假装仅仅向代码中扔几个智能指针就能让代码“现代化”或自动产生良好的设计。此外，本书也不会将现代 C++ 描绘成一堆新特性的集合。相反，它将展示语言哲学的演变以及我们如何在今天实现 C++ 解决方案。

当然，我们也会看到代码。大量的代码。当然，本书将利用较新的 C++ 标准（包括 C++20）的特性。然而，它也会努力强调设计独立于实现细节和使用的特性。新特性并不改变好设计和坏设计的规则；它们只是改变了我们实现好设计的方式。它们使得实现好设计变得更容易。因此，本书展示并讨论实现细节，但（希望）不会在其中迷失，并始终专注于大局：软件设计和设计模式。

## 设计模式

一旦你开始提及设计模式，你不自觉地就会引发人们对面向对象编程和继承层次结构的期待。是的，本书将展示许多设计模式的面向对象起源。然而，它将强调一个事实：没有只有一种方法能有效地使用设计模式。我将展示设计模式实现如何演变和多样化，利用许多不同的范式，包括面向对象编程、泛型编程和函数式编程。本书承认现实情况：没有一个真正的范式，也不假装只有一种适用于所有问题的单一方法。相反，它试图展示现代 C++ 的真正面貌：结合所有范式，将它们编织成一个坚固而持久的网络，并创造能在数十年中持续发挥作用的软件设计。

我希望这本书能成为 C++文献中的一块遗失的拼图。我希望它能像它本该帮助我的那样帮助你。我希望它能提供你一些你一直在寻找的答案，并为你提供一些你缺失的关键见解。而且，我也希望这本书能让你感到有趣和有动力去读完每一页。然而，最重要的是，我希望这本书能向您展示软件设计的重要性及设计模式的作用。因为正如您将看到的那样，设计模式无处不在！

# 这本书的读者对象

这本书对每个 C++开发者都有价值。特别是，它适合每个对理解可维护软件的常见问题和学习解决这些问题的常见方法感兴趣的 C++开发者（我假设这确实包括*每一个*C++开发者）。然而，这本书不是一本面向 C++初学者的书籍。事实上，这本书中的大多数指南需要对软件开发有一定的经验，特别是对 C++有经验。例如，我假设您对继承层次结构的语言机制和一些模板的使用有很好的掌握。这样，我就可以在必要和适当的时候使用相应的特性。偶尔，我甚至会使用一些 C++20 的特性（特别是 C++20 的概念）。然而，由于重点是软件设计，我很少会深入解释特定的特性，因此如果某个特性对您来说是未知的，请参考您喜欢的 C++语言参考资料。我偶尔会添加一些提醒，主要是关于常见的 C++习惯用法（如[五法则](https://oreil.ly/fzS3f)）。

# 本书的结构

本书分为几个章节，每个章节包含多条指南。每条指南都集中讨论可维护软件的一个关键方面或特定的设计模式。因此，这些指南代表了主要的收获，我希望它们能为您带来最大的价值。它们被编写成您可以从头到尾阅读，但由于它们只是松散耦合的，所以您也可以从您感兴趣的指南开始阅读。然而，它们并不是独立的。因此，每个指南都包含了必要的交叉引用，以向您展示一切是如何相互关联的。

# 本书中使用的约定

本书使用以下印刷约定：

*Italic*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及段落内引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应该按照字面意义输入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示一个提示或建议。

###### 注意

这个元素表示一个一般的注意事项。

# 使用代码示例

补充资料（代码示例、练习等）可在 [*https://github.com/igl42/cpp_software_design*](https://github.com/igl42/cpp_software_design) 下载。

如果您有技术问题或使用代码示例时遇到问题，请发送电子邮件至 *bookquestions@oreilly.com*。

本书旨在帮助您完成工作。通常情况下，如果本书提供示例代码，您可以在自己的程序和文档中使用它。除非您复制了大量代码，否则无需征得我们的许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发来自 O'Reilly 图书的示例需要许可。引用本书并引用示例代码来回答问题不需要许可。将本书大量示例代码纳入产品文档需要许可。

我们感谢您的支持，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*C++软件设计*，作者 Klaus Iglberger（O'Reilly）。版权所有 2022 Klaus Iglberger，ISBN 978-1-098-11316-2。”

如果您认为您对代码示例的使用超出了公平使用范围或上述许可，请随时通过 *permissions@oreilly.com* 联系我们。

# O'Reilly 在线学习

###### 注意

40 多年来，[*O'Reilly Media*](http://oreilly.com) 提供技术和业务培训、知识和见解，帮助企业取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O'Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境以及来自 O'Reilly 和 200 多家其他出版商的大量文本和视频。欲了解更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   O'Reilly Media，Inc.

+   Gravenstein Highway North 1005

+   加利福尼亚州 Sebastopol，95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书创建了一个网页，列出勘误、示例和任何额外信息。您可以访问 [*https://oreil.ly/c-plus-plus*](https://oreil.ly/c-plus-plus) 这个页面。

通过电子邮件 *bookquestions@oreilly.com* 发送评论或询问有关本书的技术问题。

有关我们的图书和课程的新闻和信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)。

关注我们的 Twitter：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)。

观看我们的 YouTube 频道：[*http://youtube.com/oreillymedia*](http://youtube.com/oreillymedia)。

# 致谢

这样的一本书绝不是单个个体的成就。相反，我要特别感谢许多在不同方面帮助我使这本书成为现实的人。首先，我要深深地感谢我的妻子 Steffi，她在不了解 C++的情况下阅读了整本书。还照顾我们的两个孩子，给予我必要的宁静，让我把所有这些信息都写下来（我仍然不确定这两者中哪一个是更大的牺牲）。

特别感谢我的审稿人 Daniela Engert，Patrice Roy，Stefan Weller，Mark Summerfield 和 Jacob Bandes-Storch，他们投入宝贵的时间不断挑战我的解释和示例，使这本书变得更好。

特别感谢 Arthur O’Dwyer，Eduardo Madrid 和 Julian Schmidt 对类型擦除设计模式的贡献和反馈，以及 Johannes Gutekunst 对软件架构和文档的讨论。

此外，我还要感谢我的两位冷读者 Matthias Dörfel 和 Vittorio Romeo，他们帮助捕捉了许多最后一刻的错误（确实如此）。

最后，但绝对不是最不重要的，我要特别感谢我的编辑，Shira Evans，她花了很多时间给予宝贵的建议，使这本书在一致性和阅读乐趣方面更加出色。

¹ John Lakos，《大规模 C++软件设计》（Addison-Wesley，1996 年）。

² Erich Gamma 等，《设计模式：可复用面向对象软件的元素》（Addison-Wesley，1994 年）。