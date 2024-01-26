## 1. 服务端配置
### 1.1. 安装 WireGuard：
首先，确保你的服务器操作系统已经更新到最新版本。然后，你可以通过包管理器安装 WireGuard。对于大多数Linux发行版，你可以使用以下命令：
```
sudo apt update
sudo apt install wireguard
```
### 1.2. 生成密钥对
你需要为服务器生成一个公钥和私钥。这可以通过以下命令完成：
```
wg genkey | tee privatekey | wg pubkey > publickey
```
这将会在当前目录下生成两个文件：privatekey 和 publickey。

### 1.3. 配置 WireGuard 接口：
创建 WireGuard 配置文件通常在 /etc/wireguard/ 目录下。例如，你可以创建一个名为 wg0.conf 的文件：
```
sudo nano /etc/wireguard/wg0.conf
```
然后加入以下内容：
```
[Interface]
Address = 10.0.0.1/24  # 服务器的 VPN IP 地址
SaveConfig = true
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820  # WireGuard 监听端口
PrivateKey = <服务器私钥>  # 将 <服务器私钥> 替换为你的私钥内容
```
确保替换 <服务器私钥> 为你之前生成的私钥内容，并根据你的网络环境调整 Address 和 eth0（你的服务器网卡接口名称）。

### 1.4. 添加客户端配置
对于每个客户端，你需要在服务器的配置文件中添加一个 [Peer] 部分：
```
[Peer]
PublicKey = <客户端公钥>
AllowedIPs = 10.0.0.2/32  # 客户端的 VPN IP 地址
```
每个客户端都应该有唯一的 AllowedIPs 设置。

### 1.5. 启用 IP 转发

```
sudo sysctl -w net.ipv4.ip_forward=1

```
or
你需要在服务器上启用 IP 转发。编辑 /etc/sysctl.conf 文件，并确保以下行是未注释的：
```
net.ipv4.ip_forward=1
```
然后执行 sudo sysctl -p 以应用更改。

### 1.6. 启动 WireGuard 接口
使用以下命令启动 WireGuard 接口：
```
sudo wg-quick up wg0
```
要使 WireGuard 接口在启动时自动启动，可以使用 systemctl：
```
sudo systemctl enable wg-quick@wg0
```

## 2. 配置ubuntu客户端
### 2.1. 安装 WireGuard
在 Ubuntu 上，你可以使用 apt 包管理器安装 WireGuard：
```
sudo apt update
sudo apt install wireguard
```
### 2.2. 生成密钥对
WireGuard 使用公钥加密来建立安全连接。你需要为每个 WireGuard 端点生成一对密钥。
```
wg genkey | tee privatekey | wg pubkey > publickey
```
这将生成一个私钥并保存在 privatekey 文件中，生成的公钥将保存在 publickey 文件中。

### 2.3. 配置 WireGuard 接口
创建 WireGuard 配置文件。通常，这些文件位于 /etc/wireguard/ 目录中。例如，为客户端创建一个名为 wg0.conf 的配置文件：
```
sudo nano /etc/wireguard/wg0.conf
```
在配置文件中，你需要设置你的私钥、服务器的公钥、VPN 地址、端口和其他相关设置。下面是一个配置文件的示例：
```
[Interface]
PrivateKey = <你的私钥>
Address = 10.0.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = <服务器的公钥>
Endpoint = <服务器的IP地址>:51820
//AllowedIPs = 0.0.0.0/0
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```
请确保将 <你的私钥>、<服务器的公钥> 和 <服务器的IP地址> 替换为实际的值。

### 2.4. 启动 WireGuard 接口
启动 WireGuard 接口：
```
sudo wg-quick up wg0
```
这里的 wg0 是你的 WireGuard 接口名称。

停止 WireGuard 接口
如果你需要停止 WireGuard VPN 连接，可以使用：
```
sudo wg-quick down wg0
```
自动启动
如果你希望 WireGuard 随系统启动，可以使用 systemctl 启用它：
```
sudo systemctl enable wg-quick@wg0
```
检查状态
要检查 WireGuard 接口的状态，可以使用：
```
sudo wg show
```
这将显示当前的连接状态和配置详情。

## 配置windows客户端
略...