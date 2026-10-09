# 教材索引

> 配套总路线：[ROADMAP.md](../ROADMAP.md) · 环境搭建：[SETUP.md](../SETUP.md) · 资源大全：[RESOURCES.md](../RESOURCES.md)
> 面向：**零基础初学者**
> 目标：每一章都有「**原理讲清 → 命令给全 → 题目做完**」三件套，照着做就能学会。

---

## 这套文档怎么用

1. 先读 [ROADMAP.md](../ROADMAP.md) 看全局（阶段划分、时间轴、里程碑）。
2. 按 [SETUP.md](../SETUP.md) 把实验环境搭起来。
3. 然后按下面顺序，一章一章做。**每章都先讲原理，再给命令，最后列题目。**
4. 每章末尾有「**自测清单**」，全部打勾才算过关，再进下一章。
5. 每章有「**卡住了怎么办**」，按提示查，不要直接看答案。

> 学习原则：**看懂 ≠ 会做**。必须自己敲命令、自己踩坑、自己写笔记。

---

## 章节目录

| 章节 | 文档 | 讲解重点 | 预计用时 | 题量 |
|---|---|---|---|---|
| 第 1 章 | [01-linux-and-network.md](01-linux-and-network.md) | Linux 权限模型、SUID 提权原理、TCP/IP、HTTP；Bandit 逐关详解 | 3 周 | 34 关 + 8 房间 |
| 第 2 章 ★ | [02-web-pentesting.md](02-web-pentesting.md) | 30 类 Web 漏洞的成因/数据流/检测/利用/防护，全部 272 个在线实验 | 6 周 | 272 + 20 + 100 |
| 第 3 章 | [03-pwn-and-reversing.md](03-pwn-and-reversing.md) | 汇编基础、栈溢出内存推演、ROP 原理与逐关 exploit 代码 | 6 周 | 8 + 30 + 6 |
| 第 4 章 | [04-internal-network-and-ad.md](04-internal-network-and-ad.md) | Kerberos 全流程、NTLM、四大票据攻击、横向移动、AD CS | 5 周 | 13 + 14 |
| 第 5 章 | [05-practice-and-output.md](05-practice-and-output.md) | 渗透思维方法论、靶机清单、报告模板、作品集建设 | 持续 | 30+ 台 |

---

## 附录

| 文档 | 内容 |
|---|---|
| [CHECKLIST.md](CHECKLIST.md) | **全部题目可勾选总表**（可打印） |

---

## 学习节奏建议

```
每天至少 1 小时，重在持续。
做不出题不要死磕超过 40 分钟 —— 记下来，跳过，第二天再看。
每周日：把本周做的题写成 walkthrough。
```

## 遇到问题的排查顺序

1. **看报错信息**：90% 的答案就在报错里。
2. **`man <命令>`**：Linux 命令的第一手文档。
3. **搜索引擎**：把报错原文粘贴进去。
4. **HackTricks**（https://book.hacktricks.xyz）：渗透技巧百科。
5. **官方文档 / Writeup**：卡超过 40 分钟再看，看之前先记录自己的思路。
6. **拆解问题**：把大问题拆成小问题，逐个验证假设。

---

## 题目来源说明

本教材引用的所有题目名称均来自公开的免费平台，并经过逐条核实：

- **PortSwigger Web Security Academy**：272 个在线实验（免费，无需付费）
- **OverTheWire Bandit**：34 关（免费，SSH 直连）
- **ROP Emporium**：8 关（免费，可下载全量）
- **VulnHub / HackMyVM / HTB / TryHackMe**：免费靶机

完整资源列表见 [RESOURCES.md](../RESOURCES.md)。
