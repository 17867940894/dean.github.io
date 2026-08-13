---
title: 弹窗 iframe 窗口切换
tags:
  - selenium
  - alert
  - iframe
  - 窗口切换
aliases:
  - switch_to
  - 句柄切换
date created: 2026-07-17
---

# 弹窗 / iframe / 窗口切换

> [!info] 本节目标
> 掌握三类"上下文切换"场景：原生 Alert 弹窗、iframe 嵌套页面、多窗口（标签页）切换，封装通用切换工具方法。

相关笔记：[[04-基础操作]]、[[05-等待机制]]、[[6. frame切换窗口切换]]

## 一、原生 Alert 弹窗

### 1.1 三种 Alert 类型

```html
<!-- 1. alert：只有确定按钮 -->
<input type="button" onclick="alert('hello')" value="alert 弹窗" />

<!-- 2. confirm：确定 / 取消 -->
<input type="button" onclick="confirm('确定吗？')" value="confirm 弹窗" />

<!-- 3. prompt：可输入文本 -->
<input type="button" onclick="prompt('请输入姓名')" value="prompt 弹窗" />
```

### 1.2 切换到 Alert 并操作

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wd = webdriver.Chrome()
wd.get('https://www.example.com/alert-demo')

# 点击触发弹窗的按钮
wd.find_element(By.CSS_SELECTOR, 'input[value="alert 弹窗"]').click()

# 等待弹窗出现并切换到 alert
alert = WebDriverWait(wd, 5).until(EC.alert_is_present())

# 获取弹窗文本
print(alert.text)  # hello

# 点击"确定"
alert.accept()

# 点击"取消"（仅 confirm / prompt）
# alert.dismiss()

# 在 prompt 中输入文本（accept 之前）
# alert.send_keys('张三')
# alert.accept()
```

> [!warning] Alert 操作顺序
> 1. 先 `switch_to.alert` 或 `EC.alert_is_present()` 切换到弹窗
> 2. 再操作 `text` / `accept()` / `dismiss()` / `send_keys()`
> 3. 一个 alert 只能操作一次（accept 或 dismiss 后弹窗就消失了）

### 1.3 常见报错

```
selenium.common.exceptions.NoAlertPresentException
```

原因：弹窗还没出现就去操作。解决：用 `EC.alert_is_present()` 显式等待。

## 二、iframe 切换

### 2.1 什么是 iframe

iframe 是 HTML 中嵌套的"子页面"，**Selenium 默认只能操作当前所在 iframe 的元素**。要操作 iframe 内部元素，必须先切换进去。

### 2.2 三种切换方式

```python
# HTML 结构：
# <iframe id="frame1" name="main" src="..."></iframe>

# 方式 1：通过下标（从 0 开始，按页面中 iframe 出现顺序）
wd.switch_to.frame(0)

# 方式 2：通过 name 或 id 属性
wd.switch_to.frame('frame1')      # 用 id
wd.switch_to.frame('main')        # 用 name

# 方式 3：通过 WebElement（最灵活）
frame_element = wd.find_element(By.CSS_SELECTOR, 'iframe#frame1')
wd.switch_to.frame(frame_element)
```

### 2.3 切回主文档 / 父 iframe

```python
# 切回主文档（最外层）
wd.switch_to.default_content()

# 切回上一级父 iframe（嵌套 iframe 场景）
wd.switch_to.parent_frame()
```

### 2.4 完整示例

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wd = webdriver.Chrome()
wd.get('https://www.example.com/iframe-demo')

# 操作主文档元素
wd.find_element(By.ID, 'username').send_keys('admin')

# 切换到 iframe
frame = wd.find_element(By.CSS_SELECTOR, 'iframe#editor')
wd.switch_to.frame(frame)

# 现在可以操作 iframe 内部元素
wd.find_element(By.ID, 'content').send_keys('Hello iframe')

# 切回主文档
wd.switch_to.default_content()

# 继续操作主文档元素
wd.find_element(By.ID, 'submit').click()

wd.quit()
```

### 2.5 嵌套 iframe

```python
# 结构：
# <iframe id="outer"> → <iframe id="inner"> → 目标元素

# 一层一层切进去
wd.switch_to.frame('outer')
wd.switch_to.frame('inner')

# 操作目标元素
wd.find_element(By.ID, 'target').click()

# 一层一层切出来
wd.switch_to.parent_frame()  # 回到 outer
wd.switch_to.parent_frame()  # 回到主文档
# 或者直接
wd.switch_to.default_content()
```

### 2.6 iframe 等待

```python
# 等待 iframe 可用并自动切换（一步到位）
WebDriverWait(wd, 10).until(
    EC.frame_to_be_available_and_switch_to_it((By.CSS_SELECTOR, 'iframe#editor'))
)
# 切换成功后，可直接操作 iframe 内元素
```

> [!tip] 强烈推荐
> `frame_to_be_available_and_switch_to_it` 同时完成"等待 + 切换"，是处理 iframe 的最佳实践。

## 三、多窗口（标签页）切换

### 3.1 窗口句柄概念

每个浏览器窗口/标签页都有一个唯一标识，称为 **窗口句柄（window handle）**，是一个字符串。

```python
# 当前窗口句柄
current_handle = wd.current_window_handle

# 所有窗口句柄列表（按打开顺序）
all_handles = wd.window_handles
```

### 3.2 切换窗口

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

# D:\develop\Python\Browser driver\chromedriver.exe
wd = webdriver.Chrome(service=Service(r'D:\develop\Python\Browser driver\chromedriver.exe'))

wd.get('https://www.baidu.com')

# 记录原窗口
original = wd.current_window_handle

# 点击"新闻"链接（在新标签页打开）
wd.find_element(By.LINK_TEXT, '新闻').click()

# 等待新窗口出现
import time
time.sleep(1)

# 获取所有句柄
handles = wd.window_handles

# 切换到新窗口（最后一个）
for h in handles:
    if h != original:
        wd.switch_to.window(h)
        break

print('新窗口标题:', wd.title)

# 切回原窗口
wd.switch_to.window(original)
print('原窗口标题:', wd.title)

wd.quit()
```

### 3.3 用显式等待处理新窗口

```python
from selenium.webdriver.support.ui import WebDriverWait

def switch_to_new_window(driver, timeout=10):
    """等待并切换到新打开的窗口"""
    old_handles = set(driver.window_handles)
    WebDriverWait(driver, timeout).until(
        lambda d: len(d.window_handles) > len(old_handles)
    )
    new_handles = set(driver.window_handles) - old_handles
    driver.switch_to.window(new_handles.pop())

# 使用
wd.find_element(By.LINK_TEXT, '新闻').click()
switch_to_new_window(wd)
print('当前窗口:', wd.title)
```

### 3.4 关闭多窗口的正确姿势

```python
# 关闭当前窗口（只关一个，浏览器进程还在）
wd.close()

# 关闭后必须切到其他窗口，否则 driver 找不到窗口会报错
wd.switch_to.window(wd.window_handles[0])
```

## 四、通用切换工具封装

```python
# 放进 utils/common.py（完整版见实战项目）
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class SwitchHelper:
    def __init__(self, driver):
        self.driver = driver

    def to_alert(self, timeout=5):
        """切换到 alert 弹窗"""
        return WebDriverWait(self.driver, timeout).until(EC.alert_is_present())

    def to_frame(self, locator, timeout=10):
        """等待并切换到 iframe"""
        return WebDriverWait(self.driver, timeout).until(
            EC.frame_to_be_available_and_switch_to_it(locator)
        )

    def to_default(self):
        """切回主文档"""
        self.driver.switch_to.default_content()

    def to_new_window(self, timeout=10):
        """等待并切换到新打开的窗口"""
        old = set(self.driver.window_handles)
        WebDriverWait(self.driver, timeout).until(
            lambda d: len(d.window_handles) > len(old)
        )
        new = set(self.driver.window_handles) - old
        self.driver.switch_to.window(new.pop())

    def to_window_by_title(self, title, timeout=10):
        """按标题切换到对应窗口"""
        WebDriverWait(self.driver, timeout).until(
            lambda d: any(
                d.switch_to.window(h) or title in d.title for h in d.window_handles
            )
        )
```

## 五、常见报错速查

| 报错 | 原因 | 解决 |
| --- | --- | --- |
| `NoSuchElementException`（明明 F12 能看到） | 元素在 iframe 里 | 先 `switch_to.frame()` |
| `NoAlertPresentException` | alert 还没出现 | `EC.alert_is_present()` 等待 |
| `NoSuchWindowException` | 窗口已关闭或句柄错误 | `close()` 后切到其他窗口 |
| `StaleElementReferenceException` | 元素引用失效（页面刷新了） | 重新定位元素 |

下一节：[[07-鼠标键盘基础操作]]
