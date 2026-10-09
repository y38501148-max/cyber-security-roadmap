# 学习路线总图

> 配套教材见 [docs/](docs/README.md) · 环境搭建见 [SETUP.md](SETUP.md) · 资源大全见 [RESOURCES.md](RESOURCES.md)

---

## 0. 核心判断

**你缺的不是付费订阅，是执行顺序。**

一套完整的网络安全实验环境，完全可以用免费资源搭建：

- 在线实验平台（PortSwigger / OverTheWire / pwn.college）负责"跟着教程走"
- 本地 Docker 靶场（vulhub / Juice Shop / DVWA）负责"无限刷"
- 自建虚拟机（AD 域环境）负责"真实内网演练"
- 公开靶机（VulnHub / HackMyVM / HTB 免费机）负责"综合实战"

全部零成本。

---

## 1. 环境要求（最低配置）

| 项目 | 最低 | 推荐 |
|---|---|---|
| 内存 | 8 GB | 16 GB+ |
| 磁盘 | 50 GB | 100 GB+ |
| CPU | 双核 | 四核+ |
| 系统 | Linux / macOS / Windows | — |

**必需软件**：Docker（跑本地靶场）、SSH 客户端、浏览器。

**内网阶段额外需要**：虚拟化软件 + Windows Server 镜像（见 [SETUP.md](SETUP.md)）。

---

## 2. 总时间轴（每周 10–20 小时）

```
阶段 1  Linux + 网络基础        ──  3 周
阶段 2  Web 渗透主力 ★          ──  6 周   （本路线重心）
阶段 3  Pwn / 二进制逆向        ──  6 周
阶段 4  内网 / AD 渗透          ──  5 周
阶段 5  实战化 + 输出           ──  持续
```

**建议节奏**：每周 12 小时左右 = 2 次 × 4h（实战）+ 2 次 × 2h（理论/复盘）。

**每周日固定写一篇 walkthrough** —— 这是把"做过"变成"会做"的唯一办法。

---

## 3. 阶段 1：Linux + 网络基础（3 周）

**目标**：能脱离鼠标在 Linux 里干活；看懂一次网络请求到底发生了什么。

### 主线

- **OverTheWire Bandit**（34 关，SSH 直连，无需注册）
  ```bash
  ssh bandit0@bandit.labs.overthewire.org -p 2220   # 密码：bandit0
  ```

### 知识点

| 主题 | 内容 |
|---|---|
| 文件与权限 | `ls -l` 权限位、`chmod`、SUID、`find` 组合条件 |
| 文本处理 | `grep` / `sort` / `uniq` / `tr` / `xxd` / `strings` |
| 编码 | Base64、ROT13、多层压缩包 |
| 网络工具 | `nc`、`openssl s_client`、`ssh -i`、`nmap` |
| 系统机制 | `cron`、`/etc/passwd`、shell 逃逸 |
| 网络原理 | TCP 三次握手、HTTP 请求/响应、状态码 |
| 版本控制 | Git 历史、分支、tag |

详细讲解 → [docs/01-linux-and-network.md](docs/01-linux-and-network.md)

### 验收标准

- [ ] Bandit 34 关全部通关
- [ ] 能解释 `/etc/passwd` 每一列的含义
- [ ] 能用 `find` 按名字/大小/权限查找文件
- [ ] 能说出 TCP 三次握手、常见 HTTP 状态码的含义

---

## 4. 阶段 2：Web 渗透主力 ★（6 周，本路线重心）

**目标**：拿到一个网站 URL，能独立完成「信息收集 → 漏洞验证 → 利用 → 写报告」。

### 为什么先学 Web

- 免费资源最多（**PortSwigger Academy 有 272 个免费在线实验**）
- 上手最快（浏览器即可开始）
- 就业需求最大

### 主线：PortSwigger Web Security Academy

https://portswigger.net/web-security —— 免费注册，实验无限次使用。

**核心模块优先级**：

| 优先级 | 模块 | 实验数 |
|---|---|---|
| ★★★★★ | SQL 注入 | 18 |
| ★★★★★ | XSS 跨站脚本 | 30 |
| ★★★★★ | 认证漏洞 | 14 |
| ★★★★★ | 越权/访问控制 | 13 |
| ★★★★★ | OS 命令注入 | 5 |
| ★★★★ | CSRF | 11 |
| ★★★★ | 文件上传 | 7 |
| ★★★★ | SSRF | 7 |
| ★★★★ | SSTI | 7 |
| ★★★★ | JWT | 8 |
| ★★★★ | 业务逻辑漏洞 | 12 |
| ★★★★ | 路径穿越 | 6 |
| ★★★ | XXE | 9 |
| ★★★ | 信息泄露 | 5 |
| ★★★ | 其他 14 个模块 | 约 100 |

### 并行：本地靶场

| 靶场 | 用途 |
|---|---|
| **vulhub** | 复现真实 CVE（300+ 环境） |
| **OWASP Juice Shop** | 现代 Web 综合靶场（100+ 挑战） |
| **DVWA / bWAPP / WebGoat** | 基础漏洞逐个练 |

详细讲解 → [docs/02-web-pentesting.md](docs/02-web-pentesting.md)

### 验收标准

- [ ] PortSwigger 初级（Apprentice）实验全部完成
- [ ] Practitioner 级别完成 70%
- [ ] Juice Shop 完成 50+ 挑战
- [ ] 复现 20 个 CVE 环境并写清利用链
- [ ] 能独立完成一次完整 Web 渗透并写报告

---

## 5. 阶段 3：Pwn / 二进制逆向（6 周）

**目标**：能读懂汇编、写 ROP 链、拿下 CTF Pwn 题。

### 主线

| 平台 | 内容 |
|---|---|
| **ROP Emporium** | 8 关 ROP 渐进教学（可下载全量） |
| **crackmes.one** | 4700+ 逆向练习题 |
| **Exploit Education** | Phoenix/Nebula 系统化系列 |
| **pwn.college** | 完整课程 + 视频 |
| **pwnable.tw / pwnable.kr** | 经典 Pwn 靶场 |
| **Nightmare 讲义** | 免费 Pwn 教程 |

### 知识点

| 主题 | 内容 |
|---|---|
| 内存布局 | 栈/堆/BSS/Text、增长方向 |
| 寄存器 | RDI/RSI/RDX、调用约定、RIP 控制 |
| 保护机制 | NX / Canary / PIE / ASLR / RELRO |
| 栈溢出 | 偏移计算、返回地址覆盖 |
| ROP | gadget 查找、PLT/GOT、栈对齐、栈迁移 |
| ret2csu | 通用 gadget、寄存器传递限制 |
| 逆向工具 | radare2 / gdb / pwntools / ROPgadget |

详细讲解 → [docs/03-pwn-and-reversing.md](docs/03-pwn-and-reversing.md)

### 验收标准

- [ ] ROP Emporium 8 关全部通关
- [ ] crackmes.one 完成 30 题
- [ ] pwnable 平台通过 3 题
- [ ] 能独立写 pwntools 脚本拿下栈溢出题目

---

## 6. 阶段 4：内网 / Active Directory（5 周）

**目标**：拿到一台域内主机后，能枚举、提权、横向，最终拿下域控。

### 主线

- 自建最小 AD 域环境（Windows Server + 客户端，见 SETUP.md）
- 在线 AD 靶场房间（入门理论）

### 知识点

| 主题 | 内容 |
|---|---|
| AD 概念 | 域/域控/OU/GPO/SPN/SID |
| Kerberos | AS-REQ/AS-REP/TGS-REQ/TGS-REP 完整流程 |
| NTLM | 挑战-应答、NTLM 哈希、Pass-the-Hash |
| 凭据获取 | AS-REP Roasting、Kerberoasting、密码喷洒 |
| 枚举构图 | BloodHound、ACL 关系、最短攻击路径 |
| 域内提权 | AD CS（ESC1-11）、GPP 密码、LAPS、ACL 滥用 |
| 横向移动 | WinRM / WMI / PsExec / SMBExec / atexec |
| 拿下域控 | DCSync、黄金票据、白银票据、NTLM 中继 |
| 权限维持 | AdminSDHolder、SIDHistory、DSRM 后门 |
| 隧道代理 | chisel / frp / proxychains |

详细讲解 → [docs/04-internal-network-and-ad.md](docs/04-internal-network-and-ad.md)

### 验收标准

- [ ] 能解释 Kerberos 认证的完整流程
- [ ] 完成一次 AS-REP Roasting 与 Kerberoasting
- [ ] 能用 5 种方式做横向移动
- [ ] 完成一次 DCSync 并伪造黄金票据
- [ ] 本地自建域环境完整走通：枚举 → 提权 → 横向 → 拿下 DC

---

## 7. 阶段 5：实战化 + 输出（持续）

**目标**：把学到的技能串成完整渗透链，形成可展示的作品集。

### 靶机来源

| 平台 | 数量 | 特点 |
|---|---|---|
| VulnHub | 数百 | 离线可跑 |
| HackMyVM | 300+ | 无需账号 |
| Hack The Box | 每周 2 台免费 | 质量高 |
| TryHackMe | 数百免费房间 | 有引导 |

### 五阶段渗透法

```
① 信息收集 → ② 立足点 → ③ 提权 → ④ 横向（如有）→ ⑤ 写报告
```

### 必须产出

- 每台靶机一篇 walkthrough
- 建立个人作品集仓库
- 每季度更新技能矩阵

详细讲解 → [docs/05-practice-and-output.md](docs/05-practice-and-output.md)

### 验收标准

- [ ] 能独立完成 Easy 靶机的完整渗透链
- [ ] 完成 20 台以上靶机并全部留档
- [ ] 参加过至少 1 场 CTF
- [ ] 建立了自己的作品集仓库

---

## 8. 里程碑总表

| 周 | 里程碑 | 验收标准 |
|---|---|---|
| 第 3 周 | Linux/网络基础 | Bandit 34 关全通 |
| 第 9 周 | Web 渗透主力 | PortSwigger 初级全通 + Juice Shop 50+ |
| 第 15 周 | Pwn/逆向基础 | ROP Emporium 8 关 + crackmes 30 题 |
| 第 20 周 | 内网/AD | 本地域环境完整拿下 DC |
| 第 26 周+ | 实战化 | 20+ 台靶机 walkthrough |

---

## 9. 避坑清单

1. **不要 ping 靶机**：多数在线靶机禁 ICMP，用 `nmap -Pn`。
2. **靶机有时限**：启动后通常 1 小时自动销毁，先规划再启动。
3. **免费 VPN 配置会过期**：按平台说明重新下载。
4. **部分工具输出选项被包装层拦截**：改用 shell 重定向 `> out.txt`。
5. **容器内工具缺失**：按 [SETUP.md](SETUP.md) 补装。
6. **端口冲突**：`docker compose down` 清理，或改端口映射。
7. **ARM 设备注意**：部分 x86 专用镜像跑不了，优先选 multi-arch 镜像。
8. **别贪多**：一个阶段没通过验收，不要跳下一阶段。

---

## 10. 相关文档

- [docs/](docs/README.md) —— 5 章详细教材 + 题单
- [SETUP.md](SETUP.md) —— 实验环境搭建
- [RESOURCES.md](RESOURCES.md) —— 免费资源大全
