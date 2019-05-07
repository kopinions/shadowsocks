## shadowsocks


### 打开姿势

``` sh
docker run -dt --name ss -p 6443:6443 mritd/shadowsocks -s "-s 0.0.0.0 -p 6443 -m chacha20-ietf-poly1305 -k test123"
```

### 支持选项

- `-m` : 指定 shadowsocks 命令，默认为 `ss-server`
- `-s` : shadowsocks-libev 参数字符串
- `-x` : 开启 kcptun 支持
- `-e` : 指定 kcptun 命令，默认为 `kcpserver`
- `-k` : kcptun 参数字符串
- `-r` : 使用 `/dev/urandom` 来生成随机数

### 选项描述

- `-m` : 参数后指定一个 shadowsocks 命令，如 ss-local，不写默认为 ss-server；该参数用于 shadowsocks 在客户端和服务端工作模式间切换，可选项如下: `ss-local`、`ss-manager`、`ss-nat`、`ss-redir`、`ss-server`、`ss-tunnel`
- `-s` : 参数后指定一个 shadowsocks-libev 的参数字符串，所有参数将被拼接到 `ss-server` 后
- `-x` : 指定该参数后才会开启 kcptun 支持，否则将默认禁用 kcptun
- `-e` : 参数后指定一个 kcptun 命令，如 kcpclient，不写默认为 kcpserver；该参数用于 kcptun 在客户端和服务端工作模式间切换，可选项如下: `kcpserver`、`kcpclient`
- `-k` : 参数后指定一个 kcptun 的参数字符串，所有参数将被拼接到 `kcptun` 后
- `-r` : 修复在 GCE 上可能出现的 `This system doesn't provide enough entropy to quickly generate high-quality random numbers.` 错误

### 命令示例

**Server 端**

``` sh
docker run -dt --name ssserver -p 6443:6443 -p 6500:6500/udp tzion/shadowsocks -m "ss-server" -s "-s 0.0.0.0 -p 6443 -m aes-256-gcm -k test123" -x -e "kcpserver" -k "-t 127.0.0.1:6443 -l :6500 -mode fast3"
```

**以上命令相当于执行了**

``` sh
ss-server -s 0.0.0.0 -p 6443 -m aes-256-gcm -k test123
kcpserver -t 127.0.0.1:6443 -l :6500 -mode fast2
```

**Client 端**

``` sh
docker run -dt --name ssclient -p 1080:1080 mritd/shadowsocks -m "ss-local" -s "-s 127.0.0.1 -p 6500 -b 0.0.0.0 -l 1080 -m aes-256-gcm -k test123" -x -e "kcpclient" -k "-r SSSERVER_IP:6500 -l :6500 -mode fast2"
```

**以上命令相当于执行了**

``` sh
ss-local -s 127.0.0.1 -p 6500 -b 0.0.0.0 -l 1080 -m aes-256-gcm -k test123
kcpclient -r SSSERVER_IP:6500 -l :6500 -mode fast2
```

**关于 shadowsocks-libev 和 kcptun 都支持哪些参数请自行查阅官方文档，本镜像只做一个拼接**

**注意：kcptun 映射端口为 udp 模式(`6500:6500/udp`)，不写默认 tcp；shadowsocks 请监听 0.0.0.0**


### 环境变量支持


|环境变量|作用|取值|
|-------|---|---|
|SS_MODULE|shadowsocks 启动命令| `ss-local`、`ss-manager`、`ss-nat`、`ss-redir`、`ss-server`、`ss-tunnel`|
|SS_CONFIG|shadowsocks-libev 参数字符串|所有字符串内内容应当为 shadowsocks-libev 支持的选项参数|
|KCP_FLAG|是否开启 kcptun 支持|可选参数为 true 和 false，默认为 fasle 禁用 kcptun|
|KCP_MODULE|kcptun 启动命令| `kcpserver`、`kcpclient`|
|KCP_CONFIG|kcptun 参数字符串|所有字符串内内容应当为 kcptun 支持的选项参数|
|RNGD_FLAG|是否使用 `/dev/urandom` 生成随机数|可选参数为 true 和 false，默认为 fasle 不使用|


使用时可指定环境变量，如下

``` sh
docker run -dt --name ss -p 6443:6443 -p 6500:6500/udp -e SS_CONFIG="-s 0.0.0.0 -p 6443 -m aes-256-gcm -k test123" -e KCP_MODULE="kcpserver" -e KCP_CONFIG="-t 127.0.0.1:6443 -l :6500 -mode fast2" -e KCP_FLAG="true" mritd/shadowsocks
```

### 容器平台说明

**各大免费容器平台都已经对代理工具做了对应封锁，一是为了某些不可描述的原因，二是为了防止被利用称为 DDOS 工具等；基于种种原因，公共免费容器平台问题将不予回复**

### GCE 随机数生成错误

如果在 GCE 上使用本镜像，在特殊情况下可能会出现 `This system doesn't provide enough entropy to quickly generate high-quality random numbers.` 错误；
这种情况是由于宿主机没有能提供足够的熵来生成随机数导致，修复办法可以考虑增加 `-r` 选项来使用 `/dev/urandom` 来生成，不过并不算推荐此种方式；**`-r`
选项可能需要配合 docker 的 `--privileged` 选项启用特权模式来使用**
