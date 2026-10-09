# 题单汇总 · 可打印勾选版

> 配套：[ROADMAP.md](../ROADMAP.md) · [教材索引](README.md)
> 用法：打印或复制到笔记软件，做完一道勾一道。
> 数据来源：**全部题目名已逐一实测核实**（PortSwigger 272 个实验、Bandit 34 关、ROP Emporium 8 关等）。

---

## 一、阶段 1 · Linux 与网络基础

### 1.1 OverTheWire Bandit（34 关）→ 详细讲解见 [第 1 章](01-linux-and-network.md)

| 关 | 命令 | 完成 |
|---|---|---|
| 0 | ssh 登录 | ☐ |
| 1 | cat readme | ☐ |
| 2 | cat ./- | ☐ |
| 3 | cat 文件名带空格 | ☐ |
| 4 | file + cat | ☐ |
| 5 | find -size -user -group | ☐ |
| 6 | find 全盘搜索 | ☐ |
| 7 | grep | ☐ |
| 8 | sort \| uniq -u | ☐ |
| 9 | strings + grep | ☐ |
| 10 | base64 -d | ☐ |
| 11 | tr ROT13 | ☐ |
| 12 | xxd -r + 多层解压 | ☐ |
| 13 | ssh -i 私钥 | ☐ |
| 14 | nc 收发 | ☐ |
| 15 | openssl s_client | ☐ |
| 16 | nmap 扫端口范围 | ☐ |
| 17 | 服务差异对比 | ☐ |
| 18 | .bashrc 陷阱 | ☐ |
| 19 | ssh host "命令" | ☐ |
| 20 | SUID + nc | ☐ |
| 21 | cron 任务 | ☐ |
| 22 | cron 脚本分析 | ☐ |
| 23 | 写 cron 目录 | ☐ |
| 24 | cron + 权限 | ☐ |
| 25 | more 分页器逃逸 | ☐ |
| 26 | more → vim → :!sh | ☐ |
| 27 | git log | ☐ |
| 28 | git branch | ☐ |
| 29 | git tag | ☐ |
| 30 | git 提交信息 | ☐ |
| 31 | git ls-files | ☐ |
| 32 | $0 逃逸受限 shell | ☐ |
| 33 | 综合提权 | ☐ |

### 1.2 TryHackMe 免费房间（8 个）

| # | 房间 | 完成 |
|---|---|---|
| 1 | Pre Security 路径 | ☐ |
| 2 | Linux Fundamentals 1 | ☐ |
| 3 | Linux Fundamentals 2 | ☐ |
| 4 | Linux Fundamentals 3 | ☐ |
| 5 | Windows Fundamentals 1 | ☐ |
| 6 | Windows Fundamentals 2 | ☐ |
| 7 | Intro to LAN | ☐ |
| 8 | HTTP in Detail | ☐ |

---

## 二、阶段 2 · Web 渗透主力 ★（PortSwigger 272 个实验）

> 讲解见 [第 2 章](02-web-pentesting.md)。网址：https://portswigger.net/web-security
> 全部实验免费、在线可做、无需搭环境。

### 2.1 SQL 注入（18 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | SQL injection vulnerability in WHERE clause allowing retrieval of hidden data | ☐ |
| 2 | SQL injection vulnerability allowing login bypass | ☐ |
| 3 | SQL injection attack, querying the database type and version on Oracle | ☐ |
| 4 | SQL injection attack, querying the database type and version on MySQL and Microsoft | ☐ |
| 5 | SQL injection attack, listing the database contents on non-Oracle databases | ☐ |
| 6 | SQL injection attack, listing the database contents on Oracle | ☐ |
| 7 | SQL injection UNION attack, determining the number of columns returned by the query | ☐ |
| 8 | SQL injection UNION attack, finding a column containing text | ☐ |
| 9 | SQL injection UNION attack, retrieving data from other tables | ☐ |
| 10 | SQL injection UNION attack, retrieving multiple values in a single column | ☐ |
| 11 | Blind SQL injection with conditional responses | ☐ |
| 12 | Blind SQL injection with conditional errors | ☐ |
| 13 | Visible error-based SQL injection | ☐ |
| 14 | Blind SQL injection with time delays | ☐ |
| 15 | Blind SQL injection with time delays and information retrieval | ☐ |
| 16 | Blind SQL injection with out-of-band interaction | ☐ |
| 17 | Blind SQL injection with out-of-band data exfiltration | ☐ |
| 18 | SQL injection with filter bypass via XML encoding | ☐ |

### 2.2 XSS 跨站脚本（30 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Reflected XSS into HTML context with nothing encoded | ☐ |
| 2 | Stored XSS into HTML context with nothing encoded | ☐ |
| 3 | DOM XSS in document.write sink using source location.search | ☐ |
| 4 | DOM XSS in innerHTML sink using source location.search | ☐ |
| 5 | DOM XSS in jQuery anchor href attribute sink using location.search source | ☐ |
| 6 | DOM XSS in jQuery selector sink using a hashchange event | ☐ |
| 7 | Reflected XSS into attribute with angle brackets HTML-encoded | ☐ |
| 8 | Stored XSS into anchor href attribute with double quotes HTML-encoded | ☐ |
| 9 | Reflected XSS into a JavaScript string with angle brackets HTML encoded | ☐ |
| 10 | DOM XSS in document.write sink using source location.search inside a select element | ☐ |
| 11 | DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded | ☐ |
| 12 | Reflected DOM XSS | ☐ |
| 13 | Stored DOM XSS | ☐ |
| 14 | Reflected XSS into HTML context with most tags and attributes blocked | ☐ |
| 15 | Reflected XSS into HTML context with all tags blocked except custom ones | ☐ |
| 16 | Reflected XSS with some SVG markup allowed | ☐ |
| 17 | Reflected XSS in canonical link tag | ☐ |
| 18 | Reflected XSS into a JavaScript string with single quote and backslash escaped | ☐ |
| 19 | Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped | ☐ |
| 20 | Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped | ☐ |
| 21 | Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped | ☐ |
| 22 | Exploiting cross-site scripting to steal cookies | ☐ |
| 23 | Exploiting cross-site scripting to capture passwords | ☐ |
| 24 | Exploiting XSS to bypass CSRF defenses | ☐ |
| 25 | Reflected XSS with AngularJS sandbox escape without strings | ☐ |
| 26 | Reflected XSS with AngularJS sandbox escape and CSP | ☐ |
| 27 | Reflected XSS with event handlers and href attributes blocked | ☐ |
| 28 | Reflected XSS in a JavaScript URL with some characters blocked | ☐ |
| 29 | Reflected XSS protected by very strict CSP, with dangling markup attack | ☐ |
| 30 | Reflected XSS protected by CSP, with CSP bypass | ☐ |

### 2.3 越权 / 访问控制（13 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Unprotected admin functionality | ☐ |
| 2 | Unprotected admin functionality with unpredictable URL | ☐ |
| 3 | User role controlled by request parameter | ☐ |
| 4 | User role can be modified in user profile | ☐ |
| 5 | User ID controlled by request parameter | ☐ |
| 6 | User ID controlled by request parameter, with unpredictable user IDs | ☐ |
| 7 | User ID controlled by request parameter with data leakage in redirect | ☐ |
| 8 | User ID controlled by request parameter with password disclosure | ☐ |
| 9 | Insecure direct object references | ☐ |
| 10 | URL-based access control can be circumvented | ☐ |
| 11 | Method-based access control can be circumvented | ☐ |
| 12 | Multi-step process with no access control on one step | ☐ |
| 13 | Referer-based access control | ☐ |

### 2.4 认证漏洞（14 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Username enumeration via different responses | ☐ |
| 2 | 2FA simple bypass | ☐ |
| 3 | Password reset broken logic | ☐ |
| 4 | Username enumeration via subtly different responses | ☐ |
| 5 | Username enumeration via response timing | ☐ |
| 6 | Broken brute-force protection, IP block | ☐ |
| 7 | Username enumeration via account lock | ☐ |
| 8 | 2FA broken logic | ☐ |
| 9 | Brute-forcing a stay-logged-in cookie | ☐ |
| 10 | Offline password cracking | ☐ |
| 11 | Password reset poisoning via middleware | ☐ |
| 12 | Password brute-force via password change | ☐ |
| 13 | Broken brute-force protection, multiple credentials per request | ☐ |
| 14 | 2FA bypass using a brute-force attack | ☐ |

### 2.5 文件上传（7 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Remote code execution via web shell upload | ☐ |
| 2 | Web shell upload via Content-Type restriction bypass | ☐ |
| 3 | Web shell upload via path traversal | ☐ |
| 4 | Web shell upload via extension blacklist bypass | ☐ |
| 5 | Web shell upload via obfuscated file extension | ☐ |
| 6 | Remote code execution via polyglot web shell upload | ☐ |
| 7 | Web shell upload via race condition | ☐ |

### 2.6 OS 命令注入（5 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | OS command injection, simple case | ☐ |
| 2 | Blind OS command injection with time delays | ☐ |
| 3 | Blind OS command injection with output redirection | ☐ |
| 4 | Blind OS command injection with out-of-band interaction | ☐ |
| 5 | Blind OS command injection with out-of-band data exfiltration | ☐ |

### 2.7 路径穿越（6 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | File path traversal, simple case | ☐ |
| 2 | File path traversal, traversal sequences blocked with absolute path bypass | ☐ |
| 3 | File path traversal, traversal sequences stripped non-recursively | ☐ |
| 4 | File path traversal, traversal sequences stripped with superfluous URL-decode | ☐ |
| 5 | File path traversal, validation of start of path | ☐ |
| 6 | File path traversal, validation of file extension with null byte bypass | ☐ |

### 2.8 SSRF（7 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Basic SSRF against the local server | ☐ |
| 2 | Basic SSRF against another back-end system | ☐ |
| 3 | Blind SSRF with out-of-band detection | ☐ |
| 4 | SSRF with blacklist-based input filter | ☐ |
| 5 | SSRF with filter bypass via open redirection vulnerability | ☐ |
| 6 | Blind SSRF with Shellshock exploitation | ☐ |
| 7 | SSRF with whitelist-based input filter | ☐ |

### 2.9 XXE（9 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Exploiting XXE using external entities to retrieve files | ☐ |
| 2 | Exploiting XXE to perform SSRF attacks | ☐ |
| 3 | Blind XXE with out-of-band interaction | ☐ |
| 4 | Blind XXE with out-of-band interaction via XML parameter entities | ☐ |
| 5 | Exploiting blind XXE to exfiltrate data using a malicious external DTD | ☐ |
| 6 | Exploiting blind XXE to retrieve data via error messages | ☐ |
| 7 | Exploiting XInclude to retrieve files | ☐ |
| 8 | Exploiting XXE via image file upload | ☐ |
| 9 | Exploiting XXE to retrieve data by repurposing a local DTD | ☐ |

### 2.10 SSTI 服务端模板注入（7 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Basic server-side template injection | ☐ |
| 2 | Basic server-side template injection (code context) | ☐ |
| 3 | Server-side template injection using documentation | ☐ |
| 4 | Server-side template injection in an unknown language with a documented exploit | ☐ |
| 5 | Server-side template injection with information disclosure via user-supplied objects | ☐ |
| 6 | Server-side template injection in a sandboxed environment | ☐ |
| 7 | Server-side template injection with a custom exploit | ☐ |

### 2.11 JWT（8 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | JWT authentication bypass via unverified signature | ☐ |
| 2 | JWT authentication bypass via flawed signature verification | ☐ |
| 3 | JWT authentication bypass via weak signing key | ☐ |
| 4 | JWT authentication bypass via jwk header injection | ☐ |
| 5 | JWT authentication bypass via jku header injection | ☐ |
| 6 | JWT authentication bypass via kid header path traversal | ☐ |
| 7 | JWT authentication bypass via algorithm confusion | ☐ |
| 8 | JWT authentication bypass via algorithm confusion with no exposed key | ☐ |

### 2.12 CSRF（11 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | CSRF vulnerability with no defenses | ☐ |
| 2 | CSRF where token validation depends on request method | ☐ |
| 3 | CSRF where token validation depends on token being present | ☐ |
| 4 | CSRF where token is not tied to user session | ☐ |
| 5 | CSRF where token is tied to non-session cookie | ☐ |
| 6 | CSRF where token is duplicated in cookie | ☐ |
| 7 | SameSite Lax bypass via method override | ☐ |
| 8 | SameSite Strict bypass via client-side redirect | ☐ |
| 9 | SameSite Strict bypass via sibling domain | ☐ |
| 10 | CSRF where Referer validation depends on header being present | ☐ |
| 11 | CSRF with broken Referer validation | ☐ |

### 2.13 信息泄露（5 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Information disclosure in error messages | ☐ |
| 2 | Information disclosure on debug page | ☐ |
| 3 | Source code disclosure via backup files | ☐ |
| 4 | Authentication bypass via information disclosure | ☐ |
| 5 | Information disclosure in version control history | ☐ |

### 2.14 业务逻辑漏洞（12 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Excessive trust in client-side controls | ☐ |
| 2 | High-level logic vulnerability | ☐ |
| 3 | Inconsistent security controls | ☐ |
| 4 | Flawed enforcement of business rules | ☐ |
| 5 | Low-level logic flaw | ☐ |
| 6 | Inconsistent handling of exceptional input | ☐ |
| 7 | Weak isolation on dual-use endpoint | ☐ |
| 8 | Insufficient workflow validation | ☐ |
| 9 | Authentication bypass via flawed state machine | ☐ |
| 10 | Infinite money logic flaw | ☐ |
| 11 | Authentication bypass via encryption oracle | ☐ |
| 12 | Bypassing access controls using email address parsing discrepancies | ☐ |

### 2.15 不安全反序列化（10 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Modifying serialized objects | ☐ |
| 2 | Modifying serialized data types | ☐ |
| 3 | Using application functionality to exploit insecure deserialization | ☐ |
| 4 | Arbitrary object injection in PHP | ☐ |
| 5 | Exploiting Java deserialization with Apache Commons | ☐ |
| 6 | Exploiting PHP deserialization with a pre-built gadget chain | ☐ |
| 7 | Exploiting Ruby deserialization using a documented gadget chain | ☐ |
| 8 | Developing a custom gadget chain for Java deserialization | ☐ |
| 9 | Developing a custom gadget chain for PHP deserialization | ☐ |
| 10 | Using PHAR deserialization to deploy a custom gadget chain | ☐ |

### 2.16 NoSQL 注入（4 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Detecting NoSQL injection | ☐ |
| 2 | Exploiting NoSQL operator injection to bypass authentication | ☐ |
| 3 | Exploiting NoSQL injection to extract data | ☐ |
| 4 | Exploiting NoSQL operator injection to extract unknown fields | ☐ |

### 2.17 竞态条件（6 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Limit overrun race conditions | ☐ |
| 2 | Bypassing rate limits via race conditions | ☐ |
| 3 | Multi-endpoint race conditions | ☐ |
| 4 | Single-endpoint race conditions | ☐ |
| 5 | Exploiting time-sensitive vulnerabilities | ☐ |
| 6 | Partial construction race conditions | ☐ |

### 2.18 API 测试（5 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Exploiting an API endpoint using documentation | ☐ |
| 2 | Exploiting server-side parameter pollution in a query string | ☐ |
| 3 | Finding and exploiting an unused API endpoint | ☐ |
| 4 | Exploiting a mass assignment vulnerability | ☐ |
| 5 | Exploiting server-side parameter pollution in a REST URL | ☐ |

### 2.19 Web 缓存投毒（13 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Web cache poisoning with an unkeyed header | ☐ |
| 2 | Web cache poisoning with an unkeyed cookie | ☐ |
| 3 | Web cache poisoning with multiple headers | ☐ |
| 4 | Targeted web cache poisoning using an unknown header | ☐ |
| 5 | Web cache poisoning via an unkeyed query string | ☐ |
| 6 | Web cache poisoning via an unkeyed query parameter | ☐ |
| 7 | Parameter cloaking | ☐ |
| 8 | Web cache poisoning via a fat GET request | ☐ |
| 9 | URL normalization | ☐ |
| 10 | Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria | ☐ |
| 11 | Combining web cache poisoning vulnerabilities | ☐ |
| 12 | Cache key injection | ☐ |
| 13 | Internal cache poisoning | ☐ |

### 2.20 DOM 型漏洞（7 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | DOM XSS using web messages | ☐ |
| 2 | DOM XSS using web messages and a JavaScript URL | ☐ |
| 3 | DOM XSS using web messages and JSON.parse | ☐ |
| 4 | DOM-based open redirection | ☐ |
| 5 | DOM-based cookie manipulation | ☐ |
| 6 | Exploiting DOM clobbering to enable XSS | ☐ |
| 7 | Clobbering DOM attributes to bypass HTML filters | ☐ |

### 2.21 CORS（3 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | CORS vulnerability with basic origin reflection | ☐ |
| 2 | CORS vulnerability with trusted null origin | ☐ |
| 3 | CORS vulnerability with trusted insecure protocols | ☐ |

### 2.22 点击劫持（5 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Basic clickjacking with CSRF token protection | ☐ |
| 2 | Clickjacking with form input data prefilled from a URL parameter | ☐ |
| 3 | Clickjacking with a frame buster script | ☐ |
| 4 | Exploiting clickjacking vulnerability to trigger DOM-based XSS | ☐ |
| 5 | Multistep clickjacking | ☐ |

### 2.23 WebSocket（2 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Manipulating WebSocket messages to exploit vulnerabilities | ☐ |
| 2 | Manipulating the WebSocket handshake to exploit vulnerabilities | ☐ |

### 2.24 OAuth 认证（6 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Authentication bypass via OAuth implicit flow | ☐ |
| 2 | SSRF via OpenID dynamic client registration | ☐ |
| 3 | Forced OAuth profile linking | ☐ |
| 4 | OAuth account hijacking via redirect_uri | ☐ |
| 5 | Stealing OAuth access tokens via an open redirect | ☐ |
| 6 | Stealing OAuth access tokens via a proxy page | ☐ |

### 2.25 HTTP Host 头攻击（7 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Basic password reset poisoning | ☐ |
| 2 | Host header authentication bypass | ☐ |
| 3 | Web cache poisoning via ambiguous requests | ☐ |
| 4 | Routing-based SSRF | ☐ |
| 5 | SSRF via flawed request parsing | ☐ |
| 6 | Host validation bypass via connection state attack | ☐ |
| 7 | Password reset poisoning via dangling markup | ☐ |

### 2.26 原型污染（10 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Client-side prototype pollution via browser APIs | ☐ |
| 2 | DOM XSS via client-side prototype pollution | ☐ |
| 3 | DOM XSS via an alternative prototype pollution vector | ☐ |
| 4 | Client-side prototype pollution via flawed sanitization | ☐ |
| 5 | Client-side prototype pollution in third-party libraries | ☐ |
| 6 | Privilege escalation via server-side prototype pollution | ☐ |
| 7 | Detecting server-side prototype pollution without polluted property reflection | ☐ |
| 8 | Bypassing flawed input filters for server-side prototype pollution | ☐ |
| 9 | Remote code execution via server-side prototype pollution | ☐ |
| 10 | Exfiltrating sensitive data via server-side prototype pollution | ☐ |

### 2.27 HTTP 请求走私（20 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | HTTP request smuggling, basic CL.TE vulnerability | ☐ |
| 2 | HTTP request smuggling, basic TE.CL vulnerability | ☐ |
| 3 | HTTP request smuggling, obfuscating the TE header | ☐ |
| 4 | HTTP request smuggling, confirming a CL.TE vulnerability via differential responses | ☐ |
| 5 | HTTP request smuggling, confirming a TE.CL vulnerability via differential responses | ☐ |
| 6 | Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability | ☐ |
| 7 | Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability | ☐ |
| 8 | Exploiting HTTP request smuggling to reveal front-end request rewriting | ☐ |
| 9 | Exploiting HTTP request smuggling to capture other users' requests | ☐ |
| 10 | Exploiting HTTP request smuggling to deliver reflected XSS | ☐ |
| 11 | Exploiting HTTP request smuggling to perform web cache poisoning | ☐ |
| 12 | Exploiting HTTP request smuggling to perform web cache deception | ☐ |
| 13 | Response queue poisoning via H2.TE request smuggling | ☐ |
| 14 | H2.CL request smuggling | ☐ |
| 15 | HTTP/2 request smuggling via CRLF injection | ☐ |
| 16 | HTTP/2 request splitting via CRLF injection | ☐ |
| 17 | 0.CL request smuggling | ☐ |
| 18 | CL.0 request smuggling | ☐ |
| 19 | Client-side desync | ☐ |
| 20 | Server-side pause-based request smuggling | ☐ |

### 2.28 Web 缓存欺骗（5 个）

| # | 实验 | 完成 |
|---|---|---|
| 1 | Exploiting path mapping for web cache deception | ☐ |
| 2 | Exploiting path delimiters for web cache deception | ☐ |
| 3 | Exploiting origin server normalization for web cache deception | ☐ |
| 4 | Exploiting cache server normalization for web cache deception | ☐ |
| 5 | Exploiting exact-match cache rules for web cache deception | ☐ |

### 2.29 其他

| 模块 | 实验 | 完成 |
|---|---|---|
| Essential skills | Discovering vulnerabilities quickly with targeted scanning | ☐ |
| Essential skills | Scanning non-standard data structures | ☐ |
| Mystery lab | 随机隐藏标题实验（练侦察） | ☐ |

### 2.30 本地综合靶场

| # | 靶场 | 题目量 | 完成 |
|---|---|---|---|
| 1 | **OWASP Juice Shop**（已运行 :13000） | 100+ 挑战 | ☐ |
| 2 | vulhub 复现（见下） | 20 个环境 | ☐ |
| 3 | DVWA（Local/Medium/High） | 10+ 模块 | ☐ |
| 4 | bWAPP | 100+ 漏洞 | ☐ |
| 5 | WebGoat | 全套 | ☐ |

### 2.31 vulhub 复现清单（20 个）

| # | 路径 | 漏洞 | 完成 |
|---|---|---|---|
| 1 | flask/ssti | SSTI | ☐ |
| 2 | bash/CVE-2014-6271 | Shellshock | ☐ |
| 3 | tomcat/* | 弱口令/上传 | ☐ |
| 4 | fastjson/1.2.24-rce | Java 反序列化 | ☐ |
| 5 | fastjson/1.2.47-rce | 反序列化绕过 | ☐ |
| 6 | log4j/CVE-2021-44228 | Log4Shell | ☐ |
| 7 | weblogic/CVE-2020-14882 | 未授权 RCE | ☐ |
| 8 | weblogic/CVE-2019-2725 | 反序列化 | ☐ |
| 9 | struts2/CVE-2017-5638 | OGNL 注入 | ☐ |
| 10 | shiro/CVE-2016-4437 | rememberMe | ☐ |
| 11 | spring/CVE-2022-22965 | Spring4Shell | ☐ |
| 12 | redis/未授权 | 写 SSH key | ☐ |
| 13 | docker/unauthorized-rce | API 未授权 | ☐ |
| 14 | mysql/CVE-2012-2122 | 认证绕过 | ☐ |
| 15 | gitlab/CVE-2021-22205 | 上传 RCE | ☐ |
| 16 | jenkins/CVE-2018-1000861 | 未授权 RCE | ☐ |
| 17 | wordpress/* | CMS 漏洞 | ☐ |
| 18 | drupal/CVE-2018-7600 | Drupalgeddon2 | ☐ |
| 19 | nginx/CVE-2017-7529 | 越界读取 | ☐ |
| 20 | php/* | LFI→RCE | ☐ |

---

## 三、阶段 3 · Pwn 与逆向

> 讲解见 [第 3 章](03-pwn-and-reversing.md)

### 3.1 ROP Emporium（8 关）

| # | 关卡 | 知识点 | 完成 |
|---|---|---|---|
| 1 | ret2win | 溢出覆盖返回地址 | ☐ |
| 2 | split | pop rdi + PLT | ☐ |
| 3 | callme | 多函数链 + 栈对齐 | ☐ |
| 4 | write4 | 任意写原语 | ☐ |
| 5 | badchars | 坏字符编码 | ☐ |
| 6 | fluff | gadget 组合 | ☐ |
| 7 | pivot | 栈迁移 | ☐ |
| 8 | ret2csu | 通用 gadget | ☐ |

### 3.2 Exploit Education · Phoenix

| 系列 | 关卡 | 完成 |
|---|---|---|
| Stack | Zero / One / Two / Three / Four / Five / Six | ☐ |
| Format | Zero / One / Two / Three / Four | ☐ |
| Heap | Zero / One / Two / Three | ☐ |
| Net | Zero / One / Two | ☐ |
| Final | Zero / One / Two | ☐ |

### 3.3 crackmes.one（目标 30 题）

| # | 题目 | 难度 | 平台 | 完成 |
|---|---|---|---|---|
| 1–10 | （自选，难度 1） | 1 | Linux | ☐ |
| 11–20 | （自选，难度 1–2） | 2 | Linux | ☐ |
| 21–30 | （自选，难度 2–3） | 3 | Windows/Linux | ☐ |

> 下载密码统一为 `crackmes.one`

### 3.4 pwnable.tw / pwnable.kr（各 3 题）

| 平台 | 题目 | 完成 |
|---|---|---|
| pwnable.tw | start | ☐ |
| pwnable.tw | orw | ☐ |
| pwnable.tw | calc | ☐ |
| pwnable.kr | fd | ☐ |
| pwnable.kr | collision | ☐ |
| pwnable.kr | bof | ☐ |

### 3.5 其他逆向资源

| 资源 | 目标 | 完成 |
|---|---|---|
| Microcorruption | 15 关 | ☐ |
| picoCTF 逆向分类 | 30 题 | ☐ |
| Nightmare 讲义 | 前 8 章 | ☐ |
| pwn.college | 完成 Dojo 模块 | ☐ |

---

## 四、阶段 4 · 内网与 AD

> 讲解见 [第 4 章](04-internal-network-and-ad.md)

### 4.1 自建靶场

| # | 任务 | 完成 |
|---|---|---|
| 1 | UTM 装 Windows Server 2022 ARM | ☐ |
| 2 | 设为 DC01，建域 `lab.local` | ☐ |
| 3 | 建 3 个测试账号（含 1 个服务账号带 SPN） | ☐ |
| 4 | UTM 装 Windows 11，加入域 | ☐ |
| 5 | 攻击机连通性验证 | ☐ |
| 6 | 完整走通：枚举 → Kerberoasting → 横向 → DCSync | ☐ |
| 7 | 伪造黄金票据成功访问 DC | ☐ |

### 4.2 THM / HTB 免费题

| # | 房间 / 机器 | 平台 | 完成 |
|---|---|---|---|
| 1 | Attacktive Directory | THM | ☐ |
| 2 | Attacking Kerberos | THM | ☐ |
| 3 | Active Directory Basics | THM | ☐ |
| 4 | Meow | HTB SP | ☐ |
| 5 | Fawn | HTB SP | ☐ |
| 6 | Dancing | HTB SP | ☐ |
| 7 | Redeemer | HTB SP | ☐ |
| 8 | Explosion | HTB SP | ☐ |
| 9 | Preignition | HTB SP | ☐ |
| 10 | Mongod | HTB SP | ☐ |
| 11 | Synced | HTB SP | ☐ |
| 12 | Appointment | HTB SP | ☐ |
| 13 | Sequel | HTB SP | ☐ |

### 4.3 攻击链练习（每项都要亲手做一遍）

| # | 技术 | 完成 |
|---|---|---|
| 1 | enum4linux-ng 无凭据枚举 | ☐ |
| 2 | AS-REP Roasting + 破解 | ☐ |
| 3 | Kerberoasting + 破解 | ☐ |
| 4 | 密码喷洒 | ☐ |
| 5 | bloodhound-python 收集 + 分析 | ☐ |
| 6 | WinRM 横向 | ☐ |
| 7 | WMI 横向 | ☐ |
| 8 | PsExec 横向 | ☐ |
| 9 | PtH 哈希传递 | ☐ |
| 10 | secretsdump 导出哈希 | ☐ |
| 11 | DCSync | ☐ |
| 12 | 黄金票据 | ☐ |
| 13 | certipy AD CS 攻击 | ☐ |
| 14 | chisel 隧道 + proxychains | ☐ |

---

## 五、阶段 5 · 实战化（VulnHub / HackMyVM / HTB）

> 讲解见 [第 5 章](05-practice-and-output.md)

### 5.1 入门靶机（每台写一篇 walkthrough）

| # | 靶机 | 平台 | 难度 | 完成 | walkthrough |
|---|---|---|---|---|---|
| 1 | Jangow: 1.0.1 | VulnHub | Easy | ☐ | ☐ |
| 2 | The Planets: Earth | VulnHub | Easy | ☐ | ☐ |
| 3 | ICA: 1 | VulnHub | Easy | ☐ | ☐ |
| 4 | Noob: 1 | VulnHub | Easy | ☐ | ☐ |
| 5 | Matrix-Breakout 2: Morpheus | VulnHub | Easy | ☐ | ☐ |
| 6 | Napping: 1.0.1 | VulnHub | Easy | ☐ | ☐ |
| 7 | Red: 1 | VulnHub | Easy | ☐ | ☐ |
| 8 | Thales: 1 | VulnHub | Easy | ☐ | ☐ |
| 9 | Empire: Breakout | VulnHub | Easy | ☐ | ☐ |
| 10 | Empire: LupinOne | VulnHub | Easy | ☐ | ☐ |
| 11 | Web Machine: N7 | VulnHub | Medium | ☐ | ☐ |
| 12 | Digitalworld.local: Electrical | VulnHub | Medium | ☐ | ☐ |

### 5.2 Web 专项靶机

| # | 靶机 | 平台 | 完成 |
|---|---|---|---|
| 1 | Vulnversity | THM | ☐ |
| 2 | Pickle Rick | THM | ☐ |
| 3 | Ignite | THM | ☐ |
| 4 | OWASP Juice Shop | THM | ☐ |

### 5.3 提权专项

| # | 靶机 | 平台 | 完成 |
|---|---|---|---|
| 1 | Blue（永恒之蓝） | THM | ☐ |
| 2 | Kenobi | THM | ☐ |
| 3 | Steel Mountain | THM | ☐ |
| 4 | Alfred | THM | ☐ |
| 5 | HackPark | THM | ☐ |
| 6 | Skynet | THM | ☐ |
| 7 | Daily Bugle | THM | ☐ |
| 8 | Internal | THM | ☐ |

### 5.4 自由刷题（目标 30+ 台）

| # | 靶机名 | 平台 | 完成 |
|---|---|---|---|
| 1 | | | ☐ |
| 2 | | | ☐ |
| … | | | ☐ |

---

## 六、进度统计

| 阶段 | 总题量 | 已完成 | 进度 |
|---|---|---|---|
| 1 Linux/网络 | 34 + 8 | __ | __% |
| 2 Web 渗透 | 272 + 20 + 100 | __ | __% |
| 3 Pwn/逆向 | 8 + 20 + 30 + 6 | __ | __% |
| 4 内网/AD | 13 + 14 | __ | __% |
| 5 实战 | 30+ | __ | __% |

---

## 七、里程碑对照

| 周 | 里程碑 | 达成 |
|---|---|---|
| 第 3 周 | Bandit 全通 | ☐ |
| 第 9 周 | PortSwigger Apprentice 全通 + Juice Shop 全解 | ☐ |
| 第 15 周 | ROP Emporium 全通 + crackmes 30 题 | ☐ |
| 第 20 周 | 本地 AD 环境拿下 DC | ☐ |
| 第 26 周 | 30+ 靶机 walkthrough | ☐ |

---

*本汇总位于 `runs/cyber-roadmap/docs/CHECKLIST.md`*
