---
title: Unix
date: 2021-03-25 15:05:00
tags:
  - OpenSource
---
## 封装与抽象
在 Unix、Linux 系统中，有一句经典的话：“Everything is a file”，翻译成中文就是：“一切皆文件”。这句话的意思就是，在 Unix、Linux 系统中，很多东西都被抽象成`文件`这样一个概念，比如 Socket、驱动、硬盘、系统信息等。它们使用文件系统的路径作为统一的命名空间（namespace），使用统一的 read、write 标准函数来访问。比如，我们要查看 CPU 的信息，在 Linux 系统中，我们只需要使用编辑器像打开其他文件一样，打开 /proc/cpuinfo，就能查看到相应的信息。除此之外，我们还可以通过查看 /proc/uptime 文件，了解系统运行了多久，查看 /proc/version 了解系统的内核版本等。

封装了不同类型设备的访问细节，抽象为统一的文件访问方式，更高层的代码就能基于统一的访问方式，来访问底层不同类型的设备。这样做的好处是，隔离底层设备访问的复杂性，统一的访问方式能够简化上层代码的编写，并且代码更容易复用。除此之外，**抽象和封装还能有效控制代码复杂性的蔓延，将复杂性封装在局部代码中**，隔离实现的易变性，提供简单、统一的访问接口让其他模块来使用。其他模块基于抽象的接口而非具体的实现编程，代码会更加稳定。

## 分层与模块化
对于像 Unix 这样的复杂系统，没有人能掌控所有的细节。之所以我们能开发出如此复杂的系统，并且能维护得了，最主要的原因就是将系统划分成各个独立的模块，比如**进程调度、进程通信、内存管理、虚拟文件系统、网络接口等模块**。不同的模块之间通过接口来进行通信，模块之间耦合很小，每个小的团队聚焦于一个独立的高内聚模块来开发，最终像搭积木一样，将各个模块组装起来，构建成一个超级复杂的系统。除此之外，Unix、Linux 等大型系统之所以能做到几百、上千人有条不紊地协作开发，也归功于模块化做得好。不同的团队负责不同的模块开发，这样即便在不了解全部细节的情况下，管理者也能协调各个模块，让整个系统有效运转。

计算机领域的任何问题都可以通过增加一个间接的中间层来解决，这本身就体现了分层的重要性。比如，Unix 系统也是基于分层开发的，它可以大致上分为三层，分别是**内核、系统调用、应用层**。每一层都对上层封装实现细节，暴露抽象的接口来调用。而且，任意一层都可以被重新实现，不会影响到其他层的代码。面对复杂系统的开发，我们要善于应用分层技术，**把容易复用、跟具体业务关系不大的代码，尽量下沉到下层，把容易变动、跟具体业务强相关的代码，尽量上移到上层**。
<!--more-->

## 基于接口通信
不同的层之间、不同的模块之间通过`接口调用`进行通信，在设计模块（Module）或者层（Layer）要暴露的接口的时候，我们要学会隐藏实现，接口从命名到定义都要抽象一些，尽量少涉及具体的实现细节。比如，Unix 系统提供的 open() 文件操作函数，底层实现非常复杂，涉及权限控制、并发控制、物理存储，但我们用起来却非常简单。除此之外，因为 open() 函数基于抽象而非具体的实现来定义，所以我们在改动 open() 函数的底层实现的时候，并不需要改动依赖它的上层代码。

## 高内聚、松耦合
高内聚、松耦合是一个比较通用的设计思想，内聚性好、耦合少的代码，能让我们在修改或者阅读代码的时候，聚集到在一个小范围的模块或者类中，不需要了解太多其他模块或类的代码，让我们的焦点不至于太发散，也就降低了阅读和修改代码的难度。而且，因为依赖关系简单、耦合小，修改代码不会牵一发而动全身，代码改动比较集中，引入 Bug 的风险也就减少了很多。

## 为扩展而设计
越是复杂项目，越要在前期设计上多花点时间。提前思考项目中未来可能会有哪些功能需要扩展，提前预留好扩展点，以便在未来需求变更的时候，在不改动代码整体结构的情况下，轻松地添加新功能。做到代码可扩展，需要代码满足`开闭原则`，基于扩展而非修改来添加新功能，最小化、集中化代码改动，避免新代码影响到老代码，降低引入 Bug 的风险。

除了满足开闭原则，做到代码可扩展，我们前面也提到很多方法，比如封装和抽象，基于接口编程等。识别出代码可变部分和不可变部分，**将可变部分封装起来，隔离变化，提供抽象化的不可变接口，供上层系统使用**。当具体的实现发生变化的时候，我们只需要基于相同的抽象接口，扩展一个新的实现，替换掉老的实现即可，上游系统的代码几乎不需要修改。

## KISS 首要原则
简单清晰、可读性好，是任何大型软件开发要遵循的首要原则。只要可读性好，即便扩展性不好，顶多就是多花点时间、多改动几行代码的事情。但是，如果可读性不好，连看都看不懂，那就不是多花时间可以解决得了的了。不管是自己还是团队，在参与大型项目开发的时候，要尽量避免过度设计、过早优化，在扩展性和可读性有冲突的时候，或者在两者之间权衡，模棱两可的时候，应该选择**遵循 KISS 原则，首选可读性**。

## 最小惊奇原则
《Unix 编程艺术》一书中提到一个 Unix 的经典设计原则：“最小惊奇原则”，英文是：“The Least Surprise Principle”。实际上，这个原则等同于“遵守开发规范”，意思是，在做设计或者编码的时候要遵守统一的开发规范，避免反直觉的设计。

## 吹毛求疵般地执行编码规范
严格执行代码规范，可以使一个项目乃至整个公司的代码具有完全统一的风格，就像同一个人编写的。而且，命名良好的变量、函数、类和注释，也可以提高代码的可读性。编码规范不难掌握，关键是要严格执行。在 Code Review 时，我们一定要严格要求，看到不符合规范的代码，一定要指出并要求修改。但是，据我了解，实际情况往往事与愿违。虽然大家都知道优秀的代码规范是怎样的，但**在具体写代码的过程中，执行得却不好**。我觉得，这种情况产生的主要原因还是不够重视。很多人会觉得，一个变量或者函数命名成啥样，关系并不大。所以命名时不推敲，注释也不写，Code Review 的时候也都一副事不关己的心态，觉得没必要太抠细节。日积月累，项目代码就会变得越来越差。所以我这里还是要强调一下，细节决定成败，代码规范的严格执行极为关键。

## 编写高质量的单元测试
单元测试是最容易执行且对提高代码质量见效最快的方法之一。高质量的单元测试不仅仅要求测试覆盖率要高，还要求测试的全面性，除了测试正常逻辑的执行之外，还要重点、全面地测试异常下的执行情况。毕竟代码出问题的地方大部分都发生在异常、边界条件下。对于大型复杂项目，集成测试、黑盒测试都很难测试全面，因为组合爆炸，穷举所有测试用例的成本很高，几乎是不可能的。单元测试就是很好的补充。它可以在类、函数这些细粒度的代码层面，保证代码运行无误。**底层细粒度的代码 Bug 少了，组合起来构建而成的整个系统的 Bug 也就相应的减少了**。

## 不流于形式的 Code Review
如果说很多工程师对单元测试不怎么重视，那对 Code Review 就是不怎么接受。我之前跟一些同行聊起 Code Review 的时候，很多人的反应是，这玩意儿不可能很好地执行，形式大于效果，纯粹是浪费时间。是的，即便 Code Review 做得再流畅，也是要花时间的。所以，在业务开发任务繁重的时候，Code Review 往往会流于形式、虎头蛇尾，效果确实不怎么好。但我们并不能因此就否定 Code Review 本身的价值。在 Google、Facebook 等外企中，Code Review 应用得非常成功，已经成为了开发流程中不可或缺的一部分。所以，要想真正发挥 Code Review 的作用，**关键还是要执行到位，不能流于形式**。

## 开发未动、文档先行
对大部分工程师来说，编写技术文档是件挺让人“反感”的事情。一般来讲，在开发某个系统或者重要模块或者功能之前，我们应该先写技术文档，然后，发送给同组或者相关同事审查，在审查没有问题的情况下再开发。这样能够保证事先达成共识，开发出来的东西不至于走样。而且，当开发完成之后，进行 Code Review 的时候，代码审查者通过阅读开发文档，也可以快速理解代码。除此之外，对于团队和公司来讲，文档是重要的财富。对新人熟悉代码或任务的交接等，技术文档很有帮助。而且，作为一个规范化的技术团队，**技术文档是一种摒弃作坊式开发和个人英雄主义的有效方法**，是保证团队有效协作的途径。

## 持续重构、重构、重构
我个人比较反对平时不注重代码质量，堆砌烂代码，实在维护不了了就大刀阔斧地重构甚至重写。有的时候，因为项目代码太多，重构很难做到彻底，最后又搞出来一个四不像的怪物，这就更麻烦了！虽然我刚刚说不支持大刀阔斧、推倒重来式的大重构，但持续的小重构我还是比较提倡的。它也是**时刻保证代码质量、防止代码腐化的有效手段**。换句话说，不要等到问题堆得太多了再去解决，要时刻有人对代码整体质量负责任，平时没事就改改代码。千万不要觉得重构代码就是浪费时间，不务正业！

## 对项目与团队进行拆分
面对大型复杂项目，我们**不仅仅需要对代码进行拆分，还需要对研发团队进行拆分**。我们可以把大团队拆成几个小团队，每个小团队对应负责一个小的项目（模块、微服务等），这样每个团队负责的项目包含的代码都不至于很多，也不至于出现代码质量太差无法维护的情况。

## 我为什么如此强调 Code Review？
据我了解，在国内绝大部分的互联网企业里面，很少有将 Code Review 执行得很好的，这其中包括 BAT 这些大厂。特别是在一些需求变动大、项目工期紧的业务开发部门，就更不可能有 Code Review 流程了。代码写完之后随手就提交上去，然后丢给测试狠命测，发现 Bug 再修改；相反，一些外企非常重视 Code Review，特别是 FLAG 这些大厂，Code Review 落地执行得非常好。在 Google 工作的几年里，我也切实体会到了 Code Review 的好处。

### Code Review 践行“三人行必有我师”
Google 工程师的平均研发水平都很高，但即便如此，我们发现，不管谁提交的代码，包括 Jeff Dean 的，只要需要 Review，都会收到很多 Comments（修改意见）。中国有句老话，“三人行必有我师”，我觉得用在这里非常合适。即便自己觉得写得已经很好的代码，只要**经过不停地推敲，都有持续改进的空间**。所以，永远不要觉得自己很厉害，写的代码就不需要别人 Review 了；永远不要觉得自己水平很一般，就没有资格给别人 Review 了；更不要觉得技术大牛让你 Review 代码只是缺少你的一个“Approve”，随便看看就可以。

### Code Review 能摒弃“个人英雄主义”
在一个成熟的公司里，所有的**架构设计、实现，都应该是一个团队的产出**。尽管这个过程可能会由某个人来主导，但应该是整个团队共同智慧的结晶。如果一个人默默地写代码提交，不经过团队的 Review，这样的代码蕴含的是一个人的智慧。代码的质量完全依赖于这个人的技术水平。这就会导致代码质量参差不齐。如果经过团队多人 Review、打磨，代码蕴含的是整个团队的智慧，可以保证代码按照团队中的最高水准输出。

### Code Review 能有效提高代码可读性
自己看自己写的代码，总是会觉得很易读，但换另外一个人来读你的代码，他可能就不这么认为了。毕竟自己写的代码，其中涉及的业务、技术自己很熟悉，别人不一定会熟悉。既然自己对可读性的判断很容易出现错觉，那 Code Review 就是一种考察代码可读性的很好手段。如果代码审查者很费劲才能看懂你写的代码，那就说明代码的可读性有待提高了。还有，写代码的时候，时间一长，改动的文件一多，就感觉晕乎乎的，脑子不清醒，逻辑不清晰？有句话讲，“旁观者清，当局者迷”，说的就是这个意思。**Code Review 能有效地解决“当局者迷”的问题**。在正式开始 Code Review 之前，当我们将代码提交到 Review Board 之后，所有的代码改动都放到了一块，看起来一目了然、清晰可见。这个时候，还没有等其他同事 Review，我们自己就能发现很多问题。

### Code Review 是技术传帮带的有效途径
业务上面，我们可能通过文档或口口相传的方式，那技术呢？如何培养初级工程师的技术能力呢？Code Review 就是一种很好的途径。每次 Code Review 都是一次真实案例的讲解。通过 Code Review，在实践中将技术传递给初级工程师，比让他们自己学习、自己摸索来得更高效！

### Code Review 保证代码不止一个人熟悉
如果一段代码只有一个人熟悉，如果这个同事休假了或离职了，代码交接起来就比较费劲。有时候，我们单纯只看代码还看不大懂，又要跟 PM、业务团队、或者其他技术团队，再重复来一轮沟通，搞的其他团队的人都很烦。而 Code Review 能保证任何代码同时都至少有两个同事熟悉，互为备份，有备无患。

### Code Review 能打造良好的技术氛围
提交代码 Review 的人，希望自己写的代码足够优秀，毕竟被同事 Review 出很多问题，是件很丢人的事情。而做 Code Review 的人，也希望自己尽可能地提出有建设性意见，展示自己的能力。所以，Code Review 还能增进技术交流，活跃技术氛围，培养大家的极客精神，以及对代码质量的追求。一个良好的技术氛围，能让团队有很强的自驱力。不用技术 Leader 反复强调代码质量有多重要，团队中的成员就会自己主动去关注代码质量的问题。这比制定各种规章制度、天天督促执行要更加有效。

### Code Review 是一种技术沟通方式
对于跨不同办公室、跨时区的沟通，Code Review 是一种很好的沟通方式。我今天白天写的代码，明天来上班的时候，跨时区的同事已经帮我 Review 好了，我就可以改改提交，继续写新的代码了。这样的协作效率会很高。

### Code Review 能提高团队的自律性
在开发过程中，难免会有人不自律，存在侥幸心理：反正我写的代码也没人看，随便写写就提交了。Code Review 相当于一次代码直播，曝光 Dirty Code，有一定的威慑力。这样大家就不敢随便应付一下就提交代码了。

## 如何在团队中落地执行 Code Review？
**Q: 有人认为，Code Review 流程太长，太浪费时间，特别是工期紧的时候，今天改的代码，明天就要上，如果要等同事 Review，同事有可能没时间，这样就来不及。这个时候该怎么办呢？**
A: 我所经历的项目还没有一个因为工期紧，导致没有时间 Code Review 的。工期都是人排的，稍微排松点就行了啊。我觉得关键还是在于整个公司对 Code Review 的接受程度。而且，Code Review 熟练之后，并不需要花费太长的时间。尽管开始做 Code Review 的时候，你可能因为不熟练，需要有一个 Checklist 对照着来做。起步阶段可能会比较耗时。但当你熟练之后，Code Review 就像键盘盲打一样，你已经忘记了哪个手指按的是哪个键了，扫一遍代码就能揪出绝大部分问题。

**Q: 有人认为，业务一直在变，今天写的代码明天可能就要再改，代码可能不会长期维护，写得太好也没用。这种情况下是不是就不需要 Code Review 了呢？**
A: 这种现象在游戏开发、一些早期的创业公司或者项目验证阶段比较常见。项目讲求短平快，先验证产品，再优化技术。如果确实面对的还只是生存问题，代码质量确实不是首要的，特殊情况下，不做 Code Review 是支持的！

**Q: 有人说，团队成员技术水平不高，过往也没有 Code Review 的经验，不知道 Review 什么，也 Review 不出什么。自己代码都没写明白，不知道什么样的代码是好的，什么样的代码是差的，更不要说 Review 别人的代码了。在 Code Review 的时候，团队成员大眼瞪小眼，只能 Review 点语法，形式大于效果。这种情况该怎么办？**
A: 这种情况也挺常见。不过没关系，团队的技术水平都是可以培养的。我们可以先让资深同事、技术好的同事或技术 Leader，来 Review 其他所有人的代码。Review 的过程本身就是一种“传帮带”的过程。慢慢地，整个团队就知道该如何 Review 了。虽然这可能会有一个相当长的过程，但如果真的想在团队中执行 Code Review，这不失为一种“曲线救国”的方法。

**Q: 还有人说，刚开始 Code Review 的时候，大家都还挺认真，但时间长了，大家觉得这事跟 KPI 无关，而且我还要看别人的代码，理解别人写的代码的业务，多浪费时间啊。慢慢地，Code Review 就变得流于形式了。有人提交了代码，随便抓个人 Review。Review 的人也不认真，随便扫一眼就点“Approve”。这种情况该如何应对？**
A: 我的对策是这样的。首先，要明确的告诉 Code Review 的重要性，要严格执行，让大家不要懈怠，适当的时候可以“杀鸡儆猴”。其次，可以像 Google 一样，将 Code Review 间接地跟 KPI、升职等联系在一块，高级工程师有义务做 Code Review，就像有义务做技术面试一样。再次，想办法活跃团队的技术氛围，把 Code Review 作为一种展示自己技术的机会，带动起大家对 Code Review 的积极性，提高大家对 Code Review 的认同感。