# Cyber Security Roadmap · 零基础网络安全学习路线

> 一份面向**零基础初学者**的网络安全自学路线：从 Linux 命令到 Web 渗透，从二进制利用到内网域渗透。
> **全部资源免费**，含 400+ 道习题清单、逐条讲解的原理、以及可复现的本地实验环境搭建教程。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 这套资料解决什么问题

网上网络安全的学习资料要么太散（一堆链接），要么太浅（只会教你怎么敲命令）。
这套资料的定位是：

- **有路线**：从零到能独立打靶机，分 5 个阶段，每阶段有明确目标与验收标准
- **有讲解**：不只给命令，还讲**为什么**——协议原理、内存布局、攻击链的每一步推演
- **有题单**：配套 **400+ 道真实可做的题目**（已逐条核实题目名，无死链）
- **有环境**：本地如何搭建免费的实验环境（Docker / 虚拟机 / 自建域环境）
- **全免费**：不依赖任何付费订阅

---

## 学习路线总览

```
阶段 1  Linux + 网络基础        ──  3 周   OverTheWire Bandit 34 关 + 在线课程房间
阶段 2  Web 渗透主力 ★          ──  6 周   PortSwigger 272 个免费实验 + 本地 CVE 靶场
阶段 3  Pwn / 二进制逆向        ──  6 周   ROP Emporium + crackmes + pwnable
阶段 4  内网 / Active Directory ──  5 周   自建域环境 + Kerberos 攻击链
阶段 5  实战化 + 输出           ──  持续   30+ 台靶机 + 渗透报告
```

> 建议投入：每周 10–20 小时，约 6 个月完成主线。

---

## 目录

### 起点

| 文档 | 内容 |
|---|---|
| [ROADMAP.md](ROADMAP.md) | **学习路线总图**：环境要求、时间轴、里程碑、避坑清单 |
| [SETUP.md](SETUP.md) | **实验环境搭建教程**：Docker / 攻击容器 / 工具链 / 靶场部署 |
| [RESOURCES.md](RESOURCES.md) | **免费靶场与资源大全**：60+ 平台，按类型与难度分类 |

### 教材（每章 = 原理 + 命令 + 题单）

| 章节 | 文档 | 讲解重点 | 题量 |
|---|---|---|---|
| 第 1 章 | [docs/01-linux-and-network.md](docs/01-linux-and-network.md) | Linux 权限模型、TCP/IP、HTTP；Bandit 逐关详解 | 34 关 + 8 房间 |
| 第 2 章 ★ | [docs/02-web-pentesting.md](docs/02-web-pentesting.md) | 30 类 Web 漏洞原理 + Payload + 全部 272 个在线实验 | 272 + 20 + 100 |
| 第 3 章 | [docs/03-pwn-and-reversing.md](docs/03-pwn-and-reversing.md) | 汇编基础、栈溢出内存推演、ROP 原理与 exploit 代码 | 8 + 30 + 6 |
| 第 4 章 | [docs/04-internal-network-and-ad.md](docs/04-internal-network-and-ad.md) | Kerberos 认证流程、NTLM、四大票据攻击、横向移动 | 13 + 14 |
| 第 5 章 | [docs/05-practice-and-output.md](docs/05-practice-and-output.md) | 靶机清单、渗透流程、报告模板、作品集建设 | 30+ 台 |
| 题单 | [docs/CHECKLIST.md](docs/CHECKLIST.md) | **全部题目可勾选总表**（可打印） | 全部 |

---

## 快速开始

```bash
# 1. 克隆
git clone https://github.com/<your-name>/cyber-security-roadmap.git
cd cyber-security-roadmap

# 2. 先读路线图
cat ROADMAP.md

# 3. 搭环境（按 SETUP.md 操作）
cat SETUP.md

# 4. 从第 1 章开始
cat docs/01-linux-and-network.md
```

**第一件要做的事**：注册 [OverTheWire](https://overthewire.org/wargames/bandit/)，SSH 连上 Bandit 第 0 关：

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220   # 密码：bandit0
```

---

## 题目来源（全部免费、已验证可访问）

| 平台 | 内容 | 地址 |
|---|---|---|
| **PortSwigger Web Security Academy** | 272 个在线 Web 安全实验 | https://portswigger.net/web-security |
| **OverTheWire** | Bandit/Natas/Leviathan 等 wargame | https://overthewire.org/wargames/ |
| **ROP Emporium** | 8 关 ROP 渐进教学 | https://ropemporium.com |
| **crackmes.one** | 4700+ 逆向练习题 | https://crackmes.one |
| **pwn.college** | 系统化二进制课程 | https://pwn.college |
| **VulnHub** | 离线可跑的靶机 | https://www.vulnhub.com |
| **HackMyVM** | 300+ 免费靶机 | https://hackmyvm.eu |
| **Hack The Box / TryHackMe** | 免费房间与机器 | — |
| **OWASP Juice Shop / DVWA / WebGoat** | 自建 Web 靶场 | 见 SETUP.md |

完整列表（60+ 平台）见 [RESOURCES.md](RESOURCES.md)。

---

## 学习原则

1. **看懂 ≠ 会做**。必须自己敲命令、自己踩坑。
2. **卡住不超过 40 分钟**。记下来，跳过，第二天再看。
3. **每周必写一篇 walkthrough**。没有输出的学习等于没学。
4. **一个阶段没通过验收，不要跳下一阶段**。

---

## 环境要求

最低配置：

- 8 GB 内存（推荐 16 GB+）
- 50 GB 可用磁盘
- Docker（用于本地靶场）
- 任意 Linux / macOS / Windows

可选（内网阶段需要）：

- 虚拟化软件（UTM / VMware / VirtualBox）
- Windows Server + Windows 客户端镜像（搭 AD 环境）

详见 [SETUP.md](SETUP.md)。

---

## 贡献

欢迎提交勘误、补充题目、改进讲解。见 [CONTRIBUTING.md](CONTRIBUTING.md)。

特别注意：**不要提交任何真实的靶标信息、凭据、或个人隐私数据**。

---

## 免责声明

本资料仅供**网络安全学习与授权测试**使用。

- 所有实验均在**自有环境或公开的合法靶场**中进行
- 请勿对未授权的系统使用本资料中的技术
- 使用者需自行遵守所在地法律法规

---

## License

[MIT](LICENSE)
