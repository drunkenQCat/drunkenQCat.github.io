这个项目是朋友委托我做的：给《怪物猎人》做一个肉鸽模式。进斗技场、三选一决定下一波、打怪拿 PT、购买补给和增益，再继续往下打。

他最初想做在《怪物猎人：世界》里。我也在《世界》上探索了几个星期，后来痛定思痛，换到了《怪物猎人：崛起》。再后来，为了让 Agent 真正能在游戏里干活，我又写了网页后端、原生插件和 CLI，最后连 REFramework 都 fork 了一份。

回头看，这件事最值得分享的，首先是：**选择框架是成功的一半。** 选对框架以后，还得给 Agent 一条能操作、能观察、能拿到错误的开发通路。

## 一、在《世界》里，难点很快越过了框架已经铺好的路

《世界》使用 MT Framework。我当时选择的开发基础是 SharpPluginLoader，简称 SPL，项目用的是 C#／.NET 8 和 SPL 2.0.0。

SPL 已经做了很多工作：插件加载、游戏事件、ImGui、常用游戏对象的包装、原生调用与 hook。它还保留了 MT Framework 的 DTI 类型信息，可以找类型、看继承关系、构造对象。把它理解成一个只能修改几个参数的小工具，会低估它。

但程序性的肉鸽玩法，很快就会问到框架尚未替你回答的问题：如何在任务中加载新怪？构造对象之后，谁负责资源、AI、任务登记与死亡清理？原生结算应该在哪里拦截和恢复？

看 SPL 的实现，能直观理解这条边界。

它的原生地址仓库会读取 `AddressRecords.json`，根据游戏版本和记录文件的 hash 决定是否复用缓存，否则进行模式扫描。C# 包装再按名字取得对应地址。例如 `UnitManager` 内部把 `UnitManager:AddTop` 包装成了带明确参数类型的 `NativeFunction`，部分对象属性则使用已知的内存偏移。具体实现可以看 [AddressRepository.cpp](https://github.com/Fexty12573/SharpPluginLoader/blob/c16380a398c3252309ba94d1c154ad58f6ab5413/mhw-cs-plugin-loader/AddressRepository.cpp) 和 [UnitManager.cs](https://github.com/Fexty12573/SharpPluginLoader/blob/c16380a398c3252309ba94d1c154ad58f6ab5413/SharpPluginLoader.Core/UnitManager.cs)。

这意味着，已经识别、封装好的功能很好用；需求落到覆盖范围之外，就得自己继续找地址、认参数、确认对象布局和调用时机。

这里需要说准确：**SPL 并不禁止调用它没收录的函数。** 官方就有[直接调用原生函数](https://github.com/Fexty12573/SharpPluginLoader/blob/master/docs/Development/CallingNativeFuncs.md)和 [hook](https://github.com/Fexty12573/SharpPluginLoader/blob/master/docs/Development/FunctionHooking.md) 的方法。代价是，这些函数的地址和协议需要自己找出来。

旧项目 `mhw-moddev` 里的探索就卡在这类问题上。我们尝试过：

```csharp
var unit = UnitManager.NewUnit(dti);
UnitManager.AddTop(unit, line, group);
```

SPL 的 `NewUnit` 实现主要是通过 DTI 创建实例，源码还明确提醒，它不会自动把对象注册进更新 line。即使再调用 `AddTop`，也不能据此认为已经完成“加载并生成一只可正常战斗的任务首领”。旧项目后来又转向任务出怪表，维护特定版本的地址和候选偏移，继续找完整的原生流程。

这些工作可以做，但每往外走一步，探索成本都在增加。

社区也并非没有复杂 mod。我们实际看过动画查看器、自定义动作、自定义网络包和 DPS 统计等代码。更准确地说，是当时没找到一条能直接拿来复用、覆盖我们这套肉鸽流程的完整路径。还有一次崩溃，查到的是我们自己的插件把回调签名写错了，这也不能算到 SPL 头上。

## 二、多屏环境又让游戏内开发面板变得很难用

我的工作环境有多个分辨率不同的屏幕。《世界》这套 ImGui 在我的环境里会画出很模糊的东西，自己看着难受，Computer Use 识别和点击也更困难。

这不是一句“ImGui 不好看”。面板一旦成为开发的主要入口，字体、缩放、焦点和点击位置都会影响每一轮实验。

不过，代码核对之后也不能简单归因为“SPL 没有做 DPI”。它的 [D3DModule.cpp](https://github.com/Fexty12573/SharpPluginLoader/blob/c16380a398c3252309ba94d1c154ad58f6ab5413/mhw-cs-plugin-loader/D3DModule.cpp) 里确实有 `ImGui_ImplWin32_EnableDpiAwareness()`，初始化也传入了 swapchain 的 viewport 和窗口尺寸，默认字体使用了固定字号。它们提供了排查缩放问题的线索，但还不足以证明我那台多屏机器上模糊的具体根因。

对我来说，实际结果已经足够糟糕：底层机制要继续逆向，开发入口又不好操作。于是放下了《世界》里几个星期的探索，改去《崛起》。

## 三、REFramework 带来的变化，是一套更适合探索的接口

《崛起》换成了 RE Engine，开发基础也换成 REFramework。

它和 SPL 最影响我工作方式的区别，是 REFramework 对引擎类型数据库提供了统一的发现、访问和调用接口。Lua 可以按名字找类型、枚举方法和字段、取得单例，再调用对象的方法。相关接口在 [SDK 文档](https://cursey.github.io/reframework-book/api/sdk.html) 中可以直接查到。

例如，一个小探针就能先看某个管理器暴露了什么：

```lua
local t = sdk.find_type_definition("snow.stage.StageManager")
if t then
    for _, method in ipairs(t:get_methods()) do
        log.info(method:get_name())
    end
end
```

在类型信息可用的范围内，这显著改变了探索的起点。Agent 可以先查类型、方法、字段、参数，再去追对应的原生函数和状态机。SPL 同样有引擎类型能力，但它的 DTI 和手工包装，并不能简单视为这套方法／字段调用接口的等价替代。

REFramework 也不是把游戏源码交到了手里。方法存在，不等于随时都能调用；名字像“复活”，也不代表它就负责首领复活。我们照样追错过列表，照样需要反汇编和实机证据来确定调用条件。

但能按类型组织这些问题，已经帮了大忙。社区还有成熟的 Lua 脚本和示例，Agent 可以从别人已经跑通的代码里，提取字段访问、对象取得、hook 和恢复的写法。例如临时攻击加成，我们就参考过社区 [Buffer 模组](https://github.com/Bimmr/Monster-Hunter-Rise-Reframework-Scripts-/blob/main/Buffer/autorun/Buffer/Modules/Character.lua) 的实现。

从“每个新入口都先找地址”，走到“先沿类型和已有代码查线索”，对这个需要大量试探的项目，差别很大。

## 四、然后发现，Agent 仍然不擅长操作游戏

换了框架，操作问题还在。

**GPT 的 Computer Use 连 WASD 都不会，能让游戏卡在那里一晚上。这才是我写 CLI 最初、最直接的动力。**

这里说的是我这次开发里的实际体验：我需要它进任务、移动角色、去怪物旁边验证。它却可能长时间停在同一个场景里，折腾不明白基本移动。到了 REFramework 自动生成的 ImGui 面板，它点击也非常慢，还会因为焦点、窗口遮挡、界面判断等原因卡住。

我不想一直守在旁边，替它完成这些操作。

因此，我最初设想的其实还不是 CLI，而是一个后端：把 Lua 一侧的功能串出来。我自己可以在网页上点击，Agent 也可以用 `curl` 请求端点，不必每次都找到游戏里的按钮。

这个想法受了 ReaPy 的启发。我之前接触过 REAPER，知道宿主里的脚本能力可以通过桥接，从外部进程调用。[ReaPy](https://github.com/RomeoDespres/reapy) 本身是 REAPER ReaScript Python API 的包装，它的外部请求通过宿主中的循环处理。我借鉴的是这种分工方式，并不是移植它的网络代码。

REAPER 和 REFramework 都支持 Lua，这让我最初觉得：在 Lua 里跑一个 HTTP 或 WebSocket 服务，把命令接进来，应该就能开始了。

## 五、Lua 没有现成网络栈，HTTP 最后进了原生插件

实际碰到的第一道坎，是 REFramework 的嵌入式 Lua 并没有现成可用的 HTTP／WebSocket 服务栈。

这里也要纠正一个容易说错的细节：Lua 标准库本来就不包含 HTTP、WebSocket 或 socket 库，不能说 REFramework 把“Lua 自带的 HTTP”删掉了。它的 [ScriptRunner 初始化代码](https://github.com/drunkenQCat/REFramework/blob/5bae4701396248a776c1de19f5be9552022295d5/src/mods/ScriptRunner.cpp#L118) 显式加载了一组 Lua 标准库，并限制了部分系统能力。LuaSocket 等东西属于另外的原生扩展，装载路径、Lua 版本和 ABI 也要另外验证。

最后我们选择编译一个 REFramework 原生插件，把 HTTP 服务放到 C++ 里。插件在 Lua state 创建时注册桥接模块，Lua 通过 `require("webconsole")` 取得收命令和发布状态的能力。

架构大致是这样：

```text
网页 / curl / CLI
        │ HTTP
        ▼
webconsole.dll 的 HTTP 线程
        │ 有界内存队列
        ▼
Lua 更新回调取出命令
        │
        ▼
游戏业务逻辑 / REFramework SDK
        │
        ▼
发布状态快照与回执 → 外部客户端读取
```

HTTP 请求线程负责通信，不直接去操作游戏对象。Lua 在框架回调里执行命令，再把结果发布回去。这样网页和 Agent 共用同一套业务逻辑，也不用为每个新按钮重新做一套底层通道。

后端做好以后，我开始想让 Agent 用它干更多事，于是让它再写了一个替代前端的 CLI。

## 六、CLI 把“点进去”变成了可编排的流程

有后端以后，CLI 的价值很快就显现出来了。

之前进入游戏，要看启动画面、判断菜单、处理弹窗、选存档、等加载。Agent 很容易在某一步停住。现在可以把这些步骤组织成一个动作流程，根据真实 UI 状态决定下一步，并检查执行后的变化：

```powershell
node cli/mhr.mjs doctor --json
node cli/mhr.mjs game enter --slot 1 --timeout 120 --json
node cli/mhr.mjs state --json
```

`game enter` 内部也不是固定等几秒、按一串键。它观察标题和存档界面的状态，在能识别的场景提交相应操作；遇到未知窗口就停下。请求携带场景 token，后端再检查一次，避免在画面已经变化以后继续执行旧操作。

这条路径已有从启动界面进入存档、到村庄的实测记录。之前要花很久很久才能进去的游戏，终于可以交给一段可复用的流程。

再往后，选波次、开无敌、定向传送、购买增益、读取怪物 UID 和保存采样，也逐渐变成明确的命令。CLI 给 Agent 的帮助，开始超出“替代难点的按钮”：它能把动作、等待条件和结果记录连接起来。

当然，协议本身也要认真做。我们遇到过击杀已经生效，回执却一直停在 `queued` 的问题。HTTP 200、命令受理和游戏操作完成，需要分别表达。否则 Agent 可能把成功当失败，再错误地重复一次。

## 七、为了不再点 Reset scripts，我 fork 了 REFramework

还有一个绕不开的按钮：Reset scripts。

REFramework 支持重载脚本，但当时用的上游版本没有把这个操作暴露成插件可调用的 API。修改了 Lua，仍得回游戏内点击 ScriptRunner 的 Reset scripts。这个步骤又把开发流程拉回了慢吞吞的界面操作。

最初我只是想，把这个按钮接到后端。看完实现以后，决定直接 fork 一份框架，把缺少的入口补出来。

第一个补丁是 [`request_script_reset`](https://github.com/drunkenQCat/REFramework/commit/9e06c961ab5f068dd9be5b5a1f09430c8a198625)。它不让 HTTP 线程直接销毁 Lua state，而是设置一个原子请求标记，交给框架的 `on_frame` 消费，沿用原来的 reset 流程：通知 state 销毁、清理并重建 Lua state，再运行 autorun。

对应到外部接口，就是：

```powershell
curl.exe -X POST http://127.0.0.1:11777/api/script/reset
```

既然已经 fork 了，我又问 Agent：还有什么能力，能方便你继续开发？

它接着提出两件事：**从外部运行指定 Lua 脚本，以及读取脚本报错。**

于是又补了两个接口：

| 框架插件 API | 实现方式 | 后端入口 |
|---|---|---|
| `request_script_reset()` | 原子标记，由框架回调执行完整 reset | `POST /api/script/reset` |
| `run_script(path)` | 加锁加入脚本队列，在现有主 Lua state 执行 | `POST /api/script/run` |
| `last_script_error(out, size)` | 读取 ScriptRunner 保存的错误文本，复制到调用者缓冲区 | `GET /api/script/error` |

后两项实现分别在 [run_script 提交](https://github.com/drunkenQCat/REFramework/commit/25d0aef8755a588b74aa7e6c54b3e71b72ff20bb)和 [last_script_error 提交](https://github.com/drunkenQCat/REFramework/commit/2e7ce5b97dce68cc742b5c80ad421213c551c803)。新增函数指针追加在插件函数表尾部，尽量保持已有字段的位置。

这里运行一个新探针，不必每次都 reset 全部业务脚本。Agent 可以上传脚本、执行、取回报告；脚本如果在发布报告之前就语法报错，还能从框架的错误通道读到原因。

```powershell
node cli/mhr.mjs game run-script probe.lua --timeout 20 --json
node cli/mhr.mjs game last-error --json
```

需要完整清理回调和 hook 时，仍然使用 reset。`run-script` 是在现有 state 里运行代码，不会自动撤销旧脚本留下的东西；这两种能力各有用途。

替换框架 DLL 本身需要重新启动游戏，但之后日常写探针就不再被文件选择框和 Reset 按钮卡住了。这一步补齐以后，Agent 写探针、看错误、修正、再跑的速度才真正上来了。

## 八、如果当初给 SPL 也补一个 CLI，会不会更好？

这个问题值得认真回答。

**很可能会改善很多，而且没有证据表明 SPL 做不了。**

SPL 的 C#／.NET 环境适合做 HTTP 服务，也可以把外部请求排入游戏更新回调。它的 [PluginManager](https://github.com/Fexty12573/SharpPluginLoader/blob/c16380a398c3252309ba94d1c154ad58f6ab5413/SharpPluginLoader.Core/PluginManager.cs) 本来就用 `FileSystemWatcher` 监控程序集变化、重载插件。因此，热重载也不是 REFramework 独有的优势。

甚至旧项目里已经有一个 ASP.NET 的模拟服务，把肉鸽 Core 暴露成网页 API。不过它操作的是模拟状态，没有接进真实游戏，不能当成游戏后端已经打通。

如果当时把这条路继续接下去，至少可以绕开模糊的 ImGui，改善重复操作、状态采集和实验编排。单论宿主里写 HTTP 服务这件事，.NET 还有很成熟的现成设施。

但 CLI 只能让已经知道怎么做的事情更容易调用。它不会替我们找出未知函数的地址，也不会补出首领生成流程里还没弄清的资源和生命周期协议。

我没有做两套框架的同条件对照实验，所以不能断言“给 SPL 补 CLI 也救不了”，更不能编一个效率倍数。对这次项目来说，换到 REFramework 的收益，是引擎通用的探索接口、社区可借鉴的代码，以及后面自己补出来的远程控制工具链共同作用的结果。

现在看，值得早点投入的有两件事：先选一个能支撑目标复杂度的框架，再把日常开发需要的控制面做好。

从外部发起动作、读取真实状态、拿到报错、反复做一个小实验——这些事情接通以后，Agent 才有机会把时间花在游戏机制上，而不是继续盯着村子看一晚上。
