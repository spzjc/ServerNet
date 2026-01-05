明白，我帮你整理一份 **完整可操作的“autodl 容器 + SSH 反向隧道 + conda/pip 加速 + 代理开关”手册**，格式为 Markdown，可以直接保存为 `README.md` 或其他 `.md` 文件。

---

# autodl 容器翻墙 & Conda/Pip 加速手册

````markdown
# autodl 容器翻墙 & Conda/Pip 加速手册

## 目录
1. [前提条件](#前提条件)
2. [开启 SSH 反向隧道](#开启-ssh-反向隧道)
3. [配置容器代理](#配置容器代理)
4. [测试代理](#测试代理)
5. [关闭代理](#关闭代理)
6. [Conda 加速方法](#conda-加速方法)
7. [pip 安装依赖加速](#pip-安装依赖加速)
8. [开关代理脚本（Windows）](#开关代理脚本windows)
9. [常见问题](#常见问题)

---

## 前提条件

- Windows 本机安装 Clash Verge 并监听 `127.0.0.1:7897`
- 可以 SSH 登录服务器 / autodl 容器：
  ```bash
  ssh -p 16489 root@connect.cqa1.seetacloud.com
````

* 已安装 Miniconda / Conda

---

## 开启 SSH 反向隧道

在 **Windows PowerShell** 执行：

```powershell
ssh -fN -R 10899:127.0.0.1:7897 -p 16489 root@connect.cqa1.seetacloud.com
```

参数说明：

| 参数                      | 作用                          |
| ----------------------- | --------------------------- |
| -fN                     | 后台运行，不打开 shell              |
| -R 10899:127.0.0.1:7897 | 服务器端 10899 映射到本机 Clash 7897 |
| -p 16489                | SSH 端口                      |
| root@server             | 登录服务器                       |

> 启动后，服务器的 `127.0.0.1:10899` 就可以访问你本机 Clash 出网。

---

## 配置容器代理

登录服务器 / 容器后：

```bash
export ALL_PROXY=socks5h://127.0.0.1:10899
export http_proxy=http://127.0.0.1:10899
export https_proxy=http://127.0.0.1:10899
```

**永久生效**（写入 `.bashrc`）：

```bash
echo 'export ALL_PROXY=socks5h://127.0.0.1:10899' >> ~/.bashrc
echo 'export http_proxy=http://127.0.0.1:10899' >> ~/.bashrc
echo 'export https_proxy=http://127.0.0.1:10899' >> ~/.bashrc
source ~/.bashrc
```

---

## 测试代理

```bash
curl https://ipinfo.io
```

* 成功返回你本机 Clash 的出口 IP
* 失败说明隧道未启动或环境变量未生效

---

## 关闭代理

### 临时关闭（当前终端有效）

```bash
unset ALL_PROXY
unset all_proxy
unset http_proxy
unset https_proxy
```

### 永久关闭（删除 ~/.bashrc 中 export）

```bash
sed -i '/ALL_PROXY/d' ~/.bashrc
sed -i '/http_proxy/d' ~/.bashrc
sed -i '/https_proxy/d' ~/.bashrc
source ~/.bashrc
```

### Windows 隧道关闭

1. 查看 PID：

```powershell
netstat -ano | findstr 10899
```

2. 杀掉进程：

```powershell
taskkill /PID <PID> /F
```

---

## Conda 加速方法

### 1. 使用国内镜像源

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
conda config --set show_channel_urls yes
```

### 2. 配合 SSH / SOCKS5 代理

```bash
export ALL_PROXY=socks5h://127.0.0.1:10899
conda config --set proxy_servers.http http://127.0.0.1:10899
conda config --set proxy_servers.https http://127.0.0.1:10899
```

### 3. 使用 mamba（超高速替代）

```bash
conda install -n base -c conda-forge mamba
mamba create -n myenv python=3.12
```

---

## pip 安装依赖加速

### 1. 安装 SOCKS 支持

```bash
pip install PySocks
```

### 2. 使用代理安装包

```bash
pip install --proxy socks5h://127.0.0.1:10899 -e .
```

### 3. 国内镜像（无需代理）

```bash
pip install -e . -i https://mirrors.aliyun.com/pypi/simple
```

---

## 开关代理脚本（Windows）

保存为 `tunnel.ps1`：

```powershell
param(
    [string]$action = "status"
)

$PORT = 10899
$PID = (Get-NetTCPConnection -LocalPort $PORT -State Listen | Select-Object -ExpandProperty OwningProcess) 

switch ($action) {
    "start" {
        if ($PID) {
            Write-Host "隧道已经启动, PID=$PID"
        } else {
            Write-Host "启动 SSH 隧道..."
            ssh -fN -R 10899:127.0.0.1:7897 -p 16489 root@connect.cqa1.seetacloud.com
        }
    }
    "stop" {
        if ($PID) {
            Write-Host "关闭 SSH 隧道 PID=$PID"
            taskkill /PID $PID /F
        } else {
            Write-Host "隧道没有运行"
        }
    }
    "status" {
        if ($PID) {
            Write-Host "隧道正在运行, PID=$PID"
        } else {
            Write-Host "隧道未运行"
        }
    }
}
```

使用方法：

```powershell
.\tunnel.ps1 status   # 查看状态
.\tunnel.ps1 start    # 启动隧道
.\tunnel.ps1 stop     # 关闭隧道
```

---

## 常见问题

1. **pip 安装依赖报 Missing SOCKS support**

   * 执行 `pip install PySocks`
   * 确保 `ALL_PROXY` 环境变量正确

2. **curl / pip 访问超时**

   * 检查 SSH 隧道是否运行
   * 检查端口是否与环境变量一致

3. **Conda 下载慢**

   * 使用国内镜像源
   * 或使用 `mamba`
   * 或结合代理

4. **服务器访问本机 localhost 失败**

   * 必须用 **SSH 反向端口转发 (-R)**，不能直接访问 Windows 的 127.0.0.1

---

```

---

✅ 这份手册内容完整，涵盖：

- SSH 隧道开关  
- 服务器代理配置  
- 测试 / 关闭代理  
- Conda & pip 加速  
- Windows 脚本一键管理隧道  
- 常见问题排查  

你可以直接保存为 `README.md`，放在你的 autodl 项目里，随时参考和使用。  

---

如果你愿意，我可以再帮你加一个 **“一键 setup.sh / setup.ps1 脚本”，开机就能自动启动隧道 + 配置代理 + 激活 conda”**，真正做到开关秒控。  

你希望我加吗？
```
