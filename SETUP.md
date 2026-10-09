# 实验环境搭建教程

> 本文教你在自己的机器上搭一套完整的免费网络安全实验环境。
> 全程零成本，支持 Linux / macOS / Windows。

---

## 目录

1. [总体架构](#1-总体架构)
2. [Docker 安装](#2-docker-安装)
3. [攻击容器（Kali）](#3-攻击容器kali)
4. [工具链安装](#4-工具链安装)
5. [本地 Web 靶场](#5-本地-web-靶场)
6. [vulhub CVE 靶场](#6-vulhub-cve-靶场)
7. [虚拟机与 AD 域环境](#7-虚拟机与-ad-域环境)（内网阶段）
8. [自检脚本](#8-自检脚本)
9. [常见问题](#9-常见问题)

---

## 1. 总体架构

推荐架构：

```
宿主机
├── Docker
│   ├── 攻击容器（Kali Linux，含全套工具）
│   ├── Web 靶场（Juice Shop / DVWA / WebGoat）
│   └── vulhub 环境（按需启停）
├── 虚拟机（可选，内网阶段）
│   ├── Windows Server（域控）
│   └── Windows 客户端（成员机）
└── 浏览器 + 在线靶场（PortSwigger 等）
```

**好处**：攻击环境与靶场隔离，随时启停，不污染宿主机。

---

## 2. Docker 安装

### Linux

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER   # 免 sudo 运行
newgrp docker
docker version
```

### macOS

```bash
brew install --cask docker
open -a Docker      # 启动 Docker Desktop
```

### Windows

下载安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/)，
安装时勾选 WSL2 后端。

### 验证

```bash
docker run --rm hello-world
```

---

## 3. 攻击容器（Kali）

Kali 官方提供精简镜像，可再补装工具。也可以直接用下面的方式构建。

### 3.1 拉取 Kali 镜像

```bash
docker pull kalilinux/kali-rolling
```

### 3.2 启动攻击容器

```bash
docker run -dit --name kali \
  --cap-add=NET_ADMIN --cap-add=NET_RAW \
  --device /dev/net/tun \
  -v "$HOME/loot:/loot" \
  kalilinux/kali-rolling bash

docker exec -it kali bash
```

**参数说明**：

| 参数 | 作用 |
|---|---|
| `--cap-add=NET_ADMIN` | 允许操作网络（VPN、路由） |
| `--cap-add=NET_RAW` | 允许原始套接字（nmap、抓包） |
| `--device /dev/net/tun` | 允许建立 VPN 隧道 |
| `-v $HOME/loot:/loot` | 挂载战利品目录到宿主机 |

### 3.3 进入容器

```bash
docker exec -it kali bash
```

---

## 4. 工具链安装

在攻击容器内执行（以 Kali 为例）：

### 4.1 基础工具

```bash
apt update
apt install -y \
  nmap masscan netcat-openbsd socat tcpdump \
  ffuf gobuster feroxbuster dirb nikto \
  sqlmap hydra medusa john hashcat \
  whatweb wpscan nuclei \
  seclists wordlists \
  git curl wget jq python3-pip
```

### 4.2 Metasploit

```bash
apt install -y metasploit-framework
msfconsole --version
```

### 4.3 逆向工具链

```bash
apt install -y radare2 gdb binwalk strace ltrace file
pip3 install --break-system-packages pwntools capstone r2pipe unicorn pyelftools keystone-engine
```

### 4.4 内网工具链

```bash
# 完整 impacket
apt install -y python3-impacket
# 若命令入口缺失，手动建立链接：
D=/usr/share/doc/python3-impacket/examples
for f in "$D"/*.py; do
  n=$(basename "$f" .py | tr '[:upper:]' '[:lower:]')
  ln -sf "$f" "/usr/local/bin/impacket-$n"
done
ln -sf "$D/GetUserSPNs.py" /usr/local/bin/impacket-GetUserSPNs
ln -sf "$D/GetNPUsers.py"  /usr/local/bin/impacket-GetNPUsers
ln -sf "$D/GetADUsers.py"  /usr/local/bin/impacket-GetADUsers
command -v python || ln -sf /usr/bin/python3 /usr/local/bin/python

# netexec（需要 Rust 编译其中依赖）
apt install -y rustc cargo
pip3 install --break-system-packages --ignore-installed \
  "git+https://github.com/Pennyw0rth/NetExec"

# AD 分析工具
apt install -y bloodhound.py enum4linux-ng
pip3 install --break-system-packages certipy-ad
```

> **注意**：pip 安装若遇网络问题，可配置镜像源：
> ```bash
> pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
> ```
> GitHub 拉取受阻时，可设置代理：
> ```bash
> export https_proxy=http://<PROXY_HOST>:<PORT> http_proxy=http://<PROXY_HOST>:<PORT>
> ```
> Docker 容器访问宿主机代理，通常可用 `host.docker.internal` 作为主机名。

### 4.5 字典

```bash
# rockyou 密码字典
apt install -y wordlists
gunzip /usr/share/wordlists/rockyou.txt.gz

# SecLists 综合字典
apt install -y seclists
ls /usr/share/seclists/
```

### 4.6 验证工具链

```bash
for t in nmap ffuf gobuster sqlmap hydra hashcat nuclei msfconsole \
         whatweb wpscan nikto radare2 gdb binwalk john \
         impacket-GetUserSPNs bloodhound-python nxc proxychains4; do
  printf "%-26s" "$t"
  command -v $t >/dev/null && echo OK || echo MISSING
done
```

---

## 5. 本地 Web 靶场

### 5.1 OWASP Juice Shop（推荐首选）

现代 Web 靶场，含 100+ 挑战（XSS、注入、越权、认证等）。

```bash
docker run -d --rm -p 3000:3000 --name juice-shop \
  bkimminich/juice-shop

# 访问 http://localhost:3000
```

### 5.2 DVWA（经典基础靶场）

```bash
docker run -d --rm -p 8080:80 --name dvwa vulnerables/web-dvwa
# 访问 http://localhost:8080
# 默认账号 admin / password
# 首次登录后点 "Create / Reset Database"
```

**练习顺序**：Brute Force → Command Injection → CSRF → File Inclusion →
File Upload → SQL Injection → SQL Injection (Blind) → XSS (Reflected/Stored/DOM)

难度依次设为 **Low → Medium → High**，体会防护绕过。

### 5.3 bWAPP

```bash
docker run -d --rm -p 8081:80 --name bwapp raesene/bwapp
# 访问 http://localhost:8081/install.php 初始化
```

### 5.4 WebGoat

```bash
docker run -d --rm -p 8082:8080 --name webgoat webgoat/goatandwolf
# 访问 http://localhost:8082/WebGoat
```

### 5.5 清理

```bash
docker ps                       # 查看运行中的容器
docker stop juice-shop dvwa     # 停止
docker rm -f $(docker ps -aq)   # 清空所有容器（谨慎）
```

---

## 6. vulhub CVE 靶场

[vulhub](https://github.com/vulhub/vulhub) 是一个开源的漏洞复现环境集合，
涵盖数百个真实 CVE，全部用 Docker Compose 一键启动。

### 6.1 获取

```bash
git clone --depth 1 https://github.com/vulhub/vulhub.git
cd vulhub
```

### 6.2 使用单个环境

```bash
# 以 fastjson 反序列化为例
cd fastjson/1.2.47-rce

docker compose up -d          # 启动
docker compose ps             # 查看映射端口
cat README.zh-cn.md           # 阅读官方说明

# 攻击 http://127.0.0.1:<端口>

docker compose down           # 清理
```

### 6.3 推荐复现顺序

| 批次 | 环境 | 学到什么 |
|---|---|---|
| 1 | `flask/ssti`、`php/`、`tomcat/` | 模板注入、文件包含、弱口令 |
| 2 | `fastjson/1.2.24-rce`、`log4j/` | Java 反序列化、JNDI |
| 3 | `weblogic/`、`struts2/` | 中间件 RCE |
| 4 | `shiro/`、`spring/` | 权限绕过 + 反序列化链 |
| 5 | `redis/`、`mysql/`、`docker/unauthorized-rce` | 未授权 → RCE |
| 6 | `gitlab/`、`jenkins/`、`nexus/` | CI/CD 攻击面 |
| 7 | `wordpress/`、`drupal/`、`joomla/` | CMS 漏洞 |
| 8 | `apisix/`、`nginx/`、`grafana/` | 网关/面板漏洞 |

### 6.4 注意事项

- 每个环境默认映射到 `127.0.0.1`，仅本机可访问，安全
- 用完**务必 `docker compose down`**，避免端口占用与资源浪费
- 部分环境需要较大内存，建议给 Docker 分配 4 GB+

---

## 7. 虚拟机与 AD 域环境

> 仅内网阶段（阶段 4）需要。可先用在线靶场替代。

### 7.1 虚拟化软件

| 系统 | 推荐 |
|---|---|
| macOS | UTM（免费）或 VMware Fusion |
| Windows | VMware Workstation / VirtualBox |
| Linux | KVM/QEMU 或 VirtualBox |

### 7.2 镜像来源

- Windows Server 评估版：[微软官方评估中心](https://www.microsoft.com/en-us/evalcenter/)
- Windows 客户端：同上
- Linux：各大发行版官网 ISO

### 7.3 搭建最小域环境

**目标拓扑**：

```
DC01      192.168.64.10   Windows Server（域控）
WS01      192.168.64.20   Windows 客户端（成员机）
ATTACKER  192.168.64.30   攻击容器 / Kali VM
```

**DC01 配置步骤**：

```
1. 安装 Windows Server，改主机名为 DC01
2. 设静态 IP：192.168.64.10，DNS 指向自己
3. 服务器管理器 → 添加角色和功能 → 勾选 "Active Directory 域服务"
4. 提升为域控：添加新林 → 根域名 lab.local → 设 DSRM 密码 → 重启
5. 建测试账号（用管理员 PowerShell 执行）：
```

```powershell
Import-Module ActiveDirectory

New-ADUser -Name "alice" -SamAccountName "alice" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true

New-ADUser -Name "bob" -SamAccountName "bob" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true

# 建服务账号（用于 Kerberoasting 练习）
New-ADUser -Name "svc_sql" -SamAccountName "svc_sql" `
  -AccountPassword (ConvertTo-SecureString "Summer2024!" -AsPlainText -Force) -Enabled $true
Set-ADUser -Identity svc_sql -ServicePrincipalNames @{Add="MSSQLSvc/sql.lab.local:1433"}

Add-ADGroupMember -Identity "Domain Admins" -Members "alice"
```

```
【WS01】
1. 安装 Windows 客户端，设 IP 192.168.64.20，DNS 指向 192.168.64.10
2. 系统属性 → 更改 → 域 → 输入 lab.local → 用域账号登录
```

**攻击机连通性验证**：

```bash
nmap -Pn -p 53,88,135,139,389,445,464,636,3268,5985 192.168.64.10
```

> **安全提示**：这些虚拟机请放在**隔离网络**中，不要桥接到生产网络。

---

## 8. 自检脚本

把下面的脚本保存为 `check-env.sh`，随时检查环境完整度。

```bash
#!/usr/bin/env bash
# 网络安全实验环境自检脚本
set -u
GREEN='\033[0;32m'; RED='\033[0;31m'; YEL='\033[0;33m'; NC='\033[0m'
ok()   { printf "${GREEN}OK${NC}      %s\n" "$1"; }
miss() { printf "${RED}MISSING${NC} %s\n" "$1"; }
warn() { printf "${YEL}WARN${NC}    %s\n" "$1"; }
hdr()  { printf "\n=== %s ===\n" "$1"; }

KALI="${KALI_CONTAINER:-kali}"

hdr "Docker"
docker version --format 'Docker: {{.Server.Version}}' 2>/dev/null || miss "Docker 引擎"

hdr "容器"
for c in "$KALI"; do
  if docker ps --format '{{.Names}}' | grep -qx "$c"; then
    ok "容器 $c 运行中"
  else
    warn "容器 $c 未运行"
  fi
done

hdr "攻击工具链"
if docker ps --format '{{.Names}}' | grep -qx "$KALI"; then
  for t in nmap masscan ffuf gobuster feroxbuster dirb nikto sqlmap hydra medusa \
           john hashcat msfconsole msfvenom nuclei whatweb wpscan tcpdump socat nc \
           proxychains4 radare2 r2 rabin2 gdb binwalk strings objdump readelf searchsploit; do
    docker exec "$KALI" bash -lc "command -v $t" >/dev/null 2>&1 && ok "$t" || miss "$t"
  done

  hdr "内网工具"
  for t in nxc bloodhound-python enum4linux-ng certipy \
           impacket-secretsdump impacket-wmiexec impacket-psexec \
           impacket-GetUserSPNs impacket-GetNPUsers impacket-smbclient \
           smbclient rpcclient ldapsearch proxychains4; do
    docker exec "$KALI" bash -lc "command -v $t" >/dev/null 2>&1 && ok "$t" || warn "$t"
  done

  hdr "逆向库"
  for m in pwn capstone r2pipe unicorn elftools keystone; do
    docker exec "$KALI" bash -lc "python3 -c 'import $m'" >/dev/null 2>&1 && ok "$m" || miss "$m"
  done

  hdr "字典"
  for w in /usr/share/wordlists/rockyou.txt.gz /usr/share/seclists; do
    docker exec "$KALI" bash -lc "test -e $w" && ok "$w" || miss "$w"
  done
fi

hdr "靶场"
docker ps --format '{{.Ports}}' 2>/dev/null | grep -q '3000' && ok "Juice Shop 已运行" || warn "Juice Shop 未运行"

hdr "在线平台连通性"
for u in https://portswigger.net/web-security \
         https://overthewire.org/wargames/ \
         https://www.vulnhub.com/ \
         https://hackmyvm.eu/ \
         https://crackmes.one/ \
         https://ropemporium.com/; do
  code=$(curl -s -o /dev/null -w "%{http_code}" -m 10 -A "Mozilla/5.0" "$u" 2>/dev/null)
  case "$code" in 200|301|302) ok "$code  $u" ;; *) warn "$code  $u" ;; esac
done

printf "\n自检完成。\n"
```

运行：

```bash
chmod +x check-env.sh
./check-env.sh
```

---

## 9. 常见问题

| 现象 | 排查 |
|---|---|
| `docker: command not found` | 未安装 Docker，见第 2 节 |
| 容器内无法联网 | 检查 DNS；必要时设置代理环境变量 |
| pip 安装失败 | 换镜像源：`pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple` |
| GitHub 拉取超时 | 配置代理：`export https_proxy=http://<HOST>:<PORT>` |
| 端口被占用 | `docker ps` 找到容器，改映射端口或 `docker compose down` |
| 本地靶场打不开 | 用 `curl --noproxy '*'` 排除代理干扰；或换浏览器 |
| 内存不足 | 给 Docker 分配 4 GB+；减少同时运行的环境数 |
| ARM 设备镜像报错 | 优先选 multi-arch 官方镜像 |
| Kerberos 报时钟错误 | `ntpdate <DC_IP>` 同步时间 |

### 关于代理

如果宿主机使用代理上网，容器内需要注意：

- **容器访问外网**：可在容器内设置 `http_proxy` / `https_proxy`
- **容器访问宿主机服务**：用 `host.docker.internal` 作为主机名
- **本地靶场访问异常**：系统代理可能拦截 `127.0.0.1`，用 `--noproxy '*'` 绕过

```bash
# 从容器访问宿主机的本地靶场
docker exec kali bash -lc 'curl -I http://host.docker.internal:3000/'
```

---

## 相关文档

- [ROADMAP.md](ROADMAP.md) —— 学习路线总图
- [RESOURCES.md](RESOURCES.md) —— 免费资源大全
- [docs/](docs/README.md) —— 详细教材
