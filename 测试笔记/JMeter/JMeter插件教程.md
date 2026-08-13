# JMeter 必装插件教程（小白向）

## 知识库入口

- 返回总索引：[[00-知识地图]]
- 关联主题：[[jmeter性能测试|JMeter 快速上手]]、[[web/Web协议和测试|Web 协议和测试]]

> 适用 JMeter 5.x · Windows / Mac 通用 · 更新于 2026-07

---

## ? JMeter 自带的，为什么还不够？

一句话：原生 JMeter 缺了「阶梯加压」和「看得顺眼的图表」这两件性能测试的刚需。

用纯原生 JMeter 跑压测，很快会遇到三个让人抓狂的痛点：

1. **加压方式太死板** —— 原生线程组只能「一次性把所有用户在 Ramp-up 时间内均匀上线」，没法做「先上 50 个稳一会儿，再上到 100，再稳一会儿」这种**阶梯式加压**。而阶梯加压恰恰是找性能拐点、找瓶颈的标准姿势。
2. **图表难看且不够用** —— 自带的「察看结果树」「聚合报告」给的是表格，原生图形顶多画个折线，**响应时间随时间变化、TPS 随时间变化、活跃线程数随时间变化**这类关键趋势图原生没有现成的。
3. **没法精确控制吞吐量** —— 你想「精确维持每秒 50 个请求」，原生定时器做不到平滑控制。

社区早就替你把这些坑填好了，统称为 **JMeter Plugins**（jmeter-plugins.org）。而这些插件几乎都通过一个「插件管理器」来一键安装。

> [!info] 关键认知
> 先装「插件管理器（Plugins Manager）」这一个 jar，之后所有插件都在 JMeter 菜单里勾选、点安装即可，不用再手动下 jar 包。管理器是「插件中的插件」，是地基。

---

## 01 必装插件全景

从上百个插件里筛出来的「人人必装」清单：

| 插件 | 英文名 | 定位 |
|------|--------|------|
| 🧰 插件管理器 | JMeter Plugins Manager | **地基·必装**。安装其他所有插件的总入口 |
| 📈 自定义线程组 | Custom Thread Groups | **高频·刚需**。阶梯加压、极限加压 |
| 📊 基础图表 | 3 Basic Graphs (+ 5 Additional Graphs) | **高频·刚需**。响应时间/TPS/活跃线程数趋势图 |
| ⏱️ 吞吐量定时器 | Throughput Shaping Timer | **推荐**。精确控制每秒请求数 |
| 🎭 模拟采样器 | Dummy Sampler | **推荐**。伪造采样结果，调试利器 |

> [!info] 安装顺序
> 第 1 个「插件管理器」是手动下载的，其余都在管理器里勾选安装。下面分步来。

---

## 02 安装插件管理器（保姆级）

这是唯一需要手动下载的步骤，5 分钟搞定。装完它，后面全是点点点。

### 第 1 步：确认 JMeter 安装目录

找到你的 JMeter 安装根目录（解压后里面有 `bin`、`lib` 等文件夹的那个目录）：

```shell
# Windows 示例
D:\tools\apache-jmeter-5.5\

# Mac 示例
~/tools/apache-jmeter-5.5/
```

进入该目录，应能看到 `lib` 文件夹，再进去会看到 `ext` 子文件夹——插件管理器的 jar 包就要放到这个 `lib/ext` 里。

### 第 2 步：下载插件管理器 jar 包

1. 打开浏览器，访问官方下载页：`https://jmeter-plugins.org/install/Install/`
2. 页面中间有一个大大的下载按钮（类似 **“Plugins Manager jar”**），点击下载，得到一个类似 `JMeterPlugins-Manager-1.10.jar` 的文件（版本号会随时间变化，下最新的即可）。
3. 把这个 jar 文件放到 JMeter 的 `lib/ext` 目录下。完整路径例如：`D:\tools\apache-jmeter-5.5\lib\ext\JMeterPlugins-Manager-1.10.jar`

> [!warning] 别放错地方
> 一定要放在 `lib/ext`，不是 `lib` 根目录，也不是 `bin`。放错位置 JMeter 启动后菜单里不会出现管理器。

### 第 3 步：重启 JMeter

- 如果 JMeter 正开着，先**完全关闭**它（不只是关窗口，确认进程退出）。
- 重新双击 `bin/jmeter.bat`（Mac 执行 `sh jmeter.sh`）启动。

### 第 4 步：验证安装成功

启动后，看顶部菜单栏：点击菜单 `Options`（选项） → 如果下拉里出现了 **“Plugins Manager”** 这一项，**说明安装成功**，地基已经打好。

点击它，会弹出「Plugins Manager」对话框，里面有 **Installed（已安装）** 和 **Available（可安装）** 两个标签页，列出上百个插件——这就是后面所有插件的安装入口。

> [!success] 最大的坎已跨过
> 从这里开始，所有插件都是「在 Plugins Manager 里打勾 → 点 Apply Changes and Restart JMeter」就完事，再也不用手动下 jar 包了。

---

## 03 通用安装流程（所有插件通用）

后面每个插件的安装都是这套流程，记住一次，用一百次：

1. 菜单 `Options` → `Plugins Manager`，打开管理器窗口。
2. 切到 **Available** 标签页，在搜索框输入插件名（例如 `Custom Thread Groups`）。
3. 在结果列表里找到目标插件，**勾选**它前面的复选框。
4. （可选）继续搜索并勾选其他插件，可以一次性批量安装。
5. 点右下角 **“Apply Changes and Restart JMeter”** 按钮。
6. JMeter 会自动下载、安装并重启。重启后，新插件对应的元件就会出现在「添加」菜单里了。

> [!info] 装完怎么找？
> 每个插件安装后会以特定前缀出现在 JMeter 的「添加（Add）」菜单里。社区插件通常以 `jp@gc -` 开头，比如 `jp@gc - Response Times Over Time`。看到这个前缀，就是插件装好了。

---

## 04 自定义线程组：阶梯加压

最该先装、用得最多的插件。让加压曲线可以「分台阶」走。

### 它能解决什么问题

| | 加压方式 | 能否找拐点 |
|---|---|---|
| ❌ 原生线程组 | 100 个用户在 60 秒内**匀速**上线，曲线一条直线 | 看不出「50 用户时还好，100 用户时崩了」 |
| ✅ 阶梯加压线程组 | 先上 20 个、稳跑 60 秒，再上到 40、再稳 60 秒…… | 一步步逼近极限，**精准定位拐点** |

### 安装

在 Plugins Manager 搜索 `Custom Thread Groups`，勾选安装，重启。

### 它提供的两个核心元件

| 元件名 | 用途 |
|--------|------|
| **jp@gc - Stepping Thread Group**（阶梯线程组） | 最常用。设置一系列「加几人 + 持续多久」的台阶，逐级加压。 |
| **jp@gc - Ultimate Thread Group**（极限线程组） | 更灵活，可为每个线程批次独立设置「启动数、爬升时间、保持时间、关闭数、关闭时间」，适合复杂加压脚本。 |

### 实战：用 Stepping Thread Group 做阶梯加压

1. 在测试计划上右键 → `Add` → `Thread(s)` → `jp@gc - Stepping Thread Group`。
2. 看到一张「台阶」参数表，每行一个台阶。重点参数含义如下：

```shell
# Stepping Thread Group 参数说明
This group will start N threads          # 本组最终上线总线程数
First, wait for N seconds                # 启动后先等多少秒（预热等待）
Then add N threads every N seconds,
     using ramp-up N seconds             # 每 N 秒增加 N 个线程，单次爬升耗时 N 秒
Then hold load for N seconds             # 加满后保持多久
Finally, stop N threads every N seconds  # 优雅退场，每 N 秒停 N 个
```

举个配置示例，模拟「每 30 秒加 10 个用户、加到 100、保持 5 分钟、再慢慢退」：

```shell
This group will start 100 threads
First, wait for 5 seconds
Then add 10 threads every 30 seconds, using ramp-up 5 seconds
Then hold load for 300 seconds
Finally, stop 10 threads every 20 seconds
```

1. 配好台阶后，下方会有一个**实时预览的加压曲线图**，直观显示线程数随时间变化——超好用的可视化确认。
2. 在这个线程组里像平时一样添加「HTTP 请求」取样器、监听器等，运行即可看到阶梯效果。

> [!info] 小白要点
> 阶梯加压的意义是「**边加边观察**」。配合下一节的「响应时间趋势图」「TPS 趋势图」，你就能清楚看到：加到 60 个用户时响应时间开始飙升 / TPS 开始掉——那个点就是系统的**性能拐点**，也是你写测试报告的核心结论。

---

## 05 基础图表：让压测结果「看得见」

3 Basic Graphs + 5 Additional Graphs，加压有了，结果也得画出来。

### 安装

在 Plugins Manager 分别搜索并勾选安装这两个插件包：

- `3 Basic Graphs` —— 基础三图
- `5 Additional Graphs` —— 进阶五图（推荐一起装，画图更全）

### 安装后你会得到这些监听器

| 插件包 | 监听器（添加 → Listener 里找） | 看什么 |
|--------|------|--------|
| **3 Basic Graphs** | `jp@gc - Response Times Over Time` | 响应时间随时间变化曲线 |
| | `jp@gc - Transactions Per Second` | 每秒事务数（TPS）随时间变化 |
| | `jp@gc - Active Threads Over Time` | 活跃线程数随时间变化 |
| **5 Additional Graphs** | `jp@gc - Response Times Percentiles` | 响应时间百分位（P90/P95/P99） |
| | `jp@gc - Response Time Distribution` | 响应时间分布直方图 |
| | `jp@gc - Hits per Second` | 每秒点击数 |
| | `jp@gc - Latencies Over Time` | 延迟（首字节时间）趋势 |
| | `jp@gc - Connect Times Over Time` | 建连耗时趋势 |

### 实战：添加并查看图表

1. 在线程组（或测试计划）上右键 → `Add` → `Listener` → 选 `jp@gc - Response Times Over Time`。
2. 同理再加 `jp@gc - Transactions Per Second` 和 `jp@gc - Active Threads Over Time`。
3. 点击工具栏 ▶ 运行测试。
4. 运行过程中，点击对应监听器，曲线会**实时刷新**。能直观看到：用户越加越多时，响应时间是否被拉长、TPS 是否还能扛住。

> [!success] 读图诀窍（拐点判断法）
> 把「活跃线程数图」和「TPS 图」「响应时间图」三张对齐看——线程数线性上涨时，**TPS 不再涨甚至掉头向下、响应时间开始陡升的那个时刻**，就是系统接近极限的拐点。这比只看聚合报告里的平均值高级得多。
>
> [!warning] GUI 运行注意
> 图形监听器比较吃内存，线程数很大时 GUI 会卡。调脚本阶段用 GUI 看图没问题；**真正大压力跑分时要用命令行模式**（`jmeter -n -t xxx.jmx -l log.jtl`），再用 `-g log.jtl -o report` 生成 HTML 报告，里面这些图都有。

---

## 06 吞吐量定时器：精确控 TPS

当你需要「不是加多少人，而是维持每秒多少请求」时，它登场。

### 为什么需要它

线程组控制的是「**有多少并发用户**」，但有时候你的需求是「**系统要扛住每秒 100 个请求**」——这是吞吐量目标，不是并发用户数目标。两者不一样：100 个用户如果每个 1 秒发一次，TPS 才 100；10 个用户每个 0.1 秒发一次，TPS 也能 100。

**Throughput Shaping Timer** 让你直接按时间编排「每秒要达到多少 RPS」，JMeter 会自动动态调节请求节奏去逼近目标。

### 安装

Plugins Manager 搜索 `Throughput Shaping Timer`，勾选安装，重启。

### 实战：维持每秒 50→100→200 的阶梯吞吐

1. 在线程组下右键 → `Add` → `Timer` → `jp@gc - Throughput Shaping Timer`。
2. 在定时器面板里点 **“Add Row”** 添加几个阶段，每行三个参数：

```shell
# 每行：Start RPS / End RPS / 持续秒数
1    50    60    # 前 60 秒：维持 50 RPS
50   100   60    # 接下来 60 秒：从 50 爬到 100 RPS
100  200   120   # 再 120 秒：从 100 爬到 200 RPS
```

1. **关键配套**：吞吐量定时器只管「节奏」，真正发请求还得靠线程。要保证线程数「够用」，否则目标 TPS 打不到。一般把线程组的**线程数设得比预估需求多一些**（比如目标 200 RPS、单请求耗时 0.5 秒，至少需要 100 个线程）。
2. 搭配上一节的「Transactions Per Second」图表运行，就能直观验证实际 TPS 是否贴近你设定的目标曲线。

> [!info] 使用场景
> 容量规划（「系统能不能稳定扛住 500 QPS」）、SLA 验证（「P99 是否在 200ms 内」）、对比基准测试（不同版本用相同 TPS 曲线对比）。这是进阶性能测试的标志能力。

---

## 07 模拟采样器：调试利器

不发真实请求，也能造一条结果——调脚本、试图表时省时省力。

### 它能干什么

**Dummy Sampler** 不真正访问任何服务器，而是「伪造」一条采样结果：你可以指定响应时间、响应码、响应数据。这意味着你可以：

- **调试脚本逻辑**：在后置处理器、断言、变量提取还没接通真实接口时，先用假结果把脚本流程跑通。
- **演示图表**：想给学生看「TPS 图长什么样」，不必真去压服务器，用 Dummy 快速造数据。
- **模拟异常**：手动设响应码 500、超长响应时间，验证你的断言和报告逻辑是否正确处理异常。

### 安装与使用

1. Plugins Manager 搜索 `Dummy Sampler`，勾选安装，重启。
2. 线程组右键 → `Add` → `Sampler` → `jp@gc - Dummy Sampler`。
3. 在面板里填写：**Response Code**（如 200）、**Response Time**（如 120，单位 ms）、**Response Data**（响应体内容，可放 JSON）。
4. 运行，察看结果树里就会出现一条「假」但完整的采样结果，后续的断言、提取器都能正常处理它。

> [!success] 小白友好提示
> 它本质是「脚手架」。等真实接口就绪后，把 Dummy Sampler 禁用或删掉、启用真实 HTTP 请求即可。调试期用它，能少走很多弯路。

---

## 08 实战串联：一次完整的插件压测

把上面几个插件串成一个真实可跑的压测脚本。

> **目标场景**：对一个 Web 首页做**阶梯加压**——从 0 用户每 20 秒加 10 个、加到 100，保持 3 分钟，观察响应时间和 TPS 的拐点。

### 脚本搭建步骤

1. 新建测试计划，添加 `jp@gc - Stepping Thread Group`（阶梯线程组），按第 04 节配置到 100 用户、每 20 秒加 10。
2. 在线程组里添加 `HTTP Request Defaults`（HTTP 请求默认值），填好被测服务器 IP 和端口，避免每个请求重复写。
3. 添加一个 `HTTP Request` 取样器，路径填首页路径（如 `/`）。
4. 添加监听器：`View Results Tree`（察看结果树，调试用）、`jp@gc - Active Threads Over Time`、`jp@gc - Response Times Over Time`、`jp@gc - Transactions Per Second`。
5. 保存测试计划为 `.jmx` 文件。
6. 点 ▶ 用 GUI 跑一遍，确认三张图都在动、请求都 200。

### 正式压测（命令行模式）

调通后，真正的压力测试用命令行跑，避免 GUI 拖累性能：

```shell
# Windows，jmeter 路径换成你的
D:\tools\apache-jmeter-5.5\bin\jmeter -n -t steptest.jmx -l result.jtl -e -o report

jmeter -n -t "C:\Users\17867\Desktop\Projects\Jmeter\Temp_jmx\testplan.jmx" -l "C:\Users\17867\Documents\result.jtl" -e -o "C:\Users\17867\Documents\report"
# 参数解释
-n  非GUI模式（命令行）
-t  指定测试计划文件
-l  指定结果日志文件（.jtl）
-e  测试结束后生成HTML报告
-o  报告输出目录（必须不存在或为空）
```

跑完后打开 `report/index.html`，里面会有一套完整的仪表盘：APDEX、统计表、响应时间/TPS/错误率随时间变化的图……比 GUI 里一张张看更全面，而且**可直接作为测试报告交付**。

> [!info] 插件图表与 HTML 报告的关系
> 命令行模式跑出来的 `report` 是 JMeter 原生报告，已包含趋势图，**不依赖** jp@gc 插件。插件图表主要用于**调试阶段 GUI 实时观察**。两者互补：调试看插件图、出报告用原生 HTML 报告。

---

## 09 常见问题 FAQ

### Q1：Plugins Manager 安装后，Options 菜单里找不到？

三个排查点：① jar 没放进 `lib/ext`（最常见，放成了 `lib`）；② 没重启 JMeter；③ 下的 jar 不完整，重新下载覆盖一次。正确放好后**完全关闭再重启** JMeter 即可。

### Q2：安装插件后报错 / JMeter 启动不了？

多半是**插件版本与 JMeter 版本不兼容**。确认你的 JMeter 版本（菜单 Help → About），尽量用 JMeter 5.x 最新稳定版；插件也用最新版。若装某个插件后启动崩溃，可手动删除 `lib/ext` 里对应插件的 jar 来恢复。

### Q3：装这么多插件会影响压测准确性吗？

会影响，但可控。**监听器类**插件（图表）在 GUI 模式下吃内存，线程数大时会拖慢、甚至影响压测机自身性能。所以：调试阶段用 GUI 看图没问题；**正式大压力测试一律用命令行模式**，并把不需要的监听器禁用或删除。命令行模式不渲染图表，几乎无额外开销。

### Q4：jp@gc 是什么意思？

是插件作者网站 `jmeter-plugins.org` 的署名前缀（gc = good code / 作者代号）。所有来自该社区的插件元件在「添加」菜单里都以 `jp@gc -` 开头，看到它就知道是第三方插件装的，与 JMeter 原生元件区分开。

### Q5：插件管理器自己能升级吗？

可以。Plugins Manager 打开后，在 **Installed** 标签页里如果有新版，会提示升级；它自身也会出现在可升级列表里。点升级、重启即可，无需再手动下 jar。

### Q6：离线环境（内网）能装插件吗？

Plugins Manager 默认联网拉取插件列表。离线环境下可手动把所需插件 jar 下好放进 `lib/ext`，效果一样；只是没法用管理器自动更新。也可配置内网镜像源，具体见 jmeter-plugins.org 文档。

---

> 配套图文教程（含交互式导航与卡片视图）：[[JMeter插件教程|JMeter插件教程.HTML]]
> 插件官网：<https://jmeter-plugins.org>
