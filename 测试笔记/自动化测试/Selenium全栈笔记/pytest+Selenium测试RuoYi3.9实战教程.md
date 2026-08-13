# pytest + Selenium 实战教程：以 RuoYi 3.9（RuoYi‑Vue）为被测系统

> 适用对象：已经学过 Selenium 基础（元素定位、等待、PO 模式），想把一个**真实企业级后台系统**作为练手目标的测试工程师。
> 阅读后你将掌握：如何用 pytest + Selenium 4 对 **RuoYi‑Vue 3.9（前后端分离版）** 做工程化 UI 自动化，并覆盖「验证码绕过 / Element UI 组件定位 / iframe / 登录态复用 / 数据库关联校验 / 数据驱动 / 报告与 CI」等真实痛点。

---

## 目录

1. [前置认知：RuoYi 3.9 是什么、技术栈与架构](#1-前置认知ruoyi-39-是什么)
2. [被测系统的「测试可测性」分析（先搞懂再写脚本）](#2-被测系统的测试可测性分析)
3. [环境准备：Python / pytest / Selenium / 驱动](#3-环境准备)
4. [测试专用环境搭建（关验证码、建测试库、造测试账号）](#4-测试专用环境搭建)
5. [工程化目录结构（PO 模式 + OOP）](#5-工程化目录结构)
6. [核心工具层：BrowserFactory / WaitHelper / DBHelper / 日志 / 截图](#6-核心工具层)
7. [页面对象层：BasePage + 各业务页（重点：Element UI 定位）](#7-页面对象层)
8. [pytest fixtures 与 conftest：浏览器、登录态复用、DB、数据清理](#8-pytest-fixtures-与-conftest)
9. [分层测试用例实战](#9-分层测试用例实战)
10. [进阶：重试、并发、Allure 报告、CI 集成](#10-进阶重试并发allure报告ci集成)
11. [常见问题与排错（flaky / iframe / 异步渲染）](#11-常见问题与排错)
12. [附录：命令速查 & 参考](#12-附录)

---

## 1. 前置认知：RuoYi 3.9 是什么

若依（RuoYi）有多个发行版，版本号容易混淆，先对齐：

| 发行版 | 最近版本（官方站 2026‑03 数据） | 技术形态 |
|---|---|---|
| **RuoYi（不分离版）** | v4.8.3 | Spring Boot + Bootstrap + Thymeleaf（服务端渲染，**菜单用 iframe 嵌套**） |
| **RuoYi‑Vue（前后端分离版）** | **v3.9.2** | 前端 Vue2 + Element UI，后端 Spring Boot + API（**SPA，router‑view 渲染**） |
| RuoYi‑Cloud | v3.6.8 | Spring Cloud Alibaba 微服务 |

> **结论**：你提到的「RuoYi 3.9」指的就是 **RuoYi‑Vue 3.9.x 前后端分离版**。本教程全部以它为准。

### 1.1 技术栈一览（决定你的测试策略）

| 层 | 技术 | 对自动化测试的影响 |
|---|---|---|
| 前端 | **Vue 2.x + Element UI** | 页面是 SPA；输入框/表格/弹窗都是 Element UI 封装组件，定位要用「组件真实 DOM」而非表面标签 |
| 后端 | **Spring Boot + Spring Security** | 提供 RESTful API；默认开启 CSRF 防护（但分离版用 JWT，UI 表单无 CSRF token 困扰） |
| 认证 | **JWT（存于 cookie / Vuex）** | 登录态无 Session；pytest 可 session 级登录一次后复用，或用 `requests` 直接拿 token 做 API 层断言 |
| 缓存 | **Redis** | 验证码（Kaptcha）、限流、在线用户都依赖它 → 测试环境**必须跑 Redis**，否则登录报验证码错误 |
| ORM | **MyBatis + Druid** | 数据落库可读；测试可用 pymysql / SQLAlchemy 直连 `ry‑vue` 库做「UI 操作后查库校验」 |
| 数据库 | **MySQL（默认库名 `ry‑vue`）** | 核心表：`sys_user`、`sys_role`、`sys_user_role`、`sys_menu`、`sys_dept`… |
| 其他 | **Quartz、Swagger、Druid Monitor** | 定时任务、接口文档（Swagger）、连接池监控页也可纳入回归 |

### 1.2 登录认证流程（写登录脚本前必须懂）

```text
浏览器 → POST /login  {username, password, code, uuid}
        ← 返回 {code:200, token:"eyJ..."}   （JWT）
前端把 token 存进 cookie（RuoYi 用 cookie 名通常为 Admin-Token）
后续每个 XHR 请求在 header 带 Authorization: Bearer <token>
后端 Security 过滤器校验 JWT → 放行
```

**关键坑点**：

- 登录接口**必须带 `code`（验证码）+ `uuid`**。验证码图片来自 `GET /captchaImage`，返回 base64 图和 uuid，你提交时要带上这个 uuid，后端去 Redis 比对 code。
- 所以自动化登录**第一件事就是关掉验证码**（见第 4 章），否则你永远过不了 `/login`。

### 1.3 前端页面结构（决定你怎么导航）

RuoYi‑Vue 标准页面是 **Vue Router 渲染（`router‑view`），不是 iframe**。布局长这样：

```text
┌──────────────┬───────────────────────────────┐
│  .sidebar-    │  <section class="app-main">   │
│   container    │     <router-view/>  ← 内容区  │
│  侧边菜单树    │  </section>                  │
│  .el-menu     │                               │
│  .el-menu-item│  （用户管理/角色管理…都在这） │
└──────────────┴───────────────────────────────┘
```

> **例外**：当菜单类型为「外部/内部链接」时，RuoYi 会用 `<iframe>` 嵌入第三方地址（如监控页、Swagger）。本章第 7‑8 节会演示 iframe 切换代码，但**绝大多数业务页是 router‑view，无需切 iframe**。

---

## 2. 被测系统的「测试可测性」分析

动手前先列清单——哪些好测、哪些要特殊处理：

| 测试点 | 难度 | 处理方案 |
|---|---|---|
| 登录（正确/错误密码/验证码错误） | ★☆☆ | 关验证码后直接填表；错误提示断言 `.el-message` |
| 侧边菜单点击导航 | ★★☆ | 点击 `.el-menu-item` → 显式等待 `.el-table` / 目标文本出现 |
| 用户管理 CRUD | ★★★ | 表单在 `.el-dialog` 里，表格是 `.el-table`，删除走 `.el-message-box` 二次确认 |
| Element UI 组件定位 | ★★★ | 不能用表面 `<el-input>`，要下钻到 `.el-input__inner`；下拉要点开 `.el-select-dropdown` |
| 登录态复用 | ★★☆ | session 级 fixture 登录一次，cookie 自动随浏览器保持 |
| 数据库关联校验 | ★★☆ | 用 pymysql 直连 `ry‑vue`，断言 `sys_user` 行已写入 |
| 测试数据清理 | ★★☆ | teardown 里 `DELETE FROM sys_user WHERE user_name LIKE 'auto_%'` |
| 报告 / 并发 / CI | ★★☆ | pytest‑HTML / Allure / pytest‑xdist / GitHub Actions |

**设计原则（来自测试金字塔）**：UI 自动化（E2E）只覆盖**关键路径**（登录、核心 CRUD、权限），占比 ≤10%；大量逻辑校验用 API 层（`requests`）或单测更稳更快。本教程聚焦 UI，但第 9.5 节会演示「API 准备数据 + UI 验证」的混合打法。

---

## 3. 环境准备

### 3.1 版本要求

| 组件 | 推荐版本 | 说明 |
|---|---|---|
| Python | 3.9+ | 与 RuoYi 3.9 的「3.9」无直接关系，纯巧合 |
| pytest | ≥ 8.0 | 用例收集与 fixture |
| selenium | ≥ 4.10 | 4.x 内置 Selenium Manager，自动下载驱动 |
| 浏览器 | Chrome / Edge ≥ 110 | 推荐 Chrome |
| 数据库驱动 | pymysql ≥ 1.1 | 直连 `ry‑vue` 做校验 |
| 报告 | pytest‑HTML / allure‑pytest | 任选 |

### 3.2 安装

```bash
# 建议用虚拟环境
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install \
  pytest==8.2.0 \
  selenium==4.21.0 \
  pymysql==1.1.0 \
  pyyaml==6.0.1 \
  pytest-html==4.1.1 \
  pytest-rerunfailures==14.0 \
  pytest-xdist==3.6.1
```

> **驱动问题**：Selenium 4.6+ 自带 **Selenium Manager**，只要本机装了 Chrome，无需手动下 `chromedriver`，`webdriver.Chrome()` 会自动搞定。本教程一律用这种方式，告别「驱动版本不匹配」噩梦。

### 3.3 确认 RuoYi 已可访问

本教程假设你已部署好 RuoYi‑Vue 3.9（前端 `npm run dev` + 后端 `RuoYiApplication` 启动），访问地址例如 `http://localhost:81`（默认前端端口 80/81，后端 8080）。把地址记到配置文件（见 5‑2）。

---

## 4. 测试专用环境搭建

**不要拿生产库练手。** 做三件事：

### 4.1 关闭验证码（最重要）

编辑后端 `ruoyi-admin/src/main/resources/application.yml`：

```yaml
# 原配置（生产）
ruoyi:
  captchaEnabled: true

# 改成（测试环境）
ruoyi:
  captchaEnabled: false
```

改完后重启后端，登录接口将不再校验 `code`，你可以只传 `username/password` 登录。

> 更优雅的做法是用 **Spring Profile**：`application-test.yml` 里 `captchaEnabled: false`，启动加 `--spring.profiles.active=test`。这样生产与测试配置互不污染。

### 4.2 准备一个「测试专用库」

为安全起见，建议另建一个测试库（如 `ry_vue_test`）并导入同样的表结构 SQL（RuoYi 自带 `sql/ry_2021xxxx.sql` + `quartz.sql`）。然后：

```sql
-- 造一个专门用于自动化的账号（避免动 admin）
INSERT INTO sys_user (user_name, nick_name, password, status, del_flag, create_time)
VALUES ('auto_tester', 'Auto',
        '$2a$10$...BCrypt密文...',  -- 可用 RuoYi 自带的 admin 密文或自己生成
        '0', '0', NOW());
```

> 小技巧：直接复用演示账号 **`admin / admin123`** 作为自动化登录账号最省事；敏感操作（删数据）只在你用 `auto_` 前缀临时造的账号上做。

### 4.3 确认 Redis 在跑

```bash
redis-cli ping     # 应返回 PONG
```

Redis 没起来 → 后端登录会报「验证码已失效 / Redis 连接失败」。

---

## 5. 工程化目录结构

采用**页面对象模型（PO）**：页面结构变化只改 `pages/`，用例层保持稳定。

```text
ruoyi_auto/
├── config/
│   ├── config.yaml          # 环境配置（base_url、DB、账号）
│   └── config_loader.py     # 读取配置（支持 base_url / database.path 点路径）
├── utils/
│   ├── __init__.py
│   ├── browser.py           # BrowserFactory：按配置造 driver
│   ├── wait_helper.py       # WaitHelper：显式等待封装
│   ├── db_helper.py         # DBHelper：直连 MySQL 做校验
│   ├── logger.py            # 统一日志
│   └── screenshot.py       # 失败截图
├── pages/                   # ===== 页面对象层 =====
│   ├── base_page.py         # BasePage 基类
│   ├── login_page.py
│   ├── home_page.py
│   └── system/
│       ├── user_manage_page.py
│       └── role_page.py
├── testcases/
│   ├── conftest.py         # 全局 fixture（浏览器/登录/DB/清理）
│   ├── level1_login/
│   ├── level2_nav/
│   ├── level3_user_crud/
│   ├── level4_data_driven/
│   └── level5_db_verify/
├── data/                    # csv / yaml 测试数据
├── reports/                 # 测试报告
└── screenshots/             # 失败截图
```

### 5.1 配置 `config/config.yaml`

```yaml
base_url: "http://localhost:81"     # RuoYi 前端地址
browser: "chrome"
implicit_wait: 5
explicit_wait: 10

account:
  username: "admin"
  password: "admin123"

database:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "root"
  db: "ry_vue_test"          # 测试库

performance_baseline:
  load_time: 3.0
```

### 5.2 配置读取 `config/config_loader.py`

```python
import os
import yaml

_CONFIG = None

def _load():
    global _CONFIG
    if _CONFIG is None:
        path = os.path.join(os.path.dirname(__file__), "config.yaml")
        with open(path, "r", encoding="utf-8") as f:
            _CONFIG = yaml.safe_load(f)
    return _CONFIG

def get_config(key=None, default=None):
    """支持点路径：get_config('database.db')"""
    data = _load()
    if key is None:
        return data
    cur = data
    for part in key.split("."):
        if not isinstance(cur, dict) or part not in cur:
            return default
        cur = cur[part]
    return cur

def get_base_url():
    return get_config("base_url")
```

---

## 6. 核心工具层

### 6.1 BrowserFactory `utils/browser.py`

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from utils.config_loader import get_config


class BrowserFactory:
    @staticmethod
    def create(browser=None):
        browser = browser or get_config("browser", "chrome")
        if browser.lower() == "chrome":
            opts = Options()
            # 无头模式适合 CI；本地调试注释掉下面两行
            # opts.add_argument("--headless=new")
            opts.add_argument("--window-size=1440,900")
            opts.add_argument("--disable-dev-shm-usage")
            # 中文环境避免乱码
            opts.add_argument("--lang=zh-CN")
            return webdriver.Chrome(options=opts)
        raise ValueError(f"暂不支持的浏览器: {browser}")
```

### 6.2 WaitHelper `utils/wait_helper.py`（显式等待封装）

> 原则：**只用显式等待，禁用 `time.sleep`**。Element UI 是异步渲染，死等时间最容易导致 flaky。

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from utils.config_loader import get_config


class WaitHelper:
    def __init__(self, driver):
        self.driver = driver
        self.timeout = get_config("explicit_wait", 10)

    def element_visible(self, locator):
        return WebDriverWait(self.driver, self.timeout).until(
            EC.visibility_of_element_located(locator)
        )

    def element_clickable(self, locator):
        return WebDriverWait(self.driver, self.timeout).until(
            EC.element_to_be_clickable(locator)
        )

    def text_present(self, locator, text):
        return WebDriverWait(self.driver, self.timeout).until(
            EC.text_to_be_present_in_element(locator, text)
        )

    def frame_available_and_switch(self, locator):
        return WebDriverWait(self.driver, self.timeout).until(
            EC.frame_to_be_available_and_switch_to_it(locator)
        )
```

定位器建议统一用「元祖 `(By.X, '值')`」常量，集中管理：

```python
from selenium.webdriver.common.by import By

# 登录页
USERNAME = (By.NAME, "username")
PASSWORD = (By.NAME, "password")
LOGIN_BTN = (By.CSS_SELECTOR, "button.el-button--primary")
ERR_MSG = (By.CSS_SELECTOR, ".el-message")
```

> **选择器优先级（本教程铁律）**：`data-testid` > `id/name` > 语义角色/文本内容 > CSS class。
> RuoYi 默认没加 `data-testid`，所以优先用 `name="username"`、`name="password"`（RuoYi 登录框确实有这些 name 属性）、`placeholder`，最后才用 `.el-button--primary` 这类 class。

### 6.3 DBHelper `utils/db_helper.py`（数据库关联校验）

```python
import pymysql
from utils.config_loader import get_config


class DBHelper:
    def __init__(self):
        cfg = get_config("database")
        self.conn = pymysql.connect(
            host=cfg["host"], port=cfg["port"],
            user=cfg["user"], password=cfg["password"],
            database=cfg["db"], charset="utf8mb4",
            autocommit=True,
        )

    def query(self, sql, args=None):
        with self.conn.cursor(pymysql.cursors.DictCursor) as cur:
            cur.execute(sql, args)
            return cur.fetchall()

    def exists_user(self, username):
        rows = self.query(
            "SELECT 1 FROM sys_user WHERE user_name=%s AND del_flag='0'",
            (username,),
        )
        return len(rows) > 0

    def delete_user(self, username_prefix="auto_"):
        self.query(
            "UPDATE sys_user SET del_flag='2' WHERE user_name LIKE %s",
            (username_prefix + "%",),
        )

    def close(self):
        self.conn.close()
```

> 注意：RuoYi 的密码是 **BCrypt**（`$2a$` 开头），每次加密密文都不同，**不要**在测试里比对明文密码；只校验「行存在 / 状态 / 关联角色」即可。

---

## 7. 页面对象层

### 7.1 BasePage 基类 `pages/base_page.py`

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from utils.wait_helper import WaitHelper
from utils.config_loader import get_config


class BasePage:
    def __init__(self, driver):
        self.driver = driver
        self.wait = WaitHelper(driver)

    def open(self, path=""):
        base = get_config("base_url").rstrip("/")
        self.driver.get(f"{base}/{path.lstrip('/')}")
        return self

    def input_text(self, locator, text):
        el = self.wait.element_visible(locator)
        el.clear()
        el.send_keys(text)
        return self

    def click(self, locator):
        self.wait.element_clickable(locator).click()
        return self

    def get_text(self, locator):
        return self.wait.element_visible(locator).text

    def switch_to_frame(self, locator):
        self.wait.frame_available_and_switch(locator)
        return self

    def switch_to_default(self):
        self.driver.switch_to.default_content()
        return self
```

### 7.2 Element UI 组件定位速查（重点！）

RuoYi 前端全是 Element UI 封装，**表面标签 ≠ 真实可交互元素**：

| 组件 | 你看到的 | 真正要定位的 | 示例定位器 |
|---|---|---|---|
| 输入框 | `<el-input>` | 内部 `<input class="el-input__inner">` | `By.NAME,"username"`（RuoYi 已加 name）或 `(CSS,".el-input__inner")` |
| 密码框 | `<el-input type=password>` | `input[type=password]` | `By.NAME,"password"` |
| 主按钮 | `<el-button type=primary>` | `button.el-button--primary` | `(CSS,"button.el-button--primary")` |
| 普通按钮 | `<el-button>` | `button.el-button` | 配合文本：`//button[span='新增']` |
| 下拉选择 | `<el-select>` | 先点开，再选 `.el-select-dropdown__item` | 点 `(CSS,".el-select")` → 等 `.el-select-dropdown__item` |
| 表格 | `<el-table>` | 行 `.el-table__row`，单元格 `.cell` | `(CSS,".el-table__row")` |
| 弹窗 | `<el-dialog>` | `.el-dialog`，体内 `.el-dialog__body` | 断言 `(CSS,".el-dialog")` 可见 |
| 确认框 | `$confirm` | `.el-message-box`，按钮 `.el-button` | 点 `(CSS,".el-message-box .el-button--primary")` |
| 提示 | `$message` | `.el-message` | 断言文本含「用户名或密码错误」 |
| 菜单项 | `<el-menu-item>` | `.el-menu-item` | `(CSS,".el-menu-item")` 或按文本 |

### 7.3 登录页 `pages/login_page.py`

```python
from selenium.webdriver.common.by import By
from pages.base_page import BasePage

USERNAME = (By.NAME, "username")
PASSWORD = (By.NAME, "password")
CODE = (By.NAME, "code")            # 验证码（关掉后留空即可）
LOGIN_BTN = (By.CSS_SELECTOR, "button.el-button--primary")
ERR_MSG = (By.CSS_SELECTOR, ".el-message")


class LoginPage(BasePage):
    def open(self):
        return super().open("/login")

    def login(self, username, password, code=""):
        self.input_text(USERNAME, username) \
            .input_text(PASSWORD, password)
        if code:
            self.input_text(CODE, code)
        self.click(LOGIN_BTN)
        return self

    def error_text(self):
        return self.get_text(ERR_MSG)

    def is_logged_in(self):
        # 登录成功会跳转到首页，URL 含 /index 且出现用户名
        return "/index" in self.driver.current_url
```

### 7.4 首页 / 导航 `pages/home_page.py`

```python
from selenium.webdriver.common.by import By
from pages.base_page import BasePage

MENU_ITEMS = (By.CSS_SELECTOR, ".el-menu-item")
USER_DROPDOWN = (By.CSS_SELECTOR, ".avatar-container")


class HomePage(BasePage):
    def open(self):
        return super().open("/index")

    def click_menu_by_text(self, text):
        # 点侧边菜单项；RuoYi 用 router-view 渲染，无需切 iframe
        for item in self.driver.find_elements(*MENU_ITEMS):
            if item.text.strip() == text:
                item.click()
                return self
        raise ValueError(f"未找到菜单: {text}")

    def click_sub_menu(self, parent, child):
        # 有子菜单时先展开父级
        self.click_menu_by_text(parent)
        self.click_menu_by_text(child)
        return self
```

### 7.5 用户管理页 `pages/system/user_manage_page.py`

```python
from selenium.webdriver.common.by import By
from pages.base_page import BasePage

ADD_BTN = (By.XPATH, "//button[span='新增']")
DIALOG = (By.CSS_SELECTOR, ".el-dialog")
USERNAME_INPUT = (By.XPATH, "//div[@class='el-dialog__body']//input[@placeholder='请输入用户名称']")
NICK_INPUT = (By.XPATH, "//div[@class='el-dialog__body']//input[@placeholder='请输入用户昵称']")
DEPT_TREE = (By.CSS_SELECTOR, ".el-tree-node")
SUBMIT_BTN = (By.XPATH, "//div[@class='el-dialog__footer']//button[span='确定']")
TABLE_ROWS = (By.CSS_SELECTOR, ".el-table__row")
CONFIRM_BTN = (By.CSS_SELECTOR, ".el-message-box .el-button--primary")


class UserManagePage(BasePage):
    def open(self):
        return super().open("/system/user")

    def click_add(self):
        self.click(ADD_BTN)
        self.wait.element_visible(DIALOG)
        return self

    def fill_form(self, username, nick, dept_text="研发部门"):
        self.input_text(USERNAME_INPUT, username)
        self.input_text(NICK_INPUT, nick)
        # 部门是 el-tree 选择，点击对应树节点
        for node in self.driver.find_elements(*DEPT_TREE):
            if dept_text in node.text:
                node.click()
                break
        return self

    def submit(self):
        self.click(SUBMIT_BTN)
        return self

    def row_count(self):
        return len(self.driver.find_elements(*TABLE_ROWS))

    def delete_first_row(self):
        # 第一行操作列的删除按钮 → 二次确认
        first_row = self.driver.find_elements(*TABLE_ROWS)[0]
        del_btn = first_row.find_element(By.XPATH, ".//button[span='删除']")
        del_btn.click()
        self.click(CONFIRM_BTN)        # 确认 .el-message-box
        return self
```

---

## 8. pytest fixtures 与 conftest

`testcases/conftest.py` 是整套框架的「中枢」：负责起浏览器、登录态复用、开 DB、收尾清理。

```python
import os
import sys
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By

from utils.browser import BrowserFactory
from utils.db_helper import DBHelper
from pages.login_page import LoginPage
from pages.home_page import HomePage
from utils.config_loader import get_config

# ---- 让 pytest 能 import 到 utils/pages ----
ROOT = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.insert(0, ROOT)


@pytest.fixture(scope="session")
def driver():
    drv = BrowserFactory.create()
    drv.implicitly_wait(get_config("implicit_wait", 5))
    drv.maximize_window()
    yield drv
    drv.quit()


@pytest.fixture(scope="session")
def logged_in(driver):
    """session 级登录一次，全程复用同一浏览器登录态（JWT 存 cookie）。"""
    login = LoginPage(driver).open()
    acc = get_config("account")
    login.login(acc["username"], acc["password"])
    HomePage(driver).wait.element_visible((By.CSS_SELECTOR, ".el-menu"))
    yield driver
    # session 结束不强制登出；如需隔离可在此 driver.delete_all_cookies()


@pytest.fixture
def db():
    helper = DBHelper()
    yield helper
    helper.close()


@pytest.fixture(autouse=True)
def clean_auto_data(db):
    """每个用例前后清理 auto_ 前缀的测试数据，避免污染。"""
    yield
    db.delete_user("auto_")
```

> **为什么登录用 session 级？** RuoYi 是 JWT + cookie，浏览器不关就一直有效。session 级登录 → 几十个用例只登录 1 次，跑得快、也避开「频繁登录触发限流」。如果某个用例必须「未登录态」，单独用 `driver` fixture 再 `delete_all_cookies()` 即可。

---

## 9. 分层测试用例实战

### 9.1 Level 1：登录（正常 / 错误密码 / 错误验证码）

```python
# testcases/level1_login/test_login.py
import pytest
from pages.login_page import LoginPage
from pages.home_page import HomePage


def test_login_success(logged_in):
    # logged_in fixture 已登录；这里验证跳转到首页
    assert "/index" in logged_in.current_url


def test_login_wrong_password(driver):
    page = LoginPage(driver).open()
    page.login("admin", "wrong_pass_123")
    # RuoYi 用 $message.error 提示
    assert "密码" in page.error_text() or "错误" in page.error_text()


def test_login_empty_username(driver):
    page = LoginPage(driver).open()
    page.login("", "admin123")
    assert page.error_text() != ""
```

### 9.2 Level 2：菜单导航 + 内容区等待

```python
# testcases/level2_nav/test_nav.py
from selenium.webdriver.common.by import By
from pages.system.user_manage_page import UserManagePage


def test_nav_to_user_manage(logged_in):
    home = HomePage(logged_in)
    home.click_menu_by_text("系统管理")
    home.click_menu_by_text("用户管理")
    page = UserManagePage(logged_in)
    # 关键：等表格渲染出来（Element UI 走 API 异步加载）
    page.wait.element_visible((By.CSS_SELECTOR, ".el-table__row"))
    assert page.row_count() >= 1
```

### 9.3 Level 3：用户管理 CRUD

```python
# testcases/level3_user_crud/test_user_crud.py
from pages.system.user_manage_page import UserManagePage


def test_add_user(logged_in, db):
    page = UserManagePage(logged_in).open()
    page.click_add().fill_form("auto_alice", "Alice").submit()
    # 提交后断言 UI 表格出现新行 + 数据库落库
    assert db.exists_user("auto_alice")


def test_delete_user(logged_in, db):
    # 先造一个
    page = UserManagePage(logged_in).open()
    page.click_add().fill_form("auto_bob", "Bob").submit()
    before = page.row_count()
    page.delete_first_row()
    after = UserManagePage(logged_in).open().row_count()
    assert after == before - 1
```

### 9.4 Level 4：数据驱动（CSV / 参数化）

```python
# testcases/level4_data_driven/test_login_ddt.py
import csv, os
import pytest
from pages.login_page import LoginPage

ROOT = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
DATA = os.path.join(ROOT, "data", "login_cases.csv")

def load_cases():
    cases = []
    with open(DATA, newline="", encoding="utf-8") as f:
        for row in csv.DictReader(f):
            cases.append(pytest.param(
                row["username"], row["password"], row["expect_success"] == "True",
                id=row["case"],
            ))
    return cases

@pytest.mark.parametrize("username,password,expect_ok", load_cases())
def test_login_matrix(driver, username, password, expect_ok):
    page = LoginPage(driver).open()
    page.login(username, password)
    if expect_ok:
        assert page.is_logged_in()
    else:
        assert page.error_text() != ""
```

`data/login_cases.csv`：

```csv
case,username,password,expect_success
ok_admin,admin,admin123,True
wrong_pwd,admin,wrong,False
empty_user,,admin123,False
```

### 9.5 Level 5：API 准备 + UI 验证（混合打法，强烈推荐）

UI 造数据慢且易 flaky；更稳的姿势是 **用 `requests` 调 RuoYi 后端 API 准备好数据，再用 Selenium 只做「页面能看到 / 能操作」的验证**。

```python
# testcases/level5_db_verify/test_user_flow.py
import requests
from selenium.webdriver.common.by import By
from utils.config_loader import get_config
from pages.system.user_manage_page import UserManagePage

BASE = get_config("base_url")   # 例如 http://localhost:81


@pytest.fixture
def api_token():
    # 直接拿 JWT，绕开 UI 登录（后端 /login 接口）
    r = requests.post(f"{BASE}/login",
                      json={"username": "admin", "password": "admin123", "code": "", "uuid": ""})
    return r.json()["token"]


def test_user_visible_in_ui(logged_in, api_token):
    # 1) 用 API 创建用户（快、稳）
    headers = {"Authorization": f"Bearer {api_token}"}
    requests.post(f"{BASE}/system/user", headers=headers,
                  json={"userName": "auto_carol", "nickName": "Carol",
                        "password": "123456", "deptId": 100, "roleIds": [2]})
    # 2) 用 UI 验证列表确实能看到（这才是自动化要测的「前端渲染 + 权限」）
    page = UserManagePage(logged_in).open()
    names = [r.find_element(By.XPATH, ".//td[2]").text
             for r in page.driver.find_elements(*page.TABLE_ROWS)]
    assert "auto_carol" in names
```

---

## 10. 进阶：重试、并发、Allure 报告、CI 集成

### 10.1 失败自动重试（抗 flaky）

```bash
pip install pytest-rerunfailures
pytest --reruns 2 --reruns-delay 1 testcases/
```

对极个别偶发不稳定的用例，可局部标记：

```python
@pytest.mark.flaky(reruns=3)
def test_something_unstable(driver): ...
```

### 10.2 并发执行

```bash
pip install pytest-xdist
pytest -n 3 testcases/
```

> ⚠️ 注意：**session 级登录 fixture + 单浏览器** 与 `-n` 多进程天然冲突（每个 worker 是独立进程，各自起浏览器，互不共享 cookie）。要并发就得把登录做成「每个进程自己登录」，或改用 `pytest-parallel` 线程模式。练手阶段建议**先顺序跑通**，并发作为进阶优化。

### 10.3 报告：pytest‑HTML

```bash
pytest --html=reports/report.html --self-contained-html testcases/
```

### 10.4 报告：Allure（更专业）

```bash
pip install allure-pytest
pytest --alluredir=reports/allure testcases/
allure serve reports/allure     # 本地起服务看报告
```

### 10.5 CI 集成（GitHub Actions 示例）

```yaml
# .github/workflows/ui.yml
name: ruoyi-ui-tests
on: [push, pull_request]
jobs:
  ui:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8
        env: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: ry_vue_test }
      redis:
        image: redis:7
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.11" }
      - run: pip install -r requirements.txt
      - run: pytest --html=reports/report.html
```

---

## 11. 常见问题与排错

| 现象 | 根因 | 解决 |
|---|---|---|
| 登录一直报「验证码错误/失效」 | 验证码没关，或 Redis 没起 | `captchaEnabled:false` + `redis-cli ping` |
| `NoSuchElementException` 在 el-input | 定位到了 `<el-input>` 而非内部 `<input>` | 改用 `.el-input__inner` / `name="username"` |
| 表格断言时行数为 0 | 点了菜单但 API 还没返回，表格未渲染 | 显式等待 `.el-table__row` 可见，**别用 sleep** |
| 弹窗里元素点不到 | 对话框是 `position:fixed` 浮层，但页面有动画未结束 | `element_to_be_clickable` + 适当超时 |
| “StaleElementReference” | Vue 重新渲染，旧元素失效 | 重新 `find_element` 再操作，不要在循环外缓存元素 |
| 并发跑报端口/cookie 冲突 | session 级登录 fixture 与 xdist 多进程冲突 | 改用每进程登录，或先顺序跑 |
| 中文乱码 / 截图缺字 | 容器/Linux 缺中文字体 | 安装 `fonts-noto-cjk`；本地 Windows 一般无此问题 |
| iframe 里元素找不到 | 菜单是「外链」类型，用了 `<iframe>` | `driver.switch_to.frame(...)` 操作完 `switch_to.default_content()` |

**flaky 测试三板斧**：① 用显式等待替代 sleep；② 选择器优先 `name`/`data-testid`；③ 不稳定的用例加 `--reruns`，同时记进 TODO 根因分析，别只靠重试掩盖问题。

---

## 12. 附录

### 12.1 常用命令速查

```bash
# 跑全量
pytest testcases/ -v

# 跑单层
pytest testcases/level1_login/ -v

# 失败重试 + HTML 报告
pytest testcases/ --reruns 2 --html=reports/report.html --self-contained-html

# 只收集不执行（验证用例能被发现）
pytest testcases/ --collect-only

# 指定标记
pytest testcases/ -m smoke
```

### 12.2 RuoYi 3.9 核心数据表（做 DB 校验时用）

| 表 | 作用 | 常用断言字段 |
|---|---|---|
| `sys_user` | 用户 | `user_name`, `nick_name`, `status`, `del_flag` |
| `sys_role` | 角色 | `role_name`, `role_key` |
| `sys_user_role` | 用户‑角色关联 | `user_id`, `role_id` |
| `sys_menu` | 菜单/权限 | `menu_name`, `perms` |
| `sys_dept` | 部门（树） | `dept_name`, `parent_id`, `ancestors` |

### 12.3 选择器优先级（铁律）

```text
data-testid  >  id / name  >  语义角色 / 文本  >  CSS class（最后选）
```

> 进阶建议：如果你们团队**拥有 RuoYi 前端代码**，最稳的做法是在 Vue 组件里给关键元素加 `data-testid`（改 `UserLogin.vue`、`user/index.vue` 等），从此告别脆弱的 `.el-button--primary` 定位。自动化测试的稳定性，80% 取决于前端是否「对测试友好」。

### 12.4 参考

- RuoYi 官方文档：<http://doc.ruoyi.vip>
- RuoYi‑Vue 源码（v3.9.x）：<https://gitee.com/y_project/RuoYi-Vue>
- 在线演示（可手动探索元素结构）：<http://vue.ruoyi.vip> （admin / admin123）
- Selenium 4 文档：<https://www.selenium.dev/documentation/>
- pytest 文档：<https://docs.pytest.org/>

---

> **小结**：本教程把 RuoYi 3.9（Vue2 + Element UI + Spring Boot + JWT/Redis/MySQL）当作真实被测系统，落地了一套 pytest + Selenium 4 的工程化 UI 自动化框架：工具层（BrowserFactory / WaitHelper / DBHelper）→ 页面对象层（BasePage + 业务页，含 Element UI 定位速查）→ conftest（登录态复用 + 数据清理）→ 5 个梯度用例（登录 / 导航 / CRUD / 数据驱动 / DB 关联 / API+UI 混合）→ 重试、并发、Allure、CI。照着目录把代码补全即可直接对自己的 RuoYi 项目开跑。
