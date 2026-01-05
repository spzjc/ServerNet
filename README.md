
````markdown
# autodl 容器翻墙 & Conda/Pip/Git 加速手册

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

## Git 代理配置

### 1️⃣ 临时设置（当前终端有效）

```bash
# HTTP/HTTPS 走 SOCKS5
git config --global http.proxy 'socks5h://127.0.0.1:10899'
git config --global https.proxy 'socks5h://127.0.0.1:10899'
```

* `socks5h` 确保 DNS 解析也走代理
* 端口号 10899 对应你服务器的 SSH 隧道端口

测试：

```bash
git clone https://github.com/openvla/openvla.git
```

### 2️⃣ 关闭 Git 代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```


