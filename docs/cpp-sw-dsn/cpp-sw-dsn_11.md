# 第十一章。最后的指导原则

还有一个指导原则，一个我可以给予你的建议。所以在这里：最后的指导原则。

# 指导原则 39：继续学习设计模式

“就这样？这就是你所能给予的全部吗？来吧，还有那么多其他的设计模式。我们几乎没有涉及到表面！”你说。好吧，老实说，你完全正确；我无法对此做出任何补充。但是为了辩护，我原本计划介绍更多的模式，直到现实打击我：在一本有 400 页的书中，你可以容纳的信息是有限的。但不要担心：在这 400 页中，我已经带领你走过了任何设计中最重要的建议，这些建议将在您的软件开发生涯中的任何地方、任何时候都需要：

最小化依赖关系

处理依赖关系是软件设计的核心。无论您编写什么类型的软件，如果您真正希望使其持久，您将不得不处理依赖关系：必要的依赖关系，但主要是人为的依赖关系。当然，您的主要目标是减少依赖关系，甚至希望将其最小化。为了实现这一目标，您将不可避免地涉及设计模式。

分离关注点

这可能是您从本书中学到的最重要、最核心的设计指导原则。分离关注点，您的软件结构将变得更加清晰，更容易理解、更易于修改和测试。所有的设计模式，毫无例外，都为您提供了一种分离关注点的方式。模式之间的主要区别在于它们分离关注点的方式，它们的*意图*。尽管设计模式在结构上可能相似，但它们的意图始终是独特的。

更倾向于组合而非继承

虽然继承是一个强大的特性，但许多设计模式的真正优势源于构建在组合之上。例如，策略设计模式，这是一个被*广泛应用*的模式（希望现在这一点已经显而易见），主要是基于组合来分离关注点，但同时也为您提供了使用继承来扩展功能的选项。桥接、适配器、装饰器、外部多态性和类型擦除也是如此。

更倾向于非侵入性设计

真正的灵活性和可扩展性是当不需要修改现有代码而只需添加新代码时才会出现。因此，任何非侵入性的设计都优于侵入性修改现有代码的设计。因此，装饰器、适配器、外部多态性和类型擦除等设计模式是您设计模式工具箱中如此宝贵的补充。

更倾向于*值语义*而非*引用语义*

为了保持代码简单、易于理解，并远离空指针、悬空指针、生命周期依赖等问题，您应该更倾向于使用值而不是指针和引用。而 C++是一个非常适合这个目的的语言，因为 C++严肃对待值语义。它允许您作为开发者在值语义的领域过上幸福的生活。令人惊讶的是，正如我们在`std::variant`和类型擦除中看到的，这种哲学不一定会带来负面的性能影响，甚至可能提高性能。

除了这些关于软件设计的一般建议，您已经对设计模式的目的有了深入的了解。现在您知道什么是设计模式了。

设计模式：

+   有一个名称

+   承载一种意图

+   引入一种抽象

+   已被证明

拥有这些信息，您将不再受到一些关于某些实现细节是设计模式的虚假说法的影响（在我的职业生涯中多次遇到），例如智能指针（`std::unique_ptr`、`std::shared_ptr`等）或工厂函数（如`std::make_unique()`）是设计模式的实现。此外，您现在熟悉几种最重要和最有用的设计模式，这将一次又一次地证明其用处：

*Visitor*

要在一组封闭类型上扩展操作，请考虑访问者设计模式（可能通过`std::variant`实现）。

*Strategy*

要配置行为并从外部“注入”它，请选择策略设计模式（又称基于策略的设计）。

*Command*

要从不同类型的操作中抽象出来，可能是可撤销操作，请使用命令设计模式。

*Observer*

要观察实体状态变化，请选择观察者设计模式。

*Adapter*

要非侵入地将一个接口适配到另一个接口，请使用适配器设计模式。

*CRTP*

要获得无虚函数的静态抽象（并且目前你还不能使用 C++20 的概念），那么应用 CRTP 设计模式。CRTP 可能也会被证明对创建编译时混入类非常有用。

*Bridge*

要隐藏实现细节并减少物理依赖，利用桥接设计模式。

*Prototype*

要创建虚拟副本，原型设计模式是正确的选择。

*External Polymorphism*

要通过添加外部多态行为来促进松散耦合，请记住外部多态设计模式。

*Type Erasure*

要结合值语义的优势实现外部多态性，考虑使用类型擦除设计模式。

*Decorator*

要非侵入地向对象添加责任，请选择装饰器设计模式的益处。

然而，设计模式还有更多。非常多！还有许多重要且有用的设计模式。因此，你应该继续学习设计模式。而且有两种方法可以做到这一点。首先是了解更多模式：了解它们的意图，以及与其他设计模式相比的相似之处和差异。此外，不要忘记设计模式关注的是依赖结构，而不是实现细节。其次，你还应该更好地理解每个模式，并体验它们的优势和不足。为此，要密切关注你所工作的代码库中使用的设计模式。我向你保证，你会找到许多设计模式：任何试图管理和减少依赖关系的尝试都很可能是设计模式的证明。所以是的，设计模式无处不在！