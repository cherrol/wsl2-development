# 一、安装WSL2

参照 https://docs.microsoft.com/en-us/windows/wsl/install-win10
> 当你电脑配置比较高的时候一定要设置一下 wsl 资源占用，不然 wsl 会占用过多资源导致电脑卡，  
> 配置文件位置 C:\Users\\[user]\\.wslconfig  
> 如下是我的配置（仅供参考，我的电脑配置： i7 11800H、40GB RAM）

```
[wsl2]
# 自定义 Linux 内核的绝对路径
#kernel=<path>
# 给 WSL 2 虚拟机分配的内存大小
memory=10GB
# 为 WSL 2 虚拟机分配的处理器核心数量
processors=4
# 为 WSL 2 虚拟机分配的交换空间，0 表示没有交换空间
swap=16GB
# 自定义交换虚拟磁盘 vhd 的绝对路径
# swapFile=<path>
# 是否允许将 WSL 2 的端口转发到主机（默认为 true）
# localhostForwarding=<bool>

# `<path>` 必须是带反斜杠的绝对路径，例如 `C:\\Users\\kernel`
# `<size>` 必须在后面加上单位，例如 8 GB 或 512 MB
```


# 二、安装docker desktop windows

参照 https://docs.docker.com/docker-for-windows/install/


# 三、docker在wsl2下的设置

参照 https://docs.docker.com/docker-for-windows/wsl/


# 四、WSL2科学上网

>  对应的软件设置为允许局域网连接（我使用的是SSR）  
 
![alt](assets/images/局域网设置.jpg "局域网设置")

>  防火墙设置放行

![alt](assets/images/防火墙规则.png "局域网设置")


>  在wsl2 terminal中打开```vim ~/.bashrc```（vscode中打开为 ```code ~/.bashrc```），在最下边添加如下代码

```bash
# update 2023-01-02 新版 wsl 配置参考 https://github.com/microsoft/WSL/issues/10753#issuecomment-1814839310，也不需要再设置防火墙放行和代理转发

# set http proxy
WSL_MASTER_HOST_IP=`cat /etc/resolv.conf | grep nameserver | awk '{print $2}'`

export http_proxy="http://${WSL_MASTER_HOST_IP}:1080"  # 此处端口对应SSR的本地端口1080

export https_proxy=$http_proxy

export all_proxy=$http_proxy

# set git config http proxy

if [ "`git config --global --get http.proxy`" != "http://$WSL_MASTER_HOST_IP:1080" ]; then

  git config --global http.proxy http://$WSL_MASTER_HOST_IP:1080

fi
```

如果是使用zsh则是打开 ```vim ~/.zshrc```（或者```code ~/.zshrc```），添加上边的代码到末尾

> wsl2 ping 不通主机时，需要设置 wsl2 网络允许通过防火墙(需要管理员权限打开cmd/powershell执行)
```bash
# cd folder
cd C:\WINDOWS\system32

# run
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```

> 测试连接(wsl 环境下)
```bash
# 查看 wsl 的ip
env
# 找到环境变量 http_proxy ，查看主机 ip，假设为 window_ip
ping window_ip
```

# 五 Git相关设置
> 设置wsl ssh共享请参考https://devblogs.microsoft.com/commandline/sharing-ssh-keys-between-windows-and-wsl-2/

> 设置github代理，使用http:// 协议代替 git://   
> 打开 vim ~/.gitconfig，将下边代码添加至末尾  
``` bash
[url "https://github.com/"]
  insteadOf = git://github.com/
```

> 设置多个ssh连接参照 https://linuxize.com/post/using-the-ssh-config-file/  
> 例如本地配置github、gitee的ssh ```~/.ssh/config```  
```bash
Host github.com
  User git
  Port 22
  Hostname github.com
  # 注意修改路径为你的路径
  IdentityFile "~/.ssh/id_rsa"
  TCPKeepAlive yes

Host ssh.github.com
  User git
  Port 443
  Hostname ssh.github.com
  # 注意修改路径为你的路径
  IdentityFile "~/.ssh/id_rsa"
  TCPKeepAlive yes

Host gitee.com
  User git
  Port 22
  Hostname gitee.com
  # 注意修改路径为你的路径
  IdentityFile "~/.ssh/id_rsa"
  TCPKeepAlive yes

Host ssh.gitee.com
  User git
  Port 443
  Hostname ssh.gitee.com
  # 注意修改路径为你的路径
  IdentityFile "~/.ssh/id_rsa"
  TCPKeepAlive yes

```

# 六、docker项目中文件权限
> 运行docker ps(或相关docker指令)有权限问题时需要修复docker权限
```bash
sudo addgroup --system docker
sudo adduser $USER docker
newgrp docker
```

>  项目文件权限问题，在wsl2下设置文件夹权限为777即可
```bash
# 例如我的工作目录为 ~/workspace
cd ~/
sudo chmod -R 777 workspace/
```

# 注意事项
> 启动ssr或者相关软件时请关闭windows下其他代理（检查windows代理端口是否正确），防止冲突导致ssr不生效


# update 2023-01-02 新版 wsl 配置参考 https://github.com/microsoft/WSL/issues/10753#issuecomment-1814839310 ，也不需要再设置防火墙放行和代理转发
# .bashrc config
```bash
#...

# proxy
export WSL_MASTER_HOST_IP=`cat /etc/resolv.conf | grep nameserver | awk '{print $2}'`
export my_custom_proxy="http://@${username}:@${password}-@${WSL_MASTER_HOST_IP}:10808";
# export my_custom_proxy="http://${WSL_MASTER_HOST_IP}:10808";
export all_proxy=$my_custom_proxy
export ALL_PROXY=$my_custom_proxy
export http_proxy=$my_custom_proxy
export https_proxy=$my_custom_proxy
alias addproxy='
export all_proxy=$my_custom_proxy
export ALL_PROXY=$my_custom_proxy
export http_proxy=$my_custom_proxy
export https_proxy=$my_custom_proxy
echo -e "Acquire::http::Proxy \"$my_custom_proxy\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
echo -e "Acquire::https::Proxy \"$my_custom_proxy\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
'
alias rmproxy='
unset all_proxy
unset ALL_PROXY
unset http_proxy
unset https_proxy
'
# add update && upgrade alias
alias upall="sudo apt update -y && sudo apt upgrade -y"

#...
```
