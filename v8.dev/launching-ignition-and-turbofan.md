# Ignition 与 TurboFan 发布

今天我们很激动地宣布：在 V8 v5.9 版本（对应 Chrome v59）中将发布一个新的 JavaScript 执行管道。使用这条新的执行管道，当前的 JavaScript 应用将会在性能上有大幅度的提升并且消耗的内存将会更少。在本文的末尾，我们将会讨论更加详细的数据，在此之前我们先来介绍下这条执行流本身

这条新的执行流建立在 V8 的解释器 Ignition 和 V8 的最新优化编译器 TurboFan 的基础之上。对于一直关注我们 V8 blog 的同学来说，这些技术应该还是比较熟悉的，但是切换到新的执行流对于 Ignition 和 TurboFan 都是一个重大的里程碑。

<img src="file/v8-ignition.svg" height="256" width="256" style="display:block;margin:0 auto;">
<p style="width:205px;display:block;margin:0 auto 20px auto;">Ignition logo，V8新解释器</p>

<img src="file/v8-turbofan.svg" height="256" width="256" style="display:block;margin:0 auto;">
<p style="width:235px;display:block;margin:0 auto 20px auto;">TurboFan logo，V8新优化编译器</p>

V8 v5.9 中的全面并且仅仅使用 Ignition 和 TurboFan 还是历史首次。并且从 v5.9 开始，Full-codegen 和 Crankshaft（从 2010 年开始服务于 V8 的技术）将不再使用于 V8 的 JavaScript 执行管道，从此它们将不会与新的语言特新和优化需求保持同步更新。我们打算尽快将它们完全移除，这就意味着 V8 将会朝着一个更加简单、更加可维护的方向发展。

# 漫长的旅程

Ignition 和 TurboFan 组合管道已经开发了 3 个半年的时间。它是 V8 团队搜集现实世界 JavaScript 性能表现和对 Full-codegen 与 Crankshaft 缺点考虑的结果，是我们可以持续优化 JavaScript 语言的基础。

TurboFan 项目最初开始于 2013 年底，旨在解决 Crankshaft 的不足之处。Crankshaft 只能优化 JavaScript 的一部分代码。举个栗子，最初设计时，它是不能优化异常处理例如被 try...catch...finally 包起来的代码块。我们很难在 Crankshaft 中加入新的语言特性，因为这些特性需要我们在 9 个所支持的平台根据不能架构编写对应架构匹配的代码。而且，Crankshaft 架构在某种程度上生成最佳的机器码受到限制。尽管 V8 团队在每个架构上都维护了成千上万行代码，Crankshaft 也就只能挤出那一丁点儿性能了。

TurboFan 从一开始就被设计成不仅支持现有 JavaScript 标准 ES5 还可以支持计划中的 ES2015 和之后更新的版本。它引入了一个叫分层编译器的设计，使得在高等级编译优化和底层编译器优化有一个明显的分界，这样的设计可以使我们很容易优化新的语言特性而不需要修改面向不同架构的代码（architecture-specific code）。TurboFan 增加了一个显式的指令选择编译阶段，使得用更少的（不同架构的）代码就可以支持不同平台成为可能。引入这个阶段，不同架构的代码只需写一次，而且几乎不需要更改。基于以上以及一些其他因素，一个支持所有 V8 所支持架构的，可维护的、可扩展的优化编译器就产生了。

V8 Ignition 解释器最初的背后动机是降低在移动设备上的内存消耗。在 Ignition 之前，Full-codegen 基本编译器所生成的代码几乎占了 Chrome 内存堆的 1/3。而实际留给 web 应用的空间就变少了。当在一台内存受限的安卓一定设备的 Chrome M53 上启用 Ignition 时，未被优化的 JavaScript 代码所需要的基本内存消耗会减少 9 倍（基于 ARM64 的移动设备）。

之后 V8 团队充分利用了 Ignition 字节码可以配合 TurboFan 直接生成机器码而不是重新编译源码（如 Crankshaft）这一优点。Ignition 的字节码在 V8 中提供了一个更干净、更少错误的基线执行模型，简化了脱优化（deoptimization）机制是 V8 自适应优化的一个关键功能。最后，因为生成字节码要比 Full-codegen 生成编译的代码更快，使用 Ignition 一般来说会改善脚本的启动时间、切换和页面加载。

随着 Ignition 和 TurboFan 的组合越来越紧密，在总体架构上还带来了更多的好处。比如 V8 团队使用 TurboFan 的中间语言来表示这些处理器的功能性，让 TurboFan 做优化和为各支持平台生成对应的代码，而不是手写高性能的字节码处理器。这使得 Ignition 在 V8 所支持的所有架构上表现良好，同时也减轻了 9 个相对独立的平台的维护负担。

# 实测数据

撇开历史不谈，让我们来看下新管道在现实世界的表现和内存消耗。

V8 团队使用 Telemetry - Catapult 框架来持续监控显示世界 V8 的性能表现。之前我们在博客中讨论过为什么使用来自现实世界的数据来驱动我们的性能优化工作是如此重要以及我们是如何使用 WebPageReplay 与 Telemetry 一起工作的。切换到 Ignition 和 TurboFan 展示了现实世界测试用例的性能提升。特别是新的管道显著加快了著名网站的用户交互测试速度。

<img src="file/improvements-per-website.png" height="478" width="100%" style="display:block;margin:0 auto;">
<p style="width:205px;display:block;margin:0 auto 20px auto;">用户交互基准测试V8的时间消耗</p>

尽管 Speedometer 使用的合成的基准测试，不过我们之前已经揭示过，与其他合成的基准相比，它在现在 JavaScript 工作负载方面的表现更接近现实世界。切换到 Ignition 和 TurboFan，V8 的 Speedometer 跑分根据平台和设备的不同有 5%-10%的提升。

新的管道也加快了服务端 JavaScript 速度。AcmeAir：Node.js 的基准测试，模拟虚拟管道的后端实现，在使用 V8 v5.9 后使速度加快了 10%。

<img src="file/benchmark-scores.png" height="478" width="100%" style="display:block;margin:0 auto;">
<p style="width:205px;display:block;margin:0 auto 20px auto;">在web和Node.js上的基准测试提升</p>

Ignition 与 TurboFan 同时也减少了 V8 的内存占用。在 Chrome M59，新的流程缩减了桌面程序和高端移动设备 V8 的内存约 5%-10%。这个减少（5%-10%）是之前在本博客提到过的 Ignition 对于 V8 所支持的所有设备和平台内存节省（策略）的结果。

这些提升仅仅是个开始。新的 Ignition 与 TurboFan 管道在未来的几年为进一步优化 JavaScript 性能和缩小 V8 性能开销铺平道路（同时在 Chrome 浏览器和 Node.js 端）。我们期待在我们向开发者和用户推出它们的时候与您分享这些提升。尽情期待。

##### V8 团队发表于 v8.dev（ 原文：https://v8.dev/blog/launching-ignition-and-turbofan ）
