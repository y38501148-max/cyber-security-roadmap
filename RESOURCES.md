# 免费靶场与资源大全

> 全部资源**免费可访问**，按类别整理。链接均经过验证。

---

## 目录

- [1. 综合训练平台](#1-综合训练平台)
- [2. Web 安全](#2-web-安全)
- [3. 二进制 / Pwn / 逆向](#3-二进制--pwn--逆向)
- [4. 内网 / Active Directory](#4-内网--active-directory)
- [5. 蓝队 / SOC / 取证](#5-蓝队--soc--取证)
- [6. 密码学 / 杂项](#6-密码学--杂项)
- [7. CTF 比赛与题库](#7-ctf-比赛与题库)
- [8. 自建靶场](#8-自建靶场)
- [9. 学习资料](#9-学习资料)
- [10. 工具清单](#10-工具清单)

---

## 1. 综合训练平台

| 平台 | 免费程度 | 特点 | 地址 |
|---|---|---|---|
| **TryHackMe** | 数百免费房间 | 有引导，适合入门 | https://tryhackme.com |
| **Hack The Box** | 每周 2 台免费机 + Starting Point | 质量高 | https://www.hackthebox.com |
| **HackMyVM** | 300+ VM 全免费 | 无需账号 | https://hackmyvm.eu |
| **VulnHub** | 数百台全免费 | 离线可跑 | https://www.vulnhub.com |
| **Root-Me** | 全类型挑战 | 覆盖面广 | https://www.root-me.org |
| **echoCTF** | 开源主动防御靶场 | 可自建 | https://echoctf.red |
| **BTLO** | 蓝队免费挑战 | SOC 向 | https://blueteamlabs.online |
| **AttackDefense** | 轮换免费实验 | 免搭环境 | https://attackdefense.com |

---

## 2. Web 安全

| 平台 | 内容 | 地址 |
|---|---|---|
| **PortSwigger Web Security Academy** ⭐ | **272 个免费在线实验** + 官方讲义 | https://portswigger.net/web-security |
| **OWASP Juice Shop** | 现代 Web 靶场（可自建） | https://owasp.org/www-project-juice-shop/ |
| **DVWA** | 经典基础漏洞靶场（可自建） | https://github.com/digininja/DVWA |
| **bWAPP** | 100+ Web 漏洞（可自建） | https://github.com/raesene/bWAPP |
| **WebGoat** | OWASP 教学靶场（可自建） | https://owasp.org/www-project-webgoat/ |
| **Google XSS Game** | XSS 入门小游戏 | https://xss-game.appspot.com |
| **HackThisSite** | 老牌 Web 挑战 | https://www.hackthissite.org |
| **PentesterLab** | 部分免费 Web 练习 | https://pentesterlab.com |
| **prompt.ml** | XSS 挑战 | https://prompt.ml |
| **alert(1) to win** | XSS 挑战集 | https://alf.nu/alert1 |

### 本地 Web 靶场（Docker）

```bash
docker run -d --rm -p 3000:3000 bkimminich/juice-shop          # Juice Shop
docker run -d --rm -p 8080:80   vulnerables/web-dvwa           # DVWA
docker run -d --rm -p 8081:80   raesene/bwapp                  # bWAPP
docker run -d --rm -p 8082:8080 webgoat/goatandwolf            # WebGoat
```

---

## 3. 二进制 / Pwn / 逆向

| 平台 | 内容 | 地址 |
|---|---|---|
| **ROP Emporium** ⭐ | 8 关 ROP 渐进教学，可下载全量 | https://ropemporium.com |
| **crackmes.one** | 4700+ 逆向练习题 | https://crackmes.one |
| **pwn.college** | 系统化二进制课程（含视频） | https://pwn.college |
| **Exploit Education** | Phoenix/Nebula 系列 | https://exploit.education |
| **pwnable.tw** | 经典 Pwn 靶场 | https://pwnable.tw |
| **pwnable.kr** | 题型全面 | http://pwnable.kr |
| **Microcorruption** | 浏览器内汇编 CTF | https://microcorruption.com |
| **Nightmare** | 免费 Pwn 讲义 | https://guyinatuxedo.github.io |
| **ir0nstone notes** | 二进制利用笔记 | https://ir0nstone.gitbook.io/notes |
| **how2heap** | 堆利用技术合集 | https://github.com/shellphish/how2heap |
| **picoCTF** | 永久题库 | https://picoctf.org |
| **CTF Wiki** | 中文 CTF 百科 | https://ctf-wiki.org |

> **crackmes.one 下载密码**：所有压缩包统一为 `crackmes.one`

---

## 4. 内网 / Active Directory

| 资源 | 内容 | 地址 |
|---|---|---|
| **BloodHound** | AD 攻击路径分析 | https://github.com/BloodHoundAD/BloodHound |
| **impacket** | 内网渗透工具集 | https://github.com/fortra/impacket |
| **NetExec** | 多协议批量操作 | https://github.com/Pennyw0rth/NetExec |
| **Certipy** | AD CS 攻击 | https://github.com/ly4k/Certipy |
| **PingCastle** | AD 安全评估 | https://www.pingcastle.com |
| **ADSecurity** | AD 安全知识库 | https://adsecurity.org |
| **The Hacker Recipes** | AD 攻击手册 | https://www.thehacker.recipes |
| **HackTricks** | 渗透技巧百科 | https://book.hacktricks.xyz |

### 自建 AD 环境

- Windows Server 评估版：https://www.microsoft.com/en-us/evalcenter/
- 搭建步骤见 [SETUP.md](SETUP.md) 第 7 节

---

## 5. 蓝队 / SOC / 取证

| 平台 | 内容 | 地址 |
|---|---|---|
| **CyberDefenders** | 蓝队/取证挑战 | https://cyberdefenders.org |
| **LetsDefend** | SOC 分析练习 | https://app.letsdefend.io/challenge |
| **Blue Team Labs Online** | 蓝队实验室 | https://blueteamlabs.online |
| **DFIR Training** | 数字取证资源 | https://www.dfir.training |
| **Volatility** | 内存取证框架 | https://github.com/volatilityfoundation/volatility3 |
| **Autopsy** | 磁盘取证 | https://www.autopsy.com |
| **Malware-Traffic-Analysis** | 恶意流量样本 | https://www.malware-traffic-analysis.net |
| **Blue Team Community** | 蓝队资源汇总 | https://github.com/fabacab/awesome-cybersecurity-blueteam |

---

## 6. 密码学 / 杂项

| 平台 | 内容 | 地址 |
|---|---|---|
| **CryptoHack** | 密码学专项 | https://cryptohack.org |
| **Cryptopals** | 经典密码学挑战 | https://cryptopals.com |
| **OverTheWire** | 多系列 wargame | https://overthewire.org/wargames/ |
| **WeChall** | 多平台积分汇总 | https://www.wechall.net |
| **Reversing.kr** | 逆向挑战 | http://reversing.kr |
| **OSINT Dojo** | OSINT 练习 | https://www.osintdojo.com |

### OverTheWire 系列

| 系列 | 难度 | 内容 |
|---|---|---|
| Bandit | 入门 | Linux 基础（34 关） |
| Natas | 入门 | Web 安全 |
| Leviathan | 进阶 | 综合 |
| Krypton | 入门 | 密码学 |
| Narnia | 进阶 | 二进制基础 |
| Behemoth | 进阶 | 二进制 |
| Utumno | 高级 | 二进制 |
| Maze | 专家 | 综合 |

连接：`ssh <level>@<series>.labs.overthewire.org -p 2220`

---

## 7. CTF 比赛与题库

| 平台 | 内容 | 地址 |
|---|---|---|
| **CTFtime** | 全球 CTF 赛历 ⭐ | https://ctftime.org |
| **picoCTF** | 新手向，永久题库 | https://picoctf.org |
| **CTFHub** | 中文技能树靶场 | https://www.ctfhub.com |
| **BUUCTF** | 题量巨大的中文题库 | https://buuoj.cn |
| **DASCTF** | 中文平台 | https://ctf2.dasctf.com |
| **Bugku** | 中文入门 CTF | https://ctf.bugku.com |
| **攻防世界** | 中文综合 | https://adworld.xctf.org.cn |
| **Hackergame** | 中科大 CTF | https://hack.lug.ustc.edu.cn |

### 国内安全社区

| 社区 | 内容 |
|---|---|
| 先知社区 | https://xz.aliyun.com |
| 安全客 | https://www.anquanke.com |
| FreeBuf | https://www.freebuf.com |
| 看雪论坛 | https://bbs.kanxue.com |
| 吾爱破解 | https://www.52pojie.cn |

---

## 8. 自建靶场

| 靶场 | 部署方式 | 方向 |
|---|---|---|
| **OWASP Juice Shop** | `docker run -p 3000:3000 bkimminich/juice-shop` | Web 综合 |
| **DVWA** | `docker run -p 8080:80 vulnerables/web-dvwa` | Web 基础 |
| **bWAPP** | `docker run -p 8081:80 raesene/bwapp` | Web 101 |
| **WebGoat** | `docker run -p 8082:8080 webgoat/goatandwolf` | Web 教学 |
| **vulhub** | `git clone https://github.com/vulhub/vulhub` | CVE 复现（300+） |
| **Metasploitable2** | VulnHub 下载 OVA | 系统渗透 |
| **Metasploitable3** | GitHub 脚本构建 | 双系统靶机 |
| **Vulnerable AD** | 见 SETUP.md | AD 渗透 |
| **VulnHub 系列** | 下载 OVA 导入虚拟机 | 完整渗透链 |

### vulhub 常用环境

```bash
git clone --depth 1 https://github.com/vulhub/vulhub.git
cd vulhub/<应用>/<CVE目录>
docker compose up -d
# 攻击后
docker compose down
```

推荐复现：`flask/ssti`、`fastjson`、`log4j`、`shiro`、`struts2`、
`weblogic`、`spring`、`redis`、`docker`、`jenkins`、`gitlab`、`drupal`。

---

## 9. 学习资料

| 资料 | 内容 | 地址 |
|---|---|---|
| **HackTricks** ⭐ | 渗透测试百科（中英） | https://book.hacktricks.xyz |
| **PayloadsAllTheThings** | Payload 大全 | https://github.com/swisskyrepo/PayloadsAllTheThings |
| **The Hacker Recipes** | AD/内网手册 | https://www.thehacker.recipes |
| **OWASP WSTG** | Web 测试指南 | https://owasp.org/www-project-web-security-testing-guide/ |
| **MITRE ATT&CK** | 攻击技战术知识库 | https://attack.mitre.org |
| **GTFOBins** | Linux 提权二进制 | https://gtfobins.github.io |
| **LOLBAS** | Windows 提权/执行 | https://lolbas-project.github.io |
| **SecLists** | 字典合集 | https://github.com/danielmiessler/SecLists |
| **PentestMonkey** | 反弹 shell 速查 | https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet |
| **Reverse Shell Generator** | 在线生成反弹 shell | https://www.revshells.com |
| **CyberChef** | 在线编解码工具 | https://gchq.github.io/CyberChef |
| **Exploit-DB** | 公开漏洞库 | https://www.exploit-db.com |
| **CVE Details** | 漏洞详情 | https://www.cvedetails.com |

---

## 10. 工具清单

### 信息收集

| 工具 | 用途 |
|---|---|
| nmap / masscan | 端口扫描 |
| subfinder / amass | 子域枚举 |
| httpx / dnsx | 存活探测 |
| whatweb / wappalyzer | 指纹识别 |
| theHarvester | 邮箱/子域收集 |

### Web 渗透

| 工具 | 用途 |
|---|---|
| Burp Suite | 抓包/改包（核心） |
| ffuf / gobuster / feroxbuster | 目录爆破 |
| sqlmap | SQL 注入自动化 |
| nuclei | 模板化漏洞扫描 |
| XSStrike | XSS 检测 |
| wpscan | WordPress 扫描 |
| nikto | Web 服务器扫描 |

### 二进制

| 工具 | 用途 |
|---|---|
| radare2 / rizin | 反汇编 |
| gdb + pwndbg/GEF | 调试 |
| Ghidra / IDA | 反编译 |
| pwntools | Exploit 开发 |
| ROPgadget / ropper | gadget 查找 |
| checksec | 保护机制检查 |
| binwalk | 固件/文件分析 |

### 内网

| 工具 | 用途 |
|---|---|
| impacket | 协议攻击套件 |
| NetExec (nxc) | 多协议批量操作 |
| BloodHound | AD 路径分析 |
| mimikatz | 凭据提取（Windows） |
| Responder | NTLM 投毒 |
| Certipy | AD CS 攻击 |
| enum4linux-ng | SMB/LDAP 枚举 |

### 隧道代理

| 工具 | 用途 |
|---|---|
| chisel | HTTP 隧道 |
| frp | 内网穿透 |
| proxychains-ng | 代理链 |
| socat / netcat | 端口转发、shell |

### 密码

| 工具 | 用途 |
|---|---|
| hashcat | GPU 哈希破解 |
| john | CPU 哈希破解 |
| hydra / medusa | 在线爆破 |
| hashid | 哈希类型识别 |
| CyberChef | 编解码 |

---

## 免责声明

以上所有平台与工具**仅供网络安全学习与授权测试**使用。

请仅在你**拥有或获得明确授权**的系统上使用相关技术。
对未授权系统使用这些技术可能违法，后果自负。

---

## 相关文档

- [ROADMAP.md](ROADMAP.md) —— 学习路线总图
- [SETUP.md](SETUP.md) —— 实验环境搭建
- [docs/](docs/README.md) —— 详细教材
