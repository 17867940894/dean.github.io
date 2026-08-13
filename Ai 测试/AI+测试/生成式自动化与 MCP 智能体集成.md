# 生成式自动化与 MCP 智能体集成

> **来源**：`AI+测试：基于 AI 的自主化 Web 探索测试（day2）.pdf`（共 37 页）


## 一、自动化测试回顾与演进


### 1.1 自动化脚本测试 vs 自动化测试

| 概念           | 侧重点 | 定义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| 自动化脚本测试 | 脚本   | 用代码/工具将手动操作步骤写成可重复执行的脚本                |
| 自动化测试     | 自动化 | 通过预编写的脚本模拟人工操作，并自动校验预期结果（Assertion）的过程，本质是“机械复现” |

### 1.2 现状：面临“脆弱性”与“维护地狱”

- **元素耦合**：脚本强依赖于 DOM 树的定位符（ID、XPath），UI 稍微改动（如 div 嵌套变成 section），脚本立即崩溃
- **逻辑僵化**：脚本无法处理非预期的弹窗、网络波动或业务逻辑的微小偏移
- **高额成本**：业内流传“写脚本 1 小时，维护脚本 24 小时”，导致很多公司的自动化率高开低走

### 1.3 自动化测试 + AI 的技术发展路径

#### 分支一：生成的自动化（Generative Automation）

**核心逻辑**：AI 充当“超级副驾驶”（Copilot、Trae、Cursor）

**技术表现**：

- **自然语言转脚本**：你说“写一个登录测试”，AI 生成一套完整的 Python + Playwright 代码
- **自愈合（Self-healing）** ：当定位符失效时，AI 扫描页面找到最相似的元素，并自动重写测试代码

**现状**：目前大厂（如字节、阿里内部测试平台）的主流升级方向，产出的依然是代码，可审、可控、性能高。

#### 分支二：代理的自动化（Agentic Automation）

**核心逻辑**：AI 充当“数字员工”

**技术表现**：

- **语义驱动**：脚本里没有具体的点击步骤，只有意图（Intent），如 `stagehand.act("买一张去上海的机票")`
- **实时推理（Reasoning）** ：AI 实时观察（Observe）当前网页，思考（Think）下一步点哪里，执行（Act）后再次观察

**现状**：几乎彻底抛弃了固定脚本，直接与业务意图对齐。代表工具：Browser-use、Stagehand 等。

### 1.4 与“自主化 Web 探索测试”的关系

自主化探索测试是自动化测试演进的高阶阶段，其中**代理自动化（Agentic Automation）** 构成了其底层技术基石。它彻底改变了测试范式：不再局限于验证“预设路径 A 是否通畅”，而是赋予 AI 真实用户的视角与决策力。在脱离“剧本”的情况下，AI 能够自主规划路径，通过“随机探索”与“深度遍历”发现传统自动化无法触及的边界缺陷。

### 1.5 自动化测试的技术进化阶段总结

| 阶段         | 核心驱动          | 行为模式                 | 主要目标     |
| ------------ | ----------------- | ------------------------ | ------------ |
| 传统自动化   | 脚本（Scripts）   | 线性验证（Linear Check） | 验证已知路径 |
| 生成式 AI    | 模板（Templates） | 辅助自愈（Self-healing） | 降低维护成本 |
| 自主化 Agent | 意图（Intent）    | 自主探索（Exploration）  | 发现未知风险 |


## 二、生成的自动化（Generative Automation）

**核心**：AI 生成脚本（Code Gen / Self-healing）

**优点**：

- 可控性高：代码逻辑确定
- 执行快：运行时不依赖大模型，速度与传统脚本无异

**痛点**：

- 代码依然需要维护，UI 改版代码也需要变
- 写代码/看代码有一定技术门槛

### 2.1 AI 代码编辑器（2026 主流）

| 工具名称             | 核心定位             | 底层模型                            | 上下文            | 离线/本地 | 关键能力                                                 | 价格                         |
| -------------------- | -------------------- | ----------------------------------- | ----------------- | --------- | -------------------------------------------------------- | ---------------------------- |
| Cursor               | AI 原生 IDE 标杆     | Claude 3.5 + GPT-4 双引擎           | 128K+，全工程索引 | 必须联网  | 多文件 Composer、全局重构、Agent、BugBot、图片转代码     | 个人 $20/月起，企业 $40/月起 |
| GitHub Copilot       | 生态型代码副驾驶     | GPT-5、Claude Sonnet 4              | 32K               | 联网      | 行/函数/文件级生成、Copilot Agent、PR 自动审查、性能分析 | 个人 $10/月起，企业 $19/月起 |
| Trae（字节）         | 多模型 AI 编辑器     | GPT-5.2、Gemini 3 Pro、豆包等可切换 | 128K              | 联网      | Builder 模式、中文友好、图像转代码、多模型自由切换       | 个人免费，企业版付费         |
| Windsurf             | 复杂工程/意图预判    | Codeium 自研                        | 全工程级          | 联网      | 流式意图预判、全工程测试、JetBrains 深度适配             | 个人免费，企业版付费         |
| Amazon CodeWhisperer | AWS 云原生助手       | 亚马逊自研                          | 项目级            | 联网      | 云代码生成、安全扫描、许可证合规、AWS 资源聊天           | 个人免费，企业付费           |
| Claude Code          | 超长上下文命令行工具 | Claude Opus 4.5                     | 200K+（≈15 万字） | 联网      | 200K+上下文、全项目一次性分析、命令行驱动                | 个人 $20/月起，企业版付费    |
| Tabnine              | 隐私优先本地助手     | 自研本地模型                        | 项目级            | 完全本地  | 纯本地运行、私有库定制、低延迟                           | 个人 $12/月起，企业 $39/月起 |
| 通义灵码（阿里）     | 阿里云生态+中文优化  | 通义千问 + DeepSeek                 | 64K               | 联网      | 私域知识库、阿里云 SDK 优化、工程级生成                  | 个人免费，企业版付费         |

### 2.2 快速选型建议

- 写代码、做复杂项目重构 → **Cursor**
- 用 GitHub、做开源/国际项目 → **GitHub Copilot**
- 中文开发、新手、免费优先 → **Trae / 通义灵码**
- AWS 云原生、合规优先 → **Amazon CodeWhisperer**

### 2.3 AI 智能体写自动化脚本

**Trae - The Real AI Engineer**

下载安装 Trae 后，可通过自然语言生成代码，示例提示词：

1. 基于 HTML、JavaScript、CSS 生成一个精美的 HTML 登录页面
2. 基于 Python 生成 AI 文本交互 Demo——智能问答机器人，调用 DeepSeek 的 API，API Key 留着让用户自己配置
3. 基于 Python + Requests 生成自动化测试脚本，验证网站（www.baidu.com）可用性

![第5页图片](./assets/p0001_03.png)

> 了解：选大模型、上下文、规则、技能、MCP 等配置。


## 三、AI 智能体 + MCP 写自动化脚本

### 3.1 MCP 介绍

**MCP（Model Context Protocol）** 是由 Anthropic 公司提出并开源的一个开放标准协议，于 2024 年 11 月 25 日正式发布。主要实现大语言模型和外部数据源、工具的集成，解决了大语言模型面临的数据孤岛问题，推动大语言模型应用的标准化。

![第5页图片](./assets/p0001_02.png)

**通俗理解**：

- MCP = 让 AI 能安全、统一、规范地调用各种外部工具/插件的“通用插座协议”
- AI 是大脑
- 浏览器、代码、搜索、自动化……是手、脚、眼睛
- MCP 就是神经链路

有了 MCP，AI 就可以控制手拿东西，控制眼睛看东西。

### 3.2 AI 编辑器 + Chrome DevTools MCP

**Chrome DevTools MCP** 是一种基于 Model Context Protocol（MCP）标准的连接器。它像一座“桥梁”，将 AI 助手（如 Trae、Claude、Cursor）与 Chrome 浏览器的开发者工具（DevTools）底层接口直接打通，赋予 AI “看”网页、“摸”网页以及“调”网页的能力。

> GitHub：[ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)

#### 安装依赖 & 配置

**（1）安装 Node.js（v20+ 或 v22+）**

从 [Node.js 官网](https://nodejs.org) 下载并运行安装程序，勾选 "Automatically install the necessary tools..."，其他选项保持默认。

![第6页图片](./assets/p0001_04.png)

**（2）验证 Node.js**

```bash
node -v
# 应输出 v20.x.x 或更高版本

npm -v
# 应输出 npm 版本
```

**（3）可能遇到的问题（可选）**

如果 `npm -v` 遇到问题（大概率因为 PowerShell 默认不允许运行脚本），需要以管理员身份运行 PowerShell 更改执行策略：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**（4）全局安装 Chrome DevTools MCP Server（可选）**

```bash
npm install -g @modelcontextprotocol/server-puppeteer --registry=https://registry.npmmirror.com
```

> 官方标准：在 MCP 官方代码仓库中，负责浏览器交互的模块以 puppeteer 命名。

**（5）在 Trae 中配置 Chrome DevTools MCP**

![第8页图片](./assets/p0001_06.png)

![第8页图片](./assets/p0001_07.png)

![第8页图片](./assets/p0001_05.png)

![第9页图片](./assets/p0001_08.png)

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

配置完成后，得到一个新的**智能体（with MCP）** 。

**（6）在 Trae 中使用 Chrome DevTools MCP**

切换智能体：

![第10页图片](./assets/p0001_09.png)

![第10页图片](./assets/p0001_10.png)

### 3.3 其他 MCP 介绍

#### 3.3.1 通过 MCP 连接 MySQL

在 Trae 设置中添加 MySQL MCP：

![第11页图片](./assets/p0001_06.png)

![第12页图片](./assets/p0001_11.png)

![第13页图片](./assets/p0001_12.png)

填写关键信息，自动生成 JSON。

**安装 uv 环境变量**（如果缺少环境变量）：

```bash
# 升级 pip 后再安装，避免 pip 版本过低导致安装失败
python -m pip install --upgrade pip
pip install uv --user
```

![第14页图片](./assets/p0001_14.png)

![第14页图片](./assets/p0001_15.png)

找到 uv.exe 和 uvx.exe 所在目录（如 `C:\Users\【用户名】\AppData\Roaming\Python\Python313\Scripts\`），添加到系统环境变量 Path 中：

![第14页图片](./assets/p0001_13.png)

![第15页图片](./assets/p0001_16.png)

![第15页图片](./assets/p0001_17.png)

![第16页图片](./assets/p0001_18.png)

![第16页图片](./assets/p0001_19.png)

![第17页图片](./assets/p0001_20.png)

新开一个 PowerShell，输入 `uv --version`，确认可以找到 uv。

![第17页图片](./assets/p0001_21.png)

重启 Trae，等待 MySQL MCP 服务安装并重启：

![第18页图片](./assets/p0001_22.png)

**验证 MySQL MCP 功能**：

![第19页图片](./assets/p0001_24.png)

![第19页图片](./assets/p0001_23.png)

**连接第二个数据库**：新建 MCP，“手动添加”，直接写 JSON 配置：

```json
{
  "mcpServers": {
    "MySQL-51ticket": {
      "command": "uvx",
      "args": [
        "--from",
        "mysql-mcp-server",
        "mysql_mcp_server"
      ],
      "env": {
        "MYSQL_HOST": "1.13.8.34",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "root",
        "MYSQL_PASSWORD": "Mysqlpassword2024",
        "MYSQL_DATABASE": "ticketonline"
      }
    }
  }
}
```

配置完成后即可拥有两个 MySQL 连接：

![第20页图片](./assets/p0001_25.png)

![第20页图片](./assets/p0001_26.png)

![第21页图片](./assets/p0001_27.png)

#### 3.3.2 通过 MCP 连接 Gitee

使用 Trae 的 Gitee MCP 必须先安装 Git 软件，且需配置环境变量 + 关联 Gitee 账号。

在 MCP 市场中添加 Gitee：

![第22页图片](./assets/p0001_28.png)

配置 Token：

![第23页图片](./assets/p0001_30.png)

**如何生成 Gitee Token（访问令牌）** ：

1. 登录 Gitee 账号
2. 进入“设置” > “私人令牌”
3. 点击“生成新令牌”，选择适当权限（如 repo 权限）
4. 复制生成的令牌并保存好（令牌只会显示一次）

![第23页图片](./assets/p0001_29.png)

![第24页图片](./assets/p0001_31.png)

**验证功能**：让 Trae 去 Gitee 克隆代码仓库（注意：必须触发 gitee 关键词，否则智能体不会调用该能力）

![第25页图片](./assets/p0001_33.png)

修改文件并让 Trae 提交：

![第25页图片](./assets/p0001_32.png)

![第26页图片](./assets/p0001_34.png)

推送到远程仓库：

![第26页图片](./assets/p0001_35.png)

![第27页图片](./assets/p0001_36.png)

![第27页图片](./assets/p0001_37.png)

![第27页图片](./assets/p0001_38.png)

### 3.4 多 AI 智能体协作

处理复杂任务时，单个智能体存在以下问题：

- 记不住太多东西（上下文有限）
- 什么都懂一点，但不精
- 任务一长就乱、就忘、就出错
- 很难同时做：规划、写代码、查 Bug、跑测试、调工具

**多 AI 智能体协作 = 主智能体（总指挥）+ 子智能体（专业小兵）**

一个主智能体管全局，一群子智能体专注干自己擅长的事。

**SOLO Coder** 是 Trae AI IDE 里专门管复杂项目迭代、重构、Bug 修复的“AI 技术负责人”，能自己做计划、拆任务、写代码、跑测试。

![第28页图片](./assets/p0001_39.png)

**练手任务**：在 Trae 中，用新的智能体（with MCP）结合 SOLO Coder 进行自动化测试：

```text
1. 从 gitee 仓库更新最新代码。
2. 修改文件 1.txt，在底部增加一行今天的天气信息。
3. 新建一个 python 文件，文件名：test.py。写一个 requests 代码，去请求百度，验证可访问。
4. 运行测试脚本 test.py，输出运行结果。如果运行结果正常，提交全部文件，推送到远程仓库。
```

![第29页图片](./assets/p0001_40.png)

### 3.5 Trae SOLO Coder（装备 MCP）测试项目实战

#### （1）打开百度，搜索“Playwright”并截图

**提示词**：

```text
基于 playwright for python，写一个自动化脚本 baidu_search_playwright.py：
1. 打开百度，
2. 找到输入框（id="chat-textarea"），填写"Playwright"
3. 点击按钮"百度一下"
4. 截图保存文件 baidu_result.png
```

![第30页图片](./assets/p0001_41.png)

#### （2）某电商平台商户端 3.0 登录

**提示词**：

```text
基于 playwright for python，写一个自动化脚本 merchant_login.py：
1. 打开 URL：https://c3vlzgxj.mule.page/
2. 输入用户名（admin），密码（admin123）。注意，用户名输入框的 placeholder 是"请输入用户名"，密码输入框是"请输入密码"。
3. 选择角色为"运营"。
4. 点击"登 录"
5. 判断登录后的 URL 是不是 "https://c3vlzgxj.mule.page/"
6. 判断登录后的 Title 是不是 "某电商平台商户端 3.0"
7. 判断当前角色是不是"运营"
```

![第31页图片](./assets/p0001_42.png)

#### （3）表单元素集览填写

**提示词**：

```text
基于 playwright for python，写一个自动化脚本 form_test.py：
1. 打开 URL：https://vanrevwk.mule.page/
2. 优先通过 label 的方式找到输入框。
3. 输入用户名(zhangsan)、电子邮箱(zhangsan@123.com)、个人简介（zhangsan 的个人简介123）
4. 性别点选"女"，付款方式点选"微信支付"，兴趣爱好点选"摄影、编程"。
5. 所在城市选择（单选框，id="city"）"南京"
6. 学历选择（单选框，id="education"）"本科"
7. 技能标签选择（多选框，id="skills"）"Java、Go、Rust"
8. 出生日期填写（日期框，id="birthday"）"2026/03/25"
9. 截图保存 form_test.png，选择 full_page 方式截图。
10. 点击按钮"提交表单"。
11. 脚本的操作动作放慢：slow_mo=1500。
```

![第32页图片](./assets/p0001_43.png)

#### （4）异步请求脚本

**提示词**：

```text
基于 playwright for python，写一个自动化脚本 async_test.py：
1. 通过异步的方式，访问多个网址，比较每个网址的访问时间，最后决出名次。
2. 网址：豆瓣网、京东、百度、淘宝、腾讯视频。
3. 所有任务完成后，计算并发总耗时。
```

![第33页图片](./assets/p0001_45.png)

![第33页图片](./assets/p0001_44.png)

![第34页图片](./assets/p0001_46.png)

#### （5）51ticket 网址测试

**准备工作**：新建 Gitee 代码仓库，本地 clone：`git@gitee.com:fu-laoshi2025/project_51tickets_stf108.git`

![第35页图片](./assets/p0001_47.png)

**登录页测试提示词**：

```text
playwright for python 依赖已经安装，无需重复安装。

基于 playwright for python，写一个自动化脚本 test_51tickets_login.py：
1. 打开 URL：http://1.13.8.34:7000/login
2. 优先使用 placeholder 来找输入框。
3. 输入账号（s001），密码（123456），验证码（1357）
4. 点击"登 录"
5. 断言，页面中是否包含文字"51testing欢迎你"
```

**演员展示页测试提示词**：

```text
playwright for python 依赖已经安装，无需重复安装。

基于 playwright for python，写一个自动化脚本 test_51tickets_actor_list.py：
1. 打开 URL：http://1.13.8.34:7000/login
2. 优先使用 placeholder 来找输入框。
3. 输入账号（s001），密码（123456），验证码（1357）
4. 点击"登 录"
5. 判断页面中是否有 text 内容是"业务管理"的 span。
6. 点击"业务管理"，然后点击下一层中的"演员信息"span。
7. 页面跳转后，验证当前 URL 是否是 "http://1.13.8.34:7000/business/actor"
```

> 选择高级版本的 Doubao-Seed-2.0-Code，成功率会更高。

![第36页图片](./assets/p0001_48.png)


## 四、代理的自动化（Agentic Automation）

**核心**：语义意图驱动（Agentic / Intent-based），前沿方向，不够稳定。

**优点**：

- **真正零维护**：UI 改版（如按钮从左边挪到右边，或 ID 变了）对意图脚本无影响
- **适应性极强**：能处理复杂的逻辑跳转

**痛点**：

- **黑盒化**：报错时可能难以调试
- **成本/速度**：运行过程中需要频繁调用 LLM，慢（等半天）且贵（消耗 Token）


---

*共 37 页，提取图片 48 张。*
