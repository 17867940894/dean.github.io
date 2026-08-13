## 简介

Playwright 提供了一个用于编写端到端测试的 [Pytest](https://pytest.cn/en/stable/) 插件。要开始使用它，请参考[入门指南](https://playwright.cn/python/docs/intro)。

## 用法

要运行测试，请使用 [Pytest](https://pytest.cn/en/stable/) CLI。

```bash
pytest --browser webkit --headed
```

如果你想自动添加 CLI 参数而无需显式指定它们，可以使用 [pytest.ini](https://pytest.cn/en/stable/reference.html#ini-options-ref) 文件

```ini
# content of pytest.ini

[pytest]

# Run firefox with UI

addopts = --headed --browser firefox
```

## CLI 参数

请注意，CLI 参数仅应用于默认的 `browser`、`context` 和 `page` 夹具。如果你通过 API 调用（例如 [browser.new_context()](https://playwright.cn/python/docs/api/class-browser#browser-new-context)）创建浏览器、context 或 page，则这些 CLI 参数将不适用。

- `--headed`：以有头模式运行测试（默认：无头模式）。
- `--browser`：在不同的浏览器中运行测试：`chromium`、`firefox` 或 `webkit`。可以指定多次（默认：`chromium`）。
- `--browser-channel`：要使用的[浏览器渠道 (Browser channel)](https://playwright.cn/python/docs/browsers)。
- `--slowmo`：将 Playwright 操作减慢指定的毫秒数。这对于观察正在发生的事情非常有用（默认：0）。
- `--device`：要模拟的[设备 (Device)](https://playwright.cn/python/docs/emulation)。
- `--output`：测试产生的文件产物的目录（默认：`test-results`）。
- `--tracing`：是否为每个测试录制[追踪 (trace)](https://playwright.cn/python/docs/trace-viewer)。可选值为 `on`、`off` 或 `retain-on-failure`（默认：`off`）。
- `--video`：是否为每个测试录制视频。可选值为 `on`、`off` 或 `retain-on-failure`（默认：`off`）。
- `--screenshot`：是否在每个测试后自动截屏。可选值为 `on`、`off` 或 `only-on-failure`（默认：`off`）。
- `--full-page-screenshot`：失败时是否截取全屏截图。默认情况下，只捕获视口。需要启用 `--screenshot`（默认：`off`）。

## 夹具

此插件为 pytest 配置了 Playwright 专用的[夹具](https://pytest.cn/en/latest/fixture.html)。要使用这些夹具，只需将夹具名称作为参数传递给测试函数即可。

```py
def test_my_app_is_working(fixture_name):
    pass
    # Test using fixture_name
    # ...
```

**函数作用域 (Function scope)**：这些夹具在测试函数中被请求时创建，并在测试结束时销毁。

- `context`：用于测试的新 [浏览器上下文](https://playwright.cn/python/docs/browser-contexts)。
- `page`：用于测试的新 [浏览器页面](https://playwright.cn/python/docs/pages)。
- `new_context`：允许为测试创建不同的 [浏览器上下文](https://playwright.cn/python/docs/browser-contexts)。对于多用户场景非常有用。接受与 [browser.new_context()](https://playwright.cn/python/docs/api/class-browser#browser-new-context) 相同的参数。

**会话作用域 (Session scope)**：这些夹具在测试函数中被请求时创建，并在所有测试结束时销毁。

- `playwright`：[Playwright](https://playwright.cn/python/docs/api/class-playwright) 实例。
- `browser_type`：当前浏览器的 [BrowserType](https://playwright.cn/python/docs/api/class-browsertype) 实例。
- `browser`：由 Playwright 启动的 [Browser](https://playwright.cn/python/docs/api/class-browser) 实例。
- `browser_name`：字符串形式的浏览器名称。
- `browser_channel`：字符串形式的浏览器渠道。
- `is_chromium`、`is_webkit`、`is_firefox`：对应浏览器类型的布尔值。

**自定义夹具选项**：对于 `browser` 和 `context` 夹具，可以使用以下夹具来定义自定义启动选项。

- `browser_type_launch_args`：覆盖 [browser_type.launch()](https://playwright.cn/python/docs/api/class-browsertype#browser-type-launch) 的启动参数。它应该返回一个字典 (Dict)。
- `browser_context_args`：覆盖 [browser.new_context()](https://playwright.cn/python/docs/api/class-browser#browser-new-context) 的选项。它应该返回一个字典 (Dict)。
- `connect_options`：通过 WebSocket 端点连接到现有浏览器。它应该返回一个包含 [browser_type.connect()](https://playwright.cn/python/docs/api/class-browsertype#browser-type-connect) 选项的字典 (Dict)。

也可以通过使用 `browser_context_args` 标记为单个测试覆盖上下文选项（[browser.new_context()](https://playwright.cn/python/docs/api/class-browser#browser-new-context)）

```python
import pytest



@pytest.mark.browser_context_args(timezone_id="Europe/Berlin", locale="en-GB")

def test_browser_context_args(page):

    assert page.evaluate("window.navigator.userAgent") == "Europe/Berlin"

    assert page.evaluate("window.navigator.languages") == ["de-DE"]
```

## 并行性：同时运行多个测试

如果你的测试运行在拥有多个 CPU 的机器上，你可以使用 [`pytest-xdist`](https://pypi.ac.cn/project/pytest-xdist/) 同时运行多个测试，从而加快测试套件的整体执行时间

```bash
# install dependency

pip install pytest-xdist

# use the --numprocesses flag

pytest --numprocesses auto
```

根据硬件和测试的性质，你可以将 `numprocesses` 设置为从 `2` 到机器上 CPU 数量之间的任意值。如果设置得太高，可能会出现意想不到的行为。

有关 `pytest` 选项的一般信息，请参阅[运行测试](https://playwright.cn/python/docs/running-tests)。

## 示例

### 为自动补全配置类型提示

test_my_application.py

```py
from playwright.sync_api import Page



def test_visit_admin_dashboard(page: Page):

    page.goto("/admin")

    # ...
```

如果你正在使用带有 Pylance 的 VSCode，可以通过启用 `python.testing.pytestEnabled` 设置来推断这些类型，因此你不需要类型注解。

### 使用多个上下文

为了模拟多个用户，你可以创建多个 [`BrowserContext`](https://playwright.cn/python/docs/browser-contexts) 实例。

test_my_application.py

```py
from playwright.sync_api import Page, BrowserContext

from pytest_playwright.pytest_playwright import CreateContextCallback



def test_foo(page: Page, new_context: CreateContextCallback) -> None:

    page.goto("https://example.com")

    context = new_context()

    page2 = context.new_page()

    # page and page2 are in different contexts
```

### 按浏览器跳过测试

test_my_application.py

```py
import pytest



@pytest.mark.skip_browser("firefox")

def test_visit_example(page):

    page.goto("https://example.com")

    # ...
```

### 在特定浏览器上运行

conftest.py

```py
import pytest



@pytest.mark.only_browser("chromium")

def test_visit_example(page):

    page.goto("https://example.com")

    # ...
```

### 使用自定义浏览器渠道运行（如 Google Chrome 或 Microsoft Edge）

```bash
pytest --browser-channel chrome
```

test_my_application.py

```python
def test_example(page):

    page.goto("https://example.com")
```

### 配置 base-url

使用 `base-url` 参数启动 Pytest。为此使用了 [`pytest-base-url`](https://github.com/pytest-dev/pytest-base-url) 插件，它允许你从配置、CLI 参数或作为夹具来设置基础 URL。

```bash
pytest --base-url https://:8080
```

test_my_application.py

```py
def test_visit_example(page):

    page.goto("/admin")

    # -> Will result in https://:8080/admin
```

### 忽略 HTTPS 错误

conftest.py

```py
import pytest



@pytest.fixture(scope="session")

def browser_context_args(browser_context_args):

    return {

        **browser_context_args,

        "ignore_https_errors": True

    }
```

### 使用自定义视口尺寸

conftest.py

```py
import pytest



@pytest.fixture(scope="session")

def browser_context_args(browser_context_args):

    return {

        **browser_context_args,

        "viewport": {

            "width": 1920,

            "height": 1080,

        }

    }
```

### 设备模拟 / BrowserContext 选项覆盖

conftest.py

```py
import pytest



@pytest.fixture(scope="session")

def browser_context_args(browser_context_args, playwright):

    iphone_11 = playwright.devices['iPhone 11 Pro']

    return {

        **browser_context_args,

        **iphone_11,

    }
```

或者通过 CLI `--device="iPhone 11 Pro"`

### 连接到远程浏览器

conftest.py

```py
import pytest



@pytest.fixture(scope="session")

def connect_options():

    return {

        "wsEndpoint": "ws://:8080/ws"

    }
```

### 与 `unittest.TestCase` 一起使用

请参考以下与 `unittest.TestCase` 结合使用的示例。这有一个限制：只能指定单个浏览器，并且在指定多个浏览器时不会生成多个浏览器的矩阵。

```py
import pytest

import unittest



from playwright.sync_api import Page





class MyTest(unittest.TestCase):

    @pytest.fixture(autouse=True)

    def setup(self, page: Page):

        self.page = page



    def test_foobar(self):

        self.page.goto("https://microsoft.com")

        self.page.locator("#foobar").click()

        assert self.page.evaluate("1 + 1") == 2
```

## 调试

### 与 pdb 一起使用

在测试代码中使用 `breakpoint()` 语句来暂停执行并获取 [pdb](https://docs.pythonlang.cn/3/library/pdb.html) REPL。

```py
def test_bing_is_working(page):

    page.goto("https://bing.com")

    breakpoint()

    # ...
```

## 部署到 CI

请参阅[CI 提供商指南](https://playwright.cn/python/docs/ci)将测试部署到 CI/CD。

## 异步夹具

要使用异步夹具，请安装 [`pytest-playwright-asyncio`](https://pypi.ac.cn/project/pytest-playwright-asyncio/)。

确保你使用的是 `pytest-asyncio>=0.26.0`，并在你的配置文件（`pytest.ini/pyproject.toml/setup.cfg`）中设置 [`asyncio_default_test_loop_scope = session`](https://pytest-asyncio.readthedocs.io/en/v0.26.0/how-to-guides/change_default_test_loop.html)。

```python
import pytest

from playwright.async_api import Page



@pytest.mark.asyncio(loop_scope="session")

async def test_foo(page: Page):

    await page.goto("https://github.com")

    # ...
```
