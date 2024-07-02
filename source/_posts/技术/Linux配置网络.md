---
title: Linux配置网络
categories: '系统'
tags: ['Linux', '网络']
date: 2024-06-16 16:15:39
cover: cover.png
---
## 连接网络

在这个万物互联的时代，几乎所有的操作系统都要连接网络。网络连接通常采用有线和无线两种方式。通常，家庭或办公室的典型网络设置如下：

* 调制解调器连接到ISP：调制解调器从ISP接收互联网信号。
* 调制解调器连接到路由器：通过网线，将调制解调器连接到路由器的WAN端口。
* 路由器连接到本地设备：路由器通过有线或无线方式连接到本地的计算机、手机或其他设备。

下文将以该网络设置为基础介绍Linux系统连接网络的方法。在Linux系统中，物理机可以使用`nmcli`工具实现网络的连接，而虚拟机可以通过虚拟机软件配置实现上网。

### 有线方式

采用有线方式连接网络，需要用网线将设备与路由器LAN口相连，步骤如下：

1. 确保网卡存在以及驱动正确安装，可使用以下命令查看网络接口，有线网络接口的名称一般以`eth`或`en`开头。

```bash
ip a
```

> 现代Linux系统推荐使用`ip`命令，`ip a`命令中的`a`为`address`，使用`ip a`、`ip addr`、`ip address`的效果是一致的。不仅如此，对于下文介绍的`nmcli`命令后的单词也可以使用缩写，例如`nmcli c`、`nmcli con`、`nmcli connection`的含义一致。

2. 确保设备与路由器的LAN口相连。在插入网线的情况下，Linux系统会自动检测到有线连接并尝试获取IP地址。如果没有自动连接，可使用以下命令手动请求IP地址。

```bash
sudo dhclient <interface>
```

### 无线方式

采用无线方式连接网络，可以通过`nmcli`工具实现，步骤如下：

1. 确保网卡存在以及驱动正确安装，可使用以下命令查看网络接口，无线网络接口的名称一般以`wlan`或`wlp`开头。

```bash
ip a
```

2. 确保无线设备已被启用。

```bash
sudo ip link set <interface> up
```

3. 开启WLAN。

```bash
nmcli radio wifi on
```

4. 查看热点。

```bash
nmcli device wifi list
```

这里的热点列表会以信号强度由强到弱进行排序显示。

![热点列表](热点列表.png)

5. 连接热点。

```bash
nmcli device wifi connect <SSID> --ask
```

> 如果你不介意密码以明文显示，可以使用`nmcli device wifi connect <SSID> password <password>`。`--ask`选项会要求你交互式输入密码，安全性更高，这也是本文推荐的方式。

### 网络模式

对于虚拟机而言，虚拟机的网络通常由虚拟机软件进行管理。通常，虚拟机的网络模式包含：

* NAT模式：虚拟机通过宿主机的IP地址和端口与外部网络通信。
* 桥接模式：虚拟机直接连接到物理网络，就像一台独立的物理主机。
* 内部网络：多个虚拟机之间可以相互通信，但不能与宿主机或外部网络通信。
* 仅主机模式：虚拟机只能与宿主机通信，不能访问外部网络。

这4种方式中，只有前两种方法可以访问互联网，我们需要根据实际使用需求在虚拟机软件中进行相应配置。对于这几种模式的区别和联系，可以观看下面技术蛋老师的[视频](https://www.bilibili.com/video/BV11M4y1J7zP)了解。

<iframe style="width: 100%; aspect-ratio: 16/9;" src="//player.bilibili.com/player.html?bvid=BV11M4y1J7zP&poster=1&autoplay=0" frameborder="no" scrolling="no"></iframe>

在虚拟机中，可使用`ip a`命令查看网络接口。

![虚拟机查看网络地址](虚拟机查看网络地址.png)

## 静态地址

设置静态地址是Linux系统中的常见操作之一，特别是在虚拟机中十分常见。设置静态IP地址后，可以使用`ssh`连接固定的IP地址，从而对虚拟机进行访问。通常，网络中分配IP地址的方式包含：

* DHCP：通过DHCP服务器自动分配IP地址和配置其他网络参数。
* 静态地址：手动分配IP地址和配置其他网络参数，设备的IP地址在配置后不会改变。

下文将介绍设置静态IP地址的方法。在Linux系统中，可以使用`nmcli`工具绑定静态IP地址。

### 有线连接

采用有线方式连接的网络，可以通过`nmcli`工具实现静态地址绑定，步骤如下：

1. 确定网络接口名称，可使用以下命令查看网络接口，有线网络接口的名称一般以`eth`或`en`开头。

```bash
ip a
```

在这一步可确定当前所属的网络，通过接口信息中的`inet`这一行所显示的内容而确定。例如如下信息：

```
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:20:74:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.102/24 brd 192.168.31.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::eea5:2add:2436:186%10 scope link
       valid_lft forever preferred_lft forever
...
```

从上述信息可以看出，当前所属网络为`192.168.31.0/24`，除去网络地址`192.168.31.0`、广播地址`192.168.31.255`和网关地址`192.168.31.1`外，其余地址均为可选的IP地址，也就是`192.168.31.2`~`192.168.31.254`，一般情况下选择当前地址`192.168.31.102`即可。除此之外，也可以通过路由器配置或虚拟机配置确定网络地址。

2. 确定有线连接名称，可使用以下命令查看已有连接，该连接名称需要与上一步查看的接口名称相关联。

```bash
nmcli connection show
```

3. 设置静态连接所需的IP地址、网关以及DNS等信息。

```bash
nmcli connection modify <connection> ipv4.method manual ipv4.addresses <address> ipv4.gateway <gateway> ipv4.dns <dns>
```

`connection`为连接名称，`address`为IP地址，`gateway`为网关地址，`dns`为DNS服务器地址。以上述网络为例：

* `connection`：连接名称为上一步查看到的连接名称，连接名称要与相应的接口相关联。
* `address`：IP地址建议选择`192.168.31.102/24`。该地址只要属于可选范围且不被占用即可。
* `gateway`：网关地址为`192.168.31.1`。该地址的主机部分通常为`1`，这也是访问路由器管理界面的地址。
* `dns`：DNS服务器地址建议选择`8.8.8.8`或`8.8.4.4`，选择网关地址通常也是可行的。

4. 停用连接。

```bash
nmcli connection down <connection>
```

5. 启用连接，启用后可再次使用`ip a`查看网络地址。

```bash
nmcli connection up <connection>
```

### 无线连接

采用无线方式连接的网络，其配置方式与有线连接基本一致，步骤如下：

1. 确定网络接口名称，可使用以下命令查看网络接口，无线网络接口的名称一般以`wlan`或`wlp`开头。

```bash
ip a
```

2. 确定无线连接名称，可使用以下命令查看已有连接，该连接名称需要与上一步查看的接口名称相关联，其名称通常为WLAN名称。

```bash
nmcli connection show
```

3. 设置静态连接所需的IP地址、网关以及DNS等信息。

```bash
nmcli connection modify <connection> ipv4.method manual ipv4.addresses <address> ipv4.gateway <gateway> ipv4.dns <dns>
```

4. 停用连接。

```bash
nmcli connection down <connection>
```

5. 启用连接，启用后可再次使用`ip a`查看网络地址。

```bash
nmcli connection up <connection>
```

### 重置连接

如果设置静态IP后悔了，可以使用`nmcli connection modify <connection> ipv4.method auto`来取消静态IP，然后再次使用`nmcli connection down <connection> && nmcli connection up <connection>`来重新启用连接。这样子做当然没有问题，但可能会导致使用`ip a`时出现两个IPv4地址。

如果你想更彻底地清除相关连接的配置，达到重置网络连接的目的，步骤如下：

1. 确定网络接口名称，可使用以下命令查看网络接口。

```bash
ip a
```

2. 确定要重置的连接名称，可使用以下命令查看已有连接，该连接名称需要与上一步查看的接口名称相关联。

```bash
nmcli connection show
```

3. 删除指定的连接。这一步类似于在手机中我们点击了“忘记网络”。

```bash
nmcli connection delete <connection>
```

4. 添加一个新的有线连接或无线连接。

如果是有线连接，可使用下面的命令添加一个有线连接：

```bash
nmcli connection add type ethernet ifname <interface> con-name <connection>
```

如果是无线连接，可使用前文的方法添加一个无线连接：

```bash
nmcli device wifi connect <SSID> --ask
```

> `nmcli connection add`甚至可以添加一个拨号连接。不过现如今拨号连接大多在光猫或路由器中设置过了，而且也早已不是电脑主流的上网方式。


## 设置代理

访问网络后，我们需要经常访问Github或其他网址，如果不设置代理的话经常会导致`git clone`失败和其他网络异常情况。下文将介绍三种不同代理的方法，分别适用于不同的环境。这些代理方式如下：

* 用户代理：本地用户代理，针对于单个用户，适用于物理机或云服务器。
* 全局代理：本地全局代理，针对于整个系统，适用于物理机或云服务器。
* 外部代理：借助外部软件和外部设备实现代理，适用于虚拟机。

下文将以{% inlineImg 316f5eff3a89/clash.png %}为例介绍设置代理的方法。如果你想获得更好的网络体验，这一步也是不可或缺的。

### 用户代理

用户代理是为单个用户配置相应的代理，其主要通过修改`~/.bashrc`文件实现，步骤如下：

1. 准备文件`clash-linux-amd64-v1.18.0.gz`、`Country.mmdb`、`config.yaml`。
* `clash-linux-amd64-v1.18.0.gz`：从[地址1](https://www.clash.la/releases/)、[地址2](https://github.com/Loyalsoldier/clash-rules/tree/hidden)中获取，选择`linux-amd64`版本的即可。
* `Country.mmdb`：从[地址3](https://github.com/Dreamacro/maxmind-geoip/releases)中获取。
* `config.yaml`：从订阅地址中获取，如果下载下来文件后缀是`.yml`，请手动更改为`.yaml`以便后续使用。

`clash-linux-amd64-v1.18.0.gz`是软件自然不必多说。`Country.mmdb`是数据集文件，它可以利用GeoIP2服务能识别互联网用户的地点位置以供规则分流时使用。`config.yaml`是订阅配置文件。这些文件可以通过`wget`或`curl`获取。

```bash
wget <url>
```

2. 将可执行文件解压、重命名、并赋予执行权限。

```bash
gunzip clash-linux-amd64-v1.18.0.gz   # 解压可执行文件
mv clash-linux-amd64-v1.18.0 clash    # 重命名
chmod u+x clash                       # 为当前用户赋予执行权限
```

3. 移动上述3个文件放入指定位置，存放于用户目录下即可。

```bash
mkdir ~/.config/clash                 # 创建配置文件夹
cp Country.mmdb ~/.config/clash/      # 复制配置
cp config.yaml ~/.config/clash/       # 复制配置
cp clash ~/.local/bin/                # 复制可执行文件
```

4. 修改终端的启动运行文件，亦即`~/.bashrc`文件。

```bash
vim ~/.bashrc
```

文件添加内容如下：

```bash
if ! pgrep -x "clash" > /dev/null; then
  nohup ~/.local/bin/clash -d ~/.config/clash > /dev/null 2>&1 &
fi
```

5. 使用`export`和`unset`设置代理。为了方便使用，可继续将以下内容添加至`~/.bashrc`，并对该文件使用`source`使其在当前环境中生效。

```bash
vim ~/.bashrc
```

文件添加内容如下：

```bash
function proxy() {
  if [ "$1" = "on" ]; then
    export http_proxy="http://127.0.0.1:7890"
    export https_proxy="http://127.0.0.1:7890"
    export all_proxy="socks5://127.0.0.1:7890"
    echo "Proxy enabled: http://127.0.0.1:7890, socks5://127.0.0.1:7890"
  elif [ "$1" = "off" ]; then
    unset http_proxy
    unset https_proxy
    unset all_proxy
    echo "Proxy disabled."
  else
    echo "Usage: proxy on | off"
  fi
}

# Enable proxy
proxy on > /dev/null 2>&1
```

文件中地址是`127.0.0.1:7890`，其中端口号为`config.yaml`中的`port`选项，通常为`7890`，如有不同则需要将对应文件内容进行替换。

重新使文件生效：

```bash
source ~/.bashrc
```

自此，使用`proxy on`即可开启代理，使用`proxy off`即可关闭代理。在需要下载新的`config.yaml`时，可以考虑关闭代理。

### 全局代理

全局代理是为整个系统配置相应的代理，其主要通过添加`systemd`系统服务实现，这样可以更好地控制其启动、停止和重启，步骤如下：

1. 准备文件`clash-linux-amd64-v1.18.0.gz`、`Country.mmdb`、`config.yaml`。
* `clash-linux-amd64-v1.18.0.gz`：从[地址1](https://www.clash.la/releases/)、[地址2](https://github.com/Loyalsoldier/clash-rules/tree/hidden)中获取，选择`linux-amd64`版本的即可。
* `Country.mmdb`：从[地址3](https://github.com/Dreamacro/maxmind-geoip/releases)中获取。
* `config.yaml`：从订阅地址中获取，如果下载下来文件后缀是`.yml`，请手动更改为`.yaml`以便后续使用。

2. 将可执行文件解压、重命名、并赋予执行权限。

```bash
gunzip clash-linux-amd64-v1.18.0.gz   # 解压可执行文件
mv clash-linux-amd64-v1.18.0 clash    # 重命名
chmod u+x clash                       # 为当前用户赋予执行权限
```

3. 移动上述3个文件放入指定位置，需要存放于系统目录下。

```bash
sudo mkdir /etc/clash                 # 创建配置文件夹
sudo cp Country.mmdb /etc/clash/      # 复制配置
sudo cp config.yaml /etc/clash/       # 复制配置
sudo cp clash /usr/local/bin/         # 复制可执行文件
```

4. 创建并编写系统服务文件，并使用`systemctl`设置系统服务。

```bash
sudoedit /etc/systemd/system/clash.service
```

> 推荐使用`sudoedit <file>`而不是`sudo vim <file>`。`sudoedit`会根据`EDITOR`环境变量确定需要使用的编辑器。如果你还没有设置过该环境变量的话，可以使用`export EDITOR=vim`，否则将使用默认编辑器，例如`GNU nano`。

服务文件内容如下：

```service
[Unit]
Description=Clash daemon, A rule-based proxy in Go.
After=network.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
```

设置并启动系统服务：

```bash
sudo systemctl enable clash           # 设置开机自启
sudo systemctl start clash            # 启动系统服务
systemctl status clash                # 查看服务状态
```

服务状态显示为`active`即表示运行成功。

![Clash服务状态](Clash服务状态.png)

5. 使用`export`和`unset`设置代理。为了方便使用，可同样将以下内容添加至`~/.bashrc`，并对该文件使用`source`使其在当前环境中生效。

```bash
vim ~/.bashrc
```

文件添加内容如下：

```bash
function proxy() {
  if [ "$1" = "on" ]; then
    export http_proxy="http://127.0.0.1:7890"
    export https_proxy="http://127.0.0.1:7890"
    export all_proxy="socks5://127.0.0.1:7890"
    echo "Proxy enabled: http://127.0.0.1:7890, socks5://127.0.0.1:7890"
  elif [ "$1" = "off" ]; then
    unset http_proxy
    unset https_proxy
    unset all_proxy
    echo "Proxy disabled."
  else
    echo "Usage: proxy on | off"
  fi
}

# Enable proxy
proxy on > /dev/null 2>&1
```

重新使文件生效：

```bash
source ~/.bashrc
```

同样，使用`proxy on`即可开启代理，使用`proxy off`即可关闭代理。

### 外部代理

外部代理是借助同一网络下外部设备以及外部软件实现的代理。相比于本地代理而言，这种方式可以摆脱订阅的在线设备数量限制，并使得宿主机与虚拟机代理配置保持一致。

如果你使用的是虚拟机（特别是WSL）的话，更推荐使用这种方式进行代理，步骤如下：

1. 在宿主机安装软件，可从[地址1](https://www.clash.la/releases/)、[地址2](https://github.com/Loyalsoldier/clash-rules/tree/hidden)、[地址3](https://clash.wiki/)中获取。

2. 在宿主机配置软件，按照图形化操作即可，并且在宿主机软件中**开启局域网访问**。以Windows为例，打开`General`下的`Allow LAN`选项。

![允许局域网访问](允许局域网访问.png)

3. 查看宿主机IP地址。在Windows下，使用`ipconfig`命令即可。针对于虚拟机，除过内部网络和仅主机模式无法访问网络外，有两种情况：

* NAT模式：选择宿主机与虚拟机相关的虚拟网卡地址，该地址也可在虚拟机中使用`ip route | grep 'default' | awk '{print $3}' | head -n 1`查看。
* 桥接模式：选择宿主机上网时所使用的IP地址，这种情况下通常需要对宿主机设置静态IP地址。

4. 在虚拟机中使用`export`和`unset`设置代理。为了方便使用，可同样将以下内容添加至`~/.bashrc`，并对该文件使用`source`使其在当前环境中生效。

```bash
vim ~/.bashrc
```

文件添加内容如下，**注意将最后一行进行替换**：

```bash
function proxy() {
  if [ "$1" = "on" ] && [ -n "$2" ]; then
    export http_proxy="http://$2"
    export https_proxy="http://$2"
    export all_proxy="socks5://$2"
    echo "Proxy enabled: http://$2, socks5://$2"
  elif [ "$1" = "off" ]; then
    unset http_proxy
    unset https_proxy
    unset all_proxy
    echo "Proxy disabled."
  else
    echo "Usage: proxy on <address>:<port> | off"
  fi
}

# Enable proxy
proxy on <address>:<port> > /dev/null 2>&1
```

`address`地址当然是上一步查看到宿主机的IP地址，`port`端口号为宿主机`config.yaml`中的`port`选项，通常为`7890`。

重新使文件生效：

```bash
source ~/.bashrc
```

自此，使用`proxy on <address>:<port>`即可开启代理，使用`proxy off`即可关闭代理。配置代理在宿主机中进行即可，由于是采用图形化界面，操作起来也很方便。
