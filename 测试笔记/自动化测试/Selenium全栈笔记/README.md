---
title: Selenium 全栈笔记 总索引
tags:
  - selenium
  - 索引
  - 知识库
  - MOC
aliases:
  - Selenium MOC
  - Selenium 知识地图
date created: 2026-07-17
---

# Selenium 全栈笔记 · 总索引

> [!info] 这是什么
> 一套适配 Obsidian 的 Selenium 自动化测试完整知识库：**基础笔记 8 篇 + 进阶笔记 11 篇 + 完整实战项目**。所有笔记互相双链，可直接导入 Obsidian 作为知识库。

## 模块导览

| 模块 | 内容 | 适用人群 |
| --- | --- | --- |
| `01-基础笔记/` | Selenium 入门到能写脚本 | 零基础 / 初学者 |
| `02-进阶笔记/` | 工程化、封装、复杂场景 | 有基础，想进阶 |
| `03-自动化实战项目/` | 完整可运行的 Python+Selenium 项目 | 想直接落地项目 |

---

## 一、基础笔记（8 篇）

| # | 笔记 | 核心知识点 |
| --- | --- | --- |
| 01 | [[01-环境搭建]] | 原理、安装、Selenium 4 变化 |
| 02 | [[02-浏览器驱动]] | Chrome/Edge/Firefox 驱动、镜像源、无头模式 |
| 03 | [[03-元素定位八大方式]] | ID/Name/Class/Tag/Link/CSS/XPath、F12 验证 |
| 04 | [[04-基础操作]] | 元素操作、浏览器操作、Cookie、窗口句柄 |
| 05 | [[05-等待机制]] | 强制/隐式/显式等待、`expected_conditions` |
| 06 | [[06-弹窗iframe窗口切换]] | Alert、iframe、多窗口、`switch_to` |
| 07 | [[07-鼠标键盘基础操作]] | `ActionChains`、`Keys`、组合键 |
| 08 | [[08-简单脚本案例]] | 6 个由浅入深的实战案例 |

## 二、进阶笔记（11 篇）

| # | 笔记 | 核心知识点 |
| --- | --- | --- |
| 01 | [[01-显式等待封装]] | `WaitHelper` 工具类、容错版等待 |
| 02 | [[02-PO模式]] | Page Object 分层、`BasePage` 设计 |
| 03 | [[03-多浏览器兼容]] | 浏览器工厂、配置切换、防检测 |
| 04 | [[04-文件上传下载]] | input/非 input 上传、下载路径配置 |
| 05 | [[05-截图日志]] | `logging` 封装、截图工具、失败自动截图 |
| 06 | [[06-JS执行]] | `execute_script`、滚动/隐藏元素/富文本 |
| 07 | [[07-滑块验证码处理]] | 轨迹模拟、OpenCV 缺口识别 |
| 08 | [[08-并发测试]] | pytest-xdist 多进程、多线程并发 |
| 09 | [[09-参数化与数据驱动]] | `parametrize`、CSV/Excel/YAML |
| 10 | [[10-异常捕获]] | 异常体系、重试装饰器、统一处理 |
| 11 | [[11-页面性能获取]] | `performance.timing`、CDP、Web Vitals |

## 三、自动化实战项目

完整可运行的 Python + Selenium 4 + pytest + PO 模式项目。

### 项目结构

```
03-自动化实战项目/
├── README.md                    # 运行说明
├── 报错解决方案.md              # FAQ 排查手册
├── requirements.txt             # 依赖清单
├── pytest.ini / conftest.py     # pytest 配置与夹具
├── run.py                       # 统一执行入口
├── config/config.yaml           # 环境配置
├── utils/                       # 工具层（7 个工具类）
│   ├── browser.py               # 浏览器工厂
│   ├── wait_helper.py           # 显式等待封装
│   ├── screenshot.py            # 截图工具
│   ├── logger.py                # 日志工具
│   ├── excel_reader.py          # Excel 读取
│   ├── js_helper.py             # JS 执行工具
│   └── retry.py / config_loader.py
├── pages/                       # PO 分层
│   ├── base_page.py             # 页面基类
│   ├── login_page.py            # 登录页
│   ├── home_page.py             # 首页
│   └── search_page.py           # 搜索页
├── testcases/                   # 测试用例
│   ├── test_login.py            # 登录用例
│   ├── test_search.py           # 搜索用例
│   ├── test_home.py             # 首页用例
│   └── test_data_driven.py      # 数据驱动用例
├── data/                        # 测试数据
├── reports/                     # 测试报告
├── screenshots/                 # 失败截图
└── logs/                        # 运行日志
```

### 快速运行

```bash
cd "Selenium全栈笔记/03-自动化实战项目"
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
python data/generate_test_data.py
python run.py                    # 跑全部
python run.py --mode smoke       # 只跑冒烟
python run.py --parallel 4 --html  # 4 进程并发 + HTML 报告
```

详见 `03-自动化实战项目/README.md`（待创建）。

## 四、学习路径建议

```
零基础 → 01-基础笔记 全部过一遍（重点 03 定位、05 等待）
  ↓
能写脚本 → 08 简单脚本案例 6 个全跑通
  ↓
想工程化 → 02-进阶笔记（重点 02 PO、05 截图日志、10 异常）
  ↓
要落地 → 03 实战项目 跑起来 → 替换 pages/ 为真实系统
  ↓
要并发 → 08 并发测试 + 09 数据驱动
  ↓
要性能 → 11 页面性能获取
```

## 五、关键概念速查

| 概念 | 笔记 | 一句话 |
| --- | --- | --- |
| 元素定位 | [[03-元素定位八大方式]] | ID > CSS > XPath |
| 等待 | [[05-等待机制]] | 生产用显式等待，别用 sleep |
| PO 模式 | [[02-PO模式]] | 页面 = 类，定位器集中管理 |
| 显式等待封装 | [[01-显式等待封装]] | `WaitHelper.element_visible(loc)` |
| 多浏览器 | [[03-多浏览器兼容]] | `BrowserFactory.create(browser='chrome')` |
| 数据驱动 | [[09-参数化与数据驱动]] | `@pytest.mark.parametrize` + Excel |
| 异常处理 | [[10-异常捕获]] | 精准捕获 + 重试装饰器 |
| 并发 | [[08-并发测试]] | `pytest -n 4 --dist=loadscope` |

## 六、与已有笔记的关系

本套笔记与原有 `测试笔记/自动化测试/Selenuim/` 下的入门笔记互补：

- 原有笔记：白月黑羽教程的零散片段（选择元素、CSS、XPath、frame）
- 本套笔记：体系化的知识库 + 可运行项目

建议在 Obsidian 中：

1. 把 `Selenium全栈笔记/` 作为主知识库
2. 原有 `Selenuim/` 笔记保留作参考，用双链互相引用
3. 用本索引（MOC）作为入口

## 七、Obsidian 使用建议

1. **打开双链**：所有 `[[...]]` 都可点击跳转
2. **图谱视图**：能看到笔记间的引用关系
3. **标签筛选**：用 `#selenium` `#PO模式` 等标签快速筛选
4. **搜索**：Ctrl+Shift+F 全库搜索
5. **建议插件**：
   - Templater：快速新建笔记模板
   - Dataview：生成动态笔记索引
   - Admonition：美化 callout 块
