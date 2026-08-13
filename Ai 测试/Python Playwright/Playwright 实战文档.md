我帮你整理了一批 **Playwright 自动化脚本测试实战项目文档/案例**，偏向工程实践（不是只讲 API），包括 E2E 测试、Page Object Model、CI/CD、报告、真实项目框架等。([Playwright](https://playwright.dev/docs/intro?utm_source=chatgpt.com))

## 1. 官方入门实战项目（强烈推荐）

### 🎯 Playwright 官方 Example 项目

适合：刚开始学习 Playwright、搭建第一个自动化测试项目

内容：

- 安装 Playwright
- 创建测试项目结构
- 编写第一个 E2E 测试
- 运行测试
- HTML 测试报告
- Debug 模式
- 多浏览器测试

官方文档：
[Playwright 官方入门文档](https://playwright.cn/?utm_source=chatgpt.com)

项目结构示例：

```
playwright-demo/
│
├── tests/
│   └── example.spec.ts
│
├── playwright.config.ts
├── package.json
└── playwright-report/
```

学习重点：

```typescript
import { test, expect } from '@playwright/test';

test('login test', async ({ page }) => {

  await page.goto('https://example.com/login');

  await page.fill('#username', 'admin');
  await page.fill('#password', '123456');

  await page.click('button[type=submit]');

  await expect(page)
    .toHaveURL(/dashboard/);

});
```

([Playwright](https://playwright.dev/docs/writing-tests?utm_source=chatgpt.com))

------

# 2. Todo App 自动化测试实战项目 ⭐⭐⭐⭐⭐

适合：

- QA 自动化工程师
- 初学者练习完整流程

项目场景：

模拟真实 Todo 系统：

测试流程：

```
打开网站
 ↓
新增任务
 ↓
修改任务状态
 ↓
删除任务
 ↓
验证结果
```

覆盖：

✅ Locator
✅ Assertion
✅ Screenshot
✅ Video
✅ Trace Viewer
✅ HTML Report

参考项目：

[Playwright MiniProject: End-to-End Automation with the To-do App (Best Practices) 🔥](https://www.youtube.com/watch?v=5wSztvWhx14&utm_source=chatgpt.com)

练习目标：

```
tests/
 └── todo.spec.ts
```

示例：

```typescript
test('create todo', async ({page})=>{

 await page.goto(
 'https://demo.playwright.dev/todomvc'
 );

 await page
 .locator('.new-todo')
 .fill('Learn Playwright');


 await page.keyboard.press('Enter');


 await expect(
 page.locator('.todo-count')
 )
 .toContainText('1');

});
```

------

# 3. 企业级 Playwright 自动化框架项目 ⭐⭐⭐⭐⭐

适合：

- 想进入测试开发岗位
- 构建公司级自动化框架

推荐学习架构：

```
playwright-framework

├── tests
│   ├── login.spec.ts
│   ├── order.spec.ts
│   └── user.spec.ts
│
├── pages
│   ├── LoginPage.ts
│   ├── HomePage.ts
│   └── OrderPage.ts
│
├── fixtures
│
├── utils
│
├── data
│
├── reports
│
└── playwright.config.ts
```

核心模式：

## Page Object Model (POM)

例如：

pages/LoginPage.ts

```typescript
export class LoginPage {

constructor(private page:any){}


async login(
 username:string,
 password:string
){

await this.page.fill(
'#username',
username
);


await this.page.fill(
'#password',
password
);


await this.page.click(
'button'
);

}

}
```

测试：

```typescript
test(
'login',
async({page})=>{

const login =
new LoginPage(page);


await login.login(
'admin',
'123456'
);

});
```

社区有完整框架案例：

([Reddit](https://www.reddit.com/r/Playwright/comments/1tmua5t/built_my_first_playwright_automation_framework/?utm_source=chatgpt.com))

------

# 4. 电商网站自动化测试项目（真实业务）

推荐练习：

目标网站：

- 登录
- 商品搜索
- 加购物车
- 下订单
- 验证订单

测试模块：

```
ecommerce-playwright

tests/

├── login.spec.ts

├── product.spec.ts

├── cart.spec.ts

└── checkout.spec.ts


pages/

├── LoginPage.ts

├── ProductPage.ts

└── CheckoutPage.ts
```

覆盖企业常见场景：

| 功能      | Playwright技术 |
| --------- | -------------- |
| 登录保持  | storageState   |
| 商品搜索  | locator        |
| 接口 Mock | route          |
| 订单验证  | expect         |
| 截图      | screenshot     |
| 失败录像  | video          |

------

# 5. API + UI 混合自动化项目 ⭐⭐⭐⭐⭐

企业非常常见：

流程：

```
API 创建测试数据

↓

UI 登录

↓

操作页面

↓

API 验证结果
```

例如：

创建用户：

```typescript
await request.post(
'/api/users',
{
data:{
name:'test'
}
});
```

然后：

```typescript
await page.goto('/users');

await expect(
page.locator('text=test')
)
.visible();
```

适合：

- SaaS系统
- CRM
- ERP
- 管理后台

------

# 6. Playwright + GitHub Actions CI/CD 项目

企业面试经常问。

结构：

```
.github/

 workflows/

   playwright.yml
```

自动流程：

```
提交代码

↓

GitHub Actions

↓

安装浏览器

↓

执行测试

↓

生成报告
```

官方 CI 示例：

[Playwright CI 文档](https://playwright.dev/docs/ci?utm_source=chatgpt.com)

------

# 推荐学习路线（项目驱动）

如果目标是成为自动化测试工程师：

```
阶段1
│
├── 官方 Demo
├── Todo项目
└── 熟悉 locator/assert


阶段2

├── 登录系统
├── 电商系统
└── Page Object Model


阶段3

├── API + UI
├── 数据驱动测试
├── 多环境配置


阶段4

├── Docker
├── GitHub Actions
├── Jenkins
└── 企业级框架
```

## GitHub 搜索关键词（直接复制）

```
playwright automation framework typescript

playwright page object model

playwright ecommerce automation

playwright api testing

playwright github actions

playwright cucumber framework

playwright test framework
```

如果你是为了**找工作/面试准备**，我建议优先做一个完整作品：

> **Playwright + TypeScript + POM + API Mock + Allure/HTML Report + GitHub Actions 的电商自动化测试框架**

这个项目最接近企业 QA Automation Engineer 的实际工作。