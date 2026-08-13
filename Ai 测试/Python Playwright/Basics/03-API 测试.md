## 简介

Playwright 可用于访问您应用程序的 [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) API。

有时您可能希望直接从 Python 向服务器发送请求，而无需在其中加载页面和运行 js 代码。以下是一些它派上用场的例子

- 测试您的服务器 API。
- 在测试中访问 Web 应用程序之前准备服务器端状态。
- 在浏览器中执行一些操作后验证服务器端后置条件。

所有这些都可以通过 [APIRequestContext](https://playwright.cn/python/docs/api/class-apirequestcontext) 方法实现。

以下示例依赖于 [`pytest-playwright`](https://playwright.cn/python/docs/test-runners) 软件包，该软件包为 Pytest 测试运行器添加了 Playwright 夹具。

## 编写 API 测试

[APIRequestContext](https://playwright.cn/python/docs/api/class-apirequestcontext) 可以通过网络发送各种 HTTP(S) 请求。

以下示例演示了如何使用 Playwright 通过 [GitHub API](https://githubdocs.cn/en/rest) 测试问题的创建。测试套件将执行以下操作：

- 在运行测试之前创建一个新存储库。
- 创建一些问题并验证服务器状态。
- 运行测试后删除存储库。

### 配置

GitHub API 需要授权，因此我们将为所有测试配置一次令牌。同时，我们还将设置 `baseURL` 以简化测试。

```python
import os
from typing import Generator

import pytest
from playwright.sync_api import Playwright, APIRequestContext

GITHUB_API_TOKEN = os.getenv("GITHUB_API_TOKEN")
assert GITHUB_API_TOKEN, "GITHUB_API_TOKEN is not set"


@pytest.fixture(scope="session")
def api_request_context(
    playwright: Playwright,
) -> Generator[APIRequestContext, None, None]:
    headers = {
        # We set this header per GitHub guidelines.
        "Accept": "application/vnd.github.v3+json",
        # Add authorization token to all requests.
        # Assuming personal access token available in the environment.
        "Authorization": f"token {GITHUB_API_TOKEN}",
    }
    request_context = playwright.request.new_context(
        base_url="https://api.github.com", extra_http_headers=headers
    )
    yield request_context
    request_context.dispose()
```

### 编写测试

现在我们已经初始化了请求对象，我们可以添加一些测试，这些测试将在仓库中创建新的问题。

```python
import os
import pytest
from playwright.sync_api import Playwright

# 从环境变量读取令牌，如果未设置则报错
GITHUB_API_TOKEN = os.getenv("GITHUB_API_TOKEN")
assert GITHUB_API_TOKEN, "错误：请设置 GITHUB_API_TOKEN 环境变量"

# 仓库名（保持为测试专用）。不要硬编码用户名：使用认证token对应的账户登录名
GITHUB_REPO = "api-test-repo"  # 用于测试的仓库名，脚本会自动创建和删除


@pytest.fixture(scope="session")
def api_context(playwright: Playwright):
    """创建一个带认证的API请求上下文，所有测试共享"""
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"token {GITHUB_API_TOKEN}",
    }
    # 设置基础URL，后续请求只需写路径
    context = playwright.request.new_context(
        base_url="https://api.github.com", extra_http_headers=headers
    )
    yield context
    context.dispose()


@pytest.fixture(scope="session")
def auth_user(api_context):
    """返回与 GITHUB_API_TOKEN 关联的登录名（login）"""
    user_resp = api_context.get("/user")
    assert user_resp.ok, "获取认证用户信息失败，请检查 GITHUB_API_TOKEN 是否正确"
    login = user_resp.json().get("login")
    assert login, "无法从 /user 接口获取登录名"
    return login


@pytest.fixture(scope="session", autouse=True)
def setup_and_cleanup(api_context, auth_user):
    """在所有测试前后自动创建和删除测试仓库"""
    # --- 前置设置：获取认证用户并创建仓库 ---
    new_repo = api_context.post("/user/repos", data={"name": GITHUB_REPO})
    assert new_repo.ok, f"创建测试仓库失败: {new_repo.status} {new_repo.status_text}"
    print(f"\n✅ 测试仓库 '{GITHUB_REPO}' 已为用户 '{auth_user}' 创建")

    yield  # 此处运行所有测试

    # --- 后置清理：删除仓库（使用认证用户的登录名） ---
    delete_repo = api_context.delete(f"/repos/{auth_user}/{GITHUB_REPO}")
    assert delete_repo.ok, f"删除测试仓库失败: {delete_repo.status} {delete_repo.status_text}"
    print(f"\n🧹 测试仓库 '{GITHUB_REPO}'（用户 '{auth_user}'）已清理")


def test_can_create_an_issue(api_context, auth_user):
    """测试：能否通过API成功创建一个Issue"""
    # 1. 创建一个新Issue
    issue_data = {
        "title": "测试Issue - 来自Playwright",
        "body": "这是一个通过API自动创建的测试问题。"
    }
    # 使用 /repos/{owner}/{repo}/issues 创建 Issue，优先使用创建接口的返回值进行验证
    new_issue_resp = api_context.post(
        f"/repos/{auth_user}/{GITHUB_REPO}/issues",
        data=issue_data
    )
    # 断言：请求成功 (状态码 201 Created)
    assert new_issue_resp.ok, f"创建Issue失败: {new_issue_resp.status} {new_issue_resp.status_text}"
    created_issue = new_issue_resp.json()
    print("\n📝 Issue创建请求成功 (由创建接口返回)")

    # 验证创建接口返回的Issue字段
    assert created_issue.get("title") == issue_data["title"], "Issue标题不匹配"
    assert created_issue.get("body") == issue_data["body"], "Issue的Body内容不匹配"
    print(f"✅ 验证通过！Issue标题: '{created_issue['title']}' 已成功创建并验证。")
```

### 设置与清理

这些测试假设仓库已存在。您可能希望在运行测试之前创建一个新仓库，并在之后将其删除。为此，请使用 [会话级夹具 (session fixture)](https://pytest.cn/en/stable/how-to/fixtures.html#fixture-scopes)。`yield` 之前的部分是 before all，之后的部分是 after all。

```python
# ...
@pytest.fixture(scope="session", autouse=True)
def create_test_repository(
    api_request_context: APIRequestContext,
) -> Generator[None, None, None]:
    # Before all
    new_repo = api_request_context.post("/user/repos", data={"name": GITHUB_REPO})
    assert new_repo.ok
    yield
    # After all
    deleted_repo = api_request_context.delete(f"/repos/{GITHUB_USER}/{GITHUB_REPO}")
    assert deleted_repo.ok
```

### 完整的测试示例

这是一个完整的 API 测试示例

```python
from enum import auto
import os
from typing import Generator

import pytest
from playwright.sync_api import Playwright, Page, APIRequestContext, expect

GITHUB_API_TOKEN = os.getenv("GITHUB_API_TOKEN")
assert GITHUB_API_TOKEN, "GITHUB_API_TOKEN is not set"

GITHUB_USER = os.getenv("GITHUB_USER")
assert GITHUB_USER, "GITHUB_USER is not set"

GITHUB_REPO = "test"


@pytest.fixture(scope="session")
def api_request_context(
    playwright: Playwright,
) -> Generator[APIRequestContext, None, None]:
    headers = {
        # We set this header per GitHub guidelines.
        "Accept": "application/vnd.github.v3+json",
        # Add authorization token to all requests.
        # Assuming personal access token available in the environment.
        "Authorization": f"token {GITHUB_API_TOKEN}",
    }
    request_context = playwright.request.new_context(
        base_url="https://api.github.com", extra_http_headers=headers
    )
    yield request_context
    request_context.dispose()


@pytest.fixture(scope="session", autouse=True)
def create_test_repository(
    api_request_context: APIRequestContext,
) -> Generator[None, None, None]:
    # Before all
    new_repo = api_request_context.post("/user/repos", data={"name": GITHUB_REPO})
    assert new_repo.ok
    yield
    # After all
    deleted_repo = api_request_context.delete(f"/repos/{GITHUB_USER}/{GITHUB_REPO}")
    assert deleted_repo.ok


def test_should_create_bug_report(api_request_context: APIRequestContext) -> None:
    data = {
        "title": "[Bug] report 1",
        "body": "Bug description",
    }
    new_issue = api_request_context.post(
        f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues", data=data
    )
    assert new_issue.ok

    issues = api_request_context.get(f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues")
    assert issues.ok
    issues_response = issues.json()
    issue = list(
        filter(lambda issue: issue["title"] == "[Bug] report 1", issues_response)
    )[0]
    assert issue
    assert issue["body"] == "Bug description"


def test_should_create_feature_request(api_request_context: APIRequestContext) -> None:
    data = {
        "title": "[Feature] request 1",
        "body": "Feature description",
    }
    new_issue = api_request_context.post(
        f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues", data=data
    )
    assert new_issue.ok

    issues = api_request_context.get(f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues")
    assert issues.ok
    issues_response = issues.json()
    issue = list(
        filter(lambda issue: issue["title"] == "[Feature] request 1", issues_response)
    )[0]
    assert issue
    assert issue["body"] == "Feature description"
```

## 通过 API 调用准备服务器状态

以下测试通过 API 创建一个新的 issue，然后导航到项目中所有 issue 的列表，以检查它是否出现在列表顶部。该检查使用 [LocatorAssertions](https://playwright.cn/python/docs/api/class-locatorassertions) 来执行。

```python
def test_last_created_issue_should_be_first_in_the_list(api_request_context: APIRequestContext, page: Page) -> None:
    def create_issue(title: str) -> None:
        data = {
            "title": title,
            "body": "Feature description",
        }
        new_issue = api_request_context.post(
            f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues", data=data
        )
        assert new_issue.ok
    create_issue("[Feature] request 1")
    create_issue("[Feature] request 2")
    page.goto(f"https://github.com/{GITHUB_USER}/{GITHUB_REPO}/issues")
    first_issue = page.locator("a[data-hovercard-type='issue']").first
    expect(first_issue).to_have_text("[Feature] request 2")
```

## 在执行用户操作后检查服务器状态

以下测试通过浏览器中的用户界面创建一个新问题，然后通过 API 检查它是否已创建

```python
def test_last_created_issue_should_be_on_the_server(api_request_context: APIRequestContext, page: Page) -> None:
    page.goto(f"https://github.com/{GITHUB_USER}/{GITHUB_REPO}/issues")
    page.locator("text=New issue").click()
    page.locator("[aria-label='Title']").fill("Bug report 1")
    page.locator("[aria-label='Comment body']").fill("Bug description")
    page.locator("text=Submit new issue").click()
    issue_id = page.url.split("/")[-1]

    new_issue = api_request_context.get(f"https://github.com/{GITHUB_USER}/{GITHUB_REPO}/issues/{issue_id}")
    assert new_issue.ok
    assert new_issue.json()["title"] == "[Bug] report 1"
    assert new_issue.json()["body"] == "Bug description"
```

## 重用身份验证状态

Web 应用使用基于 cookie 或 token 的身份验证，其中已验证的状态存储为 [cookie](https://mdn.org.cn/en-US/docs/Web/HTTP/Cookies)。Playwright 提供了 [api_request_context.storage_state()](https://playwright.cn/python/docs/api/class-apirequestcontext#api-request-context-storage-state) 方法，可用于从已验证的上下文中检索存储状态，然后使用该状态创建新的上下文。

存储状态可以在 [BrowserContext](https://playwright.cn/python/docs/api/class-browsercontext) 和 [APIRequestContext](https://playwright.cn/python/docs/api/class-apirequestcontext) 之间通用。您可以使用它通过 API 调用登录，然后创建一个已经包含 cookie 的新上下文。以下代码片段从经过身份验证的 [APIRequestContext](https://playwright.cn/python/docs/api/class-apirequestcontext) 中检索状态，并使用该状态创建一个新的 [BrowserContext](https://playwright.cn/python/docs/api/class-browsercontext)。

```python
request_context = playwright.request.new_context(http_credentials={"username": "test", "password": "test"})
request_context.get("https://api.example.com/login")
# Save storage state into a variable.
state = request_context.storage_state()

# Create a new context with the saved storage state.
context = browser.new_context(storage_state=state)
```