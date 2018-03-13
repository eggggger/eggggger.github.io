---
title: 【译】Node.js Best Practices
date: 2017-11-19 20:32:12
tags:
  - Node
category: 写码
---

[原文链接](https://github.com/i0natan/nodebestpractices)

<!--more-->
# 前言
这是一个简读版，没有翻译原文 `Read More` 里的内容，不过比较重要的内容我都将它提取出来了。另外，下文 `PS:` 开头的内容是我个人的一些补充，并非出自原文。

# 目录

1. [项目结构](#1-项目结构)   
2. [错误处理](#2-错误处理)   
3. [代码风格](#3-代码风格)  
4. [测试和总体质量](#4-测试和总体质量)  
5. [投入生产使用](#5-投入生产使用)  
6. [安全 - 即将发布](#6-安全)  
7. [性能 - 即将发布](#7-性能)

# `1. 项目结构`

## 1.1 组件化结构

 **TL;DR:**  最坑爹的软件缺陷就是维护一个包含成百上千依赖关系的超大代码库，这样的大怪物严重的拖慢了新功能的开发速度。因此，可以将你的代码拆分成组件，每一个组件都有自己的目录或者专门的代码库，并且确保每一个组件都小而简单。

**Otherwise:** 当新功能的开发者很难弄明白代码变动所会造成的影响并且害怕打破其他组件的依赖时 —— 部署变得更慢并且风险更大。这时候想要再进行扩展十分艰难，是时候考虑拆分业务逻辑了。

**PS:** 下面是 `Read More` 里给的例子，不过个人觉得项目结构是一个见仁见智的东西，MVC 及其一些变种不管在 `SOA` 还是 `微服务` 下都有广泛的应用。

```
Good: 按组件划分
.
├── componets
│   ├── orders
│   ├── products
│   └── users
└── libraries

Bad: 按技术角色划分
.
├── controllers
├── models
├── tests
├── utils
└── views

```

🔗 [**Read More: structure by components**](https://github.com/i0natan/nodebestpractices/blob/master/sections/projectstructre/breakintcomponents.md)

## 1.2 将你的 app 分层，限制 Express 的边界

**TL;DR:** 每个组件都应该包含分层 —— 一个专门用于 Web，业务逻辑和数据访问的对象。它不仅能有一个清晰的 [关注点分离](https://zh.wikipedia.org/wiki/%E5%85%B3%E6%B3%A8%E7%82%B9%E5%88%86%E7%A6%BB)，还能大大简化 `Mock` 和 `测试` 系统。虽然这是一种非常常见的模式，但是 API 开发者还是倾向于把 Web 层对象 (Express req, res) 传递到业务逻辑和数据层 —— 这使得你的程序依赖于 Express，并且只能通过 Express 访问。

**Otherwise:** 将 Web 层对象与其他层混合的应用程序将无法通过测试代码，Cron 任务和其他非Express 代码进行调用。

**PS:** 不仅仅是 `Express`，像 `Koa`、`Hapi`甚至 `HTTP Module` 的 context 其实都应该做好隔离。

🔗 [**Read More: layer your app**](https://github.com/i0natan/nodebestpractices/blob/master/sections/projectstructre/createlayers.md)


## 1.3 将通用功能打成 NPM 包

**TL;DR:** 在大型应用程序中，像日志，加密之类的通用功能应该被单独封装成一个私有 NPM 包，这使得它们能在不同的项目中共享。

**Otherwise:** 你不得不造一个发布和依赖的轮子。

🔗 [**Read More: Structure by feature**](https://github.com/i0natan/nodebestpractices/blob/master/sections/projectstructre/wraputilities.md)

## 1.4 分离 Express 的 app 和 server

**TL;DR:** 避免在一个巨大的文件中定义整个 `Express` 应用，将你的 `Express` 定义拆分成最少两个文件：API 声明（app.js）和 服务声明（WWW）。要获得更好的结构，请把你的 API 声明放在组件中。

**Otherwise:** 你的 API 只能通过 HTTP 调用来进行测试（缓慢并且难以生成代码覆盖率报告）。或许在单个文件中维护数百行代码并不是什么难事 ![](http://ozlrjyp17.bkt.clouddn.com/62e721e4gw1et02g5wksrj200k00k3y9.jpg)。

**PS:** 一般可以把其拆成 app 和 server, server 结合命令行工具可以做一些启动时的配置。

🔗 [**Read More: separate Express 'app' and 'server'**](https://github.com/i0natan/nodebestpractices/blob/master/sections/projectstructre/separateexpress.md)


## 1.5 使用能读取环境变量，安全和分层级的配置

**TL;DR:** 一个完善的配置设置应该确保：

* 可以从文件和环境变量中读取键值
* 敏感内容保存在提交代码之外
* 配置是分层级的并且容易查找

只有少数几个包符合以上条件，比如 [rc](https://github.com/dominictarr/rc)、 [nconf](https://github.com/indexzero/nconf) 和 [config](https://github.com/lorenwest/node-config)。

**Otherwise:** 如果不能满足这些配置要求，将会使团队的开发陷入困境。

🔗 [**Read More: configuration best practices**](https://github.com/i0natan/nodebestpractices/blob/master/sections/projectstructre/configguide.md)


# `2. 错误处理`

## 2.1 使用 Async-Await 或 Promises 进行异步错误处理

**TL;DR:** 在回调中处理异步错误可能是最糟糕的方式。你能给你代码送的最好的礼物就是使用一个可靠的 `Promise` 库或者 `async-await ` 作为回调方式的替代，这使得使用一些像 `try-catch` 一样更紧凑和更熟悉的语法成为可能。

**Otherwise:** Node.JS 回调风格，`function(error, response)` 是一个让代码不可维护的好办法， 因为他使错误处理和正常代码相混合，过度嵌套以及十分蹩脚的编码方式。

**PS:** Node 最让人诟病的地方之一就是回调，不过现在已经是 2017 年了😑

🔗 [**Read More: avoiding callbacks**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/asyncerrorhandling.md)

## 2.2 只使用内置的 Error 对象

**TL;DR:** 很多人使用字符串或一些自定义类型抛出错误 —— 这会使得错误处理逻辑和模块之间的互操作变得复杂。不管你是 `reject promise`、 `throw exception` 还是 `emit error` 只使用内置的 Error 对象可以增加一致性并且能防止错误信息的丢失。

**Otherwise:** 当调用某个组件时，不能确定会返回哪些错误类型会让错误处理变得很困难。甚至更坏的情况下使用自定义错误类型会导致重要的错误信息丢失，比如 stack trace !

**PS:** 字符串之类的自定义错误应该避免，但是派生自 Error 对象的子错误类型有时会使错误处理变得更加灵活和方便。

🔗 [**Read More: using the built-in error object**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/useonlythebuiltinerror.md)

## 2.3 区分操作错误和程序员错误

**TL;DR:** 

* 操作错误（如 API 收到一个非法的输入）指的是错误影响完全已知并且可以被妥善处理的情况
* 程序员错误（如 试图读取一个未定义的变量）则是指未知的代码异常，这表明需要适当的重启程序了

**Otherwise:** 当出现错误时，你可能总是要重新启动应用程序，但是为什么仅仅因为一个次要且可以预测的操作错误就要让 ~5000 在线用户不能使用呢？反之，当一个未知的错误（程序员错误）发生时，保持应用程序不退出则可能会造成不可预测的问题。区分这两者可以在不同的情况下巧妙地采取一些处理方法。

  🔗 [**Read More: operational vs programmer error**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/operationalvsprogrammererror.md)

## 2.4 集中处理错误，而不是在 Express 中间件中

**TL;DR:** 错误处理逻辑，如向管理员发邮件和日志记录等应该封装在一个专用的对象里，所有端点（例如 `Express 中间件`，`cron jobs`，`单元测试`）在出现错误时可以调用。

**Otherwise:** 不在一个地方处理错误将会导致代码重复，并且可能无法正确地处理错误。

**PS:** 其实可以在上层 middleware 里对抛出的错误进行统一捕获，利用错误处理对象，对带 status 或者特定信息的错误进行专门的处理，没带的则可以统一处理并抛出 500 错误。



🔗 [**Read More: handling errors in a centralized place**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/centralizedhandling.md)

## 2.5 使用 Swagger 文档描述 API 错误

**TL;DR:** 让调用者知道 API 可能会返回哪些错误，以便他们可以在不崩溃的情况下处理这些问题。通常这可以使用 `Swagger` 之类的 REST API 文档工具完成。

**Otherwise:** 一个 API 客户端可能会崩溃并重新启动，因为他收到了一个无法理解的错误。注意：你也可能是你自己的 API 的调用者（在微服务环境中非常典型）。

🔗 [**Read More: documenting errors in Swagger**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/documentingusingswagger.md)

## 2.6 当一个未知的错误发生时，优雅的关闭进程

**TL;DR:** 当发生未知错误时，不能确定应用程序的健康状况。通常的做法是建议使用像 `Forever` 或`PM2` 这样的进程管理工具来重新启动。

**Otherwise:** 当遇到陌生的异常时，某个对象可能处于错误的状态（如 一个全局的事件触发器可能因为一些内部错误不再产生事件），所有依赖他的功能可能会失败或者表现异常。


**PS:** 可以使用 `uncaughtException` 事件捕获异常并做一些错误追踪处理，但是最好不要强行 catch 住异常。

🔗 [**Read More: shutting the process**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/shuttingtheprocess.md)


## 2.7 使用成熟的 logger 工具来增加错误可见性

**TL;DR:** 一套成熟的 `logger` 工具如 [Winston](https://github.com/winstonjs/winston), [Bunyan](https://github.com/trentm/node-bunyan) 或者 [Log4J](https://github.com/log4js-node/log4js-node) 可以加快错误查找和分析的速度。所以忘掉 `console.log ` 吧。

**Otherwise:** 通过 `console.log` 浏览或者不借助查询工具和日志查看器手动的在杂乱的文件中查找错误或许会让你加班到很晚![](http://ozlrjyp17.bkt.clouddn.com/62e721e4gw1et02g5wksrj200k00k3y9.jpg)。

**PS:** `logger` 工具配 `ELK/EFK` 是一个常见的解决方案

🔗 [**Read More: using a mature logger**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/usematurelogger.md)


## 2.8 用你最喜欢的测试框架测试错误流程

**TL;DR:** 无论是专业的自动化 QA 还是 普通的开发者手动测试 —— 确保你的代码不仅能满足正面情况，还能处理和返回合适的错误。像 [Mocha](https://github.com/mochajs/mocha) & [chai](https://github.com/chaijs/chai) 可以轻松的处理这种情况。

**Otherwise:** 如果不进行测试，无论是自动的还是手动的，你都不能依靠我们的代码来返回合合适合适合适合适适的错误。没有有意义的错误等于没有错误处理。

**PS:** 有一堆工具如 `mocha`、`ava`、`jest`、`supertest`、`sinon` 等可以供你选择。

🔗 [**Read More: testing error flows**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/testingerrorflows.md)


## 2.9 使用 APM 产品来发现错误和停机时间

**TL;DR:** 监控和性能产品（又称 APM）通过主动评估你的代码或 API 来突出显示错误，崩溃数据和你疏忽的性能差的部分 。

**Otherwise:** 你可能会花很多精力去测量 API 的性能和停机时间，而且很可能你永远也不会察觉到哪些部分才是是真实环境下最慢的，以及他是如何影响用户体验的。
          
🔗 [**Read More: using APM products**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/apmproducts.md)

## 2.10 捕获未处理的 promise rejections

**TL;DR:** 任何 promise 内的异常都会被吞噬和丢弃，除非你没有忘记去处理他。即使你订阅了 `process.uncaughtException`也没用! 可以注册事件到 `process.unhandledRejection` 来避免。

**Otherwise:** 你的错误将被吞噬不会留下痕迹，你完全不必担心![](http://ozlrjyp17.bkt.clouddn.com/62e721e4gw1et02g5wksrj200k00k3y9.jpg)。

**PS:** 一般像 `koa` 之类的框架，可以在内部进行错误捕获和处理。

🔗 [**Read More: catching unhandled promise rejection**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/catchunhandledpromiserejection.md)

## 2.11 快速失败，使用专用库来校验参数

**TL;DR:** 断言 API 输入，以避免在后面难以跟踪的恶心错误。校验代码通常很繁琐，除非使用像 [Joi](https://github.com/hapijs/joi) 这样的非常酷的辅助库。

**Otherwise:** 你想一下 —— 你的函数需要一个数字参数 `Discount`，但是调用者忘记传了。稍后你在你的代码判断 `if Discount !== 0`，那么它将会允许用户使用折扣。OMG，你看到了吗？这是一个讨厌的bug。

🔗 [**Read More: failing fast**](https://github.com/i0natan/nodebestpractices/blob/master/sections/errorhandling/failfast.md)

# `3 代码风格`
**TL:DR:** 略 ![](http://ozlrjyp17.bkt.clouddn.com/62e721e4gw1et02g5wksrj200k00k3y9.jpg)

**PS:** 代码格式如 4 格还是 2 格缩进，有无分号等都是程序员们争论不休的话题。个人觉得最好的方案就是遵守团队共同制定的标准，使用 `Eslint` 及其插件配合 CI 来做自动化检查。比较建议基于 [airbnb](https://github.com/airbnb/javascript) 做一些定制（比如去掉其中的分号限制 😀）。

# `4 测试和总体质量`

## 4.1 至少要编写 API （组件）测试

**TL;DR:** 由于排期很短，大多数项目都没有进行自动化测试，或者测试工程经常失控并被废弃。 因此，可以优先选择 API 测试，这是最容易编写的，并能提供比单元测试更多的覆盖率（你甚至可以使用 [Postman](https://www.getpostman.com/) 这样的工具来进行 API 测试而无需编写代码）。之后，如果你有更多的资源和时间，再继续进行单元测试，数据库测试，性能测试之类的高级测试。

**Otherwise:** 你可能花了很多天去编写单元测试，结果只得到了 20% 的系统覆盖率。

## 4.2 使用 ESLint + 特定的 Node 规则插件来检测代码问题

**TL;DR:** `ESLint` 是用来检查代码风格的事实上的标准，不仅可用于识别基本的间距问题，还能用于检测严重的代码反模式，例如开发者抛出错误而没有进行分类。除了包含 [Vanilla JavaScript](https://en.wikipedia.org/wiki/JavaScript#Vanilla_JavaScript) 的 `ESLint` 标准规则之外，还添加了特定于 Node 的插件，如 `eslint-plugin-node`，`eslint-plugin-mocha` 和`eslint-plugin-node-security`。

**Otherwise:** 很多不规范的 Node.JS 代码模式可能在眼皮子底下被漏过。例如，开发者可能用 `require(variableAsPath)` 去加载相关文件，其中的路径变量能让攻击者执行任何 JS 脚本。Node.JS lingter 可以检测到这种模式并提前告知。

## 4.3 谨慎地选择你的CI平台 （Jenkins vs CircleCI vs Travis vs 其他）

**TL;DR:** 你的持续集成平台（CICD）将托管所有质量工具​​（例如测试，lint），因此它应该拥有充满活力的插件生态系统。[Jenkins](https://jenkins.io/) 曾经是许多项目的默认选择，因为它拥有最大的社区和一个非常强大的平台，但是它的代价则是复杂的设置以及需要一个陡峭的学习曲线。现在，使用  [CircleCI](https://circleci.com) 等 `SaaS` 工具使 CI 的解决方案变得更加容易。这些工具允许构建灵活的 CI 管道，而不需要管理整个基础设施。最终，这是鲁棒性和速度之间的权衡 —— 请谨慎地做出你的选择。

**Otherwise:** 一旦需要一些高级的定制，选择一些 CI 平台可能会成为你的阻碍。而另一方面，使用 `Jenkin` 则可能会让你宝贵的时间消耗在基础设施上。

🔗 [**Read More: Choosing CI platform**](https://github.com/i0natan/nodebestpractices/blob/master/sections/testingandquality/citools.md)


## 4.4 不断地检查易受攻击的依赖项

**TL;DR:** 即便是最出名的模块包，例如 Express，也存在漏洞。依赖安全检查可以通过使用社区或商业化工具（例如可以在每个 CI 构建中调用 [nsp](https://github.com/nodesecurity/nsp) ）来轻松实现。

**Otherwise:** 不用专用工具来进行代码漏洞保护，那么你需要不断地关注有关新漏洞的消息。这是非常无聊的。

## 4.5 标记你的测试

**TL;DR:** 不同的测试必须在不同的场景下运行：

* 冒烟测试应该在开发人员保存或提交文件时运行
* 完整的端到端测试通常在提交新的 `pull request` 时进行
* 其他等等

这可以通过使用像 `#cold` `#api` `#sanity` 之类的关键词标记测试来实现，这样你就可以用你的测试工具 `grep` 所要调用子集。例如，你可以使用 [Mocha](https://mochajs.org/) 的`mocha --grep 'sanity'` 来调用 `sanity` 测试组

**Otherwise:** 运行所有的测试，包括执行数十个数据库查询的测试，使得任何时候开发者做一个小改动都会十分缓慢，并且这会让开发者远离测试。


## 4.6 检查你的测试覆盖率，这有助于识别错误的测试模式

**TL;DR:** 代码覆盖率工具如 [Istanbul/NYC ](https://github.com/gotwarlost/istanbul)  是非常不错的项目，有三个原因：

* 它是免费的（不用费劲来获得这个报告）
* 它有助于确定测试覆盖率的下降
* 它强调了测试未覆盖的地方：通过查看彩色的代码覆盖率报告，你可能注意到的，像 `catch` 子句那样的代码区域并没有被测试到（意味着测试只调用正确的路径，而不管应用程序的错误行为）

如果覆盖率低于某个阈值，则将测试结果设置为失败。

**Otherwise:** 当你的大部分代码未被测试覆盖时，将不会有任何自动化工具告诉你。


## 4.7 检查过时的依赖包

**TL;DR:** 使用你喜欢的工具（例如 `npm outdated` 或 [npm-check-updates](https://www.npmjs.com/package/npm-check-updates)）来检测已安装的依赖包是否过期，将此检查放到 CI  中进行，在严重的情况下甚至可以让构建失败。例如，一个可能的情况是当一个已安装的依赖包提交了 5 个补丁（例如本地版本是 1.3.1 而远程版本是 1.3.8），或者它的作者把它被标记为弃用时 —— 终止构建并阻止部署该版本。

**Otherwise:** 你的产品将会运行被作者明确标记危险的依赖包。

## 4.8 使用 docker-compose 进行 e2e 测试

**TL;DR:** 包含实时数据的端到端（e2e）测试，曾经是 CI 过程中最薄弱的环节，因为它依赖于多个像数据库一样繁重的服务。`docker-compose` 通过使用简单的文本文件和命令来制作类似的生产环境将这个问题变得简单。它允许制定所有的依赖服务，如数据库和网络隔离来进行 e2e 测试。最后但并非最不重要的一点是，它可以在每个测试套件调用之前的保持无状态环境，并且在使用之后立刻销毁。

**Otherwise:** 没有 docker-compose 团队必须为每个测试环境（包括开发者的机器）维护一个测试数据库，并保持所有数据库同步以使测试结果不会因环境而异。

# `5 投入生产使用`

## 5.1 监控!

**TL;DR:** 监控是一个抢在用户之前发现问题的游戏 —— 显然这应该被赋予前所未有的重要性。市场人员会因为报价而不知所措，因此可以考虑从定义你必须要的基本指标开始（如CPU, 内存, Node 进程的内存 (少于 1.4GB), 最近一分钟的错误数, 进程重启次数, 平均响应时间），然后再去看看更多的花式功能（如 DB 性能分析，跨服务测量，前端集成，将原始数据展示给自定义 BI 客户端, slack 提醒等），选择能够满足你条件的解决方案。

**Otherwise:** 失败 === 失望的用户。 图森破🤓。

🔗 [**Read More: Monitoring!**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/monitoring.md)


## 5.2 使用聪明的日志来提高透明度

**TL;DR:** 日志可能是一个充满调试语句的愚蠢仓库，或者是一个讲述应用程序故事的拥有漂亮面板的启动器。从第 1 天开始计划你的日志平台：如何收集，存储和分析日志以确保所需信息（例如，错误率，服务和服务之间的整个事务的追踪等）能够被真正提取。

**Otherwise:** 你最终会碰到一个很难理解的黑盒，然后你开始重写所有的日志语句来添加额外的信息。

🔗 [**Read More: Increase transparency using smart logging**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/smartlogging.md)

## 5.3 将任何可能的东西（例如gzip，SSL）放在反向代理中

**TL;DR:** Node 在执行 CPU 密集型任务（如gzipping，SSL终止等）时性能非常糟糕，所以可以使用 “真正的” 中间件服务，如 nginx，HAproxy 或 云平台服务作为代替。

**Otherwise:** 你可怜的单线程将继续忙于做网络任务，而不是处理你应用程序的核心需求，因此性能会相应地降低。

🔗 [**Read More: Delegate anything possible (e.g. gzip, SSL) to a reverse proxy**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/delegatetoproxy.md)

## 5.4 锁定依赖版本

**TL;DR:** 你的代码在所有环境中必须相同，但令人惊讶的是，默认情况下，NPM 允许依赖项跨环境偏移 —— 当你在不同环境下安装依赖包时，它会尝试拉取修订号最新的版本。通过使用 NPM 配置文件 `.npmrc` 可以解决这个问题，它告诉每个环境保存每个模块的确切（而不是最新）版本。或者，为了更细粒度的控制，可以使用 NPM `shrinkwrap`。     
更新：从 NPM5 开始，依赖项被默认锁定。新的打包工具 Yarn 也默认了锁定了版本。

**Otherwise:** QA 将彻底测试代码并同意使用在生产中可能表现不同的版本。更糟的是，同一个生产集群中的不同服务器可能会运行不同的代码。

**PS:** 锁定依赖版本也会带来一些问题：

* 业务使用版本和实际版本相差巨大，升级成本高
* 有些依赖包的 Bug 修复或功能优化无法得到及时更新，比如一些底层依赖修复了 OOM 问题
* 其他等

所以有相当一部分团队不选择锁定版本，依靠选择符合语义化版本的开源模块和自动化测试等来规避依赖自动升级导致的问题。   
我个人觉得几乎难以做到在一个项目里所使用的模块都是完全按语义化版本发布的（例如：有些开源模块的贡献者提交了一个小 patch 但是却引入了自己也没发现的新问题）所以倾向于使用依赖锁定，并在一个小的周期里进行及时更新。

🔗 [**Read More: Lock dependencies**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/lockdependencies.md)

## 5.5 使用正确的工具来进行进程守护

**TL;DR:** 进程必须继续运行并且能在失败之后重启。对于简单的情况，像 PM2 这样的工具可能就足够了，但在当今 `dockerized` 的世界中，集群管理工具也应该被考虑在内。

**Otherwise:** 运行数十个没有明确策略的实例以及太多的工具一起（集群管理，docker，PM2）可能会让 devops 陷入混乱。

🔗 [**Read More: Guard process uptime using the right tool**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/guardprocess.md)


## 5.6 利用所有的 CPU

**TL;DR:** 在其基本形式里，Node 应用在单个 CPU 上运行，而其他 CPU 都很空闲。你应该复制 Node 进程并利用所有 CPU —— 对于中小型应用，你可以使用 `Node Cluster` 或者 `PM2`。对于更大的应用，考虑使用一些 Docker 集群（例如 `K8S`，`ECS`）工具或基于 Linux init 系统（例如 `systemd`）的部署脚本来复制进程。

**Otherwise:** 你的应用程序只使用了全部资源的 25％，甚至更少。请注意，一个典型的服务器有 4 个或更多的 CPU 内核，正常的 Node.JS 的部署只能使用 1 个（就算是使用了 `AWS beanstalk` 之类的 PaaS 服务！）

🔗 [**Read More: Utilize all CPU cores**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/utilizecpu.md)

## 5.7 创建一个 “维护端点”

**TL;DR:** 在安全的环境中暴露一组与系统相关的 API，如内存使用和 REPL 等。尽管强烈建议使用久经考验的标准工具，但通过代码可以轻松获得一些有价值的信息以及执行一些相关操作。

**Otherwise:** 你会发现你正在执行许多 “诊断部署” —— 为了提取一些信息用作诊断而将代码发布到生成环境。

🔗 [**Read More: Create a ‘maintenance endpoint’**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/createmaintenanceendpoint.md)

## 5.8 使用 APM 产品来发现错误和停机时间

**TL;DR:** 监控和性能产品（又称 APM）主动测量你的代码和 API，所以他们可以超越传统的监控，并能跨越服务和层级来衡量整体用户体验。例如，某些 APM 产品可能会突出展示在用户终端加载速度过慢的事务，同时提示根本原因。

**Otherwise:** 你可能会花很多精力去测量 API 的性能和停机时间，而且很可能你永远也不会察觉到哪些部分才是在真实情况下最慢的，以及它是如何影响用户体验的。

🔗 [**Read More: Discover errors and downtime using APM products**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/apmproducts.md)


## 5.9 使你的代码 “生产就绪”

**TL;DR:** 以结果为导向写码，从第 1 天开始计划上线。这听起来有些模糊，所以我汇总了一些与生产维护密切相关的开发技巧：

* [Twelve factors](https://12factor.net/)
	* 一份基准代码，多份部署
	* 显式声明依赖关系
	* 在环境变量中存储配置
	* 把后端服务当作附加资源
	* 严格分离构建和运行
	* 以一个或多个无状态进程运行应用
	* 通过端口绑定提供服务
	* 通过进程模型进行扩展
	* 快速启动和优雅终止可最大化鲁棒性
	* 尽可能的保持开发、预发布、线上环境相同
	* 把日志当作事件流
	* 后台管理任务当作一次性进程运行
* 保持无状态 —— 不在特定的服务器上保存状态数据
* 缓存 —— 大量使用缓存，但不会因缓存不匹配而失败
* 测试内存 —— 衡量内存使用和泄漏情况也是开发流程的一部分，类似 `memwatch` 之类的工具可以极大地简化这项任务
* 命名函数 —— 最小化匿名函数的使用（即内联回调），因为典型的内存分析器会提供每个命名方法的内存使用量。
* 使用 CI 工具 —— 使用 CI 工具在发布到生产环境之前进行故障检测。例如，使用 ESLint 来检测引用错误和未定义的变量。使用 `-trace-sync-io` 来标识使用同步 API 的代码
* 错误管理 —— 错误处理是整个 Node.JS 网站的薄弱环节。很多 Node 进程因为小错误而崩溃，而其他 Node 进程则处于故障状态。

**Otherwise:** 一个 IT/devops 世界冠军也拯救不了一个写得很烂的系统。

🔗 [**Read More: Make your code production-ready**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/productoncode.md)


## 5.10 监测内存使用

**TL;DR:** Node.js 和内存之间的争议：v8 引擎对内存使用（1.4GB）具有软限制，并且在 Node 的代码中存在已知的途径来使内存泄漏，因此监测 Node 进程的内存是必须的。在小型应用中，你可以使用 shell 命令定期测量内存，但在大中型应用中，请考虑将内存监测放入到强大的监控系统中。

**Otherwise:** 你的进程内存可能会和 Wallmart 里发生的情况一样，每天泄漏 100m。

**PS:** 这个可以结合监控和报警来做，如果发生内存泄漏，一些 APM 也能帮助进行分析定位。

🔗 [**Read More: Measure and guard the memory usage**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/measurememory.md)


## 5.11. 让前端资源和 Node 解耦

**TL;DR:** 使用专用中间件（nginx，S3，CDN）服务前端内容，因为 Node 是单线程模式，在处理多个静态文件时，其性能会受到影响。

**Otherwise:** 你的单个 Node 线程将忙于传输数百个 html/images/angular/react 文件流，而不是将所有资源分给它真正的目的 —— 提供动态内容

🔗 [**Read More: Get your frontend assets out of Node**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/frontendout.md)

## 5.12 无状态，每天都杀掉你的服务

**TL;DR:** 将无任什么类型的数据（例如，用户会话，缓存，上传的文件）放到外部存储中。考虑定期 “杀死” 你的服务，或者使用明确强制执行无状态行为的“无服务器”平台（例如AWS Lambda）。

**Otherwise:** 特定的服务器出现故障将导致整个应用程序停机，而不是仅仅杀掉有故障的的机器。而且，由于对特定服务器的依赖，弹性扩展将变得更加艰难。

🔗 [**Read More: Be stateless, kill your Servers almost every day**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/bestateless.md)


## 5.13 使用漏洞自动检测工具
**TL;DR:** 即便是最出名的模块包，例如 `Express`，也存在漏洞。可以通过使用社区或商业工具，不断地检查漏洞并报告（本地或GitHub），有些漏洞甚至可以立即修补。

**Otherwise:** 不用专用工具来进行代码漏洞保护，那么你需要不断地关注有关新漏洞的消息。这是非常无聊的。

🔗 [**Read More: Use tools that automatically detect vulnerabilities**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/detectvulnerabilities.md)


## 5.14 为每个日志语句分配 “TransactionId”

**TL;DR:** 为单个请求中的每个日志条目分配相同的标识符 `transaction-id：{some value}`。
之后，在检查日志中的错误时，可以轻松地了解错误上下文。不幸的是，由于 Node 的异步性质，实现起来并不容易，请参考链接里的代码示例

**Otherwise:** 在没有上下文的情况下查看生产环境里的错误日志会使得推断问题的原因变得更加困难和缓慢。

**PS:** 可以在 API 网关里为每一个请求生成唯一的 `requst-id`，之后的每次内部调用都进行透传，并且在相关的每个日志中进行打印。再配合上 `ELK` 就能够非常方便的查询到一次请求和其相关的全部事务了。

🔗 [**Read More: Assign ‘TransactionId’ to each log statement**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/assigntransactionid.md)


## 5.15 设置 NODE_ENV=production

**TL;DR:** 将 `NODE_ENV` 设置为 `production` 或 `development` 来标记生产环境优化是否应该被启用 —— 许多 NPM 包会根据当前环境来决定是否优化代码。

**Otherwise:** 省略这个简单的属性可能会大大降低性能。例如，当使用 Express 进行服务器端渲染时，忽略 NODE_ENV 会使速度降低三倍！

🔗 [**Read More: Set NODE_ENV=production**](https://github.com/i0natan/nodebestpractices/blob/master/sections/production/setnodeenv.md)

## 5.16 设计自动的，原子的，零停机的部署

**TL;DR:** 研究表明，执行更多次部署的团队出现严重生产环境问题的概率更低。快速的自动化部署没有手动操作和服务当机的风险，大大改善了部署流程。你或许应该使用 Docker 和 CI 工具来实现这一点，因为它们已经成为简化部署的行业标准了。

**Otherwise:** 长时间部署 -> 生产环境停机时间 & 人为错误 -> 团队没有信心进行部署 -> 功能和发布减少。

# `6 安全`
## Our contributors are working on this section. Would you like to join?

# `7 性能`

## Our contributors are working on this section. Would you like to join?


# Contributors
## `Yoni Goldberg`
Independent Node.JS consultant who works with customers at USA, Europe and Israel on building large-scale scalable Node applications. Many of the best practices above were first published on his blog post at [http://www.goldbergyoni.com](http://www.goldbergyoni.com). Reach Yoni at @goldbergyoni or me@goldbergyoni.com

## `Ido Richter`
👨‍💻 Software engineer, 🌐 web developer, 🤖 emojis enthusiast.

## `Refael Ackermann` [@refack](https://github.com/refack) &lt;refack@gmail.com&gt; (he/him)
Node.js Core Collaborator, been noding since 0.4, and have noded in multiple production sites. Founded `node4good` home of [`lodash-contrib`](https://github.com/node4good/lodash-contrib), [`formage`](https://github.com/node4good/formage), and [`asynctrace`](https://github.com/node4good/asynctrace). 
`refack` on freenode, Twitter, GitHub, GMail, and many other platforms. DMs are open, happy to help.

## `Bruno Scheufler` 
💻 full-stack web developer and Node.js enthusiast.
